# Registros de decisión de arquitectura

`CLAUDE.md` §0.5: toda decisión de arquitectura o diseño no trivial genera un ADR corto
—contexto, opciones, decisión, consecuencias— de una página como máximo.

Formato del nombre: `NNNN-titulo-en-kebab-case.md`. La numeración no se reutiliza ni se
renumera: un ADR sustituido se marca como **sustituido por** y se queda donde está.

---

| ADR | Título | Estado |
| --- | --- | --- |
| [0001](0001-arquitectura.md) | Arquitectura de la aplicación | Aceptado |
| [0002](0002-ampliacion-matriz-contraste.md) | Ampliación de la matriz de contraste a superficies `--papel-alto` | Aceptado |
| [0003](0003-extension-maquina-estados.md) | Extensión de la máquina de estados del expediente | Aceptado |

El 0001 es el único que excede la página que fija `CLAUDE.md` §0.5, y lo hace a
propósito: agrupa cinco decisiones que solo tienen sentido juntas. La justificación está
en su primera sección.
