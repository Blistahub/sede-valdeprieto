# Deuda técnica

`CLAUDE.md` §0.3: cuando se ve algo que arreglar y no toca arreglarlo ahora, se anota
aquí y se sigue con la tarea. Este documento existe para que «lo arreglo luego» sea una
entrada con fecha y no una intención.

Estados: **abierta** · **en curso** · **cerrada**.

---

| Id | Fecha | Origen | Deuda | Impacto | Estado |
| --- | --- | --- | --- | --- | --- |
| DT-01 | 2026-08-16 | `README.md` | La comprobación del topónimo contra el INE se hizo por búsqueda en fuentes públicas, no contra el fichero oficial de códigos de municipio | Bajo hasta la publicación final; entonces bloqueante, porque `docs/dominio.md` §6 lo exige antes de publicar | Abierta |
| DT-02 | 2026-08-16 | `docs/modelo-dominio.md` §1 | Tres citas de la Ley 38/2003 (arts. 24.4, 25.4 y 25.5) están verificadas en contenido pero no transcritas de forma literal: el texto consolidado del BOE se trunca antes de llegar a ellas y el espejo alternativo falla por certificado | Medio. `CLAUDE.md` §10.8 exige precisión al citar normativa | Abierta |
| DT-03 | 2026-08-16 | `docs/modelo-dominio.md` §7.2 | El desistimiento automático por vencimiento diverge del art. 68.1 de la Ley 39/2015, que exige resolución previa. Se sigue lo que fija `docs/dominio.md` §2 | Medio. Es la divergencia legal más visible del modelo. La alternativa fiel es un estado `PENDIENTE_DESISTIMIENTO` | Abierta |
| DT-04 | 2026-08-16 | `docs/modelo-dominio.md` `[D-22]` | El cómputo del plazo del art. 43.2 desde el día siguiente a la puesta a disposición es interpretación jurisprudencial, sin sentencia concreta citada | Bajo | Abierta |
| DT-05 | 2026-08-16 | `docs/maquetas.md` §5 | El cuño de registro es un rectángulo de latón con texto. `docs/diseno.md` §7 describe un cuño real, con tinta irregular y borde mordido. Sin trabajo de ilustración, el elemento firma no firma nada | Medio. Es la pieza que sostiene el caso de estudio | Abierta |
| DT-06 | 2026-08-16 | `docs/maquetas.md` §6 | El área de gestión ha recibido menos dirección visual que la ciudadana. La bandeja es una tabla con filtros, indistinguible de cualquier producto de gestión | Bajo | Abierta |
| DT-07 | 2026-08-16 | `docs/maquetas.md` | La tarjeta de convocatoria sigue siendo una caja rectangular genérica. El material del que dice salir el diseño —ficha de archivo, sobre, carpeta con solapa— no ha llegado a la forma, solo al color y a la tipografía | Bajo | Abierta |
| DT-08 | 2026-08-16 | `docs/backlog/E0-cimientos.md` | Los ejemplos de casos límite de E0-H01 nombran el caso pero no fijan fechas concretas, porque el calendario de festivos (E0-H02) todavía no existe. Hay que completarlos con valores calculados a mano en cuanto exista | Alto. Un plazo mal contado es el fallo más grave posible de este producto | Abierta |
| DT-09 | 2026-08-16 | `docs/backlog/README.md` | La velocidad de 16,6 puntos por semana es un supuesto sin dato histórico. El plan de cinco semanas depende entero de él | Alto | Abierta |
| DT-10 | 2026-08-16 | `docs/backlog/E2-solicitud-y-registro.md` | E2-H02 son 8 puntos que en realidad son cuatro problemas: validación compartida, navegación entre pasos, persistencia parcial y anuncio accesible de errores | Medio. Si en la semana 4 no cabe, se parte antes de empezar | Abierta |
| DT-11 | 2026-08-16 | `docs/backlog/E8-lenguaje-claro-ia.md` | El conjunto de 30 convocatorias con resumen de referencia revisado a mano no está estimado en ninguna historia. Es trabajo manual y lento | Medio | Abierta |
| DT-12 | 2026-08-16 | `docs/backlog/E2-solicitud-y-registro.md` | E2-H07 fija PDF/UA como objetivo del justificante descargable, sin ruta técnica decidida. La accesibilidad de un PDF es un frente entero aparte del HTML | Medio | Abierta |
| DT-13 | 2026-08-16 | `README.md` | Licencia sin decidir | Bajo, pero un repositorio público sin licencia es un repositorio sin permisos de uso | Abierta |
