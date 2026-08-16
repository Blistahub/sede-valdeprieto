# Sede Electrónica — Ayuntamiento de Valdeprieto

Portal de tramitación de ayudas y subvenciones municipales. Municipio **ficticio**.
Repositorio: `sede-valdeprieto`. Idioma de producto: **español de España**.

> Este proyecto es una pieza de portfolio con estándar de producción real. El listón no
> es "una demo que funciona": es "esto podría entrar en un pliego del sector público
> español y pasar la auditoría". Si en algún momento una decisión se justifica solo por
> "es más rápido", es la decisión equivocada.

---

## 0. Acuerdo de trabajo (leer siempre antes de actuar)

1. **Planifica antes de escribir código.** Ante cualquier tarea que toque más de un
   fichero: presenta un plan numerado (ficheros afectados, orden, riesgos, qué se testea)
   y **espera aprobación explícita**. No empieces a editar durante la fase de plan.
2. **Nunca inventes requisitos.** Si un criterio de aceptación no está en
   `docs/backlog/`, no lo deduzcas: pregunta. Es preferible una pregunta a una
   suposición plausible.
3. **Un cambio, un propósito.** No refactorices "de paso". Si ves algo que arreglar,
   anótalo en `docs/deuda-tecnica.md` y sigue con la tarea.
4. **Reporta lo que no pudiste ejecutar.** Si un comando falla o no se ejecutó, dilo de
   forma explícita. No asumas que pasó.
5. **Registra decisiones.** Toda decisión de arquitectura o diseño no trivial genera un
   ADR corto en `docs/decisiones/NNNN-titulo.md` (contexto, opciones, decisión,
   consecuencias). Máximo una página.
6. **No toques los tokens de diseño ni el contrato de testing** sin aprobación explícita.
   Son la columna vertebral del proyecto.

## 1. Los tres invariantes

Si un cambio rompe cualquiera de estos tres, no se mergea. Sin excepciones.

**I. Accesibilidad = cumplimiento legal.**
El RD 1112/2018 obliga al sector público español a cumplir la norma EN 301 549,
equiparada a **WCAG 2.1 nivel AA**, y a publicar una declaración de accesibilidad según
el modelo de la Comisión Europea. En este proyecto la accesibilidad no es una mejora:
es el requisito funcional principal. Cero violaciones `serious`/`critical` de axe-core.

**II. El expediente es una máquina de estados con valor legal.**
Ningún estado se cambia desde la UI sin pasar por la capa de dominio. Toda transición
queda registrada con sello de tiempo, actor y motivo. Los plazos se calculan en **días
hábiles** con calendario de festivos, nunca con `Date` a pelo.

**III. La calidad tiene que ser verificable por un tercero.**
Cada funcionalidad entra con sus pruebas y su evidencia publicada. Un test desactivado
para que pase CI es un fallo de proyecto, no un atajo.

## 2. Stack

| Capa | Elección | Nota |
| --- | --- | --- |
| Framework | Next.js (App Router) + React + TypeScript `strict` | Server Components por defecto |
| Estilos | Tailwind sobre tokens CSS propios | Los tokens mandan, Tailwind los consume |
| Componentes | Propios sobre primitivas accesibles headless | Nada de librerías de UI opinadas |
| Estado servidor | Server Actions + validación con Zod | Validación **compartida** cliente/servidor |
| Persistencia | PostgreSQL + Prisma | |
| Docs UI | Storybook | Un story por estado, incluido el de error |
| E2E (track A) | Playwright + TypeScript | |
| E2E (track B) | Selenium 4 + Java 21 + JUnit 5 + Cucumber | Mismos flujos, comparativa medida |
| API | Playwright `request` (A) / RestAssured (B) | |
| Unit / componente | Vitest + Testing Library | |
| Accesibilidad | axe-core en CI + NVDA manual | |
| Rendimiento | Lighthouse CI con presupuestos que rompen build | |
| CI | GitHub Actions | Allure publicado en GitHub Pages |

**Antes de añadir cualquier dependencia**: justifícala en el plan (qué problema resuelve,
peso, alternativa nativa descartada y por qué). Sin justificación, no entra.

## 3. Estructura

```
app/                    Rutas (App Router). Server Components por defecto.
  (ciudadano)/          Área pública y de tramitación
  (gestion)/            Área de tramitador y jefe de servicio
components/
  ui/                   Primitivas del sistema de diseño. Sin lógica de negocio.
  dominio/              Componentes que conocen el dominio (EstadoExpediente, Plazo…)
lib/
  dominio/              Reglas de negocio puras. Sin React, sin I/O. Testeadas al 100%.
  validacion/           Esquemas Zod compartidos
  plazos/               Cálculo de días hábiles y festivos
tests/
  e2e/                  Playwright: specs, page objects, fixtures
  integracion/
selenium/               Proyecto Maven independiente (track B)
docs/
  backlog/              Épicas e historias con criterios de aceptación
  decisiones/           ADRs
  accesibilidad/        Informes, declaración, evidencias
  evaluacion-ia/        Dataset dorado y métricas de la feature de IA
```

Regla dura: **`lib/dominio/` no importa nada de React, Next ni de la capa de datos.**
Es código puro, testeable sin navegador. Ahí viven los plazos, las transiciones de
estado y las validaciones legales.

## 4. Comandos

```bash
pnpm dev                 # desarrollo
pnpm build               # build de producción
pnpm lint                # ESLint + Prettier (debe salir limpio)
pnpm typecheck           # tsc --noEmit
pnpm test                # Vitest
pnpm test:e2e            # Playwright
pnpm test:a11y           # axe sobre todas las rutas
pnpm storybook
pnpm lhci                # Lighthouse CI con presupuestos
mvn -f selenium/pom.xml verify   # track B
```

Antes de dar una tarea por terminada, ejecuta como mínimo: `pnpm typecheck && pnpm lint
&& pnpm test && pnpm test:e2e`.

## 5. Reglas de código

- TypeScript `strict`. **Cero `any`, cero `@ts-ignore`, cero `!` de aserción no nula.**
  Si el tipo no cuadra, el modelo está mal, no el compilador.
- Nada de `useEffect` para obtener datos. Server Components o Server Actions.
- Los errores de dominio son tipos de resultado explícitos, no excepciones lanzadas al
  aire. La UI debe poder distinguir "NIF inválido" de "servicio caído".
- Nombres de dominio **en español** (`Expediente`, `calcularPlazoSubsanacion`,
  `EstadoExpediente`). Nombres técnicos en inglés (`useState`, `fetchJson`). No mezcles
  dentro del mismo identificador.
- Comentarios solo para explicar **por qué**, nunca **qué**. Si necesitas explicar el
  qué, reescribe el código.
- Nunca desactives una regla de ESLint en línea sin un comentario que explique la causa
  y enlace a un ADR.

## 6. Reglas de UI

Los tokens, la tipografía, la paleta y el elemento firma están en el documento de diseño.
Resumen de las reglas que no se negocian nunca:

- **Jamás `outline: none`** sin sustituto de foco visible y de contraste ≥ 3:1.
- Todo control interactivo alcanzable y operable **solo con teclado**, en orden lógico.
- Todo campo de formulario tiene `<label>` asociado. `placeholder` nunca sustituye a
  label.
- Los errores de formulario se anuncian a lectores de pantalla y se enlazan al campo
  (`aria-describedby`), y el foco viaja al primer campo con error.
- Toda cifra que una persona pueda tener que leer en voz alta, copiar o comparar
  (expediente, NIF, importe, fecha, plazo) va en la fuente monoespaciada con
  `font-variant-numeric: tabular-nums`.
- Funciona a **320 px** de ancho y con **zoom al 200 %** sin scroll horizontal.
- `prefers-reduced-motion: reduce` elimina toda animación, no la acorta.
- El color nunca es el único portador de información. Estado = color **+** texto **+**
  forma.

Detalle completo: @docs/diseno.md

## 7. Reglas de testing

- **Localizadores accesibles primero.** `getByRole`, `getByLabel`, `getByText`.
  `data-testid` solo cuando no existe alternativa accesible — y en ese caso **abre un
  bug de accesibilidad**, porque si no puedes localizarlo por rol, probablemente un
  lector de pantalla tampoco lo encuentra.
- **Cero esperas fijas.** Nada de `waitForTimeout`, `sleep` ni `Thread.sleep`. Solo
  aserciones con reintento automático.
- Cada test crea sus datos vía API o seed y es independiente del resto. El orden de
  ejecución no puede importar.
- Un test que falla de forma intermitente se investiga y se documenta en
  `docs/flaky/`, no se reintenta hasta que pase.

Detalle completo: @docs/testing.md

## 8. Dominio

Vocabulario administrativo, máquina de estados, reglas de plazos y datos sintéticos:
@docs/dominio.md

Nunca traduzcas los términos del dominio. Un *expediente* no es un "file", una
*subsanación* no es una "correction", una *convocatoria* no es una "call". Usa el
término español en código, en UI, en tests y en commits.

## 9. Definición de Hecho

Una historia no está hecha hasta que **todo** esto se cumple:

- [ ] `pnpm typecheck && pnpm lint` limpio
- [ ] Reglas de dominio cubiertas por tests unitarios, incluidos valores límite
- [ ] Componentes nuevos con story en Storybook: estado normal, vacío, cargando y error
- [ ] Flujo cubierto por E2E en Playwright
- [ ] Si es flujo crítico: cubierto también en el track Selenium/Cucumber
- [ ] axe-core sin violaciones `serious` ni `critical`
- [ ] Recorrido completo solo con teclado, verificado a mano
- [ ] Contraste comprobado contra la matriz de `docs/diseno.md`
- [ ] Correcto a 320 px y con zoom 200 %
- [ ] Lighthouse: accesibilidad 100, rendimiento ≥ 90
- [ ] Textos en español revisados: sin lorem ipsum, sin inglés filtrado, voz activa
- [ ] Trazabilidad actualizada: historia → caso de prueba → test automatizado
- [ ] Commit en formato Conventional Commits, en español

## 10. Prohibiciones absolutas

1. No usar nombres, escudos ni marcas de instituciones reales. El municipio es ficticio.
2. No generar NIF, NIE, IBAN, teléfonos ni direcciones que puedan corresponder a
   personas reales. Solo datos sintéticos marcados como tales
   (ver `docs/dominio.md`).
3. No registrar datos personales en logs, trazas de Playwright ni capturas.
   Enmascarado obligatorio.
4. No desactivar, saltar (`skip`) ni marcar como esperado el fallo de un test para que
   pase CI.
5. No usar `outline: none`, `tabindex` positivo, `aria-hidden` sobre contenido
   interactivo ni `div` con `onClick` en lugar de `button`.
6. No añadir animación fuera de los momentos aprobados en `docs/diseno.md`.
7. No escribir texto de interfaz en inglés. Ni un `Submit`, ni un `Loading…`.
8. No inventar normativa. Si citas el RD 1112/2018 o la EN 301 549, cita con precisión
   y enlaza a la fuente oficial.

## 11. Skills y plugins: cuándo usar cada uno

| Momento | Usar | No usar |
| --- | --- | --- |
| Inicio de épica, planificación | `superpowers` | — |
| Implementación de producto | `gstack`, `codex` | `ui-ux-pro-max` (diseño ya congelado) |
| Fase 2 únicamente: definir sistema de diseño | `ui-ux-pro-max` | — |
| Pase final de pulido (fase 12) | `impeccable`, `taste-skill` | — |
| Los 3 momentos de movimiento aprobados | `gsap`, `emil` | En cualquier otro sitio |
| Sesiones de depuración | `caveman`, `ponytail`, `i-have-adhd` | — |
| Vídeo del caso de estudio | `hyperframes` | — |

**El código de test lo escribe el humano.** Puedes generar andamiaje, datos de prueba y
page objects a partir de un patrón existente, pero cada aserción y cada estrategia de
espera se revisa línea a línea. Este proyecto se defiende en una entrevista técnica.

## 12. Git

- Ramas: `feat/expediente-subsanacion`, `fix/plazo-festivos`, `test/e2e-registro`.
- Conventional Commits en español: `feat(expediente): permitir subsanación en plazo`.
- Un commit por unidad lógica. Nada de "varios cambios".
- El mensaje explica el **porqué** en el cuerpo si no es evidente.
