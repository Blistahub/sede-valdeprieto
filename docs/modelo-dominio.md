# Modelo de dominio — Sede Electrónica de Valdeprieto

Documento normativo. Deriva de `docs/dominio.md`, que sigue siendo la fuente de
vocabulario y de datos sintéticos. Este documento lo desarrolla: entidades, máquina de
estados exhaustiva, transiciones prohibidas y reglas de cómputo de plazos.

Todo lo que aquí se define vive en `lib/dominio/` y `lib/plazos/`: código puro, sin
React, sin Next, sin capa de datos. Los identificadores del modelo son los mismos que
aparecen en la interfaz, en los tests y en los mensajes de commit.

**Estado del documento:** primera versión. Contiene 22 decisiones tomadas sin consulta
previa, marcadas en el texto como `[D-nn]` y recogidas íntegras en la sección 8. Ninguna
de ellas debe considerarse cerrada hasta su revisión.

---

## 1. Fuentes normativas

Las citas legales se han verificado contra texto consolidado. Cuando no se ha podido
transcribir el precepto de forma literal, se indica de forma expresa.

| Referencia | Contenido usado | Verificación |
| --- | --- | --- |
| Ley 39/2015, art. 30.2 | Los plazos por días son hábiles; se excluyen sábados, domingos y declarados festivos | Literal |
| Ley 39/2015, art. 30.3 | El cómputo por días empieza el día siguiente al de la notificación | Literal |
| Ley 39/2015, art. 30.4 | El cómputo por meses o años empieza el día siguiente al de la notificación | Literal |
| Ley 39/2015, art. 30.5 | Si el último día del plazo es inhábil, se prorroga al primer día hábil siguiente | Literal |
| Ley 39/2015, art. 30.6 | Es inhábil el día que lo sea en el municipio o comunidad autónoma de residencia de la persona interesada, o en la sede del órgano, o a la inversa | Literal |
| Ley 39/2015, art. 43.2 | La notificación electrónica se entiende rechazada transcurridos **diez días naturales** desde la puesta a disposición sin acceder a su contenido | Literal |
| Ley 39/2015, art. 68.1 | Requerimiento de subsanación por **diez días**, con advertencia de que en otro caso se le tendrá por desistido, **previa resolución** | Literal |
| Ley 38/2003, art. 22.1 | La concurrencia competitiva es el régimen general de concesión | Literal |
| Ley 38/2003, art. 24.4 | Propuesta de resolución provisional, examen de alegaciones y propuesta definitiva | Verificado en contenido, **pendiente de transcripción literal** |
| Ley 38/2003, art. 25.4 | El plazo máximo para resolver y notificar no puede exceder de **seis meses**, contados desde la publicación de la convocatoria | Verificado en contenido, **pendiente de transcripción literal** |
| Ley 38/2003, art. 25.5 | El vencimiento del plazo máximo sin notificar resolución legitima a entender **desestimada por silencio** | Verificado en contenido, **pendiente de transcripción literal** |
| RD 1112/2018 y EN 301 549 | Accesibilidad equiparada a WCAG 2.1 AA | Ya recogido en `CLAUDE.md` §1 |

Fuentes oficiales:
[Ley 39/2015 (BOE-A-2015-10565)](https://www.boe.es/buscar/act.php?id=BOE-A-2015-10565) ·
[Ley 38/2003 (BOE-A-2003-20977)](https://www.boe.es/buscar/act.php?id=BOE-A-2003-20977) ·
[RD 1112/2018 (BOE-A-2018-12699)](https://www.boe.es/buscar/act.php?id=BOE-A-2018-12699)

**Advertencia de alcance.** Valdeprieto es un municipio ficticio y esta sede no tiene
validez administrativa. Las normas se citan porque el modelo las imita con fidelidad,
no porque el producto sea un sistema en producción sujeto a ellas.

---

## 2. Decisiones de partida

Estas siete decisiones cierran el interrogatorio previo y condicionan todo lo demás.

**Modalidad de concesión: concurrencia competitiva** `[D-01]`. Es el régimen general
(Ley 38/2003, art. 22.1), es lo que hace un ayuntamiento con una línea de ayudas, y es
lo único que da sentido al flujo crítico 6 de `docs/testing.md` («publica listado
definitivo»). Se limita a **tres criterios de valoración** y **sin prorrateo**: el
crédito se asigna por orden de puntuación hasta agotarse `[D-02]`.

**Notificación electrónica como entidad de primera clase** `[D-03]`. El requerimiento
no se «emite y ya»: se pone a disposición, y la persona accede o no accede. Es lo que
obliga a que `lib/plazos/` sepa contar en días hábiles **y** en días naturales, que es
la parte técnicamente interesante del proyecto.

**Los vencimientos se materializan de forma perezosa** `[D-04]`. No hay tarea
programada. Toda lectura de un expediente pasa por `lib/dominio/`, que compara el reloj
inyectado con los plazos vivos y, si procede, materializa la transición y su traza antes
de devolver nada. Determinista, testeable sin infraestructura, y sin expedientes que
mienten porque el cron no ha pasado.

**Identificación simulada, sin representación** `[D-05]`. Acceso con identificador
sintético; la figura del representante queda modelada pero fuera del alcance inicial.

**Formularios fijos y tipados** `[D-06]`. Nada de motor genérico dirigido por datos: con
`strict`, cero `any` y cero aserciones no nulas, un motor genérico degenera en
`Record<string, unknown>` e incumple `CLAUDE.md` §5.

**Documentos reales, justificante con código seguro de verificación** `[D-07]`. Subida
real de ficheros con validación por número mágico, huella SHA-256, y una página pública
`/verificar/{csv}` donde un tercero comprueba la autenticidad de un justificante. Es el
invariante III convertido en funcionalidad visible.

**Cuatro roles con segregación de funciones** `[D-08]`. Quien instruye no resuelve.

---

## 3. Entidades

### 3.1 Convocatoria

| Atributo | Tipo | Notas |
| --- | --- | --- |
| `id` | `IdConvocatoria` | |
| `codigo` | `string` | Formato `CONV-{año}-{secuencia}`. Dato monoespaciado en interfaz |
| `titulo` | `string` | |
| `basesReguladoras` | `BasesReguladoras` | Texto legal íntegro más metadatos |
| `creditoTotal` | `Importe` | Céntimos enteros. **Nunca coma flotante** `[D-09]` |
| `creditoComprometido` | `Importe` | Derivado de las resoluciones favorables |
| `importeMaximoPorSolicitud` | `Importe` | |
| `puntuacionMinima` | `Puntuacion` | Umbral por debajo del cual la resolución es desfavorable |
| `fechaPublicacion` | `FechaCivil` | Inicia el cómputo del plazo máximo de resolución |
| `inicioPresentacion` | `InstanteUTC` | |
| `finPresentacion` | `InstanteUTC` | `23:59:59.999` de `Europe/Madrid` del último día |
| `estadoConvocatoria` | `EstadoConvocatoria` | `BORRADOR` · `PUBLICADA` · `CERRADA` · `RESUELTA` |
| `criterios` | `CriterioValoracion[]` | Exactamente 3 |
| `organoInstructor` | `string` | |

`BasesReguladoras`: `documentoId`, `huella`, `resumenLenguajeClaro` (opcional,
generado por IA, siempre etiquetado como orientativo y sin valor jurídico, según
`docs/diseno.md` §9).

`CriterioValoracion`: `id`, `nombre`, `descripcion`, `puntuacionMaxima`, `orden`.
El `orden` no es decorativo: es el criterio de desempate `[D-10]`.

### 3.2 PersonaInteresada

| Atributo | Tipo | Notas |
| --- | --- | --- |
| `id` | `IdPersona` | |
| `documentoIdentidad` | `NifSintetico \| NieSintetico` | Rangos reservados de `docs/dominio.md` §4 |
| `nombre`, `apellidos` | `string` | Del fichero de nombres sintéticos |
| `correo` | `CorreoSintetico` | Dominio `@ejemplo.valdeprieto.test` |
| `telefono` | `TelefonoSintetico` | |
| `municipioResidencia` | `CodigoMunicipio` | **Relevante para plazos**: art. 30.6 |
| `iban` | `IbanSintetico` | Solo si la convocatoria lo exige |
| `sintetico` | `true` | Literal, no booleano. Un registro no sintético no compila `[D-11]` |
| `runId` | `string` | Ejecución que lo creó |

### 3.3 Solicitud

Mientras es borrador, es mutable y no tiene efectos. Al registrarse queda **congelada**.

| Atributo | Tipo | Notas |
| --- | --- | --- |
| `id` | `IdSolicitud` | |
| `convocatoriaId` | `IdConvocatoria` | |
| `interesadaId` | `IdPersona` | |
| `datos` | `DatosSolicitud` | Tipado fijo por convocatoria `[D-06]` |
| `documentos` | `Documento[]` | |
| `versionCongelada` | `HuellaSHA256 \| null` | Se calcula en el registro y no cambia jamás |

### 3.4 Documento

| Atributo | Tipo | Notas |
| --- | --- | --- |
| `id`, `tipoDocumental`, `nombreOriginal` | | El nombre original **nunca** aparece en la URL |
| `almacenamientoId` | `string` | Nombre opaco |
| `mimeReal` | `string` | Detectado por número mágico, no por extensión `[D-12]` |
| `tamanoBytes` | `number` | |
| `huella` | `HuellaSHA256` | |
| `aportadoEn` | `InstanteUTC` | Sello de servidor |
| `origen` | `SOLICITUD \| SUBSANACION \| ALEGACION` | |

No hay análisis antivirus: queda fuera de alcance y así se declara `[D-13]`.

### 3.5 Expediente

Es la entidad con estado y con valor legal. Uno por solicitud registrada.

| Atributo | Tipo | Notas |
| --- | --- | --- |
| `id` | `IdExpediente` | |
| `numeroExpediente` | `string` | `EXP-{año}-{secuencia}`. Dato monoespaciado |
| `solicitudId` | `IdSolicitud` | 1:1, inmutable |
| `estado` | `EstadoExpediente` | Sección 4 |
| `asientoRegistral` | `AsientoRegistral` | Sección 3.6 |
| `tramitadorAsignadoId` | `IdUsuario \| null` | |
| `valoracion` | `Valoracion \| null` | |
| `plazosVivos` | `Plazo[]` | Como máximo uno a la vez `[D-14]` |
| `traza` | `AsientoTraza[]` | Inmutable, solo se añade |
| `silencioDesestimatorio` | `boolean` | **Derivado**, no almacenado. Sección 6.6 |

### 3.6 AsientoRegistral y Justificante

El registro es el acto irreversible. Genera número, sello y huella.

`AsientoRegistral`: `numeroRegistro` (`REG-{año}-{secuencia}`), `selloTiempo`
(`InstanteUTC`, **tomado en servidor**), `huellaSolicitud`, `csv`.

`Justificante`: `csv` (código seguro de verificación, 16 caracteres del alfabeto
**Crockford base32**, que excluye `I`, `L`, `O` y `U` para que no se confundan con `1`,
`0` y sus vecinas al dictarlo por teléfono `[D-15]`), `numeroRegistro`, `selloTiempo`,
`huella`, `expedienteId`, `urlVerificacion`.

**Un solo formato por identificador.** `numeroRegistro` es siempre `REG-2026-001427`: es
lo que se almacena, se copia, se busca y se compara. La agrupación con puntos medios
—`REG · 2026 · 001427`— es exclusivamente de **presentación**, para poder dictarlo. Dos
formatos para el mismo identificador legal es garantía de que tarde o temprano uno de los
dos miente.

El CSV se consulta en `/verificar/{csv}`, pública y sin autenticación, que devuelve
únicamente: número de registro, sello de tiempo, convocatoria y huella. **Ningún dato
personal** `[D-16]`.

### 3.7 Notificacion

| Atributo | Tipo | Notas |
| --- | --- | --- |
| `id`, `expedienteId`, `actoNotificado` | | `REQUERIMIENTO` · `PROPUESTA_PROVISIONAL` · `RESOLUCION` |
| `puestaADisposicion` | `InstanteUTC` | Arranca el plazo de 10 días naturales |
| `accedidaEn` | `InstanteUTC \| null` | |
| `estadoNotificacion` | `PUESTA_A_DISPOSICION \| ACCEDIDA \| RECHAZADA_POR_TRANSCURSO` | |
| `fechaEfectosNotificacion` | `FechaCivil` | Derivada. Sección 6.4 |

### 3.8 Requerimiento, Alegacion, Propuesta, Resolucion

`Requerimiento`: `expedienteId`, `defectos: DefectoTipificado[]`, `notificacionId`,
`plazo: Plazo`, `emitidoPorId`. Los defectos son un enumerado cerrado, nunca texto
libre: así el requerimiento es comparable, testeable y traducible a lenguaje claro `[D-17]`.

`Alegacion`: `expedienteId`, `texto`, `documentos`, `presentadaEn`, `resueltaEn`,
`sentido: ESTIMADA | DESESTIMADA | PARCIALMENTE_ESTIMADA`, `motivacion`.

`PropuestaResolucion`: `convocatoriaId`, `caracter: PROVISIONAL | DEFINITIVA`,
`lineas: LineaPropuesta[]`, `emitidaPorId`, `emitidaEn`, `notificacionIds`.
`LineaPropuesta`: `expedienteId`, `puntuacion`, `orden`, `importePropuesto`,
`sentidoPropuesto`.

`Resolucion`: `expedienteId`, `sentido: FAVORABLE | DESFAVORABLE`, `importeConcedido`,
`motivacion`, `firmadaPorId`, `firmadaEn`, `notificacionId`.

### 3.9 Actores

| Rol | Puede | Alcance inicial |
| --- | --- | --- |
| `persona_interesada` | Crear y registrar solicitud, aportar subsanación, alegar, desistir | Sí |
| `tramitador` | Asignarse expedientes, requerir subsanación, valorar, proponer | Sí |
| `jefe_servicio` | Inadmitir, resolver, publicar listados | Sí |
| `auditor` | Leer expedientes y trazas. **Ninguna** transición | Fase posterior `[D-18]` |

**Segregación de funciones** `[D-08]`: la persona que firma la propuesta no puede
firmar la resolución del mismo expediente. Es una precondición de dominio, no una regla
de interfaz, y tiene su transición prohibida (P-15).

### 3.10 AsientoTraza

Inmutable. Solo admite inserción; no hay actualización ni borrado.

`id`, `expedienteId`, `estadoOrigen`, `estadoDestino`, `transicionId` (`T-nn`),
`actorId | 'sistema'`, `rolActor`, `selloTiempo` (servidor), `motivo`,
`datosAdicionales`, `simulado: boolean`.

`simulado` marca las transiciones producidas con el reloj desplazado del modo
demostración. Una traza de demostración jamás se confunde con una real `[D-19]`.

### 3.11 CalendarioLaboral

`ambito: ESTATAL | AUTONOMICO | LOCAL`, `codigoAmbito`, `anio`, `festivos: FechaCivil[]`,
`version`, `fuente`.

Es un dato de configuración versionado en `lib/plazos/calendarios/`, nunca constantes
en el código. Cambiar un festivo cambia el resultado de un plazo: es un cambio con
consecuencias jurídicas y debe verse en el historial.

### 3.12 Relaciones

```
Convocatoria 1 ──── N Solicitud 1 ──── 1 Expediente
     │                                      │
     │ 1                                    │ 1
     │                                      ├── 1 AsientoRegistral ── 1 Justificante
     N                                      ├── N Documento
CriterioValoracion                          ├── N Notificacion
     │                                      ├── 0..1 Requerimiento
     │ valorado en                          ├── N Alegacion
     └──── N Valoracion ──── 1 Expediente   ├── 0..1 Resolucion
                                            └── N AsientoTraza   (inmutable)

Convocatoria 1 ──── 0..2 PropuestaResolucion   (provisional y definitiva)
PersonaInteresada 1 ──── N Solicitud
Usuario(rol) 1 ──── N AsientoTraza
```

---

## 4. Máquina de estados del expediente

### 4.1 Estados

| Código | Estado | Terminal | Token de color (`docs/diseno.md` §3) |
| --- | --- | --- | --- |
| `BOR` | `BORRADOR` | No | `--gris` + icono de lápiz |
| `REG` | `REGISTRADO` | No | `--sello` + cuño y número |
| `REV` | `EN_REVISION` | No | `--azul` + icono de reloj |
| `SUB` | `SUBSANACION_REQUERIDA` | No | `--lacre` + días hábiles restantes |
| `PPR` | `PROPUESTA_PROVISIONAL` | No | `--azul` + plazo de alegaciones `[D-20]` |
| `PDE` | `PROPUESTA_DEFINITIVA` | No | `--azul` + texto «pendiente de resolución» `[D-20]` |
| `RFA` | `RESUELTO_FAVORABLE` | **Sí** | `--verde-500` + check y fecha |
| `RDE` | `RESUELTO_DESFAVORABLE` | **Sí** | `--gris` + texto y motivo |
| `DES` | `DESISTIDO` | **Sí** | `--gris` + texto tachado y motivo |
| `INA` | `INADMITIDO` | **Sí** | `--gris` + texto tachado y motivo |

Cuatro decisiones sobre el conjunto de estados, todas divergencias respecto del diagrama
de `docs/dominio.md` §2:

- `RESUELTO` se parte en dos estados en lugar de un estado con atributo `sentido`
  `[D-21]`. `docs/diseno.md` §3 da color propio a «resuelto favorable»; con dos estados
  el token se deriva del estado y no de un campo, y las transiciones prohibidas se
  pueden enumerar.
- Se añade `INADMITIDO`, que no existía y es imprescindible con la decisión de la
  sección 2 sobre presentación fuera de plazo.
- Se añaden `PROPUESTA_PROVISIONAL` y `PROPUESTA_DEFINITIVA` donde el diagrama tenía
  una única `PROPUESTA`, porque la concurrencia competitiva exige las dos (Ley 38/2003,
  art. 24.4).
- **No** se añade un estado `ALEGACIONES`. El plazo de alegaciones es una ventana
  temporal del estado `PROPUESTA_PROVISIONAL`, no un estado: presentar una alegación no
  cambia el estado del expediente. Un estado más aquí sería ruido con coste de test.

No existe estado `CADUCADO`. El vencimiento del plazo máximo de resolución no cambia el
estado: produce silencio desestimatorio, que es un derecho de la persona interesada, no
una transición. Sección 6.6.

### 4.2 Tabla exhaustiva de transiciones permitidas

| Id | Origen | Destino | Disparador | Actor autorizado | Precondiciones | Efectos secundarios |
| --- | --- | --- | --- | --- | --- | --- |
| **T-01** | `BOR` | `REG` | `registrarSolicitud` | `persona_interesada` titular del borrador | Convocatoria `PUBLICADA` o `CERRADA`; formulario válido según el esquema Zod compartido; documentación obligatoria presente; declaración responsable aceptada; información de tratamiento mostrada antes del envío | Sello de tiempo de servidor; número de registro; huella SHA-256 de la solicitud; congelación de la solicitud; creación del expediente; generación de justificante y CSV; traza |
| **T-02** | `REG` | `REV` | `asignarInstructor` | `tramitador` (autoasignación desde bandeja) o `jefe_servicio` | Expediente sin instructor; convocatoria no `RESUELTA` | `tramitadorAsignadoId`; traza |
| **T-03** | `REG` | `INA` | `inadmitir` | `jefe_servicio` | `selloTiempo` posterior a `finPresentacion`, **o** motivo tipificado de inadmisión; motivo obligatorio | Resolución de inadmisión; notificación; traza |
| **T-04** | `REG` | `DES` | `desistir` | `persona_interesada` titular | Escrito de desistimiento registrado | Asiento de registro del escrito; justificante; traza con motivo `desistimiento_voluntario` |
| **T-05** | `REV` | `SUB` | `requerirSubsanacion` | `tramitador` asignado | Al menos un `DefectoTipificado`; **no existe requerimiento previo** en el expediente; convocatoria no `RESUELTA` | `Requerimiento`; `Notificacion` en `PUESTA_A_DISPOSICION`; apertura del plazo de 10 días hábiles; traza |
| **T-06** | `REV` | `PPR` | `publicarPropuestaProvisional` | `tramitador` instructor de la convocatoria | Convocatoria `CERRADA`; **todos** los expedientes valorables valorados en los 3 criterios; crédito y orden calculados | Acto sobre la convocatoria que alcanza a todos sus expedientes en `REV`; `PropuestaResolucion` provisional; notificación a cada interesada; apertura del plazo de alegaciones; traza por expediente |
| **T-07** | `REV` | `INA` | `inadmitir` | `jefe_servicio` | Motivo tipificado (falta de legitimación, duplicidad, convocatoria equivocada); motivo obligatorio | Resolución de inadmisión; notificación; traza |
| **T-08** | `REV` | `DES` | `desistir` | `persona_interesada` titular | Escrito registrado | Igual que T-04 |
| **T-09** | `SUB` | `REV` | `aportarSubsanacion` | `persona_interesada` titular | **Dentro de plazo**; al menos un documento nuevo válido | Registro de entrada del documento; justificante; cierre del plazo; traza |
| **T-10** | `SUB` | `DES` | `vencimientoPlazoSubsanacion` | `sistema` | Plazo vencido sin aportación; materializado de forma perezosa | Traza con `actorId: 'sistema'` y motivo `vencimiento_plazo_subsanacion`; notificación de la resolución de desistimiento |
| **T-11** | `SUB` | `DES` | `desistir` | `persona_interesada` titular | Escrito registrado dentro o fuera de plazo | Igual que T-04 |
| **T-12** | `PPR` | `PDE` | `elevarAPropuestaDefinitiva` | `tramitador` instructor | Plazo de alegaciones vencido para **todos** los expedientes de la convocatoria; todas las alegaciones resueltas con motivación | `PropuestaResolucion` definitiva; notificación; traza por expediente |
| **T-13** | `PPR` | `DES` | `desistir` | `persona_interesada` titular | Escrito registrado | Igual que T-04; libera el importe propuesto |
| **T-14** | `PDE` | `RFA` | `resolverFavorable` | `jefe_servicio` | **No ser quien firmó la propuesta**; puntuación ≥ `puntuacionMinima`; crédito disponible ≥ importe; dentro del plazo máximo de resolución | `Resolucion` favorable; incremento de `creditoComprometido`; notificación; traza |
| **T-15** | `PDE` | `RDE` | `resolverDesfavorable` | `jefe_servicio` | **No ser quien firmó la propuesta**; motivo tipificado (puntuación insuficiente, crédito agotado, incumplimiento de requisitos); motivación obligatoria | `Resolucion` desfavorable; notificación; traza |
| **T-16** | `PDE` | `DES` | `desistir` | `persona_interesada` titular | Escrito registrado antes de la resolución | Igual que T-04 |

Fuera de la máquina de estados, y por eso no figuran como transiciones: la eliminación
de un borrador a petición de la persona interesada (`docs/dominio.md` §5), que es un
borrado real porque un borrador no tiene efectos; y el acceso a una notificación, que
cambia el estado de la `Notificacion`, no el del expediente.

### 4.3 Matriz completa de adyacencia

Diez estados, cien pares ordenados. **Quince** pares permitidos, que contienen dieciséis
transiciones — `SUB → DES` admite dos disparadores distintos. **Ochenta y cinco** pares
prohibidos.

| Origen \ Destino | BOR | REG | REV | SUB | PPR | PDE | RFA | RDE | DES | INA |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| **BOR** | ✕ | T-01 | ✕ | ✕ | ✕ | ✕ | ✕ | ✕ | ✕ | ✕ |
| **REG** | ✕ | ✕ | T-02 | ✕ | ✕ | ✕ | ✕ | ✕ | T-04 | T-03 |
| **REV** | ✕ | ✕ | ✕ | T-05 | T-06 | ✕ | ✕ | ✕ | T-08 | T-07 |
| **SUB** | ✕ | ✕ | T-09 | ✕ | ✕ | ✕ | ✕ | ✕ | T-10 / T-11 | ✕ |
| **PPR** | ✕ | ✕ | ✕ | ✕ | ✕ | T-12 | ✕ | ✕ | T-13 | ✕ |
| **PDE** | ✕ | ✕ | ✕ | ✕ | ✕ | ✕ | T-14 | T-15 | T-16 | ✕ |
| **RFA** | ✕ | ✕ | ✕ | ✕ | ✕ | ✕ | ✕ | ✕ | ✕ | ✕ |
| **RDE** | ✕ | ✕ | ✕ | ✕ | ✕ | ✕ | ✕ | ✕ | ✕ | ✕ |
| **DES** | ✕ | ✕ | ✕ | ✕ | ✕ | ✕ | ✕ | ✕ | ✕ | ✕ |
| **INA** | ✕ | ✕ | ✕ | ✕ | ✕ | ✕ | ✕ | ✕ | ✕ | ✕ |

**Mundo cerrado.** Toda celda marcada `✕` es una transición prohibida que debe fallar de
forma explícita con un error de dominio tipado. No existe transición por defecto, no
existe `default:` que deje pasar, y la función de transición es exhaustiva sobre el
enumerado: si mañana se añade un estado, el compilador señala todos los puntos que hay
que decidir.

---

## 5. Transiciones prohibidas

Esto es lo que de verdad hay que testear. La tabla de la sección 4.2 dice qué debe
funcionar; esta dice qué no debe poder ocurrir nunca, ni desde la interfaz, ni desde una
Server Action, ni desde una llamada directa a la capa de dominio.

### 5.1 Prohibiciones estructurales

| Id | Regla | Pares afectados | Error de dominio |
| --- | --- | --- | --- |
| **P-01** | Ningún estado vuelve a `BORRADOR`. El registro es irreversible | 9 | `RegistroEsIrreversible` |
| **P-02** | Ningún estado vuelve a `REGISTRADO` | 8 | `TransicionNoContemplada` |
| **P-03** | Desde `RFA`, `RDE`, `DES` o `INA` no sale ninguna transición | 36 | `ExpedienteEnEstadoTerminal` |
| **P-04** | Ningún estado transita a sí mismo | 10 | `TransicionNoContemplada` |
| **P-05** | `REG → PPR`, `REG → PDE`, `REG → RFA`, `REG → RDE`, `REG → SUB`: no se instruye sin asignar instructor | 5 | `InstruccionNoIniciada` |
| **P-06** | `REV → PDE`, `REV → RFA`, `REV → RDE`: no se resuelve sin propuesta provisional | 3 | `PropuestaProvisionalRequerida` |
| **P-07** | `SUB → PPR`, `SUB → PDE`, `SUB → RFA`, `SUB → RDE`, `SUB → INA`: con un requerimiento vivo no se avanza | 5 | `PlazoDeSubsanacionAbierto` |
| **P-08** | `PPR → RFA`, `PPR → RDE`: no se resuelve saltándose la propuesta definitiva | 2 | `PropuestaDefinitivaRequerida` |
| **P-09** | `PPR → SUB`, `PDE → SUB`, `PDE → PPR`, `PPR → REV`, `PDE → REV`: no se retrocede la instrucción | 5 | `TransicionNoContemplada` |
| **P-10** | `BOR → *` salvo `REG`: un borrador no se inadmite, ni se desiste, ni se resuelve. Se elimina | 8 | `TransicionNoContemplada` |
| **P-11** | `PPR → INA`, `PDE → INA`: emitida una propuesta ya no se inadmite; se resuelve desfavorablemente | 2 | `InadmisionFueraDePlazoProcesal` |

Estas once reglas cubren los 85 pares prohibidos de la matriz. Las cuentas de la columna
«pares afectados» se solapan a propósito —`RFA → BOR` lo prohíben P-01 y P-03 a la vez—;
lo que se garantiza no es una partición, sino que **ningún par prohibido queda sin al
menos una regla que lo rechace**. El test parametrizado de la sección 5.4 lo comprueba
recorriendo la matriz entera, que es la única forma de que esta afirmación no envejezca.

### 5.2 Prohibiciones por actor

| Id | Regla | Error de dominio |
| --- | --- | --- |
| **P-12** | Una `persona_interesada` no ejecuta T-02, T-03, T-05, T-06, T-07, T-12, T-14 ni T-15 | `ActorNoAutorizado` |
| **P-13** | Una `persona_interesada` no actúa sobre un expediente del que no es titular, ni siquiera para leerlo | `ExpedienteAjeno` |
| **P-14** | Un `tramitador` no ejecuta T-03, T-07, T-14 ni T-15: no inadmite y no resuelve | `ActorNoAutorizado` |
| **P-15** | Un `jefe_servicio` no ejecuta T-14 ni T-15 sobre un expediente cuya propuesta firmó él mismo | `SegregacionDeFuncionesVulnerada` |
| **P-16** | Un `tramitador` no requiere subsanación sobre un expediente que no tiene asignado | `ExpedienteNoAsignado` |
| **P-17** | El actor `sistema` solo ejecuta T-10. Ninguna otra transición admite actor automático | `ActorNoAutorizado` |
| **P-18** | El rol `auditor` no ejecuta ninguna transición | `ActorNoAutorizado` |

### 5.3 Prohibiciones por precondición

| Id | Regla | Error de dominio |
| --- | --- | --- |
| **P-19** | T-09 fuera de plazo, aunque sea por un milisegundo del sello de servidor | `PlazoDeSubsanacionVencido` |
| **P-20** | Un segundo requerimiento de subsanación sobre el mismo expediente | `SubsanacionYaRequerida` |
| **P-21** | T-06 con algún expediente de la convocatoria sin valorar | `ValoracionIncompleta` |
| **P-22** | T-12 con el plazo de alegaciones abierto, o con alegaciones sin resolver | `AlegacionesPendientes` |
| **P-23** | T-14 con `creditoComprometido + importe > creditoTotal` | `CreditoInsuficiente` |
| **P-24** | T-14 con puntuación por debajo de `puntuacionMinima` | `PuntuacionInsuficiente` |
| **P-25** | Cualquier transición que exige motivo, ejecutada con motivo vacío o solo espacios | `MotivoRequerido` |
| **P-26** | T-01 sobre una convocatoria en estado `BORRADOR` | `ConvocatoriaNoPublicada` |
| **P-27** | Cualquier transición sobre un expediente de convocatoria `RESUELTA` salvo las de desistimiento | `ConvocatoriaCerrada` |
| **P-28** | Cualquier escritura sobre `Solicitud` una vez existe `versionCongelada` | `SolicitudInmutable` |
| **P-29** | Cualquier actualización o borrado sobre `AsientoTraza` | `TrazaInmutable` |

### 5.4 Cómo se testea

Cada regla `P-nn` es un test unitario en `lib/dominio/`. Las prohibiciones estructurales
se generan recorriendo la matriz completa: el test itera los 100 pares, comprueba que los
15 permitidos devuelven un resultado correcto y que los 85 restantes devuelven el error
tipado que corresponde. Es una sola prueba parametrizada y cubre la máquina entera; si
alguien añade una transición sin actualizar la matriz, falla.

Los errores son **tipos de resultado explícitos**, no excepciones lanzadas al aire
(`CLAUDE.md` §5): la interfaz tiene que poder distinguir `PlazoDeSubsanacionVencido` de
un fallo de infraestructura, porque el primero se le explica a la persona y el segundo no.

---

## 6. Cómputo de plazos

Vive en `lib/plazos/`. Sin React, sin I/O, cobertura del 100 %. Es la parte del proyecto
donde un error no se ve en pantalla y sin embargo tiene consecuencias jurídicas.

### 6.1 Reglas de implementación innegociables

1. **Nunca aritmética de milisegundos.** Sumar `10 * 24 * 60 * 60 * 1000` es un error en
   los dos cambios de hora del año. Se opera sobre fechas civiles y se convierte a
   instante solo en los bordes.
2. **Zona `Europe/Madrid` fija**, explícita en cada conversión. Nunca la zona del proceso
   ni la del navegador.
3. **El reloj es una dependencia inyectada**, nunca `new Date()` dentro de una regla. Sin
   esto no hay forma honesta de testear un plazo de diez días hábiles.
4. **El sello de tiempo se toma en servidor**, en el momento de confirmarse la
   transacción, nunca en el cliente y nunca antes de que la escritura sea firme.
5. **El calendario es dato versionado**, no constantes. Un festivo mal puesto es un bug
   con efectos legales y tiene que verse en el historial.

### 6.2 Definición de día hábil

Un día es inhábil si es sábado, domingo o festivo declarado (Ley 39/2015, art. 30.2).

El art. 30.6 añade una regla que casi nadie implementa y que este proyecto sí implementa
`[D-14]`: es inhábil el día que lo sea **en el municipio de residencia de la persona
interesada o en la sede del órgano**. El cómputo usa por tanto la **unión** de dos
calendarios: el de Valdeprieto (sede) y el del municipio de residencia. Un festivo local
en cualquiera de los dos hace inhábil el día.

Consecuencia de diseño: el cálculo de un plazo necesita saber dónde reside la persona.
Por eso `municipioResidencia` está en el modelo y por eso su ausencia es un error, no un
campo opcional que se rellena luego.

### 6.3 Plazos por días hábiles

```
calcularVencimientoHabil(fechaNotificacion, dias, calendarios) -> FechaCivil
```

1. El cómputo arranca el **día siguiente** al de la notificación (art. 30.3), sea hábil o
   no ese día siguiente.
2. Se cuentan solo los días hábiles, uno a uno, hasta alcanzar `dias`.
3. Si el día resultante es inhábil, se prorroga al primer día hábil siguiente
   (art. 30.5).
4. El plazo vence a las `23:59:59.999` de ese día en `Europe/Madrid`.

Plazos del producto: subsanación **10 días hábiles** (art. 68.1); alegaciones tras la
propuesta provisional **10 días hábiles** `[D-15]`, plazo que las bases reguladoras de la
convocatoria ficticia fijan de forma expresa.

### 6.4 Plazos por días naturales: el rechazo de la notificación

Regla distinta y por eso función distinta. La notificación puesta a disposición y no
accedida se entiende **rechazada** a los diez días **naturales** (art. 43.2). El cómputo
arranca el día siguiente a la puesta a disposición `[D-22]`. El precepto no lo dice de
forma expresa: es la lectura que hace la jurisprudencia por coherencia con el art. 30.3 y
por seguridad jurídica. Queda marcada como interpretación, no como texto de la norma, y
pendiente de citar sentencia concreta antes de darla por firme.

La `fechaEfectosNotificacion` es, por tanto:

- Si se accedió: la fecha del acceso.
- Si no se accedió y transcurrieron los diez días naturales: la fecha en que se cumplen.

Y **sobre esa fecha** se calcula después el plazo de subsanación en días hábiles. Es una
composición de los dos cómputos, y es el caso que más se equivoca en producción.

### 6.5 Plazos por meses

El plazo máximo de resolución es de seis meses desde la publicación de la convocatoria
(Ley 38/2003, art. 25.4). Cómputo de fecha a fecha desde el día siguiente (art. 30.4).
Si el mes de vencimiento no tiene día equivalente —31 de enero más un mes—, vence el
último día del mes.

### 6.6 Silencio desestimatorio

Vencido el plazo máximo sin resolución notificada, la persona interesada puede entender
desestimada su solicitud (Ley 38/2003, art. 25.5). **No es una transición.** Es un
atributo derivado que la interfaz muestra junto al estado real del expediente, con su
explicación en lenguaje claro y el enlace al precepto. El expediente sigue donde estaba y
la Administración sigue obligada a resolver.

### 6.7 Casos límite obligatorios

Los diez de `docs/dominio.md` §3, más seis que añade este modelo. Todos con test
unitario y valores esperados calculados a mano, no generados por el propio código.

| # | Caso | Por qué rompe una implementación ingenua |
| --- | --- | --- |
| 1 | Vencimiento en sábado | Exige la prórroga del art. 30.5 |
| 2 | Vencimiento en festivo local | Exige que el calendario local esté cargado |
| 3 | Plazo que cruza Navidad | Varios inhábiles consecutivos |
| 4 | Plazo que cruza el cambio de hora de marzo | El día tiene 23 horas |
| 5 | Plazo que cruza el cambio de hora de octubre | El día tiene 25 horas |
| 6 | Plazo de 0 días | ¿Vence hoy o mañana? Debe estar decidido, no descubierto |
| 7 | Notificación a las 23:59:59 | El día siguiente es el mismo con o sin el segundo |
| 8 | Notificación en día inhábil | El arranque es el día siguiente aunque sea inhábil |
| 9 | Año bisiesto | 29 de febrero en cómputo por meses |
| 10 | Festivo que cae en domingo | No se cuenta dos veces ni se traslada solo |
| 11 | Plazo que cruza fin de año | Cambia el calendario aplicable a mitad de plazo |
| 12 | **Festivo local solo en el municipio de residencia** | Art. 30.6: unión de calendarios |
| 13 | **Festivo local solo en la sede** | Art. 30.6, caso simétrico |
| 14 | **Cómputo compuesto: rechazo por transcurso y después subsanación** | Encadena días naturales con días hábiles |
| 15 | **Registro a las 23:59:59.999 del último día de presentación** | Frontera exacta entre admitir e inadmitir |
| 16 | **Cómputo por meses hacia un mes más corto** | 31 de enero más un mes vence el 28 o el 29 |

---

## 7. Discrepancias detectadas en los documentos normativos

Cuatro, todas resueltas en este documento pero pendientes de tu confirmación.

**7.1 `docs/dominio.md` §2 contradice `docs/testing.md` §6.** El diagrama ASCII lleva la
flecha «aporta» desde `SUBSANACION_REQ` hasta `PROPUESTA`. El ejemplo Gherkin de
`docs/testing.md` dice: «*Cuando* la persona interesada aporta el certificado *Entonces*
el expediente pasa a estado "en revisión"». **Se ha resuelto a favor de `testing.md`**
(T-09): quien aporta documentación devuelve el expediente a instrucción, porque alguien
tiene que revisar lo aportado. La lectura del diagrama permitiría proponer sin revisar,
que es indefendible.

**7.2 El desistimiento automático diverge de la norma citada.** `docs/dominio.md` §2 dice
que el vencimiento del plazo «produce `DESISTIDO` de forma automática», con actor
`sistema`. El art. 68.1 de la Ley 39/2015 dice que se le tendrá por desistido «**previa
resolución** que deberá ser dictada en los términos previstos en el artículo 21». Es
decir, en Derecho hace falta un acto expreso. **Manda el documento** (T-10 automática),
pero queda anotado: la alternativa fiel sería un estado intermedio
`PENDIENTE_DESISTIMIENTO` que el `jefe_servicio` confirma. Si quieres fidelidad legal
completa, este es el punto a cambiar y merece su propio ADR.

**7.3 `docs/diseno.md` §3 agrupa «Desistido / caducado» en una fila.** Este modelo no
tiene estado `CADUCADO`; esa fila cubre `DESISTIDO` e `INADMITIDO`, ambos en `--gris`
con texto tachado y motivo. Si en algún momento aparece caducidad por inactividad, será
un estado nuevo con su propia entrada en la tabla.

**7.4 Rutas.** Los tres documentos normativos estaban en la raíz mientras `CLAUDE.md` los
importaba desde `docs/`. Se han movido a `docs/` y se han creado `docs/backlog/`,
`docs/decisiones/`, `docs/accesibilidad/` y `docs/flaky/`. El repositorio se ha
inicializado en git, que `CLAUDE.md` §12 daba por hecho.

---

## 8. Decisiones tomadas sin consulta

Todas están tomadas por indicación expresa de responder al interrogatorio en lugar de
esperar respuestas. Ninguna es irreversible en esta fase, y las cinco primeras son las
que más código mueven si cambian.

| Id | Decisión | Si se revierte |
| --- | --- | --- |
| **D-01** | Concurrencia competitiva, no orden de entrada | Desaparecen `PPR`, `PDE`, valoración y alegaciones: la máquina baja de 16 transiciones a 9 |
| **D-02** | Tres criterios de valoración, sin prorrateo del crédito | Cambia solo la valoración, no la máquina |
| **D-03** | Notificación como entidad, con rechazo por transcurso | Desaparece el cómputo en días naturales y media sección 6 |
| **D-04** | Vencimientos materializados de forma perezosa, sin tarea programada | Hace falta infraestructura de planificación y los tests dejan de ser deterministas |
| **D-05** | Identificación simulada, sin representante | Añadir representación duplica las reglas de autorización |
| **D-06** | Formularios fijos y tipados, no dirigidos por datos | Un motor genérico obliga a revisar `CLAUDE.md` §5 |
| **D-07** | Documentos reales, justificante con CSV y página pública de verificación | Se pierde la funcionalidad que hace visible el invariante III |
| **D-08** | Cuatro roles con segregación de funciones | Desaparece P-14 |
| **D-09** | Importes en céntimos enteros, nunca coma flotante | — |
| **D-10** | Desempate por el criterio de menor `orden`, y después por sello de registro | Hay que definir otro desempate; no puede no haberlo |
| **D-11** | `sintetico` como tipo literal `true`, no booleano | Se pierde la garantía en tiempo de compilación de `docs/dominio.md` §4 |
| **D-12** | Validación de ficheros por número mágico, no por extensión | — |
| **D-13** | Sin análisis antivirus, declarado como exclusión | — |
| **D-14** | Un solo plazo vivo por expediente; unión de calendarios del art. 30.6 | Simplificar a un solo calendario haría el cómputo más fácil y menos correcto |
| **D-15** | CSV de 16 caracteres dictables; alegaciones de 10 días hábiles | — |
| **D-16** | La página de verificación no muestra ningún dato personal | — |
| **D-17** | Defectos de subsanación como enumerado cerrado, no texto libre | Se pierde comparabilidad y resumen en lenguaje claro |
| **D-18** | Rol `auditor` modelado pero fuera del alcance inicial | — |
| **D-19** | Marca `simulado` en la traza para el modo demostración | Sin ella, una demo con el reloj desplazado es indistinguible de un acto real |
| **D-20** | `PPR` y `PDE` usan el token `--azul` de «en revisión» | `docs/diseno.md` §3 no les asigna token: hay que decidirlo o crear uno |
| **D-21** | `RESUELTO` partido en `RFA` y `RDE` en lugar de un atributo `sentido` | El token de color pasa a derivarse de un campo y no del estado |
| **D-22** | El plazo de rechazo del art. 43.2 arranca el día siguiente a la puesta a disposición, no el mismo día | Si arranca el mismo día, todos los plazos encadenados se adelantan una jornada |

---

## 9. Qué falta para poder implementar

Este documento define el dominio, no el sistema. Antes de la primera línea de código
hacen falta: el backlog con criterios de aceptación (Trabajo 3), la aplicación del
sistema de diseño a las pantallas (Trabajo 4), el ADR de arquitectura (Trabajo 5) y el
plan de pruebas (Trabajo 6).
