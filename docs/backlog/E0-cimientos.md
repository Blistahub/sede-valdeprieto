# E0 · Cimientos de dominio y sistema de diseño

**41 puntos.** Semanas 1–2, salvo E0-H08 que va en la 6.

Estas ocho historias no enseñan pantallas nuevas y son las únicas que no se pueden
recortar. Todas están redactadas desde el valor real que producen, no como tareas
técnicas: un motor de plazos correcto es lo que impide que a alguien se le pase un plazo
de verdad.

---

## E0-H01 · Cómputo de plazos en días hábiles

> Como **persona interesada** quiero que **el plazo que se me concede se cuente en días
> hábiles con los festivos que me aplican** para **no perder un derecho por un error de
> calendario del propio portal**.

```gherkin
# language: es
Característica: Cómputo de plazos en días hábiles

  Antecedentes:
    Dado el calendario laboral de Valdeprieto para el año en curso

  Escenario: El cómputo empieza el día hábil siguiente a la notificación
    Dado un requerimiento notificado un martes hábil
    Cuando se calcula un plazo de 10 días hábiles
    Entonces el primer día del cómputo es el miércoles siguiente
    Y el día de la notificación no se cuenta

  Escenario: El vencimiento en día inhábil se traslada
    Dado un plazo cuyo último día cae en sábado
    Cuando se calcula la fecha de vencimiento
    Entonces el plazo vence el primer día hábil siguiente

  Esquema del escenario: Casos límite del cómputo
    Dado un requerimiento notificado el "<notificación>"
    Cuando se calcula un plazo de <días> días hábiles
    Entonces el plazo vence el "<vencimiento>"

    Ejemplos:
      | notificación | días | vencimiento | motivo                          |
      | 20-12        | 10   | 09-01       | cruza Navidad y el fin de año   |
      | 22-03        | 10   | 05-04       | cruza el cambio de hora de marzo|
      | 21-10        | 10   | 06-11       | cruza el cambio de hora octubre |
      | 27-02        | 3    | 04-03       | año bisiesto                    |

  Escenario: Un festivo local del municipio de residencia hace inhábil el día
    Dado que la persona interesada reside en un municipio con festivo local propio
    Y ese festivo no lo es en la sede del órgano
    Cuando se calcula su plazo
    Entonces ese día no se cuenta como hábil

  Escenario: Una notificación en el último segundo del día no adelanta el cómputo
    Dado un requerimiento notificado a las 23:59:59
    Cuando se calcula el plazo
    Entonces el cómputo empieza el mismo día que si se hubiera notificado a las 00:00:01
```

**Criterios de accesibilidad**

- Ninguno: es código puro sin interfaz. Su consumo accesible se especifica en E1-H03 y
  E4-H01.

**Estimación** 5 · **Depende de** E0-H02 · **Flujo crítico** 4 · **Semana** 1

**Notas.** Los 16 casos límite de `docs/modelo-dominio.md` §6.7 son obligatorios, con
valores esperados calculados a mano y no generados por el propio código. Cobertura 100 %.
Prohibida la aritmética de milisegundos.

---

## E0-H02 · Calendario de festivos versionado

> Como **auditor** quiero que **el calendario de festivos sea un dato versionado con su
> fuente** para **poder reconstruir por qué un plazo venció el día que venció**.

```gherkin
# language: es
Característica: Calendario laboral como dato versionado

  Escenario: El calendario declara ámbito, año y fuente
    Dado el calendario del año en curso
    Entonces contiene los festivos estatales, los autonómicos y los locales por separado
    Y cada conjunto indica su fuente y su versión

  Escenario: Un festivo que cae en domingo no se cuenta dos veces
    Dado un festivo declarado que coincide en domingo
    Cuando se calcula un plazo que lo atraviesa
    Entonces ese día se descuenta una sola vez

  Escenario: Un plazo que cruza fin de año usa los dos calendarios
    Dado un plazo que empieza en diciembre y vence en enero
    Cuando se calcula el vencimiento
    Entonces se aplican los festivos del año de cada día del cómputo
```

**Criterios de accesibilidad**

- Sin interfaz.

**Estimación** 3 · **Depende de** — · **Semana** 2

---

## E0-H03 · Máquina de estados del expediente

> Como **jefe de servicio** quiero que **ningún expediente pueda cambiar de estado por
> una vía distinta de la capa de dominio** para **que el historial del expediente tenga
> el mismo valor que un libro de registro**.

```gherkin
# language: es
Característica: Transiciones del expediente

  Escenario: Una transición permitida se ejecuta y queda registrada
    Dado un expediente en estado "registrado"
    Cuando el tramitador se lo asigna
    Entonces el expediente pasa a estado "en revisión"
    Y la transición queda registrada con actor, sello de tiempo y motivo

  Escenario: Una transición no contemplada se rechaza de forma explícita
    Dado un expediente en estado "registrado"
    Cuando se intenta resolverlo de forma favorable
    Entonces la operación se rechaza con el error de dominio "instrucción no iniciada"
    Y el expediente permanece en estado "registrado"
    Y no se registra ninguna transición

  Escenario: El registro es irreversible
    Dado un expediente en estado "registrado"
    Cuando se intenta devolverlo a "borrador"
    Entonces la operación se rechaza con el error de dominio "el registro es irreversible"

  Escenario: Desde un estado terminal no sale ninguna transición
    Dado un expediente en estado "resuelto favorable"
    Cuando se intenta cualquier transición
    Entonces la operación se rechaza con el error de dominio "expediente en estado terminal"

  Escenario: La matriz completa se comprueba entera
    Dado los 10 estados del expediente
    Cuando se recorren los 100 pares ordenados de origen y destino
    Entonces los 15 pares permitidos devuelven un resultado correcto
    Y los 85 pares restantes devuelven el error de dominio que les corresponde
```

**Criterios de accesibilidad**

- Los errores de dominio son tipos de resultado, no excepciones: la interfaz tiene que
  poder distinguir «plazo vencido» de «servicio caído» para redactar un mensaje útil
  (`CLAUDE.md` §5 y `docs/diseno.md` §9).

**Estimación** 8 · **Depende de** E0-H01 · **Flujo crítico** 2, 4, 5, 6 · **Semana** 2

---

## E0-H04 · Traza inmutable de transiciones

> Como **auditor** quiero que **la traza del expediente no se pueda modificar ni borrar**
> para **poder certificar qué ocurrió, cuándo y por orden de quién**.

```gherkin
# language: es
Característica: Traza del expediente

  Escenario: Cada transición añade un asiento completo
    Cuando un expediente cambia de estado
    Entonces se añade un asiento con estado origen, estado destino, actor, rol,
      sello de tiempo de servidor y motivo

  Escenario: La traza no admite modificación
    Dado un expediente con transiciones registradas
    Cuando se intenta modificar o borrar un asiento existente
    Entonces la operación se rechaza con el error de dominio "traza inmutable"

  Escenario: Las transiciones automáticas se identifican
    Dado un plazo de subsanación vencido
    Cuando el sistema materializa el desistimiento
    Entonces el asiento indica como actor "sistema" y como motivo el vencimiento del plazo

  Escenario: Una transición producida con el reloj desplazado queda marcada
    Dado el modo demostración activo
    Cuando se materializa una transición por vencimiento
    Entonces el asiento queda marcado como simulado
```

**Criterios de accesibilidad**

- Sin interfaz propia. Su presentación se especifica en E3-H02.

**Estimación** 3 · **Depende de** E0-H03 · **Semana** 2

---

## E0-H05 · Tokens de diseño y primitivas accesibles

> Como **persona con discapacidad** quiero que **todos los controles del portal se
> comporten igual y sean operables con mi tecnología de apoyo** para **no tener que
> aprender cada pantalla por separado**.

```gherkin
# language: es
Característica: Primitivas del sistema de diseño

  Escenario: Todo control es alcanzable y operable con teclado
    Dado cualquier primitiva del sistema de diseño
    Cuando se recorre la pantalla solo con teclado
    Entonces la primitiva recibe el foco en orden lógico
    Y muestra un indicador de foco visible
    Y se activa con las teclas propias de su rol

  Escenario: Un campo de formulario siempre tiene etiqueta
    Dado un campo de formulario
    Entonces tiene una etiqueta asociada
    Y el texto de ayuda no sustituye a la etiqueta

  Escenario: El estado del expediente nunca se comunica solo con color
    Dado la insignia de estado de un expediente
    Entonces comunica el estado con color, con texto y con forma

  Escenario: Los datos legibles en voz alta usan la fuente de dato
    Dado un número de expediente, un NIF, un importe o una fecha
    Entonces se muestra en la fuente monoespaciada con cifras tabulares
```

**Criterios de accesibilidad**

- Anillo de foco de dos capas de `docs/diseno.md` §5. Contraste del indicador ≥ 3:1
  sobre `--papel` **y** sobre `--verde-700`. Prohibido `outline: none`.
- Área táctil mínima 44 × 44 px en todos los controles.
- Ningún token de la matriz de contraste se usa fuera de su fila permitida. En concreto:
  `--sello` nunca como texto ni borde único (2,81:1), `--linea` nunca como borde
  funcional (1,39:1).
- Cada primitiva tiene story de Storybook en estado normal, vacío, cargando y error, y
  todas pasan axe sin violaciones `serious` ni `critical`.
- `prefers-reduced-motion: reduce` elimina la animación, no la acorta.

**Estimación** 8 · **Depende de** — · **Semana** 1

**Notas.** Alcance: botón, campo de texto, campo de fichero, grupo de radio y casilla,
insignia de estado, filete, tabla, migas de pan, mensaje de error y raíl de progreso.
Nada más. El justificante y el cuño son E2-H06.

---

## E0-H06 · Semilla y limpieza de datos de prueba

> Como **responsable de calidad** quiero que **cada prueba cree sus propios datos por API
> y los limpie al terminar** para **que la suite se pueda ejecutar en paralelo y el orden
> no importe**.

```gherkin
# language: es
Característica: Datos de prueba aislados

  Escenario: Una prueba crea su escenario sin navegar por la interfaz
    Cuando una prueba solicita un expediente en estado "subsanación requerida"
      con 7 días hábiles restantes
    Entonces el escenario queda disponible sin haber pasado por ninguna pantalla

  Escenario: Todo dato creado queda marcado y es rastreable
    Cuando una prueba crea datos
    Entonces cada registro queda marcado como sintético
    Y lleva el identificador de la ejecución que lo creó

  Escenario: Los identificadores generados están en los rangos reservados
    Cuando se genera una persona interesada sintética
    Entonces su NIF pertenece al rango reservado del proyecto y su letra de control es correcta
    Y su correo usa el dominio de ejemplo del proyecto
```

**Criterios de accesibilidad**

- Sin interfaz. La semilla nunca vuelca datos personales sin enmascarar en logs ni en
  artefactos de CI, aunque sean sintéticos.

**Estimación** 3 · **Depende de** E0-H03 · **Semana** 2

---

## E0-H07 · Reloj inyectable y modo demostración

> Como **persona que enseña el producto** quiero **poder adelantar el reloj del sistema
> de forma controlada y visible** para **demostrar en dos minutos un procedimiento que
> en la realidad dura diez días hábiles**.

```gherkin
# language: es
Característica: Control del tiempo

  Escenario: El dominio nunca lee el reloj del sistema directamente
    Dado cualquier regla de dominio que dependa de la fecha
    Entonces recibe el instante como dependencia
    Y su resultado es reproducible con el mismo instante de entrada

  Escenario: El modo demostración adelanta el reloj y lo declara
    Dado el modo demostración activo
    Cuando se adelanta el reloj más allá del vencimiento de un plazo
    Entonces las transiciones que se produzcan quedan marcadas como simuladas
    Y la interfaz avisa de que el reloj está desplazado

  Escenario: El modo demostración no existe fuera de los entornos de demostración
    Dado un entorno que no es de demostración
    Cuando se intenta desplazar el reloj
    Entonces la operación se rechaza
```

**Criterios de accesibilidad**

- El aviso de reloj desplazado es texto permanente en la interfaz, no un color ni un
  icono, y se anuncia a lectores de pantalla al activarse.

**Estimación** 3 · **Depende de** — · **Semana** 1

---

## E0-H08 · Andamiaje del track Selenium / Cucumber

> Como **candidato en una entrevista técnica** quiero **el mismo flujo automatizado en
> Playwright y en Selenium con Cucumber** para **poder comparar los dos enfoques con
> datos medidos y no con opiniones**.

```gherkin
# language: es
Característica: Track B de automatización

  Escenario: El flujo 2 pasa en los dos tracks
    Dado el flujo de presentación de solicitud
    Cuando se ejecuta en Playwright y en Selenium
    Entonces ambos tracks verifican los mismos criterios de negocio

  Escenario: Las métricas de comparación se recogen de la propia CI
    Cuando termina una ejecución completa de la integración continua
    Entonces se registran tiempo de suite, tiempo del flujo aislado y tasa de fallo
      intermitente de cada track
```

**Criterios de accesibilidad**

- Los localizadores se eligen por rol, etiqueta y texto, en ese orden. Cada
  `data-testid` que haga falta abre un issue con etiqueta `a11y` antes de escribirse: si
  el test no encuentra el elemento por su rol, un lector de pantalla probablemente
  tampoco.

**Estimación** 8 · **Depende de** E2-H06 · **Flujo crítico** 2 · **Semana** 6
