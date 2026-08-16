# Dominio — Procedimiento de concesión de ayudas

Documento normativo. El vocabulario de aquí es el que se usa en código, interfaz, tests,
commits y documentación. **No se traduce al inglés en ningún caso.**

---

## 1. Glosario

| Término | Significado en este proyecto |
| --- | --- |
| **Convocatoria** | Llamamiento público a solicitar una ayuda. Tiene bases, plazo de presentación, crédito y órgano instructor. |
| **Bases reguladoras** | Documento legal que fija requisitos, criterios de valoración y obligaciones. |
| **Persona interesada** | Quien solicita. Puede actuar por sí misma o mediante representante. |
| **Representante** | Quien actúa en nombre de la persona interesada, acreditando la representación. |
| **Solicitud** | Formulario cumplimentado. Mientras no se registra, es un **borrador** sin efectos. |
| **Registro** | Acto que da entrada oficial a la solicitud. Genera número y sello de tiempo. Es irreversible. |
| **Justificante** | Documento que acredita el registro. Contiene número, fecha, hora y huella. |
| **Expediente** | Conjunto ordenado de la solicitud, la documentación y las actuaciones. Tiene estado. |
| **Subsanación** | Corrección de defectos u omisiones a requerimiento de la Administración, en plazo. |
| **Requerimiento** | Comunicación que abre un plazo de subsanación. |
| **Plazo** | Tiempo para actuar, contado en **días hábiles** salvo indicación expresa. |
| **Desistimiento** | La persona renuncia a continuar, o se le tiene por desistida al no subsanar en plazo. |
| **Propuesta de resolución** | Documento del instructor previo a la resolución. |
| **Resolución** | Acto que concede o deniega. Pone fin al procedimiento. |
| **Listado provisional / definitivo** | Publicación de personas beneficiarias, antes y después de alegaciones. |
| **Sede electrónica** | El propio portal, con validez para relacionarse con la Administración. |
| **Órgano instructor** | Unidad que tramita. |

Errores frecuentes a evitar: una *solicitud* no es un *expediente*; una *convocatoria* no
es una *ayuda*; *registrar* no es *guardar*; *presentar* no es *enviar*.

## 2. Máquina de estados del expediente

```
                       ┌──────────────┐
                       │   BORRADOR   │  sin efectos legales, editable
                       └──────┬───────┘
                              │ registrar  (irreversible)
                       ┌──────▼───────┐
                       │  REGISTRADO  │  nº de registro + sello de tiempo
                       └──────┬───────┘
                              │ asignar a instructor
                       ┌──────▼───────┐
              ┌────────┤ EN_REVISION  │
     requerir │        └──────┬───────┘
  ┌───────────▼───────────┐   │
  │ SUBSANACION_REQUERIDA │   │ valoración completa de la convocatoria
  └───────────┬───────────┘   │
              │ aporta en plazo
              └────────►──────┤
                              │
                 ┌────────────▼────────────┐
                 │  PROPUESTA_PROVISIONAL  │  abre el plazo de alegaciones
                 └────────────┬────────────┘
                              │ alegaciones resueltas
                 ┌────────────▼────────────┐
                 │  PROPUESTA_DEFINITIVA   │
                 └───┬─────────────────┬───┘
          resolver   │                 │   resolver
         favorable   │                 │   desfavorable
  ┌─────────────────▼───┐         ┌────▼─────────────────────┐
  │ RESUELTO_FAVORABLE  │         │  RESUELTO_DESFAVORABLE   │
  └─────────────────────┘         └──────────────────────────┘

  Terminales alcanzables en paralelo:

  INADMITIDO  ◄── inadmitir (jefe de servicio), desde REGISTRADO y EN_REVISION.
                  Presentación fuera de plazo, falta de legitimación o duplicidad.
                  Deja de ser posible una vez emitida la propuesta provisional.

  DESISTIDO   ◄── desistir (persona interesada), desde REGISTRADO, EN_REVISION,
                  SUBSANACION_REQUERIDA, PROPUESTA_PROVISIONAL y PROPUESTA_DEFINITIVA.
              ◄── vencimiento del plazo de subsanación, con actor «sistema».
```

Diez estados, dieciséis transiciones, ochenta y cinco pares prohibidos. El desarrollo
completo —actores autorizados, precondiciones, efectos y la matriz de adyacencia entera—
está en `docs/modelo-dominio.md` §4. La extensión de ocho a diez estados está justificada
en `docs/decisiones/0003-extension-maquina-estados.md`.

**Invariantes:**

- Toda transición se ejecuta en `lib/dominio/expediente.ts`. Nunca desde un componente,
  nunca desde una Server Action directamente.
- Toda transición registra: estado origen, estado destino, actor, sello de tiempo,
  motivo. La traza es inmutable y consultable.
- Desde `REGISTRADO` en adelante, la solicitud es inmutable. Los cambios se hacen
  aportando documentación nueva, no editando la original.
- **Quien aporta documentación de subsanación devuelve el expediente a `EN_REVISION`**,
  nunca directamente a la propuesta: alguien tiene que revisar lo aportado. Proponer sin
  revisar es indefendible ante una reclamación.
- El vencimiento del plazo de subsanación produce `DESISTIDO` de forma automática. Esa
  transición también se registra, con actor `sistema`.
- La inadmisión tiene ventana: procede hasta la propuesta provisional. A partir de ahí lo
  que corresponde es resolver de forma desfavorable, no inadmitir.
- Una transición no contemplada en el diagrama lanza un error de dominio tipado. No hay
  transición «por defecto».
- **El vencimiento del plazo máximo de resolución no cambia el estado.** Produce silencio
  desestimatorio, que es un derecho de la persona interesada (Ley 38/2003, art. 25.5), no
  una transición. La Administración sigue obligada a resolver.

> **Divergencia conocida con la norma.** El desistimiento automático que fija este
> documento no encaja del todo con el art. 68.1 de la Ley 39/2015, que dice que se le
> tendrá por desistido «previa resolución que deberá ser dictada en los términos
> previstos en el artículo 21». En Derecho hace falta un acto expreso. Se mantiene aquí
> la transición automática por decisión de producto, y la divergencia queda anotada como
> DT-03 en `docs/deuda-tecnica.md`. La alternativa fiel sería un estado intermedio
> `PENDIENTE_DESISTIMIENTO` que el jefe de servicio confirma, y necesitaría su propio ADR.

Cada transición del diagrama tiene su test unitario, incluidas las **transiciones
prohibidas**, que deben fallar de forma explícita.

## 3. Plazos: la regla más delicada del proyecto

Vive en `lib/plazos/`. Sin React, sin I/O, 100 % de cobertura.

- Los plazos por defecto se cuentan en **días hábiles**.
- No son hábiles: sábados, domingos, festivos nacionales, autonómicos y locales.
  El calendario es un dato de configuración versionado, no constantes en el código.
- El cómputo empieza el **día hábil siguiente** a la notificación, no el mismo día.
- Si el vencimiento cae en inhábil, se traslada al siguiente hábil.
- Zona horaria fija `Europe/Madrid`. Cuidado con los cambios de hora: un plazo que
  vence el día del cambio no puede perder ni ganar una hora.
- El sello de tiempo del registro se toma **en servidor**, jamás del cliente.

**Casos de prueba obligatorios** (valores límite, todos con test unitario):

vencimiento en sábado · vencimiento en festivo local · plazo que cruza Navidad ·
plazo que cruza el cambio de hora de marzo y el de octubre · plazo de 0 días ·
notificación a las 23:59:59 · notificación en día inhábil · año bisiesto ·
festivo que cae en domingo · plazo que cruza fin de año.

## 4. Datos sintéticos

**Prohibido** generar identificadores que puedan corresponder a personas reales.

- **NIF**: usar exclusivamente el rango reservado del proyecto `99000000` a `99999999`
  con su letra de control correctamente calculada. Nunca números que empiecen por otra
  cifra.
- **NIE**: prefijo `Z` y el mismo rango reservado.
- **IBAN**: usar el prefijo de pruebas `ES00 0000` con dígitos de control válidos.
  Nunca un IBAN de banco real.
- **Teléfonos**: rango `600 00 00 00`–`600 00 99 99`.
- **Correo**: dominio `@ejemplo.valdeprieto.test`.
- **Nombres**: del fichero `tests/datos/nombres-sinteticos.json`. Nunca nombres de
  personas conocidas.

Todo registro sintético lleva la marca `sintetico: true` y el `run-id` de la ejecución
que lo creó.

## 5. Protección de datos

- Minimización: no se pide un dato que el procedimiento no necesite. Si se pide, hay que
  poder justificar por qué.
- Enmascarado obligatorio de NIF, IBAN y correo en logs, trazas de Playwright, capturas
  de CI y mensajes de error.
- Todo formulario que recoja datos personales muestra la información de tratamiento
  antes del envío, no después.
- Existe y funciona el borrado de borradores a petición.

## 6. El municipio

**Valdeprieto** es ficticio, igual que su escudo, su logotipo y sus direcciones. Antes de
publicar el proyecto, comprueba que no coincide con ningún municipio real del INE y
deja constancia de la comprobación en el README. En todas las pantallas debe verse un
aviso discreto pero inequívoco:

> Portal de demostración. Municipio ficticio. Sin validez administrativa.
