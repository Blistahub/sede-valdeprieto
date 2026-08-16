# Trazabilidad

`docs/testing.md` §11: historia de usuario → criterio de aceptación → caso de prueba →
test automatizado → estado. **Se actualiza en el mismo commit que la funcionalidad,
nunca después.**

Esta tabla es lo que permite responder en una auditoría a la pregunta «enséñame dónde se
verifica este requisito» sin buscar en el código.

---

## Convenciones

**Criterio de aceptación.** Se referencia por el nombre del escenario Gherkin de la
historia, entre comillas. Si un escenario cambia de nombre, esta tabla cambia en el mismo
commit.

**Caso de prueba.** `CP-{historia}-{n}`. Un criterio puede necesitar varios casos.

**Test automatizado.** Ruta y nombre completos. Si hay dos tracks, se listan los dos.

**Estado.** `pendiente` · `en curso` · `verde` · `roja` · `intermitente`. Ningún test
puede estar `intermitente` sin su ficha en `docs/flaky/`.

---

## Tabla

Vacía todavía: no hay código de aplicación, así que no hay tests. Se rellena a partir de
la primera historia implementada.

| Historia | Criterio de aceptación | Caso | Test automatizado | Estado |
| --- | --- | --- | --- | --- |
| — | — | — | — | — |

---

## Cobertura por historia

Se rellena en paralelo. Sirve para detectar la historia que se dio por hecha sin test,
que es el fallo que esta tabla existe para impedir.

| Épica | Historias | Con test | Cobertura |
| --- | --- | --- | --- |
| E0 · Cimientos | 8 | 0 | 0 % |
| E1 · Convocatorias | 3 | 0 | 0 % |
| E2 · Solicitud y registro | 9 | 0 | 0 % |
| E3 · Seguimiento | 4 | 0 | 0 % |
| E4 · Subsanación | 4 | 0 | 0 % |
| E5 · Instrucción | 6 | 0 | 0 % |
| E6 · Propuesta y resolución | 6 | 0 | 0 % |
| E7 · Sede y verificación | 4 | 0 | 0 % |
| E8 · Lenguaje claro | 4 | 0 | 0 % |
| **Total** | **48** | **0** | **0 %** |
