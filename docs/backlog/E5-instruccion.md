# E5 · Instrucción y bandeja del tramitador

**23 puntos.** Semanas 6–8. Es el **flujo crítico 5**: el tramitador revisa un expediente
y solicita subsanación.

El área de gestión tiene exactamente el mismo listón de accesibilidad que el área
ciudadana. Un tramitador puede ser una persona con discapacidad, y además pasa aquí ocho
horas al día: la accesibilidad de esta épica también es ergonomía.

---

## E5-H01 · Bandeja de expedientes

> Como **tramitador** quiero **ver los expedientes que me tocan con su estado y su
> antigüedad** para **decidir por dónde empiezo sin abrirlos uno a uno**.

```gherkin
# language: es
Característica: Bandeja del tramitador

  Escenario: Se listan los expedientes de las convocatorias del órgano
    Dado un tramitador del órgano instructor
    Cuando consulta su bandeja
    Entonces ve los expedientes de sus convocatorias con número, estado, fecha de
      registro y persona asignada

  Escenario: Se puede filtrar por estado y por convocatoria
    Dado una bandeja con expedientes en varios estados
    Cuando el tramitador filtra por estado "registrado"
    Entonces solo ve los expedientes en ese estado
    Y conoce cuántos hay en total

  Escenario: Los expedientes con plazo próximo a vencer se distinguen
    Dado un expediente con menos de 3 días hábiles de plazo restante
    Cuando el tramitador consulta su bandeja
    Entonces ese expediente aparece identificado como plazo en riesgo

  Escenario: La bandeja no muestra expedientes de otros órganos
    Dado un expediente de una convocatoria de otro órgano instructor
    Cuando el tramitador consulta su bandeja
    Entonces ese expediente no aparece
```

**Criterios de accesibilidad**

- Tabla con `<caption>` que describe qué contiene y bajo qué filtro, `<th scope="col">`
  y orden accesible anunciado con `aria-sort`.
- Al aplicar un filtro se anuncia el número de resultados en región `polite`, y el foco
  no salta al principio de la página.
- «Plazo en riesgo» se comunica con `--lacre`, icono **y** el número de días en texto.
- Paginación con números y total; prohibido el scroll infinito.
- La tabla completa es navegable con teclado incluidas las acciones de fila, sin
  trampas de foco. A 320 px se reorganiza sin scroll horizontal.
- Los datos personales de la bandeja se muestran minimizados: solo lo necesario para
  identificar el expediente.

**Estimación** 5 · **Depende de** E2-H06, E0-H05 · **Flujo crítico** 5 · **Semana** 6

---

## E5-H02 · Asignarse un expediente

> Como **tramitador** quiero **asignarme un expediente registrado** para **que conste
> quién lo está instruyendo**.

```gherkin
# language: es
Característica: Asignación de expedientes

  Escenario: Un expediente registrado pasa a instrucción al asignarse
    Dado un expediente en estado "registrado" sin instructor
    Cuando un tramitador se lo asigna
    Entonces el expediente pasa a estado "en revisión"
    Y consta el tramitador asignado en la traza

  Escenario: No se asigna un expediente ya asignado
    Dado un expediente ya asignado a otro tramitador
    Cuando un tramitador intenta asignárselo
    Entonces la operación se rechaza y se indica quién lo tiene asignado
```

**Criterios de accesibilidad**

- Tras asignarse, el cambio de estado se anuncia en región `polite` y el foco permanece
  en un punto útil, no vuelve al principio de la bandeja.

**Estimación** 2 · **Depende de** E5-H01, E0-H03 · **Flujo crítico** 5 · **Semana** 6

---

## E5-H03 · Revisar la solicitud y su documentación

> Como **tramitador** quiero **revisar lo presentado con su huella y su fecha** para
> **poder comprobar que nada se ha alterado desde el registro**.

```gherkin
# language: es
Característica: Revisión del expediente

  Escenario: Se consulta lo presentado tal como se registró
    Dado un expediente en revisión
    Cuando el tramitador lo consulta
    Entonces ve los datos de la solicitud y todos los documentos con su fecha y huella

  Escenario: La integridad de lo presentado es comprobable
    Cuando el tramitador comprueba la integridad de un documento
    Entonces la huella calculada coincide con la registrada en el momento del registro

  Escenario: El tramitador no puede alterar lo presentado
    Cuando se intenta modificar un dato de la solicitud registrada
    Entonces la operación se rechaza con el error de dominio "solicitud inmutable"
```

**Criterios de accesibilidad**

- Las huellas y fechas van en fuente de dato con cifras tabulares, y las huellas son
  seleccionables y copiables como texto.
- La comparación de integridad se comunica con texto y forma además de color:
  «Íntegro» / «No coincide».

**Estimación** 3 · **Depende de** E5-H02 · **Flujo crítico** 5 · **Semana** 7

---

## E5-H04 · Requerir subsanación

> Como **tramitador** quiero **requerir la subsanación indicando exactamente qué falta**
> para **que la persona interesada sepa qué hacer sin tener que interpretar nada**.

```gherkin
# language: es
Característica: Requerimiento de subsanación

  Escenario: El tramitador requiere subsanación con defectos concretos
    Dado un expediente en revisión al que le falta el certificado de empadronamiento
    Cuando el tramitador requiere la subsanación indicando ese defecto
    Entonces el expediente pasa a estado "subsanación requerida"
    Y se pone a disposición de la persona interesada la notificación del requerimiento
    Y se abre un plazo de 10 días hábiles

  Escenario: No se requiere sin indicar al menos un defecto
    Cuando se intenta requerir subsanación sin indicar ningún defecto
    Entonces la operación se rechaza con el error de dominio "motivo requerido"

  Escenario: No cabe un segundo requerimiento
    Dado un expediente que ya tuvo un requerimiento de subsanación
    Cuando el tramitador intenta requerir de nuevo
    Entonces la operación se rechaza con el error de dominio "subsanación ya requerida"

  Escenario: Solo requiere quien tiene el expediente asignado
    Dado un expediente asignado a otro tramitador
    Cuando un tramitador distinto intenta requerir subsanación
    Entonces la operación se rechaza con el error de dominio "expediente no asignado"
```

**Criterios de accesibilidad**

- Los defectos se eligen de una lista cerrada mediante casillas con etiquetas completas
  y visibles, agrupadas en un `fieldset` con `legend`.
- Antes de emitir se muestra una previsualización del texto que va a recibir la persona
  interesada, y esa previsualización es texto real, no una imagen.
- El resultado se anuncia en región `polite` con la fecha de vencimiento del plazo.

**Estimación** 5 · **Depende de** E5-H03, E3-H03 · **Flujo crítico** 5 · **Semana** 7

---

## E5-H05 · Valorar el expediente según los criterios

> Como **tramitador** quiero **puntuar cada expediente con los criterios de las bases y
> motivar cada puntuación** para **que la propuesta sea defendible ante una reclamación**.

```gherkin
# language: es
Característica: Valoración

  Escenario: Se puntúa cada criterio dentro de su máximo
    Dado un expediente en revisión y una convocatoria con tres criterios
    Cuando el tramitador puntúa cada criterio dentro de su máximo y lo motiva
    Entonces la valoración queda registrada con la puntuación total

  Escenario: No se admite una puntuación fuera de rango
    Cuando el tramitador puntúa un criterio por encima de su máximo
    Entonces la operación se rechaza indicando el máximo admitido

  Escenario: No se admite una puntuación sin motivación
    Cuando el tramitador puntúa un criterio sin motivarlo
    Entonces la operación se rechaza con el error de dominio "motivo requerido"

  Escenario: El desempate está reglado
    Dado dos expedientes con la misma puntuación total
    Cuando se ordenan
    Entonces se ordena primero por la puntuación del criterio de menor orden
    Y si persiste el empate, por la fecha y hora de registro
```

**Criterios de accesibilidad**

- Cada criterio es un campo numérico con etiqueta que incluye su máximo, y el máximo
  también se expone en el texto de ayuda enlazado con `aria-describedby`.
- El total se recalcula en servidor y se anuncia al cambiar, sin interrumpir la
  escritura.
- Las puntuaciones se muestran en fuente de dato con cifras tabulares para poder
  compararlas en columna.

**Estimación** 5 · **Depende de** E5-H03 · **Semana** 8

---

## E5-H06 · Inadmitir un expediente

> Como **jefe de servicio** quiero **inadmitir un expediente con motivo tipificado** para
> **cerrar los casos que no pueden tramitarse dejando constancia motivada**.

```gherkin
# language: es
Característica: Inadmisión durante la instrucción

  Escenario: Se inadmite por falta de legitimación
    Dado un expediente en revisión cuya persona solicitante no reúne los requisitos
      subjetivos de las bases
    Cuando el jefe de servicio lo inadmite indicando el motivo
    Entonces el expediente pasa a estado "inadmitido"
    Y la persona interesada recibe la resolución motivada

  Escenario: El tramitador no puede inadmitir
    Dado un expediente en revisión
    Cuando un tramitador intenta inadmitirlo
    Entonces la operación se rechaza con el error de dominio "actor no autorizado"

  Escenario: No se inadmite una vez emitida la propuesta provisional
    Dado un expediente con propuesta de resolución provisional
    Cuando se intenta inadmitirlo
    Entonces la operación se rechaza
    Y se indica que procede resolver de forma desfavorable
```

**Criterios de accesibilidad**

- Confirmación en modal con foco atrapado y devuelto, y con la consecuencia escrita.
- El motivo tipificado se elige de una lista con etiquetas completas; el campo de
  motivación adicional es obligatorio y su obligatoriedad se indica con texto.

**Estimación** 3 · **Depende de** E5-H03, E0-H03 · **Semana** 8
