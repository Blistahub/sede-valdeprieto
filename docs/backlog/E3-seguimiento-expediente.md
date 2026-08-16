# E3 · Seguimiento del expediente

**16 puntos.** Semanas 5–6.

Es la épica que resuelve la pregunta que de verdad se hace la gente: *«¿en qué punto
está lo mío y cuánto tiempo tengo?»*.

---

## E3-H01 · Consultar mis solicitudes

> Como **persona interesada** quiero **ver de un vistazo todas mis solicitudes y su
> estado** para **saber cuál requiere algo de mí ahora mismo**.

```gherkin
# language: es
Característica: Mis solicitudes

  Escenario: Se listan los borradores y los expedientes
    Dado una persona interesada con un borrador y dos expedientes registrados
    Cuando consulta sus solicitudes
    Entonces ve las tres, distinguiendo el borrador de las presentadas
    Y conoce el estado de cada expediente

  Escenario: Lo que requiere acción se distingue de lo que no
    Dado un expediente con una subsanación requerida
    Cuando la persona interesada consulta sus solicitudes
    Entonces ese expediente aparece identificado como pendiente de su actuación
    Y conoce los días hábiles que le quedan

  Escenario: Sin solicitudes se invita a actuar
    Dado una persona interesada sin ninguna solicitud
    Cuando consulta sus solicitudes
    Entonces se le indica que todavía no tiene solicitudes
    Y se le ofrece consultar las convocatorias abiertas

  Escenario: Nadie ve expedientes ajenos
    Dado un expediente de otra persona
    Cuando se intenta consultarlo
    Entonces la operación se rechaza con el error de dominio "expediente ajeno"
```

**Criterios de accesibilidad**

- Tabla con `<caption>`, `<th scope="col">` y orden accesible por columnas: el control
  de orden es un botón dentro del encabezado y anuncia el sentido con `aria-sort`.
- Paginación con números y total. Prohibido el scroll infinito.
- «Pendiente de tu actuación» es **texto**, no un punto de color.
- Números de expediente, fechas y días restantes en fuente de dato con cifras tabulares.
- A 320 px la tabla se reorganiza en fichas sin perder la relación entre dato y
  concepto, y sin scroll horizontal.

**Estimación** 3 · **Depende de** E2-H06 · **Semana** 5

---

## E3-H02 · Consultar el detalle de mi expediente

> Como **persona interesada** quiero **ver el recorrido completo de mi expediente con sus
> fechas** para **entender qué ha pasado y qué falta sin tener que llamar por teléfono**.

```gherkin
# language: es
Característica: Detalle del expediente

  Escenario: Se muestra el recorrido con fechas reales
    Dado un expediente en revisión
    Cuando la persona interesada consulta su detalle
    Entonces ve las etapas del procedimiento en orden
    Y ve la fecha de cada etapa ya cumplida
    Y ve la etapa en la que se encuentra

  Escenario: Se muestra el plazo vivo si lo hay
    Dado un expediente con una subsanación requerida
    Cuando la persona interesada consulta su detalle
    Entonces conoce los días hábiles restantes y la fecha exacta de vencimiento
    Y conoce qué ocurre si no actúa

  Escenario: Se puede consultar la documentación presentada
    Dado un expediente registrado
    Cuando la persona interesada consulta su detalle
    Entonces ve la relación de documentos presentados con su fecha de aportación
    Y puede descargar cada uno

  Escenario: Un expediente resuelto muestra el sentido y la motivación
    Dado un expediente resuelto de forma desfavorable
    Cuando la persona interesada consulta su detalle
    Entonces conoce el sentido de la resolución y su motivación íntegra
```

**Criterios de accesibilidad**

- El raíl de estado es una lista ordenada real. El estado actual se identifica con
  `aria-current` **y** con texto, nunca solo con el punto de color.
- Las etapas futuras se anuncian como pendientes; no se ocultan a la tecnología de
  apoyo.
- A 320 px el raíl colapsa a una barra fija con dos datos —estado actual y días hábiles
  restantes— tal como fija `docs/diseno.md` §6, y esa barra no tapa contenido ni atrapa
  el foco.
- El plazo vivo se anuncia con equivalente textual completo, incluido qué pasa al
  vencer.
- Toda cifra del expediente en fuente de dato con cifras tabulares.

**Estimación** 5 · **Depende de** E3-H01, E0-H04 · **Semana** 6

---

## E3-H03 · Buzón de notificaciones

> Como **persona interesada** quiero **acceder a mis notificaciones sabiendo lo que
> implica abrirlas** para **controlar desde cuándo me corren los plazos**.

```gherkin
# language: es
Característica: Notificaciones

  Escenario: Una notificación puesta a disposición se identifica como no accedida
    Dado un requerimiento puesto a disposición
    Cuando la persona interesada consulta su buzón
    Entonces la notificación aparece como pendiente de acceso
    Y conoce la fecha de puesta a disposición

  Escenario: Acceder a la notificación fija la fecha de efectos
    Dado una notificación pendiente de acceso
    Cuando la persona interesada accede a su contenido
    Entonces la notificación queda como accedida con su fecha y hora
    Y el plazo asociado empieza a contar desde el día hábil siguiente

  Escenario: Se advierte antes de acceder
    Dado una notificación pendiente de acceso
    Cuando la persona interesada se dispone a abrirla
    Entonces se le advierte de que al acceder empezará a contar el plazo

  Escenario: La notificación no accedida se rechaza por transcurso
    Dado una notificación puesta a disposición hace 10 días naturales sin acceder
    Cuando se consulta su estado
    Entonces consta como rechazada por transcurso del plazo
    Y el plazo asociado se cuenta desde esa fecha
```

**Criterios de accesibilidad**

- El aviso previo al acceso es un diálogo modal con foco atrapado correctamente, cierre
  con `Escape` y devolución del foco. Explica la consecuencia con texto, no con un
  icono de alerta.
- El estado de cada notificación se comunica con texto: «Pendiente de acceso»,
  «Accedida el …», «Rechazada por transcurso el …».
- Las fechas de puesta a disposición y de acceso van en fuente de dato con segundos.

**Estimación** 5 · **Depende de** E0-H01, E0-H07 · **Flujo crítico** 4 · **Semana** 6

---

## E3-H04 · Desistir de la solicitud

> Como **persona interesada** quiero **desistir de mi solicitud dejando constancia** para
> **cerrar el asunto de forma ordenada cuando ya no me interesa**.

```gherkin
# language: es
Característica: Desistimiento voluntario

  Escenario: La persona interesada desiste antes de la resolución
    Dado un expediente en revisión
    Cuando la persona interesada desiste indicando el motivo
    Entonces el expediente pasa a estado "desistido"
    Y obtiene justificante del escrito de desistimiento

  Escenario: No se desiste de un expediente ya resuelto
    Dado un expediente resuelto
    Cuando se intenta desistir
    Entonces la operación se rechaza con el error de dominio "expediente en estado terminal"

  Escenario: El desistimiento es irreversible
    Dado un expediente desistido
    Cuando se intenta reanudarlo
    Entonces la operación se rechaza
    Y se explica que para volver a solicitar hay que presentar una solicitud nueva
```

**Criterios de accesibilidad**

- Confirmación en modal con foco atrapado y devuelto, y con la consecuencia escrita en
  el propio diálogo: «Tu solicitud dejará de tramitarse. Esta acción no se puede
  deshacer».
- El botón nombra la acción del dominio: «Desistir de la solicitud». Nunca «Cancelar»,
  que además se confundiría con cerrar el diálogo.
- El estado «Desistido» se comunica con `--gris`, texto tachado y motivo.

**Estimación** 3 · **Depende de** E3-H02, E0-H03 · **Semana** 6
