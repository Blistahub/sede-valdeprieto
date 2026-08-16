# E2 · Solicitud, registro y justificante

**37 puntos.** Semanas 3–5. Es el **flujo crítico 2** y el corazón de la meta de la
semana 5. También el **flujo crítico 3** (reanudar borrador).

---

## E2-H01 · Iniciar una solicitud

> Como **persona interesada** quiero **empezar una solicitud para una convocatoria
> concreta** para **poder rellenarla con calma sabiendo que todavía no he presentado
> nada**.

```gherkin
# language: es
Característica: Inicio de la solicitud

  Escenario: Se crea un borrador sin efectos
    Dado una convocatoria con el plazo de presentación abierto
    Cuando una persona interesada inicia una solicitud
    Entonces se crea un borrador
    Y el borrador no tiene número de registro
    Y se le informa de que todavía no ha presentado nada

  Escenario: No se puede solicitar una convocatoria sin publicar
    Dado una convocatoria en preparación
    Cuando se intenta iniciar una solicitud
    Entonces la operación se rechaza con el error "convocatoria no publicada"

  Escenario: Una persona no puede tener dos borradores de la misma convocatoria
    Dado una persona interesada con un borrador abierto para una convocatoria
    Cuando inicia otra solicitud para la misma convocatoria
    Entonces se le ofrece continuar el borrador que ya tiene
```

**Criterios de accesibilidad**

- La diferencia entre borrador y solicitud presentada se comunica con texto explícito en
  la propia página, no con un matiz de color.
- El aviso de tratamiento de datos aparece **antes** de que se pida el primer dato
  personal, no después (`docs/dominio.md` §5).

**Estimación** 3 · **Depende de** E1-H02, E0-H05 · **Flujo crítico** 2 · **Semana** 3

---

## E2-H02 · Cumplimentar la solicitud por pasos

> Como **persona interesada** quiero **rellenar la solicitud en pasos cortos y que los
> errores se me expliquen cuando los cometo** para **no descubrir al final que tengo que
> volver atrás**.

```gherkin
# language: es
Característica: Cumplimentación de la solicitud

  Escenario: Los datos válidos permiten avanzar
    Dado un borrador de solicitud
    Cuando la persona interesada completa correctamente los datos personales
    Entonces puede continuar al paso siguiente
    Y sabe en qué paso está y cuántos quedan

  Escenario: Un dato inválido se explica y no deja avanzar
    Dado un borrador de solicitud
    Cuando la persona interesada indica un NIF con formato incorrecto
    Entonces se le explica qué está mal y cómo corregirlo
    Y no puede continuar al paso siguiente

  Escenario: La validación es la misma en el navegador y en el servidor
    Dado unos datos que el navegador considera válidos
    Cuando se envían al servidor
    Entonces el servidor aplica exactamente las mismas reglas
    Y unos datos manipulados fuera del navegador se rechazan igual

  Escenario: Se puede volver a un paso anterior sin perder lo escrito
    Dado una solicitud en el paso 3
    Cuando la persona interesada vuelve al paso 1
    Entonces conserva todo lo que había escrito
```

**Criterios de accesibilidad**

- Todo campo tiene `<label>` asociado. **El texto de ejemplo nunca hace de etiqueta.**
- Al fallar la validación, el foco viaja al **primer** campo con error, el campo se
  marca con `aria-invalid` y el mensaje se enlaza con `aria-describedby`.
- El resumen de errores del paso se anuncia a lectores de pantalla y cada entrada lleva
  al campo correspondiente.
- La validación se dispara al salir del campo, no solo al enviar
  (`docs/diseno.md` §10: prohibido el error que aparece cuando el campo ya no está en
  pantalla).
- El indicador de paso comunica «Paso 3 de 5» con texto, no solo con puntos de color, y
  se anuncia al cambiar de paso.
- Los campos obligatorios se identifican con texto, no solo con un asterisco de color.
- La columna del formulario no supera 640 px; funciona a 320 px y con zoom al 200 %.
- La transición entre pasos es uno de los tres movimientos aprobados: 8 px y opacidad,
  180 ms. Con `prefers-reduced-motion: reduce` se elimina.

**Estimación** 8 · **Depende de** E2-H01 · **Flujo crítico** 2 · **Semana** 4

**Notas.** Es la historia de mayor riesgo del backlog. Si en la semana 4 se ve que no
cabe, se parte en «datos personales y de contacto» y «datos de la ayuda» **antes** de
empezar.

---

## E2-H03 · Aportar la documentación exigida

> Como **persona interesada** quiero **adjuntar los documentos que exigen las bases y
> comprobar que han quedado bien** para **no que me requieran después por algo que creía
> haber entregado**.

```gherkin
# language: es
Característica: Documentación de la solicitud

  Escenario: Se aporta un documento válido
    Dado una convocatoria que exige certificado de empadronamiento
    Cuando la persona interesada aporta un documento admitido
    Entonces el documento queda asociado a la solicitud
    Y puede comprobar su nombre, su tamaño y su huella

  Escenario: Un documento con formato no admitido se rechaza explicando por qué
    Cuando la persona interesada aporta un documento con un formato no admitido
    Entonces se rechaza indicando qué formatos se admiten
    Y la solicitud conserva el resto de documentos

  Escenario: El formato se comprueba por el contenido, no por la extensión
    Cuando se aporta un fichero cuya extensión no corresponde con su contenido real
    Entonces el documento se rechaza

  Escenario: Falta documentación obligatoria
    Dado una solicitud sin el certificado exigido
    Cuando la persona interesada intenta continuar
    Entonces se le indica qué documento falta y por qué es obligatorio

  Escenario: Un documento aportado se puede retirar mientras sea borrador
    Dado un borrador con un documento aportado
    Cuando la persona interesada lo retira
    Entonces el documento deja de estar asociado a la solicitud
```

**Criterios de accesibilidad**

- El control de subida es un campo de fichero nativo con etiqueta asociada, operable con
  teclado. Si además hay zona de arrastre, es un complemento, nunca la única vía.
- El resultado de cada subida se anuncia en una región activa: «Certificado de
  empadronamiento aportado correctamente».
- El progreso de subida tiene equivalente textual con porcentaje, no solo una barra.
- La acción de retirar un documento es un botón con **texto**, no un icono suelto
  (`docs/diseno.md` §10: iconos sin texto en acciones destructivas).
- La lista de documentos aportados es una tabla con `<caption>` y `<th scope>`, y los
  tamaños y huellas van en fuente de dato.

**Estimación** 5 · **Depende de** E2-H02 · **Flujo crítico** 2 · **Semana** 4

---

## E2-H04 · Guardar y reanudar un borrador

> Como **persona interesada que se ha quedado sin tiempo** quiero **dejar la solicitud a
> medias y continuarla otro día** para **no tener que reunir toda la documentación de una
> sentada**.

```gherkin
# language: es
Característica: Reanudación del borrador

  Escenario: El borrador conserva lo cumplimentado
    Dado un borrador cumplimentado hasta el paso 3
    Cuando la persona interesada vuelve más tarde
    Entonces recupera todos los datos y documentos que había aportado
    Y continúa desde el paso 3

  Escenario: El borrador indica desde cuándo está sin tocar
    Dado un borrador sin modificar desde hace días
    Cuando la persona interesada accede a sus solicitudes
    Entonces conoce la fecha de la última modificación
    Y conoce el plazo que le queda para presentarlo

  Escenario: Un borrador de una convocatoria cerrada avisa
    Dado un borrador de una convocatoria cuyo plazo ha terminado
    Cuando la persona interesada lo abre
    Entonces se le advierte de que el plazo de presentación ha terminado
```

**Criterios de accesibilidad**

- Al reanudar, el foco se sitúa en el encabezado del paso recuperado y se anuncia qué se
  ha recuperado.
- Las fechas de última modificación van en fuente de dato y en formato completo, no en
  «hace 3 días» como único texto.

**Estimación** 3 · **Depende de** E2-H02 · **Flujo crítico** 3 · **Semana** 3

---

## E2-H05 · Revisar antes de presentar

> Como **persona interesada** quiero **ver todo lo que voy a presentar en una sola
> pantalla antes de registrar** para **corregir un error mientras todavía puedo**.

```gherkin
# language: es
Característica: Revisión previa al registro

  Escenario: Se muestra el contenido completo de la solicitud
    Dado una solicitud completa
    Cuando la persona interesada llega al paso de revisión
    Entonces ve todos los datos que ha cumplimentado y todos los documentos aportados

  Escenario: Se puede corregir cualquier apartado
    Dado el paso de revisión
    Cuando la persona interesada decide corregir los datos de contacto
    Entonces vuelve a ese apartado y regresa a la revisión conservando el resto

  Escenario: Se advierte de que el registro es irreversible
    Dado el paso de revisión
    Entonces se advierte de que una vez registrada la solicitud no se puede modificar
    Y se explica que los cambios posteriores se hacen aportando documentación nueva

  Escenario: No se registra sin aceptar la declaración responsable
    Dado una solicitud completa
    Cuando la persona interesada no acepta la declaración responsable
    Entonces no puede registrar la solicitud
```

**Criterios de accesibilidad**

- El resumen usa una estructura de lista de descripción, con cada dato asociado a su
  concepto, para que un lector de pantalla lo recorra por parejas.
- Cada enlace de corrección indica en su texto accesible **qué** corrige: «Corregir los
  datos de contacto», nunca «Editar».
- La declaración responsable es una casilla con etiqueta completa y visible; su texto no
  se sustituye por un enlace.
- El botón dice exactamente lo que va a pasar: «Registrar solicitud». Nunca «Enviar» ni
  «Aceptar» (`docs/diseno.md` §9).

**Estimación** 3 · **Depende de** E2-H03 · **Flujo crítico** 2 · **Semana** 4

---

## E2-H06 · Registrar la solicitud y obtener el justificante sellado

> Como **persona interesada** quiero **registrar mi solicitud y recibir un justificante
> con número y sello de tiempo** para **poder demostrar que la presenté, cuándo y qué
> presenté**.

```gherkin
# language: es
Característica: Registro de la solicitud

  Escenario: El registro genera número, sello y justificante
    Dado una solicitud completa y revisada
    Cuando la persona interesada registra la solicitud
    Entonces obtiene un número de registro
    Y obtiene un sello de tiempo tomado en el servidor con segundos
    Y obtiene un justificante con la huella de lo presentado
    Y se crea el expediente en estado "registrado"

  Escenario: La solicitud registrada es inmutable
    Dado una solicitud registrada
    Cuando se intenta modificar cualquiera de sus datos
    Entonces la operación se rechaza con el error de dominio "solicitud inmutable"

  Escenario: El sello de tiempo lo pone el servidor
    Dado un cliente cuya hora local está adelantada
    Cuando registra una solicitud
    Entonces el sello del justificante es el del servidor, no el del cliente

  Escenario: Un fallo al registrar no deja la solicitud a medias
    Dado una solicitud completa
    Cuando el registro falla por un problema del servicio
    Entonces no se genera número de registro
    Y la solicitud sigue siendo un borrador editable
    Y se explica que no se ha presentado nada y qué hacer a continuación

  Escenario: El registro fuera de plazo se acepta y se advierte
    Dado una convocatoria con el plazo de presentación terminado
    Cuando la persona interesada registra su solicitud
    Entonces la solicitud se registra y obtiene justificante
    Y se le advierte de que se inadmitirá por presentación fuera de plazo
```

**Criterios de accesibilidad**

- Al confirmarse el registro, el foco viaja al botón **Descargar justificante** y se
  anuncia en una región `polite`: «Solicitud registrada con el número …».
- El número de registro se presenta con los grupos de dígitos separados para poder
  dictarlo por teléfono, en fuente de dato con cifras tabulares y tamaño Display.
- El cuño de latón es **decorativo**: se oculta a la tecnología de apoyo y toda su
  información está en texto.
- El sellado es uno de los tres movimientos aprobados: escala 1,06 → 1,00 y opacidad
  0 → 1 en 320 ms, un solo movimiento, sin rebote. Con `prefers-reduced-motion: reduce`
  el justificante aparece completo, y el anuncio y el foco funcionan igual.
- **Prohibido** confeti, sonido, icono animado o marca de verificación que se dibuja.
- El mensaje de error de registro distingue «no se ha presentado» de «no sabemos si se
  ha presentado», porque son situaciones distintas para quien lo lee.

**Estimación** 8 · **Depende de** E2-H05, E0-H03, E0-H04 · **Flujo crítico** 2 ·
**Semana** 5

---

## E2-H07 · Descargar el justificante

> Como **persona interesada** quiero **descargar y conservar mi justificante** para
> **tener la prueba fuera del portal**.

```gherkin
# language: es
Característica: Justificante descargable

  Escenario: El justificante contiene todo lo necesario para acreditar la presentación
    Cuando la persona interesada descarga su justificante
    Entonces el documento contiene número de registro, fecha y hora con segundos,
      convocatoria, relación de documentos presentados y huella
    Y contiene el código seguro de verificación

  Escenario: El justificante se puede recuperar más adelante
    Dado una solicitud registrada hace semanas
    Cuando la persona interesada consulta su expediente
    Entonces puede volver a descargar el mismo justificante
    Y su huella coincide con la original
```

**Criterios de accesibilidad**

- El documento descargado lleva idioma declarado, título, estructura de encabezados y
  orden de lectura correcto. El objetivo declarado es **PDF/UA**; hasta alcanzarlo, la
  limitación se documenta en `docs/accesibilidad/` en lugar de ocultarse.
- Existe siempre una versión en página web accesible del mismo justificante: la
  descarga nunca es la única forma de leerlo.
- El enlace de descarga indica formato y tamaño en su texto accesible.

**Estimación** 3 · **Depende de** E2-H06 · **Flujo crítico** 2 · **Semana** 5

---

## E2-H08 · Eliminar un borrador

> Como **persona interesada** quiero **borrar una solicitud que no voy a presentar** para
> **que mis datos no se queden guardados sin motivo**.

```gherkin
# language: es
Característica: Eliminación de borradores

  Escenario: Un borrador se elimina a petición
    Dado un borrador de solicitud
    Cuando la persona interesada lo elimina y confirma
    Entonces el borrador y sus documentos dejan de existir

  Escenario: Una solicitud registrada no se elimina
    Dado una solicitud registrada
    Cuando se intenta eliminarla
    Entonces la operación se rechaza
    Y se explica que para no continuar hay que desistir
```

**Criterios de accesibilidad**

- La confirmación es un diálogo modal que atrapa el foco correctamente, se cierra con
  `Escape` y **devuelve el foco** al control que lo abrió.
- La acción destructiva usa `--lacre` con texto explícito, nunca un icono suelto, y su
  contraste se comprueba contra la matriz.
- El texto del botón nombra lo que se borra: «Eliminar el borrador», no «Eliminar».

**Estimación** 2 · **Depende de** E2-H01 · **Semana** 6

---

## E2-H09 · Inadmisión por presentación fuera de plazo

> Como **jefe de servicio** quiero **inadmitir las solicitudes presentadas fuera de
> plazo dejando constancia motivada** para **no rechazarlas de forma silenciosa**.

```gherkin
# language: es
Característica: Inadmisión

  Escenario: Se inadmite una solicitud presentada fuera de plazo
    Dado un expediente registrado con sello posterior al fin del plazo de presentación
    Cuando el jefe de servicio lo inadmite indicando el motivo
    Entonces el expediente pasa a estado "inadmitido"
    Y la persona interesada recibe la resolución de inadmisión

  Escenario: No se inadmite sin motivo
    Cuando se intenta inadmitir sin indicar motivo
    Entonces la operación se rechaza con el error de dominio "motivo requerido"

  Escenario: La frontera del plazo es exacta
    Dado una solicitud registrada en el último segundo del plazo de presentación
    Entonces se considera presentada en plazo
```

**Criterios de accesibilidad**

- El estado «Inadmitido» se comunica con `--gris`, texto tachado **y** motivo visible.
  El tachado nunca es el único portador de la información.
- El motivo se muestra íntegro en la página, no truncado tras un enlace.

**Estimación** 2 · **Depende de** E2-H06, E0-H03 · **Semana** 6
