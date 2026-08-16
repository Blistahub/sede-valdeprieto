# Maquetas — Aplicación del sistema de diseño

Documento derivado. `docs/diseno.md` es normativo y aquí no se cambia ni un valor: se
aplica. Si alguna pantalla necesitara un token que no existe, el camino es un ADR, no un
ajuste local.

---

## 0. Calibración: el diseño que habría propuesto sin haber leído el documento

Antes de aplicar nada, el ejercicio honesto. Esto es, con bastante exactitud, lo que
habría salido si me hubieran pedido «un portal de sede electrónica municipal» sin más
contexto:

**Color.** Azul institucional en torno a `#1E4B8F` o `#0B5FFF`, con un gris azulado
`#F5F7FA` de fondo, `#64748B` para texto secundario y la tríada semántica de siempre:
verde `#16A34A` para éxito, ámbar `#F59E0B` para aviso, rojo `#DC2626` para error. Nada
de eso sale del dominio; sale de que es lo que hay en cualquier librería de componentes.

**Tipografía.** Inter para todo, 16 px de cuerpo, 1.5 de interlínea, escala de 12/14/16/20/24/32.
Números en la misma fuente que el texto, sin cifras tabulares, sin distinguir un
identificador legal de una palabra cualquiera.

**Composición.** Un héroe a pantalla completa con una fotografía del ayuntamiento
—probablemente en tonos azulados y con un degradado oscuro encima para que el titular en
blanco se leyera—, un buscador grande centrado, y debajo una rejilla de tres tarjetas con
esquinas de 12 o 16 px, sombra `0 4px 12px rgba(0,0,0,0.08)`, un icono de línea de
Lucide arriba a la izquierda, un titular y un enlace «Saber más →».

**Detalles.** Insignias de estado como píldoras de color pastel con texto del mismo tono
saturado. Iconos sin texto en las acciones de tabla. Una barra de progreso con puntos
azules para los pasos del formulario. Alguna transición de 300 ms con `ease-in-out` en
casi todo. Y un modo oscuro que nadie ha pedido.

**Por qué es peor.** No es feo. Es *idéntico*. No dice nada sobre lo que hace el
producto, trata igual un número de expediente que un titular, comunica el estado legal
con la paleta genérica de una aplicación de tareas, y el héroe con foto ocupa la primera
pantalla completa sin ayudar a nadie a presentar una solicitud. Es el suelo del que hay
que despegar, y lo dejo escrito para poder comprobar en cada pantalla que no he vuelto a
él sin darme cuenta.

---

## 1. Ampliación de la matriz de contraste

`docs/diseno.md` §2 calcula las parejas sobre `--papel`. Estas pantallas usan además
superficies `--papel-alto`, así que faltan filas. Calculadas con la fórmula de luminancia
relativa de WCAG 2.1, no estimadas:

| Combinación | Ratio | Uso |
| --- | --- | --- |
| `--verde-700` sobre `--papel-alto` | 11.08:1 | Titulares sobre tarjeta |
| `--azul` sobre `--papel-alto` | 8.66:1 | Estado «en revisión» sobre tarjeta |
| `--lacre` sobre `--papel-alto` | 8.19:1 | Error y plazo en riesgo sobre tarjeta |
| `--verde-500` sobre `--papel-alto` | 6.39:1 | Estado favorable sobre tarjeta |
| `--gris` sobre `--papel-alto` | 6.06:1 | Texto secundario sobre tarjeta |
| `--linea-fuerte` sobre `--papel-alto` | 3.81:1 | Bordes funcionales sobre tarjeta |
| `--sello` sobre `--papel-alto` | 3.19:1 | **Solo** anillo de foco y cuño. Nunca texto |
| `--linea` sobre `--papel-alto` | 1.59:1 | **Solo** filete decorativo, nunca funcional |

**Estas siete filas son una ampliación de un documento normativo y necesitan ADR antes
de darse por firmes.** No contradicen ninguna fila existente: la prohibición de
`--sello` como texto se mantiene y de hecho se agrava, porque 3.19:1 tampoco llega a los
4.5:1 que exige el texto normal.

**Por qué el anillo de foco tiene dos capas.** Sobre `--papel`, `--sello` da 2.81:1, por
debajo del mínimo de 3:1 de WCAG 1.4.11; la capa interior de `--tinta` lo resuelve con
15.42:1. Sobre `--verde-700`, `--tinta` casi desaparece y es `--sello` quien cumple, con
3.47:1. Ninguna capa sola funciona en los dos fondos. Por eso son dos.

---

## 2. Portada

### Wireframe

```
┌────────────────────────────────────────────────────────────────────────┐
│ Portal de demostración · Municipio ficticio · Sin validez administrativa│ ← aviso
├────────────────────────────────────────────────────────────────────────┤
│  AYUNTAMIENTO DE VALDEPRIETO                    Mis solicitudes  Entrar │ ← verde-700
│  Sede electrónica                                                       │
├────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│  AYUDAS Y SUBVENCIONES                                                  │ ← eyebrow
│                                                                         │
│  Presenta tu solicitud                                                  │ ← Display 56
│  sin salir de casa                                                      │
│                                                                         │
│  Cada trámite tiene su plazo y aquí lo ves siempre en días hábiles.     │ ← cuerpo largo
│                                                                         │
│  ┌──────────────────────────┐  ┌──────────────────────────┐            │
│  │ Ver convocatorias abiertas│  │ Verificar un justificante│            │
│  └──────────────────────────┘  └──────────────────────────┘            │
│   primario, verde-700           secundario, filete                      │
│                                                                         │
│ ─────────────────────────────────────────────────────────────────────── │ ← filete 1px
│                                                                         │
│  CONVOCATORIAS ABIERTAS                                          3      │
│                                                                         │
│  ┌───────────────────────────────────────────────────────────────────┐ │
│  │ CONV-2026-004                                     ● Plazo abierto  │ │
│  │ Ayudas al alquiler de vivienda habitual                            │ │
│  │ Hasta 2.400,00 €   ·   Quedan 12 días hábiles                      │ │
│  │ Termina el 03/09/2026 a las 23:59                                  │ │
│  │ → Ver bases y solicitar                                            │ │
│  └───────────────────────────────────────────────────────────────────┘ │
│  ┌───────────────────────────────────────────────────────────────────┐ │
│  │ CONV-2026-003                                     ● Plazo abierto  │ │
│  │ …                                                                  │ │
│  └───────────────────────────────────────────────────────────────────┘ │
│                                                                         │
├────────────────────────────────────────────────────────────────────────┤
│ Declaración de accesibilidad · Reclamaciones · Privacidad · Contacto    │
└────────────────────────────────────────────────────────────────────────┘
```

**No hay héroe con fotografía.** El primer elemento útil de la página es el listado de
convocatorias abiertas, y está por encima del pliegue en cualquier pantalla de portátil.
La foto del ayuntamiento habría empujado ese listado 700 px hacia abajo a cambio de
nada — y además el ayuntamiento es ficticio, así que la foto tendría que ser falsa.

### Jerarquía tipográfica

| Elemento | Rol de la escala | Valor |
| --- | --- | --- |
| «AYUDAS Y SUBVENCIONES» | Etiqueta / eyebrow | 13/16, Archivo 600, `letter-spacing: 0.08em`, versalitas |
| «Presenta tu solicitud sin salir de casa» | Display | 56/60, Archivo Expanded 800 |
| Párrafo de entrada | Cuerpo largo | 19/32, Source Sans 3 400 |
| «CONVOCATORIAS ABIERTAS» | Etiqueta / eyebrow | 13/16, Archivo 600 |
| Título de cada convocatoria | H3 | 21/28, Archivo 600 |
| `CONV-2026-004`, importe, días, fecha | Dato | 16/24, IBM Plex Mono 500, `tabular-nums` |
| «Plazo abierto» | Auxiliar | 15/24, Source Sans 3 400 |
| Pie | Auxiliar | 15/24 |

La portada es uno de los dos únicos sitios donde se permite el Display de 56 px. El otro
es el justificante.

### Tokens y contraste

| Pareja | Ratio | Dónde |
| --- | --- | --- |
| `--papel-alto` sobre `--verde-700` | 11.08:1 | Texto de la cabecera institucional |
| `--tinta` sobre `--papel` | 15.42:1 | Display, párrafo, títulos |
| `--gris` sobre `--papel` | 5.33:1 | Eyebrow y texto auxiliar |
| `--papel-alto` sobre `--verde-700` | 11.08:1 | Botón primario |
| `--verde-700` sobre `--papel` | 9.74:1 | Texto del botón secundario y su filete |
| `--tinta` sobre `--papel-alto` | 17.54:1 | Contenido de cada tarjeta de convocatoria |
| `--linea-fuerte` sobre `--papel-alto` | 3.81:1 | Borde de la tarjeta |
| `--verde-500` sobre `--papel-alto` | 6.39:1 | Punto e insignia «Plazo abierto» |
| `--linea` sobre `--papel` | 1.39:1 | Filete separador, **decorativo**, `aria-hidden` |

El punto verde de «Plazo abierto» nunca va solo: color, punto y texto. Con las tres
cosas, quitar el color no quita información.

### Orden de tabulación

1. Enlace de salto «Ir al contenido principal»
2. Enlace del escudo / nombre del ayuntamiento (vuelve a portada)
3. «Mis solicitudes»
4. «Entrar»
5. Botón «Ver convocatorias abiertas»
6. Botón «Verificar un justificante»
7. Enlace de la tarjeta 1 → «Ver bases y solicitar. Ayudas al alquiler de vivienda habitual»
8. Enlace de la tarjeta 2
9. Enlace de la tarjeta 3
10. «Declaración de accesibilidad»
11. «Reclamaciones»
12. «Privacidad»
13. «Contacto»

El aviso de demostración **no** está en el orden de tabulación: es texto, no un control.
Sí lo lee el lector de pantalla, porque es lo primero del documento.

### Lector de pantalla

**Al llegar:**

> «Sede electrónica del Ayuntamiento de Valdeprieto. Portal de demostración, municipio
> ficticio, sin validez administrativa. Enlace, ir al contenido principal. Navegación
> principal, lista de 2 elementos… Contenido principal. Encabezado de nivel 1: Presenta
> tu solicitud sin salir de casa. … Encabezado de nivel 2: Convocatorias abiertas, 3
> convocatorias.»

**Al completar la acción principal** (activar el enlace de una convocatoria): navegación
de página completa. El foco se sitúa en el `<h1>` del detalle y se anuncia:

> «Ayudas al alquiler de vivienda habitual. Convocatoria CONV guion 2026 guion 004.»

El número se marca con `aria-label` para que se lea agrupado y no como «cero cero
cuatro» seguido.

### A 320 px

- Cabecera institucional a dos líneas; «Mis solicitudes» y «Entrar» pasan a un menú con
  `button` y `aria-expanded`, nunca una casilla oculta con CSS.
- Display baja a 36/44 (rol H1 de la escala). El Display de 56 px no cabe en 320 px sin
  partir palabras.
- Los dos botones se apilan a ancho completo, 44 px de alto mínimo, 12 px de separación.
- Las tarjetas pasan a una columna. Dentro de cada una, el orden se mantiene: código,
  estado, título, importe, plazo.
- El pie apila sus cuatro enlaces en vertical con 44 px de área táctil cada uno.
- Sin scroll horizontal a 320 px y con zoom al 200 %.

### Autocrítica

**Resuelto.** No hay héroe decorativo: lo primero accionable es la lista de
convocatorias con su plazo en días hábiles. El Display institucional en Archivo Expanded
da autoridad sin recurrir a una fotografía ni a un escudo, que además serían falsos.

**Todavía genérico.** La tarjeta de convocatoria es una tarjeta. Filete de 1 px en lugar
de sombra, esquinas de 4 px, sin degradado — pero sigue siendo la misma caja rectangular
con título y enlace que tiene cualquier portal. El material del que dice salir el diseño
—la ficha de archivo, el sobre, la carpeta con solapa— no ha llegado a esta pantalla.

**Qué eliminaría.** El botón «Verificar un justificante» de la primera pantalla. Es una
función de tercero, no de quien llega a solicitar, y compite con la acción principal. Su
sitio está en el pie y en el propio justificante.

---

## 3. Detalle de convocatoria

### Wireframe

```
┌────────────────────────────────────────────────────────────────────────┐
│ Portal de demostración · Municipio ficticio · Sin validez administrativa│
├────────────────────────────────────────────────────────────────────────┤
│  AYUNTAMIENTO DE VALDEPRIETO                    Mis solicitudes  Entrar │
├────────────────────────────────────────────────────────────────────────┤
│ Inicio › Convocatorias › Ayudas al alquiler                             │ ← migas
├──────────────────────────────────────────┬─────────────────────────────┤
│                                          │  PLAZO DE PRESENTACIÓN      │
│ CONV-2026-004                            │  ─────────────────────      │
│                                          │                             │
│ Ayudas al alquiler de vivienda           │  Quedan                     │
│ habitual                                 │  12 días hábiles            │ ← dato 21px
│                                          │                             │
│ ┌──────────────────────────────────────┐ │  Termina el                 │
│ │ RESUMEN ORIENTATIVO                  │ │  03/09/2026 a las 23:59     │
│ │ Sin valor jurídico                   │ │                             │
│ │                                      │ │  ┌───────────────────────┐  │
│ │ Puedes pedir esta ayuda si estás     │ │  │ Solicitar esta ayuda  │  │
│ │ empadronada en Valdeprieto desde     │ │  └───────────────────────┘  │
│ │ hace un año…                         │ │                             │
│ └──────────────────────────────────────┘ │  ┌───────────────────────┐  │
│                                          │  │ Descargar las bases   │  │
│ Bases reguladoras                        │  │ PDF · 412 KB          │  │
│ ─────────────────────────────────────    │  └───────────────────────┘  │
│                                          │                             │
│ Primera. Objeto y finalidad.             │  DATOS DE LA CONVOCATORIA   │
│ Las presentes bases tienen por objeto    │  ─────────────────────      │
│ regular la concesión…                    │  Importe máximo             │
│                                          │  2.400,00 €                 │
│ [texto legal íntegro, 60-72 caracteres]  │  Crédito total              │
│                                          │  120.000,00 €               │
│                                          │  Órgano instructor          │
│                                          │  Servicio de Bienestar      │
├──────────────────────────────────────────┴─────────────────────────────┤
│ Declaración de accesibilidad · Reclamaciones · Privacidad · Contacto    │
└────────────────────────────────────────────────────────────────────────┘
```

El resumen en lenguaje claro va **antes** del texto legal en el orden de lectura, con su
etiqueta como parte del encabezado. La columna de texto legal no pasa de 72 caracteres
de medida aunque la pantalla tenga 2.000 px.

### Jerarquía tipográfica

| Elemento | Rol | Valor |
| --- | --- | --- |
| Migas de pan | Auxiliar | 15/24 |
| `CONV-2026-004` | Dato | 16/24, IBM Plex Mono 500 |
| «Ayudas al alquiler de vivienda habitual» | H1 | 36/44, Archivo 700 |
| «RESUMEN ORIENTATIVO / Sin valor jurídico» | Etiqueta | 13/16, Archivo 600, versalitas |
| Texto del resumen | Cuerpo | 17/28 |
| «Bases reguladoras» | H2 | 28/36, Archivo 700 |
| «Primera. Objeto y finalidad.» | H3 | 21/28, Archivo 600 |
| Texto legal | Cuerpo largo | 19/32 |
| «PLAZO DE PRESENTACIÓN» | Etiqueta | 13/16 |
| «12 días hábiles» | Dato, ampliado | 21/28, IBM Plex Mono 500, `tabular-nums` |
| Fechas e importes | Dato | 16/24 |

### Tokens y contraste

| Pareja | Ratio | Dónde |
| --- | --- | --- |
| `--tinta` sobre `--papel` | 15.42:1 | H1, texto legal, datos |
| `--gris` sobre `--papel` | 5.33:1 | Migas, etiquetas de sección |
| `--tinta` sobre `--papel-alto` | 17.54:1 | Interior del bloque de resumen |
| `--linea-fuerte` sobre `--papel` | 3.35:1 | Filete del bloque de resumen y del raíl |
| `--papel-alto` sobre `--verde-700` | 11.08:1 | Botón «Solicitar esta ayuda» |
| `--verde-700` sobre `--papel` | 9.74:1 | Botón secundario «Descargar las bases» |
| `--verde-500` sobre `--papel` | 5.62:1 | Insignia de plazo abierto |
| `--lacre` sobre `--papel` | 7.20:1 | Aviso de último día, si procede |
| `--linea` sobre `--papel` | 1.39:1 | Filete bajo «Bases reguladoras», decorativo |

El bloque del resumen se distingue por superficie (`--papel-alto`) y filete, no por un
fondo de color pastel. Un fondo pastel con texto del mismo tono es el patrón que rompe
el contraste sin que nadie lo note.

### Orden de tabulación

1. «Ir al contenido principal»
2. Escudo / nombre del ayuntamiento
3. «Mis solicitudes»
4. «Entrar»
5. Miga «Inicio»
6. Miga «Convocatorias»
7. Botón «Solicitar esta ayuda»
8. Enlace «Descargar las bases. PDF, 412 KB»
9. Enlaces dentro del texto legal, en orden de aparición
10. «Declaración de accesibilidad»
11. «Reclamaciones»
12. «Privacidad»
13. «Contacto»

El raíl lateral aparece **después** de la columna de trabajo en el DOM, pero sus dos
botones se colocan antes en el orden de tabulación mediante su posición real en el
documento, no con `tabindex` positivo, que está prohibido. Es decir: el bloque de
plazo y acción se escribe en el HTML antes del texto legal, y la maquetación lo sitúa a
la derecha. El orden visual y el de lectura coinciden.

### Lector de pantalla

**Al llegar:**

> «Ayudas al alquiler de vivienda habitual. Encabezado de nivel 1. Convocatoria CONV
> 2026 004. Región complementaria: Plazo de presentación. Quedan 12 días hábiles. El
> plazo termina el 3 de septiembre de 2026 a las 23:59. Botón, solicitar esta ayuda.»

**Al completar la acción principal** (descargar las bases): el enlace anuncia por sí
mismo «Descargar las bases reguladoras. PDF, 412 kilobytes», y tras la descarga se
anuncia en región `polite`:

> «Descarga iniciada. Bases reguladoras de la convocatoria CONV 2026 004.»

### A 320 px

- El raíl lateral se convierte en el **primer** bloque bajo el H1: plazo, botón
  «Solicitar esta ayuda» y botón «Descargar las bases», en ese orden y a ancho completo.
  Los datos de la convocatoria bajan al final.
- El resumen orientativo mantiene su posición previa al texto legal.
- El texto legal reduce su medida pero conserva 19/32: no se baja el tamaño de cuerpo en
  móvil, porque el público objetivo lo necesita igual o más.
- Sin barra fija: en esta pantalla no hay expediente y por tanto no hay estado que fijar
  arriba.

### Autocrítica

**Resuelto.** El resumen en lenguaje claro va antes del texto legal, etiquetado en el
propio encabezado, y el texto legal se muestra íntegro. La jerarquía deja claro que el
resumen orienta y la norma manda, sin ocultar ninguno de los dos.

**Todavía genérico.** El raíl derecho con botones y datos es el patrón de barra lateral
de cualquier producto. Funciona, pero no es lo que sugiere `docs/diseno.md` §1: no hay
en él nada del cuño, del sobre ni de la ficha de archivo. Es una barra lateral con
tipografía buena.

**Qué eliminaría.** El bloque «Datos de la convocatoria» del raíl. Importe máximo,
crédito y órgano instructor ya están en las bases, y repetirlos obliga a mantener dos
fuentes de la misma cifra. Con dos versiones, tarde o temprano una miente.

---

## 4. Formulario, paso 3 de 5 — Documentación

### Wireframe

```
┌────────────────────────────────────────────────────────────────────────┐
│ Portal de demostración · Municipio ficticio · Sin validez administrativa│
├────────────────────────────────────────────────────────────────────────┤
│  AYUNTAMIENTO DE VALDEPRIETO                            Marta R. · Salir│
├────────────────────────────────────────────────────────────────────────┤
│ Inicio › Convocatorias › Ayudas al alquiler › Solicitud                 │
├──────────────────────────────────────────┬─────────────────────────────┤
│                                          │  TU SOLICITUD               │
│ Paso 3 de 5 · Documentación              │  ─────────────────────      │
│                                          │  ① Datos personales    ✓    │
│ Documentación                            │  ② Datos de la ayuda   ✓    │
│                                          │  ③ Documentación       ●    │
│ Necesitas aportar dos documentos. Puedes │  ④ Revisión            —    │
│ subirlos en PDF, JPG o PNG, hasta 10 MB  │  ⑤ Registro            —    │
│ cada uno.                                │                             │
│                                          │  Borrador guardado          │
│ ┌────────────────────────────────────────┤  16/08/2026 12:41           │
│ │ ⚠ Falta un documento obligatorio       │                             │
│ │   · Certificado de empadronamiento     │  Quedan 12 días hábiles     │
│ └────────────────────────────────────────┤  para presentar             │
│                                          │                             │
│ Certificado de empadronamiento *         │                             │
│ Lo expide el propio ayuntamiento. Debe   │                             │
│ tener menos de 3 meses.                  │                             │
│ ┌──────────────────────────────────────┐ │                             │
│ │ Seleccionar archivo                  │ │                             │
│ └──────────────────────────────────────┘ │                             │
│ ⚠ Necesitas aportar este documento.      │ ← lacre                     │
│                                          │                             │
│ Contrato de arrendamiento *              │                             │
│ Firmado por las dos partes.              │                             │
│ ┌──────────────────────────────────────┐ │                             │
│ │ Seleccionar archivo                  │ │                             │
│ └──────────────────────────────────────┘ │                             │
│ ✓ contrato-alquiler.pdf · 1,2 MB         │                             │
│   Huella 9f2a…c71d      [ Quitar ]       │                             │
│                                          │                             │
│ ┌──────────┐          ┌────────────────┐ │                             │
│ │  Atrás   │          │  Continuar     │ │                             │
│ └──────────┘          └────────────────┘ │                             │
├──────────────────────────────────────────┴─────────────────────────────┤
│ Declaración de accesibilidad · Reclamaciones · Privacidad · Contacto    │
└────────────────────────────────────────────────────────────────────────┘
```

Columna de trabajo a 640 px máximo. El raíl numerado está justificado porque el
procedimiento *es* una secuencia legal ordenada, no una decoración.

### Jerarquía tipográfica

| Elemento | Rol | Valor |
| --- | --- | --- |
| «Paso 3 de 5 · Documentación» | Etiqueta | 13/16, Archivo 600, versalitas |
| «Documentación» | H1 | 36/44, Archivo 700 |
| Texto de introducción | Cuerpo | 17/28 |
| Resumen de errores | Cuerpo | 17/28, Source Sans 3 400 |
| Etiqueta de campo | Cuerpo | 17/28, Source Sans 3 600 |
| Texto de ayuda del campo | Auxiliar | 15/24 |
| Mensaje de error del campo | Auxiliar | 15/24 |
| Nombre y tamaño del fichero | Dato | 16/24, IBM Plex Mono 500 |
| Huella | Dato | 16/24, IBM Plex Mono 500 |
| Raíl: pasos | Auxiliar | 15/24 |
| Raíl: fecha de guardado y días hábiles | Dato | 16/24 |

La etiqueta de campo va a 17 px en peso 600, no a 15 px. Reducir la etiqueta para que
«respire» es el error clásico: la etiqueta es el elemento más importante del campo.

### Tokens y contraste

| Pareja | Ratio | Dónde |
| --- | --- | --- |
| `--tinta` sobre `--papel` | 15.42:1 | H1, etiquetas, texto |
| `--gris` sobre `--papel` | 5.33:1 | Texto de ayuda y raíl |
| `--linea-fuerte` sobre `--papel` | 3.35:1 | Borde de los campos y del botón secundario |
| `--lacre` sobre `--papel` | 7.20:1 | Icono y texto de error, borde del campo en error |
| `--papel-alto` sobre `--verde-700` | 11.08:1 | Botón «Continuar» |
| `--verde-700` sobre `--papel` | 9.74:1 | Botón «Atrás» y su filete |
| `--verde-500` sobre `--papel` | 5.62:1 | Marca de paso completado y de fichero correcto |
| `--azul` sobre `--papel` | 7.62:1 | Punto del paso en curso |
| `--tinta` + `--sello` | 15.42 / 3.47 | Anillo de foco de dos capas |

**El campo en error se marca con tres cosas**: borde `--lacre` de 2 px, icono de aviso y
texto. Quien no distinga el rojo del gris sigue viendo el icono y leyendo el mensaje.

### Orden de tabulación

1. «Ir al contenido principal»
2. Escudo / nombre del ayuntamiento
3. Menú de sesión «Marta R.»
4. «Salir»
5. Miga «Inicio»
6. Miga «Convocatorias»
7. Miga «Ayudas al alquiler»
8. Enlace del resumen de errores → «Certificado de empadronamiento»
9. Campo de fichero «Certificado de empadronamiento»
10. Campo de fichero «Contrato de arrendamiento»
11. Botón «Quitar el contrato de arrendamiento»
12. Botón «Atrás»
13. Botón «Continuar»
14. «Declaración de accesibilidad»
15. «Reclamaciones»
16. «Privacidad»
17. «Contacto»

La última miga —«Solicitud», la página actual— no es un enlace y por tanto no ocupa
posición. Marcarla con `aria-current="page"` y dejarla como texto evita un enlace que no
lleva a ninguna parte.

El raíl de pasos **no** es tabulable: es un indicador de progreso, no un navegador. Se
lee como lista ordenada y el paso actual lleva `aria-current="step"`.

### Lector de pantalla

**Al llegar** (tras un intento fallido de continuar):

> «Documentación. Encabezado de nivel 1. Paso 3 de 5. Alerta: Falta un documento
> obligatorio. Lista de 1 elemento. Enlace: Certificado de empadronamiento.»

El foco viaja al resumen de errores, que es una región `role="alert"`. Desde ahí, activar
el enlace lleva el foco al campo.

**En el campo en error:**

> «Certificado de empadronamiento. Obligatorio. Campo de selección de archivo. No
> válido. Lo expide el propio ayuntamiento, debe tener menos de 3 meses. Necesitas
> aportar este documento.»

El texto de ayuda y el mensaje de error se enlazan los dos con `aria-describedby`, en
ese orden.

**Al completar la acción principal** (aportar un documento válido), en región `polite`:

> «Certificado de empadronamiento aportado. certificado-empadronamiento.pdf, 840
> kilobytes. Ya no falta ningún documento obligatorio.»

El foco no se mueve: la persona sigue donde estaba. Mover el foco tras una subida
correcta desorienta.

**Al continuar al paso 4:** el foco va al `<h1>` del paso siguiente y se anuncia «Revisión.
Encabezado de nivel 1. Paso 4 de 5». La transición es uno de los tres movimientos
aprobados: 8 px y opacidad, 180 ms, eliminada con `prefers-reduced-motion: reduce`.

### A 320 px

- El raíl colapsa a una **barra fija superior** con dos datos y nada más: «Paso 3 de 5 ·
  Documentación» y «Quedan 12 días hábiles». No tapa contenido —el cuerpo compensa con
  padding superior— y no atrapa el foco.
- Los campos ocupan el ancho completo; el control de archivo mantiene 44 px de alto.
- «Atrás» y «Continuar» se apilan con «Continuar» arriba: la acción principal primero.
- El resumen de errores queda inmediatamente bajo el H1, antes del primer campo.
- La huella del fichero se parte en dos líneas manteniendo la fuente de dato; no se
  trunca con puntos suspensivos, porque una huella truncada no sirve para comparar.

### Autocrítica

**Resuelto.** El circuito completo del error: resumen con `role="alert"`, foco al primer
campo con error, enlace desde el resumen, `aria-describedby` con ayuda y error, y tres
portadores de la información (borde, icono, texto). Y la huella del documento visible
desde el momento de aportarlo, no solo en el justificante.

**Todavía genérico.** El paso 3 es un formulario vertical con dos campos y dos botones.
Cualquier producto lo resuelve igual. La única decisión propia es la tipografía del dato,
y no basta para que esta pantalla se distinga de un formulario de alta de cualquier SaaS.

**Qué eliminaría.** El texto «Borrador guardado 16/08/2026 12:41» del raíl. Compite con
el dato que de verdad importa —los días hábiles restantes— y quien está subiendo
documentos no necesita saber a qué hora se guardó. Su sitio es la pantalla de «Mis
solicitudes».

---

## 5. Justificante sellado

Es el elemento firma. Todo lo demás calla para que este momento suene.

### Wireframe

```
┌────────────────────────────────────────────────────────────────────────┐
│ Portal de demostración · Municipio ficticio · Sin validez administrativa│
├────────────────────────────────────────────────────────────────────────┤
│  AYUNTAMIENTO DE VALDEPRIETO                            Marta R. · Salir│
├────────────────────────────────────────────────────────────────────────┤
│                                                                         │
│    ┌───────────────────────────────────────────────────────────────┐   │
│    │                                                    ╭────────╮ │   │
│    │  JUSTIFICANTE DE PRESENTACIÓN                      │ ██████ │ │   │
│    │                                                    │ CUÑO   │ │   │
│    │  Solicitud registrada                              │ REG.   │ │   │
│    │                                                    ╰────────╯ │   │
│    │  REG · 2026 · 001427                                          │   │ ← Display 56 mono
│    │                                                               │   │
│    │  16 de agosto de 2026, 12:47:03                                │   │
│    │                                                               │   │
│    │  ───────────────────────────────────────────────────────────  │   │
│    │                                                               │   │
│    │  Convocatoria      CONV-2026-004                              │   │
│    │                    Ayudas al alquiler de vivienda habitual    │   │
│    │  Expediente        EXP-2026-000318                            │   │
│    │  Documentos        2 documentos presentados                   │   │
│    │  Huella            9f2a41c8…e07c71d3                          │   │
│    │  Código seguro     K7M4 P2XR 9TQB 5FHN                        │   │
│    │  de verificación                                              │   │
│    │                                                               │   │
│    └───────────────────────────────────────────────────────────────┘   │
│                                                                         │
│    ┌────────────────────────┐   ┌────────────────────────────────┐     │
│    │ Descargar justificante │   │ Ver el estado de mi expediente │     │
│    └────────────────────────┘   └────────────────────────────────┘     │
│                                                                         │
│    Guarda este código. Con él cualquiera puede comprobar que este       │
│    justificante es auténtico en valdeprieto.test/verificar              │
│                                                                         │
├────────────────────────────────────────────────────────────────────────┤
│ Declaración de accesibilidad · Reclamaciones · Privacidad · Contacto    │
└────────────────────────────────────────────────────────────────────────┘
```

El número de registro es `REG-2026-001427` —un único formato, el que fija
`docs/modelo-dominio.md` §3.6— y se **presenta** con los grupos separados para poder
dictarlo por teléfono: «registro, dos mil veintiséis, cero cero uno cuatro dos siete».
El separador es de presentación; el identificador que se copia, se busca y se compara es
siempre `REG-2026-001427`.

### Jerarquía tipográfica

| Elemento | Rol | Valor |
| --- | --- | --- |
| «JUSTIFICANTE DE PRESENTACIÓN» | Etiqueta | 13/16, Archivo 600, versalitas |
| «Solicitud registrada» | H1 | 36/44, Archivo 700 |
| `REG-2026-001427`, mostrado `REG · 2026 · 001427` | Display | 56/60, IBM Plex Mono 500, `tabular-nums` |
| Sello de tiempo | Dato | 16/24, IBM Plex Mono 500 |
| Etiquetas de la ficha | Auxiliar | 15/24 |
| Valores de la ficha | Dato | 16/24, IBM Plex Mono 500 |
| Texto sobre el código | Cuerpo | 17/28 |

Es el único sitio, junto con la portada, donde se usa el Display de 56 px — y aquí en la
monoespaciada, no en Archivo Expanded, porque lo que se agranda es un identificador
legal, no un titular.

### Tokens y contraste

| Pareja | Ratio | Dónde |
| --- | --- | --- |
| `--tinta` sobre `--papel-alto` | 17.54:1 | Todo el texto del justificante |
| `--linea-fuerte` sobre `--papel-alto` | 3.81:1 | Filete perimetral de 1 px |
| `--gris` sobre `--papel-alto` | 6.06:1 | Etiquetas de la ficha |
| `--tinta` sobre `--sello` | 5.50:1 | Texto dentro del cuño |
| `--sello` sobre `--papel-alto` | 3.19:1 | Contorno del cuño, elemento no textual |
| `--papel-alto` sobre `--verde-700` | 11.08:1 | Botón «Descargar justificante» |
| `--verde-700` sobre `--papel` | 9.74:1 | Botón secundario |
| `--linea` sobre `--papel-alto` | 1.59:1 | Filete horizontal interior, decorativo |

**Sin sombra.** El justificante es papel sobre papel: se separa por superficie
(`--papel-alto` sobre `--papel`) y por filete. Una sombra flotante lo convertiría en una
tarjeta de aplicación.

### Orden de tabulación

Al confirmarse el registro, el foco se coloca por programa en el botón **Descargar
justificante**. A partir de ahí:

1. Botón «Descargar justificante» ← **foco inicial**
2. Botón «Ver el estado de mi expediente»
3. Enlace «valdeprieto.test/verificar»
4. Escudo / nombre del ayuntamiento
5. Menú de sesión
6. «Salir»
7–10. Enlaces del pie

El cuño es decorativo: `aria-hidden="true"` y fuera del orden de tabulación. Toda su
información —que la solicitud está registrada, y su número— está en texto.

### Lector de pantalla

**Al completar la acción principal** (registrar), en `aria-live="polite"`:

> «Solicitud registrada con el número REG 2026 001427, el 16 de agosto de 2026 a las
> 12:47 y 3 segundos.»

Y a continuación el foco aterriza en el botón, que se anuncia:

> «Descargar justificante. Botón.»

**Al recorrer la ficha**, cada pareja se lee junta porque es una lista de descripción:

> «Convocatoria: CONV 2026 004, Ayudas al alquiler de vivienda habitual. Expediente: EXP
> 2026 000318. Documentos: 2 documentos presentados. Huella: 9f2a41c8, puntos
> suspensivos, e07c71d3. Código seguro de verificación: K7M4 P2XR 9TQB 5FHN.»

El código se marca para que se deletree grupo a grupo. Un lector de pantalla que lea
«K7M4» como una palabra hace inservible el código.

**Movimiento.** El formulario se retira en 200 ms con `ease-out`; el cuño se asienta con
escala 1.06 → 1.00 y opacidad 0 → 1 en 320 ms, un solo movimiento, sin rebote. Con
`prefers-reduced-motion: reduce` el justificante aparece completo y el anuncio y el foco
funcionan exactamente igual. **Nada de confeti, sonido, icono animado ni marca de
verificación que se dibuja.**

### A 320 px

- El justificante ocupa el ancho completo menos 16 px de margen a cada lado.
- El Display de 56 px baja a 36/44 manteniendo la monoespaciada y los grupos separados;
  a 320 px, `REG · 2026 · 001427` en 56 px se saldría de la pantalla.
- El cuño se coloca bajo el número, centrado, a 96 px. No se elimina: es el elemento
  firma y es lo único que se permite ocupar espacio sin aportar texto.
- La ficha pasa de dos columnas a etiqueta sobre valor.
- Los dos botones se apilan a ancho completo con «Descargar justificante» arriba.
- La huella se parte en dos líneas; el código de verificación **nunca** se parte, porque
  hay que poder seleccionarlo entero.

### Autocrítica

**Resuelto.** El número de registro es el elemento más grande de la pantalla, en la
fuente correcta, agrupado para dictarse, con el sello de tiempo al segundo debajo. El
foco y el anuncio funcionan igual con o sin animación. Y el código de verificación
convierte el justificante en algo que un tercero puede comprobar, que es el invariante
III hecho pantalla.

**Todavía genérico.** El cuño es, ahora mismo, un rectángulo de latón con texto. La
descripción de `docs/diseno.md` §1 habla de un cuño de registro real: tinta irregular,
borde mordido, ligera rotación. Sin ese trabajo de ilustración, el elemento firma no
firma nada; es una insignia dorada.

**Qué eliminaría.** El botón «Ver el estado de mi expediente». En el momento del
registro solo hay una cosa que hacer: guardar el justificante. El estado del expediente
está a un clic desde «Mis solicitudes» y aquí compite con la única acción que importa.

---

## 6. Bandeja del tramitador

### Wireframe

```
┌────────────────────────────────────────────────────────────────────────┐
│ Portal de demostración · Municipio ficticio · Sin validez administrativa│
├────────────────────────────────────────────────────────────────────────┤
│  AYUNTAMIENTO DE VALDEPRIETO · GESTIÓN            N. Tejedor · Tramitadora │
├────────────────────────────────────────────────────────────────────────┤
│ Inicio › Bandeja                                                        │
├────────────────────────────────────────────────────────────────────────┤
│ Bandeja de expedientes                                                  │
│                                                                         │
│ Convocatoria [ Todas          ▾ ]  Estado [ Todos        ▾ ]  [Filtrar] │
│                                                                         │
│ 47 expedientes · 6 con plazo en riesgo                                  │
│                                                                         │
│ ┌───────────┬────────────┬──────────────┬──────────┬───────────┬──────┐ │
│ │ Expediente│ Registro   │ Estado       │ Plazo    │ Asignado  │      │ │
│ ├───────────┼────────────┼──────────────┼──────────┼───────────┼──────┤ │
│ │ EXP-2026- │ 12/08/2026 │ ⚠ Subsanación│ 2 días   │ N.Tejedor │ Abrir│ │
│ │ 000318    │ 09:14:22   │   requerida  │ hábiles  │           │      │ │
│ ├───────────┼────────────┼──────────────┼──────────┼───────────┼──────┤ │
│ │ EXP-2026- │ 12/08/2026 │ ⏱ En revisión│ —        │ N.Tejedor │ Abrir│ │
│ │ 000317    │ 08:52:07   │              │          │           │      │ │
│ ├───────────┼────────────┼──────────────┼──────────┼───────────┼──────┤ │
│ │ EXP-2026- │ 11/08/2026 │ ⬤ Registrado │ —        │ Sin       │ Abrir│ │
│ │ 000316    │ 17:03:41   │              │          │ asignar   │      │ │
│ └───────────┴────────────┴──────────────┴──────────┴───────────┴──────┘ │
│                                                                         │
│ Página 1 de 5 · [1] 2 3 4 5 · Siguiente                                 │
├────────────────────────────────────────────────────────────────────────┤
│ Declaración de accesibilidad · Reclamaciones · Privacidad · Contacto    │
└────────────────────────────────────────────────────────────────────────┘
```

Tabla con `<caption>`, `<th scope="col">`, radio 0 y orden accesible. Paginación con
números y total. **Nada de scroll infinito** en un listado de expedientes.

### Jerarquía tipográfica

| Elemento | Rol | Valor |
| --- | --- | --- |
| «Bandeja de expedientes» | H1 | 36/44, Archivo 700 |
| Etiquetas de filtro | Auxiliar | 15/24 |
| «47 expedientes · 6 con plazo en riesgo» | Cuerpo | 17/28 |
| Encabezados de columna | Etiqueta | 13/16, Archivo 600, versalitas |
| Número de expediente | Dato | 16/24, IBM Plex Mono 500 |
| Fecha y hora de registro | Dato | 16/24, `tabular-nums` |
| Texto de estado | Auxiliar | 15/24 |
| Días hábiles restantes | Dato | 16/24, `tabular-nums` |
| Paginación | Dato | 16/24 |

Las cifras tabulares no son un capricho aquí: son lo que permite comparar 47 fechas de
registro en columna sin leerlas una a una.

### Tokens y contraste

| Pareja | Ratio | Dónde |
| --- | --- | --- |
| `--papel-alto` sobre `--verde-700` | 11.08:1 | Cabecera de gestión |
| `--tinta` sobre `--papel` | 15.42:1 | Contenido de las celdas |
| `--gris` sobre `--papel` | 5.33:1 | Encabezados de columna, «Sin asignar» |
| `--linea-fuerte` sobre `--papel` | 3.35:1 | Filetes de la tabla y bordes de los `select` |
| `--lacre` sobre `--papel` | 7.20:1 | Estado «Subsanación requerida» y días en riesgo |
| `--azul` sobre `--papel` | 7.62:1 | Estado «En revisión» |
| `--sello` sobre `--papel` | 2.81:1 | **Prohibido como texto.** El estado «Registrado» usa fondo `--sello` con texto `--tinta` |
| `--tinta` sobre `--sello` | 5.50:1 | Texto del estado «Registrado» |
| `--verde-700` sobre `--papel` | 9.74:1 | Enlace «Abrir» |

La fila «Registrado» es la trampa del sistema: si alguien escribiera el texto en
`--sello` sobre `--papel` obtendría 2.81:1 y un fallo de accesibilidad. El estado se
resuelve al revés, con el latón de fondo y la tinta encima.

### Orden de tabulación

1. «Ir al contenido principal»
2. Escudo / nombre (área de gestión)
3. Menú de sesión «N. Tejedor · Tramitadora»
4. Miga «Inicio»
5. Selector «Convocatoria»
6. Selector «Estado»
7. Botón «Filtrar»
8. Botón de orden del encabezado «Expediente»
9. Botón de orden del encabezado «Registro»
10. Botón de orden del encabezado «Estado»
11. Botón de orden del encabezado «Plazo»
12. Enlace «Abrir el expediente EXP-2026-000318»
13. Enlace «Abrir el expediente EXP-2026-000317»
14. Enlace «Abrir el expediente EXP-2026-000316»
15. Enlaces de paginación, en orden: 2, 3, 4, 5, «Siguiente»
16–19. Enlaces del pie

Cada «Abrir» lleva en su nombre accesible el número de expediente. Diez enlaces que se
llaman «Abrir» son diez enlaces indistinguibles para quien navega por lista de enlaces.

### Lector de pantalla

**Al llegar:**

> «Bandeja de expedientes. Encabezado de nivel 1. 47 expedientes, 6 con plazo en riesgo.
> Tabla: Expedientes de las convocatorias asignadas al Servicio de Bienestar, ordenada
> por fecha de registro descendente. 6 columnas, 10 filas.»

**Al recorrer una fila:**

> «EXP 2026 000318. Registro: 12 de agosto de 2026, 09:14:22. Estado: Subsanación
> requerida. Plazo: quedan 2 días hábiles. Asignado a: Nuria Tejedor Peñalver. Enlace: Abrir el
> expediente EXP 2026 000318.»

**Al completar la acción principal** (aplicar un filtro), en región `polite`:

> «12 expedientes con subsanación requerida. 6 con plazo en riesgo.»

El foco **no** vuelve al principio de la página: se queda en el botón «Filtrar». Devolver
el foco arriba obliga a rehacer todo el recorrido con cada filtro, y en una bandeja de
trabajo eso son cientos de tabulaciones al día.

**Al ordenar una columna:** el botón del encabezado cambia su `aria-sort` y se anuncia
«Registro, ordenado ascendente».

### A 320 px

- La tabla se reorganiza en fichas, una por expediente, **sin perder la relación entre
  dato y concepto**: cada valor conserva su etiqueta visible.
- Orden dentro de la ficha: número de expediente, estado, plazo, fecha de registro,
  asignación, acción.
- Los filtros se apilan a ancho completo y el botón «Filtrar» queda fijo bajo ellos.
- La paginación mantiene números y total; a 320 px se muestran la página actual, la
  anterior y la siguiente, más «Primera» y «Última». Nunca se sustituye por «cargar
  más».
- Sin scroll horizontal a 320 px ni con zoom al 200 %.

### Autocrítica

**Resuelto.** El estado «Registrado» está resuelto al revés —latón de fondo, tinta
encima— para no caer en la trampa de 2.81:1 del propio sistema. Los enlaces de acción
llevan el número de expediente en su nombre accesible. Y el foco no salta al filtrar,
que es la diferencia entre una tabla usable ocho horas al día y una que no.

**Todavía genérico.** La bandeja es una tabla con filtros y paginación. Es exactamente
lo que sería en cualquier producto de gestión. El área de gestión ha recibido menos
atención de dirección visual que el área ciudadana, y se nota: la cabecera verde es lo
único que la ata al sistema.

**Qué eliminaría.** La columna «Asignado». En una bandeja que ya está filtrada por el
propio tramitador, repite en cada fila un dato que casi siempre es el mismo. Convertirla
en un filtro —«Míos» / «Sin asignar» / «Todos»— libera una columna entera, que a 320 px
es media pantalla.

---

## 7. Lo que este documento deja pendiente

1. **Las siete filas nuevas de la matriz de contraste** necesitan un ADR antes de darse
   por firmes, porque amplían un documento normativo.
2. **El cuño de registro necesita trabajo de ilustración.** Sin él, el elemento firma es
   una insignia dorada y `docs/diseno.md` §7 promete otra cosa.
3. **El área de gestión necesita una pasada de dirección propia.** Cuatro de las cinco
   pantallas tienen decisiones defendibles; la bandeja todavía no.
4. **La autocrítica coincide en las cinco pantallas**: lo resuelto es el comportamiento
   accesible, lo genérico es la forma. El sistema de tokens está bien aplicado y el
   material del que dice salir el diseño —el archivo municipal— apenas ha llegado más
   allá del color y de la tipografía.
