# Contrato de testing

Documento normativo. Este es el entregable que se defiende en la entrevista técnica.

---

## 1. Pirámide real de este proyecto

| Nivel | Herramienta | Qué cubre | Volumen |
| --- | --- | --- | --- |
| Unitario | Vitest | `lib/dominio/`: plazos, transiciones, validaciones | Mayoría |
| Componente | Vitest + Testing Library | Comportamiento accesible de primitivas | Alto |
| Integración API | Playwright `request` / RestAssured | Server Actions, contratos, códigos de error | Medio |
| E2E | Playwright (A) / Selenium+Cucumber (B) | 6 flujos críticos, no más | Bajo |
| Accesibilidad | axe-core + NVDA manual | Todas las rutas | Todas |
| Rendimiento | Lighthouse CI | Rutas de entrada | 5 rutas |

**Los seis flujos críticos E2E** (los únicos que se duplican en ambos tracks):

1. Consultar convocatoria y descargar bases.
2. Presentar solicitud completa hasta obtener justificante.
3. Reanudar un borrador guardado y presentarlo.
4. Responder a un requerimiento de subsanación dentro de plazo.
5. Tramitador revisa expediente y solicita subsanación.
6. Jefe de servicio resuelve y publica listado definitivo.

Todo lo demás se cubre por debajo. Un E2E que valida una regla de negocio es un E2E mal
colocado.

## 2. Estrategia de localizadores

Orden obligatorio de preferencia:

1. `getByRole('button', { name: 'Registrar solicitud' })`
2. `getByLabel('NIF de la persona solicitante')`
3. `getByText(...)` para contenido no interactivo
4. `getByTestId(...)` — **último recurso**

**Regla del proyecto:** si necesitas `data-testid` para localizar un elemento
interactivo, eso es un síntoma de accesibilidad, no una preferencia de estilo. Abre un
issue con etiqueta `a11y` antes de escribir el test. El resultado es que la suite E2E
funciona como una auditoría de accesibilidad continua y gratuita.

Cuando el `data-testid` sea inevitable (por ejemplo, la fila N de una tabla), el formato
es `{dominio}-{bloque}-{elemento}` en kebab-case:
`expediente-tabla-fila`, `solicitud-paso3-adjuntos`.

Prohibidos siempre: XPath, selectores CSS por clase, `nth-child`, texto parcial ambiguo.

## 3. Esperas

- Cero `waitForTimeout`, `sleep`, `Thread.sleep`, `implicitlyWait`.
- Solo aserciones con reintento: `expect(locator).toBeVisible()`,
  `expect(locator).toHaveText()`, y en Selenium `WebDriverWait` con
  `ExpectedConditions` explícitas.
- Si algo necesita una espera fija para pasar, hay una condición observable que no
  estás usando. Búscala.

## 4. Datos de prueba

- Cada test crea sus datos por **API o seed**, nunca navegando por la interfaz.
  Excepción: el propio flujo bajo prueba.
- Cada test es independiente. El orden de ejecución no puede importar y la suite debe
  poder ejecutarse en paralelo con sharding.
- Aislamiento por identificador único de ejecución (`run-id`) en cada registro creado,
  con limpieza al final.
- Todos los datos personales son **sintéticos y marcados como tales**. Ver
  `docs/dominio.md` para los rangos permitidos de NIF, IBAN y teléfonos.
- Prohibido volcar datos personales, aunque sean sintéticos, en trazas, capturas o logs
  de CI sin enmascarar.

## 5. Estructura Playwright (track A)

```
tests/e2e/
  paginas/            Page Objects. Sin aserciones dentro.
    SolicitudPagina.ts
  flujos/             Composición de acciones de negocio
    presentarSolicitud.ts
  fixtures/           Fixtures de Playwright: sesión, datos, limpieza
  specs/
    solicitud.spec.ts
  a11y/
    axe.spec.ts       Recorre todas las rutas
```

Reglas del Page Object: expone **acciones de negocio** (`rellenarDatosPersonales`), no
mecánica de clics. Nunca contiene `expect`. Nunca conoce datos concretos: los recibe.

## 6. Estructura Selenium / Cucumber (track B)

```
selenium/
  src/test/java/.../paginas/
  src/test/java/.../pasos/        Step definitions
  src/test/resources/features/    Gherkin en español
```

Gherkin en español (`# language: es`), con vocabulario del dominio:

```gherkin
# language: es
Característica: Subsanación de solicitud

  Escenario: La persona subsana dentro de plazo
    Dado un expediente en estado "subsanación requerida" con 7 días hábiles restantes
    Cuando la persona interesada aporta el certificado de empadronamiento
    Entonces el expediente pasa a estado "en revisión"
    Y se genera un justificante con sello de tiempo
```

Nada de pasos técnicos en Gherkin. Prohibido `Cuando hago clic en el botón #enviar`.

## 7. La comparativa A vs B

Este es el entregable diferencial. En `docs/comparativa-automatizacion.md`, medido con
datos reales de la propia CI, no de opinión:

| Métrica | Playwright | Selenium |
| --- | --- | --- |
| Tiempo de suite completa (frío / caliente) | | |
| Tiempo del flujo 2 aislado | | |
| Tasa de fallo intermitente sobre 50 ejecuciones | | |
| Líneas de código para el flujo 2 | | |
| Ficheros a tocar al renombrar un campo | | |
| Calidad del diagnóstico cuando falla | | |

Y una conclusión escrita: en qué contexto elegirías cada uno. Con matices, no con
titulares.

## 8. Accesibilidad

**Automatizado (cubre ~40 % de los criterios, dilo así en el informe):**
`pnpm test:a11y` ejecuta axe-core sobre cada ruta, en estado normal, con formulario
vacío y con formulario en error. Cero violaciones `serious` o `critical` rompe la build.

**Manual (el 60 % restante):**

- Recorrido completo de los 6 flujos solo con teclado. Orden de tabulación, trampas de
  foco, atajos, `Escape` en modales.
- NVDA sobre los flujos 2 y 4. Se graba en vídeo y se guarda en
  `docs/accesibilidad/evidencias/`.
- Zoom al 200 % y ancho de 320 px en cada pantalla.
- Verificación de contraste contra la matriz de `docs/diseno.md`.
- Revisión de textos alternativos, encabezados jerárquicos y `lang`.

**Entregables:** informe de revisión siguiendo la metodología del Observatorio de
Accesibilidad Web, y declaración de accesibilidad según el modelo de la Comisión
Europea, publicada en el propio sitio junto al mecanismo de reclamación.

## 9. Evaluación de la funcionalidad de IA

El resumen en lenguaje claro es no determinista. No se testea con igualdad exacta.

- **Dataset dorado**: 30 convocatorias reales anonimizadas con su resumen de referencia
  revisado a mano, en `docs/evaluacion-ia/dataset/`.
- **Métricas automatizadas por ejecución**:
  - Índice de legibilidad (Fernández Huerta o INFLESZ) — umbral mínimo definido.
  - Cobertura de hechos clave: importe, plazo, requisitos, órgano. Extraídos y
    comparados. Umbral: 100 % de los cuatro.
  - Ausencia de hechos inventados: ninguna cifra en el resumen que no esté en el
    original.
  - Longitud dentro de rango.
- **Seguridad**: batería de inyección de prompt en el texto de la convocatoria. El
  sistema debe ignorar instrucciones embebidas.
- **Regla de producto**: el resumen se muestra siempre etiquetado como orientativo y
  junto al texto legal íntegro, nunca en su lugar.
- La bajada de cualquier umbral rompe la build.

## 10. Fallos intermitentes

Cuando un test falle de forma intermitente:

1. **No** se reintenta hasta que pase. **No** se marca `skip`.
2. Se aísla y se ejecuta 50 veces para medir la tasa real.
3. Se documenta en `docs/flaky/NNN-nombre.md`: síntoma, hipótesis, causa raíz
   encontrada, corrección, tasa antes y después.
4. Se corrige la causa, no el síntoma.

Este directorio, con dos o tres casos bien analizados, vale más en una entrevista que
doscientos tests verdes.

## 11. Trazabilidad

`docs/trazabilidad.md` mantiene la tabla: historia de usuario → criterio de aceptación →
caso de prueba → test automatizado (ruta y nombre) → estado. Se actualiza en el mismo
commit que la funcionalidad, nunca después.

## 12. CI

Pipeline en GitHub Actions:

1. `typecheck` + `lint`
2. `test` unitario y de componente, con umbral de cobertura sobre `lib/dominio/`
3. `build`
4. `test:e2e` en paralelo con sharding, matriz Chromium/Firefox/WebKit
5. `test:a11y`
6. `lhci` con presupuestos
7. `mvn verify` del track B
8. Publicación de Allure en GitHub Pages

Artefactos siempre: trazas de Playwright, vídeos de fallo, informe de axe, informe de
Lighthouse. Sin datos personales sin enmascarar.
