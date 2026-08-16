# 0002 · Ampliación de la matriz de contraste a superficies `--papel-alto`

**Estado:** aceptado · **Fecha:** 2026-08-16 · **Afecta a:** `docs/diseno.md` §2

---

## Contexto

`docs/diseno.md` §2 publica una matriz de contraste calculada, y su regla es tajante:
«calculada, no estimada». Todas sus parejas están medidas contra `--papel` (`#EFF1EC`),
el fondo de página.

Al aplicar el sistema a las cinco pantallas clave (`docs/maquetas.md`) aparecieron
superficies elevadas: tarjetas de convocatoria, el bloque del resumen orientativo y el
propio justificante, todas sobre `--papel-alto` (`#FFFFFF`). Ninguna pareja de texto
sobre esa superficie tenía ratio publicado.

Sin esas filas, cada componente que use una tarjeta estaría eligiendo colores sin dato, o
—peor— reutilizando por analogía el ratio de la fila de `--papel`, que es distinto.

## Opciones consideradas

**A · Prohibir `--papel-alto` como fondo de texto.** Todo el contenido iría sobre
`--papel`. Descartada: `docs/diseno.md` §2 define `--papel-alto` explícitamente como
«superficies elevadas: tarjetas, modales», y §7 construye el justificante sobre ella.
Prohibirla contradiría el propio documento.

**B · Estimar por analogía.** Asumir que si una pareja pasa sobre `--papel` también pasa
sobre `--papel-alto`, porque el blanco es más claro. Es cierto que los ratios suben, pero
«sube, luego pasa» no es un dato. Y en el caso de `--sello` el razonamiento lleva
directamente al error: sube de 2.81 a 3.19 y sigue sin llegar a los 4.5:1 del texto
normal. Descartada.

**C · Calcular las filas que faltan e incorporarlas a la matriz.** Elegida.

## Decisión

Se añaden siete filas a la matriz de `docs/diseno.md` §2, calculadas con la fórmula de
luminancia relativa de WCAG 2.1 (§1.4.3 y §1.4.11), no estimadas:

| Combinación | Ratio | Uso permitido |
| --- | --- | --- |
| `--verde-700` sobre `--papel-alto` | 11.08:1 | Todo |
| `--azul` sobre `--papel-alto` | 8.66:1 | Todo |
| `--lacre` sobre `--papel-alto` | 8.19:1 | Todo |
| `--verde-500` sobre `--papel-alto` | 6.39:1 | Todo |
| `--gris` sobre `--papel-alto` | 6.06:1 | Texto secundario |
| `--linea-fuerte` sobre `--papel-alto` | 3.81:1 | Bordes funcionales |
| `--sello` sobre `--papel-alto` | 3.19:1 | **Solo** cuño y anillo de foco. Nunca texto |
| `--linea` sobre `--papel-alto` | 1.59:1 | **Solo** filete decorativo |

**Validación del método.** Antes de aceptar los valores nuevos se reprodujeron dos filas
ya publicadas con el mismo procedimiento: `--tinta` sobre `--papel` da 15.42:1 y
`--papel-alto` sobre `--verde-700` da 11.08:1, ambas coincidentes con `docs/diseno.md`.
Un método que no reproduce los valores conocidos no sirve para calcular los desconocidos.

## Consecuencias

**Se agrava, no se relaja, la prohibición de `--sello` como texto.** Sobre `--papel` da
2.81:1 y sobre `--papel-alto` da 3.19:1. Ninguno de los dos llega a 4.5:1. El latón no es
un color de texto en ninguna superficie del sistema. Donde haya que escribir sobre el
cuño, se hace al revés: fondo `--sello`, texto `--tinta`, 5.50:1.

**Queda explicado por qué el anillo de foco tiene dos capas.** Sobre `--papel`, `--sello`
da 2.81:1, por debajo del mínimo de 3:1 de WCAG 1.4.11, y lo resuelve la capa interior de
`--tinta` con 15.42:1. Sobre `--verde-700` ocurre lo contrario y es `--sello` quien
cumple, con 3.47:1. Ninguna capa sola funciona en los dos fondos: por eso son dos, y por
eso `outline: none` rompe el cumplimiento, no solo la estética.

**`--linea` sigue siendo decorativo.** 1.59:1 sobre `--papel-alto` está aún más lejos del
umbral que 1.39:1 sobre `--papel`. Nunca es borde funcional, y donde aparezca lleva
`aria-hidden`.

**Coste de mantenimiento.** Cualquier token nuevo obliga a calcular sus parejas sobre las
dos superficies, no sobre una. Es el precio de tener dos fondos.
