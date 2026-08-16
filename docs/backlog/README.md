# Backlog — Sede Electrónica de Valdeprieto

Este directorio es la fuente de los criterios de aceptación. `CLAUDE.md` §0.2 lo dice sin
ambigüedad: **si un criterio no está aquí, no se deduce, se pregunta**. Nada se
implementa contra una historia que no exista en este backlog.

---

## 1. Convención

**Identificadores.** `E{épica}-H{historia}`. El identificador no se reutiliza ni se
renumera aunque la historia se descarte: la trazabilidad de `docs/trazabilidad.md`
depende de que sea estable.

**Criterios de aceptación.** Gherkin en español (`# language: es`), escrito en lenguaje
de negocio. Prohibido el paso técnico: `Cuando hago clic en el botón #enviar` no es un
criterio de aceptación, es una instrucción de robot. Un criterio debe seguir siendo
válido si mañana se rediseña la pantalla entera.

**Criterios de accesibilidad.** Concretos y verificables por historia. «Debe ser
accesible» no es un criterio; «el foco viaja al primer campo con error y el mensaje se
enlaza con `aria-describedby`» sí lo es. Los generales de `CLAUDE.md` §6 y la Definición
de Hecho de §9 se dan por incluidos en todas y no se repiten.

**Estimación.** Puntos relativos en escala de Fibonacci: 1 · 2 · 3 · 5 · 8 · 13. Miden
esfuerzo con la **Definición de Hecho completa**, no «cuánto tarda el código en
funcionar». Una historia de 3 puntos incluye sus tests unitarios, su story de Storybook
con los cuatro estados, su cobertura E2E, su pasada de axe y su recorrido con teclado.
Una historia de 13 no entra en un sprint: se parte.

**Flujo crítico.** Cuando la historia forma parte de uno de los seis flujos de
`docs/testing.md` §1, se indica. Esos son los únicos que se duplican en el track
Selenium.

---

## 2. Épicas

| Épica | Título | Puntos | Alcance |
| --- | --- | --- | --- |
| [E0](E0-cimientos.md) | Cimientos de dominio y sistema de diseño | 41 | Semanas 1–2 (E0-H08 en la 6) |
| [E1](E1-convocatorias.md) | Convocatorias y bases reguladoras | 8 | Semana 3 |
| [E2](E2-solicitud-y-registro.md) | Solicitud, registro y justificante | 37 | Semanas 3–5 |
| [E3](E3-seguimiento-expediente.md) | Seguimiento del expediente | 16 | Semanas 5–6 |
| [E4](E4-subsanacion.md) | Subsanación y notificación | 18 | Semanas 6–7 |
| [E5](E5-instruccion.md) | Instrucción y bandeja del tramitador | 23 | Semanas 6–8 |
| [E6](E6-propuesta-y-resolucion.md) | Propuesta, alegaciones y resolución | 29 | Semanas 9–11 |
| [E7](E7-sede-y-verificacion.md) | Sede, verificación pública y cumplimiento | 9 | Transversal |
| [E8](E8-lenguaje-claro-ia.md) | Resumen en lenguaje claro y su evaluación | 26 | Semanas 8–9 |

**Total: 207 puntos.**

---

## 3. Meta de la semana 5

> Una persona entra en la sede, encuentra una convocatoria abierta, lee sus bases,
> presenta una solicitud completa con documentación adjunta, obtiene un justificante
> sellado con número de registro, se descarga ese justificante, y **cualquier tercero
> puede verificar su autenticidad** en una página pública sin identificarse.

Es el **flujo crítico 2** de `docs/testing.md` de punta a punta, más la verificación por
CSV. Se puede grabar en vídeo, se puede enseñar en una entrevista y se puede auditar sin
tocar el código.

### Corrección de la estimación previa

En la sesión de interrogatorio asumí que en cinco semanas entraban los flujos 2 **y** 4
en los dos tracks de automatización. Hecha la cuenta, no entran: son unos 100 puntos
contra los 83 que caben. La meta de la semana 5 es el **flujo 2 solo**. El flujo 4
(subsanación) queda en semanas 6–7 y el track Selenium arranca en la 6 sobre el flujo 2
ya estabilizado.

Prefiero un flujo impecable y una fecha que se cumple a dos flujos a medias.

### Supuesto de velocidad, y qué pasa si falla

Se asume **una persona, 20 horas por semana, y 1 punto ≈ 1,2 h con la Definición de
Hecho completa** — es decir, **16,6 puntos por semana**. Es un supuesto optimista y está
marcado como tal: la DoD de este proyecto (Storybook con cuatro estados, E2E, axe,
recorrido con teclado a mano, Lighthouse, trazabilidad) pesa más que en un proyecto
normal.

Si la velocidad real es menor, el orden de recorte es este y no otro:

1. **E1-H03** (cuenta atrás de días hábiles en el detalle de convocatoria) — 2 puntos.
2. **E3-H01** (bandeja «Mis solicitudes») — 3 puntos. El justificante ya demuestra el
   flujo.
3. **E7-H01** (verificación pública por CSV) — 3 puntos. Duele, porque es la pieza que
   más impresiona, pero es la única de la meta que se puede enseñar una semana después.

**No se recorta nunca**: nada de E0 (los cimientos mal puestos se pagan tres veces), ni
la accesibilidad de ninguna historia, ni un test. Recortar un test no adelanta la fecha:
la mueve a después de la entrevista.

### Plan por semanas

| Semana | Historias | Puntos | Al final de la semana se puede enseñar |
| --- | --- | --- | --- |
| **1** | E0-H05, E0-H01, E0-H07, E7-H03 | 17 | Storybook con las primitivas accesibles y el motor de plazos con sus 16 casos límite en verde |
| **2** | E0-H03, E0-H02, E0-H04, E0-H06 | 17 | La máquina de estados completa, con la matriz de 100 pares testeada y la traza inmutable |
| **3** | E1-H01, E1-H02, E1-H03, E2-H01, E2-H04, E7-H04 | 16 | Navegar convocatorias, abrir un borrador y reanudarlo |
| **4** | E2-H02, E2-H03, E2-H05 | 16 | El formulario completo con validación compartida y adjuntos |
| **5** | E2-H06, E2-H07, E7-H01, E3-H01 | 17 | **El flujo 2 entero, con justificante sellado y verificación pública** |

Total comprometido: **83 puntos**.

### Después de la semana 5

| Semanas | Contenido |
| --- | --- |
| 6–7 | E3 completa, E4 completa, arranque del track Selenium sobre el flujo 2 |
| 8 | E5 completa; flujos críticos 4 y 5 cerrados en ambos tracks |
| 9 | E8: resumen en lenguaje claro con su arnés de evaluación |
| 10–11 | E6 completa; flujo crítico 6 |
| 12 | Pase de pulido, informe de accesibilidad, comparativa A vs B, caso de estudio |

---

## 4. Orden de dependencias

```
E0-H05 (primitivas) ──┬─> toda historia con interfaz
E0-H01 (plazos) ──────┼─> E1-H03, E4-H01, E4-H02, E4-H03, E4-H04, E6-H02
E0-H02 (calendario) ──┘
E0-H03 (máquina) ─────┬─> E2-H06, E4-*, E5-*, E6-*
E0-H04 (traza) ───────┘
E0-H06 (semilla) ─────> toda historia con E2E
E0-H07 (reloj) ───────> E4-H03, E4-H04, E6-H02

E1-H02 ──> E2-H01 ──> E2-H02 ──> E2-H03 ──> E2-H05 ──> E2-H06 ──> E2-H07
                                                          │
                                                          ├──> E7-H01
                                                          ├──> E3-H01 ──> E3-H02
                                                          └──> E5-H01 ──> E5-H02 ──> E5-H03 ──> E5-H04 ──> E4-H01 ──> E4-H02
                                                                                                                 └──> E4-H03
E5-H05 ──> E6-H01 ──> E6-H02 ──> E6-H03 ──> E6-H04 ──> E6-H05 ──> E6-H06
```

El camino crítico de las cinco semanas es la cadena `E0-H05 → E1-H02 → E2-H01 → E2-H02 →
E2-H03 → E2-H05 → E2-H06`. Cualquier retraso ahí mueve la fecha; retrasos fuera de esa
cadena, no.

---

## 5. Riesgos del propio backlog

- **E2-H02 (formulario en pasos) es la historia más peligrosa.** Ocho puntos que en
  realidad son cuatro problemas: validación compartida, navegación entre pasos,
  persistencia parcial y anuncio accesible de errores. Si en la semana 4 se ve que no
  cabe, se parte en E2-H02a y E2-H02b antes de empezar, no a mitad.
- **La accesibilidad no tiene historia propia y es deliberado.** Repartirla por historia
  es lo único que impide que se convierta en una tarea de la semana 12 que nadie hace.
- **E8 depende de un dataset de 30 convocatorias con resumen de referencia revisado a
  mano.** Ese trabajo es manual, es lento y no aparece en los 8 puntos de E8-H03. Se
  empieza a construir en la semana 6, en segundo plano.
