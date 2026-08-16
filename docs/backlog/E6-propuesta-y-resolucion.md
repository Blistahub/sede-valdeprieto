# E6 · Propuesta, alegaciones y resolución

**29 puntos.** Semanas 9–11. Es el **flujo crítico 6**: el jefe de servicio resuelve y
publica el listado definitivo.

Aquí se reparte dinero público. Todas las historias de esta épica exigen motivación
registrada y todas dejan traza.

---

## E6-H01 · Publicar la propuesta de resolución provisional

> Como **tramitador instructor** quiero **publicar la propuesta provisional con el orden
> y los importes** para **que cada persona conozca su posición antes de la resolución y
> pueda alegar**.

```gherkin
# language: es
Característica: Propuesta de resolución provisional

  Escenario: Se publica la propuesta con el orden de puntuación
    Dado una convocatoria cerrada con todos sus expedientes valorados
    Cuando el instructor publica la propuesta provisional
    Entonces los expedientes quedan ordenados por puntuación
    Y cada uno conoce su puntuación, su posición y el importe propuesto
    Y todos pasan a estado "propuesta provisional"

  Escenario: El crédito se asigna por orden hasta agotarse
    Dado una convocatoria cuyo crédito no alcanza para todos los expedientes
    Cuando se publica la propuesta provisional
    Entonces se propone importe hasta agotar el crédito siguiendo el orden
    Y los siguientes constan como propuestos desfavorablemente por agotamiento del crédito

  Escenario: No se publica con expedientes sin valorar
    Dado una convocatoria con un expediente sin valorar
    Cuando se intenta publicar la propuesta provisional
    Entonces la operación se rechaza con el error de dominio "valoración incompleta"

  Escenario: La publicación abre el plazo de alegaciones
    Cuando se publica la propuesta provisional
    Entonces se notifica a cada persona interesada
    Y se abre un plazo de 10 días hábiles para alegar
```

**Criterios de accesibilidad**

- El listado publicado es una tabla con `<caption>`, `<th scope>` y orden accesible.
  Nunca una imagen ni un PDF como única forma de consulta.
- Cada persona identifica su propia posición sin tener que buscar entre todas: su
  expediente se destaca con texto además de con fondo.
- Los datos personales del listado público se muestran minimizados conforme a
  `docs/dominio.md` §5.
- Puntuaciones, posiciones e importes en fuente de dato con cifras tabulares, alineadas
  para poder compararse en columna.

**Estimación** 8 · **Depende de** E5-H05, E0-H03 · **Flujo crítico** 6 · **Semana** 9

---

## E6-H02 · Presentar alegaciones

> Como **persona interesada** quiero **alegar contra la propuesta provisional dentro de
> plazo** para **corregir un error de valoración antes de que sea definitiva**.

```gherkin
# language: es
Característica: Alegaciones

  Escenario: Se presenta una alegación dentro de plazo
    Dado un expediente con propuesta provisional y plazo de alegaciones abierto
    Cuando la persona interesada presenta una alegación con documentación
    Entonces la alegación queda registrada con sello de tiempo
    Y obtiene justificante
    Y el expediente permanece en estado "propuesta provisional"

  Escenario: No se admite una alegación fuera de plazo
    Dado un plazo de alegaciones vencido
    Cuando la persona interesada intenta alegar
    Entonces la operación se rechaza indicando la fecha en que venció el plazo

  Escenario: Se pueden presentar varias alegaciones dentro del plazo
    Dado un expediente con una alegación ya presentada y el plazo abierto
    Cuando la persona interesada presenta otra
    Entonces ambas quedan registradas de forma independiente
```

**Criterios de accesibilidad**

- Presentar una alegación **no** cambia el estado del expediente, y la interfaz lo dice
  con texto para que nadie crea que su expediente se ha detenido.
- El campo de alegación tiene etiqueta, indica si hay límite de extensión y anuncia los
  caracteres restantes en texto, no solo con un contador de color.
- El justificante de la alegación se anuncia y recibe el foco igual que el del registro.

**Estimación** 5 · **Depende de** E6-H01, E0-H01 · **Flujo crítico** 6 · **Semana** 10

---

## E6-H03 · Resolver las alegaciones

> Como **tramitador instructor** quiero **resolver cada alegación de forma motivada**
> para **que la propuesta definitiva se sostenga ante una reclamación**.

```gherkin
# language: es
Característica: Resolución de alegaciones

  Escenario: Cada alegación se resuelve con sentido y motivación
    Dado una alegación presentada
    Cuando el instructor la resuelve
    Entonces consta su sentido y su motivación
    Y la persona interesada puede consultarla

  Escenario: Una alegación estimada puede cambiar la valoración
    Dado una alegación estimada que corrige la puntuación de un criterio
    Cuando se actualiza la valoración
    Entonces la nueva puntuación queda registrada junto a la anterior
    Y consta la alegación como causa del cambio

  Escenario: No se resuelve una alegación sin motivación
    Cuando se intenta resolver una alegación sin motivarla
    Entonces la operación se rechaza con el error de dominio "motivo requerido"
```

**Criterios de accesibilidad**

- El sentido de la resolución se comunica con texto completo —«Estimada»,
  «Desestimada», «Parcialmente estimada»— además de con color y forma.
- El historial de cambios de puntuación se presenta como tabla accesible, no como
  texto corrido.

**Estimación** 3 · **Depende de** E6-H02 · **Flujo crítico** 6 · **Semana** 10

---

## E6-H04 · Elevar a propuesta definitiva

> Como **tramitador instructor** quiero **elevar la propuesta a definitiva una vez
> resueltas todas las alegaciones** para **cerrar la instrucción y dejarla lista para
> resolver**.

```gherkin
# language: es
Característica: Propuesta de resolución definitiva

  Escenario: Se eleva con todas las alegaciones resueltas
    Dado una convocatoria con el plazo de alegaciones vencido y todas resueltas
    Cuando el instructor eleva la propuesta a definitiva
    Entonces los expedientes pasan a estado "propuesta definitiva"
    Y se notifica a cada persona interesada

  Escenario: No se eleva con el plazo de alegaciones abierto
    Dado una convocatoria con el plazo de alegaciones todavía abierto
    Cuando se intenta elevar la propuesta
    Entonces la operación se rechaza con el error de dominio "alegaciones pendientes"

  Escenario: No se eleva con alegaciones sin resolver
    Dado una alegación presentada y sin resolver
    Cuando se intenta elevar la propuesta
    Entonces la operación se rechaza con el error de dominio "alegaciones pendientes"
```

**Criterios de accesibilidad**

- La diferencia entre propuesta provisional y definitiva se explica con texto en la
  pantalla de la persona interesada; los dos estados comparten token de color y por eso
  el texto es obligatorio.

**Estimación** 3 · **Depende de** E6-H03 · **Flujo crítico** 6 · **Semana** 10

---

## E6-H05 · Resolver el expediente

> Como **jefe de servicio** quiero **resolver conceder o denegar de forma motivada y sin
> haber instruido yo el expediente** para **que la decisión sea válida y auditable**.

```gherkin
# language: es
Característica: Resolución

  Escenario: Se resuelve de forma favorable dentro del crédito
    Dado un expediente con propuesta definitiva favorable y crédito disponible
    Cuando el jefe de servicio lo resuelve de forma favorable
    Entonces el expediente pasa a estado "resuelto favorable"
    Y consta el importe concedido y la motivación
    Y se notifica la resolución a la persona interesada

  Escenario: Quien instruyó no puede resolver
    Dado un expediente cuya propuesta firmó el propio jefe de servicio
    Cuando intenta resolverlo
    Entonces la operación se rechaza con el error de dominio
      "segregación de funciones vulnerada"

  Escenario: No se concede sin crédito suficiente
    Dado una convocatoria con el crédito agotado
    Cuando se intenta resolver de forma favorable
    Entonces la operación se rechaza con el error de dominio "crédito insuficiente"

  Escenario: Se resuelve de forma desfavorable con motivo tipificado
    Dado un expediente con puntuación por debajo del mínimo
    Cuando el jefe de servicio lo resuelve de forma desfavorable
    Entonces el expediente pasa a estado "resuelto desfavorable"
    Y consta el motivo y la motivación íntegra

  Escenario: Vencido el plazo máximo, la persona puede entender desestimada su solicitud
    Dado una convocatoria publicada hace más de seis meses sin resolución notificada
    Cuando la persona interesada consulta su expediente
    Entonces se le informa de que puede entender desestimada su solicitud por silencio
    Y el expediente conserva su estado real
    Y consta que la Administración sigue obligada a resolver
```

**Criterios de accesibilidad**

- La resolución se muestra íntegra en página web accesible, además de en el documento
  descargable.
- El sentido se comunica con texto, icono y color: «Concedida» / «Denegada». El importe
  y la fecha van en fuente de dato.
- La información sobre el silencio administrativo se explica en lenguaje claro, con el
  precepto citado y enlazado, sin sustituir al texto legal.

**Estimación** 5 · **Depende de** E6-H04, E0-H03 · **Flujo crítico** 6 · **Semana** 11

---

## E6-H06 · Publicar el listado definitivo

> Como **vecina de Valdeprieto** quiero **consultar el listado definitivo de personas
> beneficiarias** para **conocer el resultado de la convocatoria aunque no haya
> participado**.

```gherkin
# language: es
Característica: Listado definitivo

  Escenario: Se publica el listado con las personas beneficiarias
    Dado una convocatoria con todos sus expedientes resueltos
    Cuando el jefe de servicio publica el listado definitivo
    Entonces cualquier persona puede consultarlo sin identificarse
    Y el listado indica los importes concedidos y el crédito total comprometido

  Escenario: El listado no publica más datos de los necesarios
    Cuando se consulta el listado definitivo
    Entonces no se publican datos personales más allá de los exigidos por las bases

  Escenario: No se publica con expedientes sin resolver
    Dado una convocatoria con un expediente sin resolver
    Cuando se intenta publicar el listado definitivo
    Entonces la operación se rechaza indicando cuántos quedan sin resolver
```

**Criterios de accesibilidad**

- Tabla accesible con `<caption>`, `<th scope>`, orden accesible y paginación con total.
  El listado **nunca** se publica solo como PDF o como imagen.
- Los importes se alinean en columna con cifras tabulares y llevan la unidad en el
  encabezado, no repetida en cada celda.
- El total comprometido se presenta en un pie de tabla asociado semánticamente.
- Funciona a 320 px reorganizándose en fichas, y con zoom al 200 %.

**Estimación** 5 · **Depende de** E6-H05 · **Flujo crítico** 6 · **Semana** 11
