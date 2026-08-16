# E1 · Convocatorias y bases reguladoras

**8 puntos.** Semana 3. Cubre el **flujo crítico 1**: consultar convocatoria y descargar
bases.

---

## E1-H01 · Consultar las convocatorias abiertas

> Como **vecina de Valdeprieto** quiero **ver qué ayudas puedo solicitar ahora mismo**
> para **saber si me interesa antes de meterme en un formulario**.

```gherkin
# language: es
Característica: Listado de convocatorias

  Escenario: Se muestran las convocatorias con el plazo abierto
    Dado una convocatoria publicada con el plazo de presentación abierto
    Cuando una persona consulta las convocatorias
    Entonces esa convocatoria aparece en el listado
    Y se indica su importe máximo y la fecha de fin de presentación

  Escenario: Las convocatorias cerradas se muestran identificadas como tales
    Dado una convocatoria cuyo plazo de presentación ha terminado
    Cuando una persona consulta las convocatorias
    Entonces la convocatoria aparece marcada como cerrada
    Y no se ofrece iniciar una solicitud desde el listado

  Escenario: Sin convocatorias abiertas se orienta sobre qué hacer
    Dado que no hay ninguna convocatoria con el plazo abierto
    Cuando una persona consulta las convocatorias
    Entonces se le informa de que no hay ninguna abierta
    Y se le ofrece consultar las convocatorias ya cerradas
```

**Criterios de accesibilidad**

- El listado es una lista semántica, no una rejilla de `div`. Cada elemento tiene un
  encabezado de nivel coherente con la jerarquía de la página.
- El estado de la convocatoria se comunica con texto («Plazo abierto», «Cerrada»),
  nunca solo con el color de la insignia.
- Las fechas y los importes van en la fuente de dato con cifras tabulares.
- Paginación con números y total. **Prohibido el scroll infinito**
  (`docs/diseno.md` §10).
- Funciona a 320 px y con zoom al 200 % sin scroll horizontal.

**Estimación** 3 · **Depende de** E0-H05 · **Flujo crítico** 1 · **Semana** 3

---

## E1-H02 · Leer las bases y descargarlas

> Como **persona interesada** quiero **leer las bases reguladoras íntegras y poder
> descargarlas** para **comprobar si cumplo los requisitos antes de solicitar**.

```gherkin
# language: es
Característica: Bases reguladoras

  Escenario: Las bases se muestran íntegras
    Dado una convocatoria publicada
    Cuando una persona consulta su detalle
    Entonces puede leer el texto íntegro de las bases reguladoras
    Y conoce los requisitos, los criterios de valoración y la documentación exigida

  Escenario: Las bases se pueden descargar
    Cuando una persona descarga las bases de una convocatoria
    Entonces obtiene el documento oficial
    Y la huella del documento descargado coincide con la publicada

  Escenario: El detalle indica qué documentación habrá que aportar
    Dado una convocatoria que exige certificado de empadronamiento
    Cuando una persona consulta su detalle
    Entonces conoce esa exigencia antes de iniciar la solicitud
```

**Criterios de accesibilidad**

- Texto legal con la escala de cuerpo largo de `docs/diseno.md` §3 y medida de línea de
  60–72 caracteres. **Nunca a todo el ancho.**
- Jerarquía de encabezados correcta y sin saltos de nivel; el texto legal se puede
  recorrer con la navegación por encabezados del lector de pantalla.
- El enlace de descarga declara formato y tamaño en su texto accesible, no solo un
  icono.
- El elemento del documento declara su idioma con `lang`.

**Estimación** 3 · **Depende de** E1-H01 · **Flujo crítico** 1 · **Semana** 3

---

## E1-H03 · Saber cuánto plazo queda para presentar

> Como **persona interesada con prisa** quiero **saber cuántos días hábiles me quedan
> para presentar** para **no confiarme contando días naturales de cabeza**.

```gherkin
# language: es
Característica: Plazo de presentación

  Escenario: Se indican los días hábiles restantes
    Dado una convocatoria cuyo plazo de presentación termina en 7 días hábiles
    Cuando una persona consulta su detalle
    Entonces se le indica que quedan 7 días hábiles
    Y se le indica la fecha y la hora exactas de fin de plazo

  Escenario: El último día se advierte de forma inequívoca
    Dado una convocatoria cuyo plazo de presentación termina hoy
    Cuando una persona consulta su detalle
    Entonces se le advierte de que hoy es el último día
    Y se le indica la hora límite

  Escenario: Terminado el plazo se explica la consecuencia
    Dado una convocatoria con el plazo terminado
    Cuando una persona consulta su detalle
    Entonces se le informa de que puede registrar una solicitud
    Y se le advierte de que se inadmitirá por presentación fuera de plazo
```

**Criterios de accesibilidad**

- La cuenta de días tiene **equivalente textual completo**: «Quedan 7 días hábiles. El
  plazo termina el viernes 24 de abril a las 23:59». Prohibida la cuenta atrás que solo
  se ve (`docs/diseno.md` §10).
- El aviso de último día usa `--lacre` **más** icono **más** texto, y su contraste se
  comprueba contra la matriz.
- La cifra de días va en fuente de dato con cifras tabulares.
- Ningún elemento se actualiza solo de forma que interrumpa a un lector de pantalla: la
  cuenta se calcula en servidor y no parpadea en cliente.

**Estimación** 2 · **Depende de** E0-H01, E1-H02 · **Flujo crítico** 1 · **Semana** 3
