# E4 · Subsanación y vencimiento de plazos

**18 puntos.** Semanas 6–7. Es el **flujo crítico 4**.

Es la épica donde el producto se juega su credibilidad: un plazo mal contado aquí
significa que alguien pierde una ayuda.

---

## E4-H01 · Conocer el requerimiento y el plazo que abre

> Como **persona interesada** quiero **entender qué me falta y hasta cuándo tengo para
> aportarlo** para **poder subsanar a tiempo sin interpretar lenguaje administrativo**.

```gherkin
# language: es
Característica: Requerimiento de subsanación

  Escenario: El requerimiento explica qué falta y hasta cuándo
    Dado un expediente con subsanación requerida
    Cuando la persona interesada accede al requerimiento
    Entonces conoce cada defecto que debe subsanar
    Y conoce la fecha exacta de vencimiento del plazo
    Y conoce que dispone de 10 días hábiles

  Escenario: El requerimiento advierte de la consecuencia de no actuar
    Dado un requerimiento notificado
    Entonces se advierte de que, si no se subsana en plazo, se le tendrá por desistida

  Escenario: El plazo se cuenta desde el día hábil siguiente a la notificación
    Dado un requerimiento notificado un viernes hábil
    Cuando se calcula su plazo de subsanación
    Entonces el cómputo empieza el lunes hábil siguiente

  Escenario: El plazo de una notificación rechazada por transcurso también corre
    Dado un requerimiento no accedido durante 10 días naturales
    Cuando se calcula su plazo de subsanación
    Entonces el cómputo empieza el día hábil siguiente a la fecha de rechazo
```

**Criterios de accesibilidad**

- El texto legal del requerimiento se muestra íntegro. Si existe resumen en lenguaje
  claro (E8-H02), va junto al original y **etiquetado como orientativo sin valor
  jurídico**, nunca en su lugar.
- Los días hábiles restantes tienen equivalente textual completo con la fecha y hora de
  vencimiento; prohibida la cuenta atrás solo visual.
- El estado «Subsanación requerida» se comunica con `--lacre`, icono de aviso **y** el
  número de días hábiles restantes en texto.
- La lista de defectos es una lista semántica; cada defecto enlaza con la acción que lo
  resuelve.

**Estimación** 3 · **Depende de** E3-H03, E5-H04 · **Flujo crítico** 4 · **Semana** 6

---

## E4-H02 · Aportar la documentación de subsanación

> Como **persona interesada** quiero **aportar lo que me piden y recibir constancia
> inmediata** para **quedarme tranquila de que llegó dentro de plazo**.

```gherkin
# language: es
Característica: Subsanación de la solicitud

  Escenario: La persona subsana dentro de plazo
    Dado un expediente en estado "subsanación requerida" con 7 días hábiles restantes
    Cuando la persona interesada aporta el certificado de empadronamiento
    Entonces el expediente pasa a estado "en revisión"
    Y se genera un justificante con sello de tiempo

  Escenario: Lo aportado se suma, no sustituye
    Dado un expediente con documentación ya presentada
    Cuando la persona interesada aporta documentación de subsanación
    Entonces la documentación original se conserva íntegra
    Y la nueva queda identificada como aportada en subsanación

  Escenario: No se admite la subsanación fuera de plazo
    Dado un expediente cuyo plazo de subsanación ha vencido
    Cuando la persona interesada intenta aportar documentación
    Entonces la operación se rechaza con el error de dominio "plazo de subsanación vencido"
    Y se le explica su situación y qué puede hacer

  Escenario: El último instante del plazo todavía es plazo
    Dado un plazo de subsanación que vence hoy
    Cuando la persona interesada aporta la documentación antes de las 23:59:59
    Entonces la subsanación se admite
```

**Criterios de accesibilidad**

- El resultado se anuncia en región `polite` con el número de registro del nuevo
  asiento: «Documentación aportada. Registro número …».
- Tras aportar, el foco viaja al justificante de la aportación.
- El mensaje de rechazo por plazo vencido no culpa a la persona y explica la
  consecuencia real y qué le queda por hacer (`docs/diseno.md` §9).
- Se reutilizan las mismas primitivas de subida de E2-H03: mismo comportamiento, misma
  forma de anunciar.

**Estimación** 5 · **Depende de** E4-H01, E2-H03 · **Flujo crítico** 4 · **Semana** 7

---

## E4-H03 · Vencimiento del plazo sin subsanar

> Como **jefe de servicio** quiero que **el vencimiento del plazo de subsanación quede
> registrado automáticamente y con constancia** para **que el expediente refleje la
> realidad sin depender de que alguien lo mire**.

```gherkin
# language: es
Característica: Vencimiento del plazo de subsanación

  Escenario: El vencimiento produce el desistimiento
    Dado un expediente en estado "subsanación requerida"
    Cuando vence el plazo sin que se aporte documentación
    Entonces el expediente pasa a estado "desistido"
    Y la transición queda registrada con actor "sistema" y motivo el vencimiento del plazo

  Escenario: El expediente refleja el vencimiento en cuanto se consulta
    Dado un expediente cuyo plazo venció mientras nadie lo consultaba
    Cuando cualquier persona consulta el expediente
    Entonces ya consta en estado "desistido"
    Y el sello de la transición es el del vencimiento, no el de la consulta

  Escenario: Subsanar en el último día evita el desistimiento
    Dado un plazo de subsanación que vence hoy
    Cuando la persona interesada aporta la documentación dentro del día
    Entonces el expediente pasa a "en revisión"
    Y no se produce ningún desistimiento automático
```

**Criterios de accesibilidad**

- Al consultar un expediente que ha cambiado de estado por vencimiento, el cambio se
  explica con texto en la propia página: qué ha pasado, cuándo y por qué.
- El estado «Desistido» se comunica con `--gris`, texto tachado y motivo íntegro.

**Estimación** 5 · **Depende de** E0-H07, E4-H02 · **Flujo crítico** 4 · **Semana** 7

**Notas.** El sello de la transición es la fecha del vencimiento, no la de la consulta
que la materializa. Es la diferencia entre un registro veraz y uno que se inventa la
fecha. Ver `docs/modelo-dominio.md` §7.2: la norma exigiría resolución expresa; esta
implementación sigue lo que fija `docs/dominio.md` §2 y la divergencia está anotada.

---

## E4-H04 · Rechazo de la notificación por transcurso

> Como **órgano instructor** quiero que **una notificación no accedida se entienda
> rechazada a los diez días naturales** para **que el procedimiento no se paralice porque
> alguien no abre su buzón**.

```gherkin
# language: es
Característica: Rechazo de notificación por transcurso

  Escenario: Transcurridos diez días naturales sin acceso, la notificación se rechaza
    Dado una notificación puesta a disposición
    Cuando transcurren 10 días naturales sin que se acceda a su contenido
    Entonces la notificación consta como rechazada por transcurso
    Y se tiene por practicada a todos los efectos

  Escenario: El plazo de rechazo se cuenta en días naturales, no hábiles
    Dado una notificación puesta a disposición un viernes
    Cuando se calcula la fecha de rechazo
    Entonces los sábados, domingos y festivos del intervalo se cuentan

  Escenario: Acceder el último día evita el rechazo
    Dado una notificación puesta a disposición hace 9 días naturales
    Cuando la persona interesada accede a su contenido
    Entonces la notificación consta como accedida
    Y no se produce rechazo por transcurso

  Escenario: El plazo de subsanación se encadena a la fecha de efectos
    Dado una notificación rechazada por transcurso
    Cuando se calcula el plazo de subsanación
    Entonces se cuentan 10 días hábiles desde el día siguiente a la fecha de rechazo
```

**Criterios de accesibilidad**

- La distinción entre días naturales y días hábiles se explica en texto donde aparece,
  no se da por sabida. Es la confusión más común del procedimiento administrativo.
- El buzón indica en texto cuántos días naturales quedan antes del rechazo.

**Estimación** 5 · **Depende de** E3-H03, E0-H01 · **Flujo crítico** 4 · **Semana** 7

**Notas.** Ley 39/2015, art. 43.2. El cómputo desde el día siguiente a la puesta a
disposición está marcado como interpretación en `docs/modelo-dominio.md` `[D-22]`.
