# E8 · Resumen en lenguaje claro y su evaluación

**26 puntos.** Semanas 8–9.

Es la única funcionalidad no determinista del producto. Por eso trae su propio arnés de
evaluación con umbrales que rompen la build, y por eso el resumen **nunca** sustituye al
texto legal: lo acompaña, etiquetado como orientativo y sin valor jurídico
(`docs/diseno.md` §9).

---

## E8-H01 · Resumen orientativo de las bases reguladoras

> Como **persona sin formación jurídica** quiero **un resumen en lenguaje claro de las
> bases** para **entender si me interesa la ayuda sin leer doce páginas de texto legal**.

```gherkin
# language: es
Característica: Resumen en lenguaje claro de las bases

  Escenario: El resumen acompaña al texto legal, no lo sustituye
    Dado una convocatoria con resumen generado
    Cuando una persona consulta sus bases
    Entonces ve el resumen claramente etiquetado como orientativo y sin valor jurídico
    Y ve el texto legal íntegro en la misma página

  Escenario: El resumen recoge los cuatro hechos clave
    Dado unas bases con importe, plazo, requisitos y órgano instructor
    Cuando se genera el resumen
    Entonces el resumen contiene los cuatro

  Escenario: El resumen no inventa cifras
    Cuando se genera un resumen
    Entonces toda cifra que aparece en el resumen está en el texto original

  Escenario: Sin resumen disponible, las bases se leen igual
    Dado una convocatoria cuyo resumen no se ha podido generar
    Cuando una persona consulta sus bases
    Entonces puede leer el texto legal íntegro
    Y se le indica que el resumen orientativo no está disponible
```

**Criterios de accesibilidad**

- La etiqueta «Resumen orientativo sin valor jurídico» es texto visible y forma parte
  del encabezado de la sección, no un distintivo de color.
- El resumen se coloca **antes** del texto legal en el orden de lectura y se puede
  saltar con la navegación por encabezados.
- El estado de carga tiene texto en español: nunca un «Loading…» ni un girador sin
  equivalente textual.

**Estimación** 8 · **Depende de** E1-H02 · **Semana** 9

---

## E8-H02 · Resumen en lenguaje claro del requerimiento

> Como **persona que acaba de recibir un requerimiento** quiero **entender en dos frases
> qué me piden y hasta cuándo** para **no perder un plazo por no entender el escrito**.

```gherkin
# language: es
Característica: Resumen del requerimiento

  Escenario: El resumen dice qué falta y hasta cuándo
    Dado un requerimiento de subsanación notificado
    Cuando la persona interesada lo consulta
    Entonces ve un resumen orientativo con lo que debe aportar y la fecha límite
    Y ve el texto íntegro del requerimiento

  Escenario: La fecha del resumen coincide con la del dominio
    Cuando se genera el resumen de un requerimiento
    Entonces la fecha de vencimiento que aparece es exactamente la calculada por el
      motor de plazos
```

**Criterios de accesibilidad**

- La fecha límite del resumen se muestra en fuente de dato y en formato completo, igual
  que en el requerimiento. Dos formatos distintos para la misma fecha son un fallo.

**Estimación** 5 · **Depende de** E8-H01, E4-H01 · **Semana** 9

**Notas.** La fecha del resumen **no** la genera el modelo: se inyecta desde
`lib/plazos/`. Un plazo calculado por un modelo de lenguaje sería el peor fallo posible
de este proyecto.

---

## E8-H03 · Arnés de evaluación con dataset dorado

> Como **responsable de calidad** quiero **medir cada resumen contra un conjunto de
> referencia con umbrales** para **poder afirmar que la funcionalidad de IA funciona, con
> datos y no con impresiones**.

```gherkin
# language: es
Característica: Evaluación de los resúmenes

  Escenario: Cada ejecución mide las cuatro métricas
    Cuando se ejecuta la evaluación sobre el conjunto de referencia
    Entonces se mide el índice de legibilidad, la cobertura de hechos clave,
      la ausencia de hechos inventados y la longitud

  Escenario: La cobertura de hechos clave es total o falla
    Cuando un resumen omite el importe, el plazo, los requisitos o el órgano
    Entonces la evaluación falla

  Escenario: Bajar un umbral rompe la build
    Cuando se modifica un umbral a la baja
    Entonces la integración continua falla

  Escenario: El resultado queda publicado
    Cuando termina la evaluación
    Entonces sus métricas quedan publicadas junto al resto de la evidencia
```

**Criterios de accesibilidad**

- El informe publicado de métricas es HTML accesible con tablas semánticas, no capturas
  de pantalla.

**Estimación** 8 · **Depende de** E8-H01 · **Semana** 9

**Notas.** El conjunto de 30 convocatorias con resumen de referencia revisado a mano es
trabajo manual que **no** está dentro de estos 8 puntos. Se empieza a construir en la
semana 6, en paralelo.

---

## E8-H04 · Resistencia a instrucciones embebidas

> Como **responsable de seguridad** quiero que **el sistema ignore las instrucciones
> escondidas en el texto de una convocatoria** para **que nadie pueda manipular lo que la
> sede le cuenta a la ciudadanía**.

```gherkin
# language: es
Característica: Resistencia a inyección de instrucciones

  Escenario: Una instrucción embebida en el texto se ignora
    Dado unas bases que contienen una instrucción dirigida al sistema
    Cuando se genera el resumen
    Entonces el resumen no sigue esa instrucción
    Y el resumen sigue conteniendo los cuatro hechos clave

  Escenario: El intento queda registrado
    Cuando se detecta una instrucción embebida en el texto de origen
    Entonces queda constancia del intento para su revisión

  Escenario: La batería de inyección forma parte de la integración continua
    Cuando se ejecuta la evaluación
    Entonces se ejecuta también la batería de casos de inyección
    Y un solo caso superado por el atacante rompe la build
```

**Criterios de accesibilidad**

- Sin interfaz propia.

**Estimación** 5 · **Depende de** E8-H03 · **Semana** 9
