# Sede Electrónica del Ayuntamiento de Valdeprieto

> **Portal de demostración. Municipio ficticio. Sin validez administrativa.**

Portal de tramitación de ayudas y subvenciones municipales. Es una pieza de portfolio
construida con estándar de producción real: el listón no es «una demo que funciona», es
«esto podría entrar en un pliego del sector público español y pasar la auditoría».

---

## Qué se demuestra aquí

**Accesibilidad como requisito funcional, no como mejora.** El RD 1112/2018 obliga al
sector público español a cumplir la norma EN 301 549, equiparada a WCAG 2.1 nivel AA. En
este proyecto eso se traduce en cero violaciones `serious` o `critical` de axe-core en
CI, recorrido completo con teclado verificado a mano, y contraste calculado —no
estimado— contra una matriz publicada.

**Un dominio administrativo con valor legal.** El expediente es una máquina de estados
con diez estados, dieciséis transiciones y ochenta y cinco pares prohibidos, todos
testeados. Los plazos se cuentan en días hábiles con calendario de festivos versionado,
nunca con aritmética de fechas.

**Calidad verificable por un tercero.** Cada funcionalidad entra con sus pruebas y su
evidencia publicada, y el justificante de registro se puede comprobar en una página
pública mediante código seguro de verificación.

**Dos tracks de automatización medidos.** Los mismos seis flujos críticos en Playwright
y en Selenium con Cucumber, comparados con datos de la propia CI.

## Estado

En fase de diseño. **Todavía no hay código de aplicación.** Lo que existe ahora mismo es
la documentación que lo precede: modelo de dominio, backlog con criterios de aceptación
y aplicación del sistema de diseño a las pantallas clave.

| Documento | Contenido |
| --- | --- |
| [CLAUDE.md](CLAUDE.md) | Acuerdo de trabajo, invariantes, stack y prohibiciones |
| [docs/dominio.md](docs/dominio.md) | Vocabulario administrativo y reglas de plazos |
| [docs/diseno.md](docs/diseno.md) | Sistema de diseño con matriz de contraste calculada |
| [docs/testing.md](docs/testing.md) | Contrato de testing |
| [docs/modelo-dominio.md](docs/modelo-dominio.md) | Entidades, máquina de estados y transiciones prohibidas |
| [docs/backlog/](docs/backlog/) | Nueve épicas, cuarenta y ocho historias |
| [docs/maquetas.md](docs/maquetas.md) | Las cinco pantallas clave con contraste, foco y anuncios |
| [docs/decisiones/](docs/decisiones/) | Registros de decisiones de arquitectura |
| [docs/deuda-tecnica.md](docs/deuda-tecnica.md) | Lo que se ha visto y no se ha arreglado todavía |
| [docs/trazabilidad.md](docs/trazabilidad.md) | Historia → criterio → caso de prueba → test |

## Stack

Next.js con App Router · React · TypeScript `strict` · Tailwind sobre tokens propios ·
PostgreSQL con Prisma · Zod compartido cliente/servidor · Storybook · Vitest con Testing
Library · Playwright · Selenium 4 con Java 21, JUnit 5 y Cucumber · axe-core ·
Lighthouse CI · GitHub Actions.

## Comandos

```bash
pnpm dev                        # desarrollo
pnpm build                      # build de producción
pnpm lint                       # ESLint + Prettier
pnpm typecheck                  # tsc --noEmit
pnpm test                       # Vitest
pnpm test:e2e                   # Playwright
pnpm test:a11y                  # axe sobre todas las rutas
pnpm storybook
pnpm lhci                       # Lighthouse CI con presupuestos
mvn -f selenium/pom.xml verify  # track B
```

---

## Sobre el municipio ficticio

Valdeprieto no existe. Su nombre, su escudo, su logotipo, sus direcciones, sus
convocatorias y todas las personas que aparecen en el proyecto son inventados. Ninguna
pantalla, ningún documento y ningún dato de prueba reproduce marcas, escudos ni
identificadores de instituciones o personas reales.

### Comprobación contra el registro de municipios del INE

`docs/dominio.md` §6 exige comprobar que el nombre no coincide con ningún municipio real
y dejar constancia aquí.

**Fecha de la comprobación:** 16 de agosto de 2026.

**Método:** búsqueda del topónimo «Valdeprieto» contra la relación de municipios,
provincias y comunidades autónomas del Instituto Nacional de Estadística y contra los
listados públicos de municipios españoles.

**Resultado:** no existe ningún municipio español denominado Valdeprieto. Sí existen
topónimos próximos, que **no** se usan en este proyecto y de los que conviene dejar
constancia para que la diferencia sea deliberada y no casual:

| Municipio real | Provincia |
| --- | --- |
| Valdeprados | Segovia |
| Valdeprado | Soria |
| Valdeprado del Río | Cantabria |

**Límite de la comprobación, declarado de forma expresa:** se ha realizado por búsqueda
sobre fuentes públicas, no por comparación byte a byte contra el fichero oficial de
códigos de municipio del INE. Antes de dar el proyecto por publicable en su versión
final, la comprobación debe repetirse descargando el diccionario oficial de municipios y
buscando el topónimo en él. Queda anotado en
[docs/deuda-tecnica.md](docs/deuda-tecnica.md).

**Fuente:** [INEbase · Relación de municipios, provincias, comunidades autónomas y sus
códigos](https://www.ine.es/dynt3/inebase/index.htm?padre=525)

### Datos sintéticos

Ningún dato personal de este proyecto puede corresponder a una persona real. Los rangos
reservados están fijados en `docs/dominio.md` §4:

- **NIF/NIE:** rango reservado `99000000`–`99999999`, con letra de control correcta.
- **IBAN:** prefijo de pruebas `ES00 0000`.
- **Teléfonos:** rango `600 00 00 00`–`600 00 99 99`.
- **Correo:** dominio `@ejemplo.valdeprieto.test`.
- **Nombres:** exclusivamente de `tests/datos/nombres-sinteticos.json`.

Todo registro sintético lleva la marca `sintetico: true` y el identificador de la
ejecución que lo creó.

## Licencia

Pendiente de decidir.
