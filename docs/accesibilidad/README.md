# Accesibilidad

`CLAUDE.md` §1, invariante I: la accesibilidad no es una mejora, es el requisito
funcional principal. El RD 1112/2018 obliga al sector público español a cumplir la norma
EN 301 549, equiparada a WCAG 2.1 nivel AA, y a publicar una declaración de accesibilidad
según el modelo de la Comisión Europea.

## Qué vive aquí

| Contenido | Estado |
| --- | --- |
| `declaracion.md` — declaración de accesibilidad según el modelo de la Comisión Europea | Pendiente (historia E7-H02) |
| `informe-revision.md` — informe siguiendo la metodología del Observatorio de Accesibilidad Web | Pendiente |
| `evidencias/` — grabaciones de NVDA sobre los flujos 2 y 4 | Pendiente |
| `contraste.md` — verificación de cada pareja contra la matriz de `docs/diseno.md` | Pendiente |

## La separación honesta

`docs/testing.md` §8 obliga a decirlo así en el informe, y aquí se repite porque es la
frase que más se falsea en los proyectos de accesibilidad:

**La automatización cubre en torno al 40 % de los criterios.** `pnpm test:a11y` ejecuta
axe-core sobre cada ruta en estado normal, con formulario vacío y con formulario en
error. Cero violaciones `serious` o `critical` rompe la build.

**El 60 % restante exige revisión manual**: recorrido completo de los seis flujos solo con
teclado, NVDA sobre los flujos 2 y 4 con grabación en vídeo, zoom al 200 % y ancho de
320 px en cada pantalla, verificación de contraste contra la matriz, y revisión de textos
alternativos, jerarquía de encabezados e idioma declarado.

Un informe que presente el resultado de axe como «la web es accesible» es un informe
falso. Este directorio existe para que no lo sea.

## Regla sobre las evidencias

Ninguna captura, grabación o traza puede contener datos personales sin enmascarar,
aunque sean sintéticos (`docs/dominio.md` §5). Antes de añadir una evidencia aquí, se
comprueba el enmascarado de NIF, IBAN y correo.
