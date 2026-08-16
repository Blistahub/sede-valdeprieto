# Evaluación de la funcionalidad de IA

`docs/testing.md` §9. El resumen en lenguaje claro es no determinista, así que no se
testea con igualdad exacta: se mide contra un conjunto de referencia y contra umbrales
que rompen la build al bajar.

## Qué vive aquí

| Contenido | Estado |
| --- | --- |
| `dataset/` — 30 convocatorias anonimizadas con su resumen de referencia revisado a mano | Pendiente (arranca en la semana 6) |
| `umbrales.md` — valor mínimo de cada métrica y su justificación | Pendiente |
| `inyeccion/` — batería de casos de instrucción embebida | Pendiente (historia E8-H04) |
| `resultados/` — métricas por ejecución, publicadas junto al resto de la evidencia | Pendiente |

## Las cuatro métricas

1. **Índice de legibilidad** (Fernández Huerta o INFLESZ), con umbral mínimo definido.
2. **Cobertura de hechos clave**: importe, plazo, requisitos y órgano. Se extraen del
   original y del resumen y se comparan. **Umbral: 100 % de los cuatro.**
3. **Ausencia de hechos inventados**: ninguna cifra del resumen puede faltar en el
   original.
4. **Longitud dentro de rango.**

## Seguridad

Batería de inyección de instrucciones en el texto de la convocatoria. El sistema debe
ignorar las instrucciones embebidas y seguir produciendo los cuatro hechos clave. Un solo
caso superado por el atacante rompe la build.

## Regla de producto, que no es negociable

El resumen se muestra **siempre** etiquetado como orientativo y sin valor jurídico, y
**siempre** junto al texto legal íntegro. Nunca en su lugar.

Y una regla que se salta con facilidad y no se puede saltar: **las fechas del resumen no
las genera el modelo**. Se inyectan desde `lib/plazos/`. Un plazo calculado por un modelo
de lenguaje sería el peor fallo posible de este proyecto.

## Nota sobre el conjunto de referencia

Las 30 convocatorias se construyen a mano y su coste no está estimado en ninguna historia
del backlog. Está anotado como DT-11 en `docs/deuda-tecnica.md`. Las convocatorias reales
que sirvan de base se anonimizan por completo: ni nombres de municipios reales, ni
identificadores, ni importes que permitan reconocer la convocatoria original.
