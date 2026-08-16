# 0003 · Extensión de la máquina de estados del expediente

**Estado:** aceptado · **Fecha:** 2026-08-16 · **Afecta a:** `docs/dominio.md` §2

---

## Contexto

El diagrama de `docs/dominio.md` §2 define ocho estados y un recorrido lineal. Al
desarrollarlo hasta el detalle que exige el invariante II —tabla exhaustiva de
transiciones y enumeración de las prohibidas— aparecieron cuatro huecos que impiden
modelar el procedimiento sin inventar requisitos:

1. **No hay estado de inadmisión.** Una convocatoria tiene plazo de presentación. Si
   alguien registra fuera de él, el derecho a registrar no desaparece: lo que procede es
   inadmitir después. Sin estado, ese expediente se queda en `REGISTRADO` para siempre.
2. **`PROPUESTA` es un solo estado.** La concesión en concurrencia competitiva exige
   propuesta provisional, trámite de alegaciones y propuesta definitiva (Ley 38/2003,
   art. 24.4). Con un único estado no se puede saber si el plazo de alegaciones está
   abierto.
3. **`RESUELTO` no distingue sentido.** `docs/diseno.md` §3 asigna un token de color
   propio a «resuelto favorable», así que el sentido ya es información visible.
4. **El diagrama contradice `docs/testing.md` §6.** La flecha «aporta» sale de
   `SUBSANACION_REQ` hacia `PROPUESTA`, mientras el ejemplo Gherkin del contrato de
   testing dice que el expediente pasa a «en revisión».

## Opciones consideradas

**A · Dejar el diagrama como está y resolver los huecos con atributos.** Un campo
`inadmitido: boolean`, un campo `sentido`, un campo `faseDePropuesta`. Descartada: el
invariante II dice que el estado del expediente es la máquina de estados. Un booleano que
cambia lo que se puede hacer con un expediente **es** un estado, solo que uno que no
aparece en el diagrama y que nadie testea.

**B · Extender el diagrama con estados nuevos.** Elegida.

**C · Rehacer el procedimiento como concesión por orden de entrada.** Eliminaría la
necesidad de las dos propuestas y de las alegaciones. Descartada por decisión de producto
(concurrencia competitiva, `docs/modelo-dominio.md` `[D-01]`), y porque el flujo crítico
6 de `docs/testing.md` habla explícitamente de publicar un listado definitivo.

## Decisión

La máquina pasa de 8 a **10 estados**:

| Cambio | Estado |
| --- | --- |
| Añadido | `INADMITIDO` (terminal) |
| Desdoblado | `PROPUESTA` → `PROPUESTA_PROVISIONAL` y `PROPUESTA_DEFINITIVA` |
| Desdoblado | `RESUELTO` → `RESUELTO_FAVORABLE` y `RESUELTO_DESFAVORABLE` |
| Corregido | «aporta» va de `SUBSANACION_REQUERIDA` a `EN_REVISION`, no a la propuesta |

**No se añade un estado `ALEGACIONES`.** El plazo de alegaciones es una ventana temporal
del estado `PROPUESTA_PROVISIONAL`: presentar una alegación no cambia el estado del
expediente. Un estado más aquí sería ruido con coste de test y sin significado jurídico
propio.

**No se añade un estado `CADUCADO`.** El vencimiento del plazo máximo de resolución no
cambia el estado del expediente: produce silencio desestimatorio, que es un derecho de la
persona interesada (Ley 38/2003, art. 25.5), no una transición. Se modela como atributo
derivado, y la Administración sigue obligada a resolver.

**La flecha «aporta» se corrige a favor de `docs/testing.md`.** Quien aporta
documentación devuelve el expediente a instrucción, porque alguien tiene que revisar lo
aportado. La lectura del diagrama original permitiría proponer sin revisar, que es
indefendible ante una reclamación.

## Consecuencias

**La máquina crece de forma no lineal.** Diez estados son cien pares ordenados: quince
permitidos, ochenta y cinco prohibidos. Cada par prohibido debe fallar con un error de
dominio tipado. Esto se cubre con un único test parametrizado que recorre la matriz
entera, no con ochenta y cinco tests escritos a mano — y si alguien añade una transición
sin actualizar la matriz, ese test falla.

**Aparece la segregación de funciones como precondición de dominio.** Con `RESUELTO`
desdoblado y actor obligatorio, quien firma la propuesta no puede firmar la resolución.
Es la regla P-15 y no es una comprobación de interfaz.

**La inadmisión tiene ventana.** Una vez emitida la propuesta provisional ya no se
inadmite: se resuelve de forma desfavorable. Es la regla P-11, y sin el estado nuevo no
habría dónde colgarla.

**El token de color de las dos propuestas queda sin asignar en `docs/diseno.md` §3.** Se
usa `--azul` provisionalmente, el mismo de «en revisión», y por eso la diferencia entre
provisional y definitiva se comunica obligatoriamente con texto. Es una deuda del sistema
de diseño, no del dominio.

**Divergencia legal conocida y no resuelta.** El desistimiento automático por vencimiento
que fija `docs/dominio.md` §2 no encaja con el art. 68.1 de la Ley 39/2015, que exige
resolución previa. Este ADR no la resuelve: la deja anotada en `docs/deuda-tecnica.md`
como DT-03. La alternativa fiel sería un estado intermedio `PENDIENTE_DESISTIMIENTO` que
el jefe de servicio confirma, y merece su propio ADR.
