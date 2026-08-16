# E7 · Sede, verificación pública y cumplimiento

**9 puntos.** Transversal; E7-H03 y E7-H04 en semanas 1 y 3, E7-H01 en la semana 5.

Es la épica que convierte el invariante III —«la calidad tiene que ser verificable por un
tercero»— en algo que se puede abrir en una pestaña del navegador.

---

## E7-H01 · Verificar un justificante sin identificarse

> Como **cualquier persona con un justificante delante** quiero **comprobar si es
> auténtico introduciendo su código** para **no tener que fiarme del papel que me
> enseñan**.

```gherkin
# language: es
Característica: Verificación por código seguro de verificación

  Escenario: Un código válido confirma la autenticidad
    Dado un justificante emitido por la sede
    Cuando cualquier persona introduce su código seguro de verificación
    Entonces se confirma que el documento es auténtico
    Y se muestran el número de registro, el sello de tiempo, la convocatoria y la huella

  Escenario: La verificación no expone datos personales
    Cuando se verifica un código válido
    Entonces no se muestra ningún dato personal de la persona solicitante

  Escenario: Un código inexistente se rechaza sin dar pistas
    Cuando se introduce un código que no existe
    Entonces se indica que no corresponde a ningún documento emitido
    Y no se revela ninguna información sobre otros documentos

  Escenario: La verificación no exige identificación
    Cuando una persona sin identificar accede a la página de verificación
    Entonces puede verificar un código igualmente
```

**Criterios de accesibilidad**

- El campo del código tiene etiqueta visible que explica dónde encontrarlo en el
  justificante, y admite el código con o sin los espacios de agrupación.
- El resultado se anuncia en región `polite` y el foco viaja al resultado.
- El resultado se comunica con texto y forma además de color: «Documento auténtico» /
  «Código no encontrado».
- El código y la huella se muestran en fuente de dato, agrupados para poder dictarlos, y
  son seleccionables como texto.

**Estimación** 3 · **Depende de** E2-H06 · **Semana** 5

---

## E7-H02 · Declaración de accesibilidad y reclamaciones

> Como **persona usuaria con discapacidad** quiero **saber en qué grado cumple esta sede
> y cómo reclamar si algo no me funciona** para **poder ejercer mis derechos**.

```gherkin
# language: es
Característica: Declaración de accesibilidad

  Escenario: La declaración es accesible desde cualquier página
    Cuando una persona consulta cualquier página de la sede
    Entonces puede llegar a la declaración de accesibilidad desde el pie

  Escenario: La declaración contiene lo que exige el modelo
    Cuando una persona consulta la declaración de accesibilidad
    Entonces conoce el grado de cumplimiento, el contenido no accesible y su motivo,
      la fecha de la declaración, el método de evaluación empleado
      y el procedimiento de reclamación

  Escenario: El mecanismo de reclamación funciona
    Cuando una persona presenta una comunicación sobre accesibilidad
    Entonces obtiene constancia de su presentación
    Y conoce el plazo de respuesta
```

**Criterios de accesibilidad**

- La declaración es la página que mejor tiene que funcionar de todo el portal: cero
  violaciones de axe de cualquier severidad, no solo `serious` y `critical`.
- Redactada en lenguaje claro, con la estructura del modelo de la Comisión Europea y con
  las referencias al RD 1112/2018 y a la norma EN 301 549 citadas con precisión y
  enlazadas a su fuente oficial.
- El enlace del pie es visible sin desplegar menús y forma parte del orden de tabulación
  natural.

**Estimación** 3 · **Depende de** — · **Semana** 12

**Notas.** Se redacta al final porque tiene que describir el estado **real**, no el
deseado. Una declaración de accesibilidad escrita antes de la auditoría es una
declaración falsa.

---

## E7-H03 · Aviso permanente de portal de demostración

> Como **cualquier visitante** quiero **saber en todo momento que esto no es una sede
> real** para **no creer que he presentado algo con efectos administrativos**.

```gherkin
# language: es
Característica: Aviso de demostración

  Escenario: El aviso está en todas las pantallas
    Cuando una persona consulta cualquier página de la sede
    Entonces ve el aviso de que es un portal de demostración de un municipio ficticio
      sin validez administrativa

  Escenario: El aviso aparece también en los documentos emitidos
    Cuando se emite un justificante o una resolución
    Entonces el documento incluye el mismo aviso
```

**Criterios de accesibilidad**

- El aviso es texto real en el flujo del documento, no una imagen ni una marca de agua
  meramente visual, y forma parte del contenido que lee un lector de pantalla.
- Su contraste cumple la matriz de `docs/diseno.md`: discreto no significa ilegible.
- No es descartable, no flota sobre el contenido y no atrapa el foco.

**Estimación** 1 · **Depende de** E0-H05 · **Semana** 1

---

## E7-H04 · Información de tratamiento de datos antes de pedirlos

> Como **persona interesada** quiero **saber qué se hace con mis datos antes de
> escribirlos** para **decidir con conocimiento si continúo**.

```gherkin
# language: es
Característica: Información de tratamiento

  Escenario: La información se muestra antes de pedir el primer dato personal
    Dado un formulario que recoge datos personales
    Cuando la persona interesada llega a él
    Entonces conoce la finalidad del tratamiento, su base jurídica, el plazo de
      conservación y cómo ejercer sus derechos
    Y esa información aparece antes de los campos, no después del envío

  Escenario: No se piden datos que el procedimiento no necesite
    Cuando se revisa un formulario del procedimiento
    Entonces cada dato solicitado tiene justificación en las bases reguladoras
      o en el procedimiento
```

**Criterios de accesibilidad**

- La información se muestra como texto en la propia página. Si se resume, el texto
  íntegro está a un enlace visible, nunca detrás de un icono.
- Si se presenta en un bloque desplegable, el control es un `button` con estado
  `aria-expanded`, y el bloque forma parte del orden de lectura.
- Redactada en lenguaje claro y en segunda persona, conforme a `docs/diseno.md` §9.

**Estimación** 2 · **Depende de** E2-H01 · **Semana** 3
