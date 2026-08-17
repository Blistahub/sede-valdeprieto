# 0001 · Arquitectura de la aplicación

**Estado:** aceptado · **Fecha:** 2026-08-17 · **Afecta a:** `app/`, `components/`,
`lib/`, `tests/`

---

## Nota sobre la extensión

`CLAUDE.md` §0.5 fija un máximo de una página por ADR. Este lo excede a propósito y es el
único que lo hace: es el ADR fundacional y agrupa cinco decisiones que solo tienen sentido
juntas, porque cada una apoya a la siguiente. Partirlas en cinco documentos obligaría a
leer los cinco para entender cualquiera. Cada bloque se mantiene compacto, y cualquiera de
ellos que más adelante necesite profundidad tendrá su propio ADR.

## Contexto

El invariante II de `CLAUDE.md` dice que el expediente es una máquina de estados con valor
legal y que ningún estado cambia desde la interfaz sin pasar por la capa de dominio. El
invariante III exige que todo sea verificable por un tercero. La arquitectura no es aquí
una preferencia de estilo: es el mecanismo que hace que esos dos invariantes sean ciertos
por construcción y no por disciplina de quien programa.

Todo lo que sigue existe para responder a una pregunta: **¿qué impide que dentro de tres
meses alguien con prisa cambie un estado desde un componente?**

---

## D1 · Frontera servidor/cliente

### Opciones

**A · Aplicación de cliente con API REST.** El navegador pide datos y los pinta. Es lo
habitual y es lo peor aquí: obliga a duplicar en cliente el cálculo de plazos y estados, o
a confiar en que el servidor los mande siempre bien calculados sin que nada lo garantice.

**B · Server Components por defecto, Server Actions para escribir.** Elegida.

**C · Renderizado en servidor con hidratación total.** Todo se vuelve interactivo aunque no
lo necesite. Coste de rendimiento sin beneficio: la mayoría de estas pantallas son
documentos, no aplicaciones.

### Decisión

Server Components por defecto. Un componente pasa a cliente **solo** cuando hay una
interacción que el servidor no puede resolver: la subida de ficheros con su progreso, el
sellado del justificante, el movimiento del foco y los anuncios en regiones activas. Nada
más.

**Regla dura: el cliente nunca calcula un dato de dominio.** Ni un día hábil, ni un estado,
ni un vencimiento. Esos valores llegan calculados desde el servidor y el cliente los pinta.
`lib/dominio/` y `lib/plazos/` se ejecutan en servidor; desde cliente solo se importan sus
**tipos**.

### Consecuencias

Desaparece `useEffect` para obtener datos, que `CLAUDE.md` §5 ya prohibía. El borrador de
la solicitud vive en servidor, no en memoria del navegador: reanudarlo (E2-H04) deja de ser
una funcionalidad y pasa a ser el comportamiento natural, y cerrar la pestaña no pierde
nada. A cambio, cada paso del formulario es una ida y vuelta al servidor, y hay que medirlo
contra el presupuesto de Lighthouse.

---

## D2 · Dónde vive la validación compartida

### Opciones

**A · Validar solo en servidor.** Correcto y hostil: la persona descubre el error después
de enviar.

**B · Dos validaciones, una por lado.** Es lo que pasa cuando nadie decide. Divergen, y el
día que divergen el cliente deja pasar algo que el servidor rechaza sin explicación útil.

**C · Un solo esquema, ejecutado en los dos lados.** Elegida.

### Decisión

Los esquemas Zod viven en `lib/validacion/` y los importan tanto el componente cliente como
la Server Action. **Un solo esquema, dos ejecuciones.** El cliente valida para dar respuesta
inmediata; el servidor valida porque es el único que manda.

Y una separación que no se mezcla nunca:

| Tipo de regla | Dónde vive | Ejemplo |
| --- | --- | --- |
| **Formato** | `lib/validacion/` (Zod) | El NIF termina en letra y la letra de control cuadra |
| **Negocio** | `lib/dominio/` y `lib/plazos/` | ¿Está dentro del plazo? ¿Queda crédito? ¿Procede esta transición? |

Zod no sabe de festivos y no debe aprender. Un esquema que consulte el calendario laboral
es una regla de negocio disfrazada de validación de formato.

Los errores son **tipos de resultado explícitos**, no excepciones: la interfaz tiene que
poder distinguir «el NIF no es válido» de «el servicio está caído», porque a la persona se
le dice una cosa distinta en cada caso.

### Consecuencias

Un cambio de regla se hace en un sitio. El coste es que `lib/validacion/` no puede importar
nada de servidor —ni Prisma, ni `node:fs`—, porque se empaqueta también para el navegador.
Esa restricción se vigila con lint, no con buena voluntad.

---

## D3 · Cómo se garantiza que ninguna transición escapa a la capa de dominio

Esta es la decisión que sostiene el invariante II, y una sola barrera no basta. Son cuatro,
y cada una atrapa lo que se le escapa a la anterior.

### Opciones

**A · Confiar en la convención.** «Todo el mundo sabe que las transiciones van en
`lib/dominio/`.» Funciona hasta el primer viernes por la tarde.

**B · Revisión de código.** Necesaria, pero no es un mecanismo: es una persona cansada.

**C · Barreras de tipo, de acceso, de lint y de test.** Elegida.

### Decisión

**1 · Barrera de tipos.** `EstadoExpediente` no se asigna, se obtiene. La única forma de
producir un expediente con estado nuevo es la función de transición de
`lib/dominio/expediente.ts`, que devuelve un `TransicionAplicada` —un tipo que solo ese
módulo puede construir—. Fuera de ahí, el estado es de solo lectura.

**2 · Barrera de acceso a datos.** El repositorio **no expone** una operación del tipo
`actualizar({ estado })`. Su única entrada para cambiar un expediente es un
`TransicionAplicada`. Prisma queda encapsulado en `lib/datos/` y nadie lo importa desde
fuera.

**3 · Barrera de lint.** Regla `no-restricted-imports`: `app/` y `components/` no pueden
importar `lib/datos/`, y nadie fuera de `lib/datos/` puede importar el cliente de Prisma.
Cualquier desactivación en línea de esta regla exige comentario con la causa y enlace a
este ADR, como manda `CLAUDE.md` §5.

**4 · Barrera de test.** Dos pruebas. La parametrizada que recorre los 100 pares de la
matriz de adyacencia, y una prueba de arquitectura que falla si algún fichero fuera de
`lib/dominio/` contiene un literal de estado.

**Y la traza va en la misma transacción que el cambio de estado.** Si el asiento de traza
no se escribe, la transición no ocurre. Un expediente que cambió de estado sin dejar
constancia es peor que un expediente que no cambió: el primero miente y el segundo solo
está desactualizado.

### Consecuencias

Escribir una transición nueva cuesta más que asignar un campo, y ese es exactamente el
efecto buscado. La capa de datos queda encapsulada, lo que además permite cambiar Prisma
sin tocar el dominio. El precio es un `lib/datos/` con más superficie de la que tendría un
acceso directo.

---

## D4 · Sesión e identificación simulada

### Opciones

**A · Auth.js o similar.** Resuelve OAuth, proveedores externos y refresco de tokens. Aquí
no hay ningún proveedor externo que integrar: la identificación es simulada. Sería una
dependencia grande sin un problema que resolver, y `CLAUDE.md` §2 obliga a justificar cada
dependencia. Descartada.

**B · Cookie de sesión firmada propia.** Elegida.

**C · Sin sesión, identidad por parámetro.** Haría imposible demostrar la autorización, que
es justo lo que hay que enseñar.

### Decisión

Cookie de sesión firmada, `HttpOnly`, `Secure`, `SameSite=Lax`, con contenido mínimo:
identificador de la persona o del usuario de gestión, y nada más.

**El rol no viaja en la cookie como fuente de verdad.** Se resuelve en servidor en cada
Server Action a partir del identificador. Una cookie manipulada no concede un rol.

La pantalla de identificación imita el aspecto de un selector de método de acceso, pero
ofrece una lista de identidades sintéticas de demostración, etiquetada de forma inequívoca
como tal. No se imita ninguna marca ni ningún sistema real de identificación
(`CLAUDE.md` §10.1).

**La interfaz oculta; el dominio impide.** Que un botón no se pinte para un rol no es una
comprobación de seguridad. Cada transición recibe el actor y verifica rol y titularidad
dentro de `lib/dominio/`, y devuelve `ActorNoAutorizado` o `ExpedienteAjeno` si no procede.
Las reglas P-12 a P-18 del modelo de dominio son tests, no buenas intenciones.

### Consecuencias

Sin dependencia de autenticación. A cambio, hay que escribir y probar la firma y la
expiración de la cookie, que es código de seguridad propio y por tanto necesita sus tests.
Cuando llegue una identificación real, esta decisión se sustituye por un ADR nuevo: la
frontera —el servidor resuelve el rol en cada acción— seguirá siendo válida.

---

## D5 · Cómo se genera el justificante

### Opciones

**A · Generar el PDF y guardarlo, y calcular la huella sobre sus bytes.** Parece lo obvio y
es una trampa: cualquier cambio de plantilla, de fuente o de versión de la librería produce
bytes distintos, y con ellos una huella distinta para el mismo acto administrativo.

**B · Huella sobre una representación canónica de los datos; el PDF y la página son
representaciones.** Elegida.

**C · Sin huella.** Incumple `docs/dominio.md`, que exige huella en el justificante.

### Decisión

Al confirmarse el registro, y **dentro de la misma transacción**, el servidor construye un
objeto canónico con lo presentado: número de registro, sello de tiempo, convocatoria,
expediente, relación de documentos con sus huellas individuales. Se serializa de forma
estable —claves ordenadas, sin espacios variables— y se calcula su **SHA-256**. Esa es la
huella, y es lo que se almacena.

El PDF descargable y la página web del justificante son **dos representaciones del mismo
canónico**, generadas bajo demanda. La huella acredita el contenido presentado, no los
píxeles de una plantilla. Si mañana cambia el diseño del PDF, la huella sigue siendo
válida, que es justo lo que debe ocurrir.

**Código seguro de verificación.** 16 caracteres del alfabeto Crockford base32, generados
con un generador criptográficamente seguro y almacenados con índice único. **No se deriva
del contenido**: un código derivado es un código adivinable.

**Página `/verificar/{csv}`.** Pública, sin identificación. Devuelve número de registro,
sello de tiempo, convocatoria y huella. **Ningún dato personal.** La respuesta para un
código inexistente y para un código mal formado es idéntica, para no permitir enumeración,
y va limitada en frecuencia.

### Consecuencias

El justificante se puede regenerar siempre y no ocupa almacenamiento. La verificación por
un tercero deja de ser una promesa del documento y pasa a ser una URL, que es el invariante
III convertido en algo que se puede abrir en una pestaña.

Queda pendiente elegir la librería de PDF y resolver su accesibilidad, que es un frente
aparte del HTML. Anotado como DT-12 en `docs/deuda-tecnica.md`.

---

## Lo que este ADR no decide

Y no lo decide a propósito, porque decidirlo ahora sería decidir sin datos:

- **La librería de generación de PDF** y la ruta hacia PDF/UA (DT-12).
- **El almacenamiento de los ficheros aportados**: sistema de ficheros u objeto remoto.
- **El destino de despliegue concreto** y su base de datos gestionada.
- **La estrategia de caché** de las páginas públicas de convocatorias.

Cada uno tendrá su ADR cuando haya una razón para elegir, no antes.
