# Sistema de diseño — Sede de Valdeprieto

Este documento es normativo. Los valores de aquí no se improvisan ni se ajustan "a ojo"
en un componente. Cambiarlos requiere un ADR.

---

## 1. Dirección

**El mundo del que sale este diseño.** No sale de una app SaaS. Sale del archivo
municipal: la tinta sobre papel de oficio, el verde de las encuadernaciones de los
boletines oficiales, el lacre rojo de los sellos, el latón del cuño de registro, los
números de expediente escritos con máquina. Ese es el material.

**El trabajo que hace la interfaz.** Que una persona —de cualquier edad, con cualquier
capacidad, probablemente con prisa y algo de miedo a equivocarse— presente una solicitud
sin cometer errores y sepa en todo momento en qué punto está y cuánto plazo le queda.

**La tensión que resuelve el diseño.** Autoridad institucional sin frialdad
burocrática. Debe parecer serio y oficial, porque lo que hace tiene consecuencias
legales, pero no debe intimidar.

**Lo que este diseño NO es.** No es un clon de GOV.UK. No es azul corporativo genérico.
No es crema con serif de alto contraste y acento terracota, que es a lo que tiende por
defecto cualquier generador. No lleva degradados, ni glassmorphism, ni tarjetas con
sombra flotante, ni esquinas de 16 px.

**El riesgo estético asumido.** Verde archivo profundo como color institucional en lugar
del azul de administración. Es una decisión derivada del material del dominio, se
sostiene en contraste, y hace que el producto no se confunda con ningún otro portfolio.

## 2. Color

### Marca

```css
--tinta:        #0E1C17;  /* texto principal, casi negro con matiz verde */
--verde-900:    #10241C;
--verde-700:    #1B4332;  /* color institucional: cabeceras, botón primario */
--verde-500:    #2D6A4F;  /* hover, estados favorables */
--papel:        #EFF1EC;  /* fondo de página */
--papel-alto:   #FFFFFF;  /* superficies elevadas: tarjetas, modales */
--gris:         #5A6560;  /* texto secundario */
--linea:        #C9CFC8;  /* filete decorativo — NUNCA borde funcional */
--linea-fuerte: #7C857E;  /* borde de controles, tablas, inputs */
--lacre:        #8C2F2F;  /* plazo en riesgo, error, acción destructiva */
--azul:         #1F4E79;  /* información, estado "en revisión" */
--sello:        #B08B2E;  /* latón. SOLO sello de registro y anillo de foco */
```

### Regla estructural: el color codifica la máquina de estados

La paleta semántica **no** es "éxito / aviso / error" genérico. Es el estado legal del
expediente. Esto es más veraz y además es testeable: cada estado tiene su token.

| Estado | Color | Refuerzo no cromático |
| --- | --- | --- |
| Borrador | `--gris` | Icono de lápiz + texto |
| Registrado | `--sello` (fondo) + `--tinta` (texto) | Cuño + número de registro |
| En revisión | `--azul` | Icono de reloj + texto |
| Subsanación requerida | `--lacre` | Icono de aviso + **días hábiles restantes** |
| Resuelto favorable | `--verde-500` | Icono de check + fecha de resolución |
| Desistido / caducado | `--gris` | Texto tachado + motivo |

El color nunca va solo. Siempre color **+** texto **+** forma.

### Matriz de contraste (calculada, no estimada)

| Combinación | Ratio | Uso permitido |
| --- | --- | --- |
| `--tinta` sobre `--papel` | 15.42:1 | Todo |
| `--tinta` sobre `--papel-alto` | 17.54:1 | Todo |
| `--verde-700` sobre `--papel` | 9.74:1 | Todo |
| `--papel-alto` sobre `--verde-700` | 11.08:1 | Botón primario |
| `--verde-500` sobre `--papel` | 5.62:1 | Texto y UI |
| `--lacre` sobre `--papel` | 7.20:1 | Texto y UI |
| `--papel-alto` sobre `--lacre` | 8.19:1 | Botón destructivo |
| `--azul` sobre `--papel` | 7.62:1 | Texto y UI |
| `--gris` sobre `--papel` | 5.33:1 | Texto secundario |
| `--linea-fuerte` sobre `--papel` | 3.35:1 | Bordes funcionales (WCAG 1.4.11) |
| `--tinta` sobre `--sello` | 5.50:1 | Texto sobre el cuño |
| **`--sello` sobre `--papel`** | **2.81:1** | **PROHIBIDO como texto o borde único** |
| **`--linea` sobre `--papel`** | **1.39:1** | **Solo filete decorativo, nunca funcional** |

Las dos últimas filas son las trampas del sistema. Si alguna vez ves texto dorado sobre
papel, es un bug de accesibilidad.

## 3. Tipografía

```css
--fuente-titular: 'Archivo', system-ui, sans-serif;      /* 600–800 */
--fuente-texto:   'Source Sans 3', system-ui, sans-serif;
--fuente-dato:    'IBM Plex Mono', ui-monospace, monospace;
```

**Por qué Archivo.** Es una grotesca de alto rendimiento con anchos extendidos que a
pesos altos tiene peso institucional sin caer en el serif editorial de manual. Y se
llama, literalmente, como aquello que gestiona el producto. Esa coincidencia es el tipo
de detalle que hace memorable un caso de estudio.

**Por qué Source Sans 3.** Altura de x generosa, diseñada para interfaz, y trata los
acentos y la ñ del español con corrección real, no como un añadido.

**Por qué una monoespaciada.** Los números de este producto no son decorativos: son
identificadores legales que la gente lee en voz alta por teléfono, copia y compara.

### Regla del dato

Todo número que una persona pueda necesitar leer, copiar o comparar va en
`--fuente-dato` con `font-variant-numeric: tabular-nums`:

número de expediente · NIF/NIE · IBAN · importes · fechas · sellos de tiempo · días
hábiles restantes · número de registro.

Es simultáneamente decisión de diseño, de accesibilidad y de dominio.

### Escala

| Rol | Tamaño / interlínea | Fuente | Peso |
| --- | --- | --- | --- |
| Display (solo portada y justificante) | 56 / 60 px | Archivo Expanded | 800 |
| H1 | 36 / 44 px | Archivo | 700 |
| H2 | 28 / 36 px | Archivo | 700 |
| H3 | 21 / 28 px | Archivo | 600 |
| Cuerpo | 17 / 28 px | Source Sans 3 | 400 |
| Cuerpo largo (bases, resoluciones) | 19 / 32 px | Source Sans 3 | 400 |
| Auxiliar | 15 / 24 px | Source Sans 3 | 400 |
| Etiqueta / eyebrow | 13 / 16 px, `letter-spacing: 0.08em`, versalitas | Archivo | 600 |
| Dato | 16 / 24 px | IBM Plex Mono | 500 |

17 px de cuerpo, no 16. El público incluye personas mayores y la diferencia se nota.
Medida de línea: **60–72 caracteres**. Nunca un párrafo a todo el ancho.

## 4. Espacio, forma, elevación

- Base de 4 px. Escala: 4 · 8 · 12 · 16 · 24 · 32 · 48 · 64 · 96.
- Radios: `2px` en inputs, `4px` en tarjetas y botones, `0` en tablas. Nada mayor.
  Las esquinas muy redondeadas leen "startup" y aquí restan credibilidad.
- **Sin sombras** salvo en superficies que flotan de verdad (modal, menú desplegable,
  toast). La jerarquía se construye con filete de 1 px y con espacio.
- Anchura de la columna de trabajo en formularios: **640 px máximo**. Un formulario
  ancho es un formulario con más errores.
- Área táctil mínima **44 × 44 px** en todo control. Excede el mínimo legal a propósito:
  el público objetivo lo necesita.

## 5. Foco

Anillo de dos capas, para que funcione sobre fondo claro y oscuro:

```css
:focus-visible {
  outline: 3px solid var(--tinta);
  outline-offset: 0;
  box-shadow: 0 0 0 6px var(--sello);
}
```

La capa interior de tinta garantiza el contraste sobre `--papel` (15.42:1); la exterior
de latón lo garantiza sobre `--verde-700` (3.47:1). Nunca uses una sola capa. Nunca
uses `outline: none`.

## 6. Disposición

```
┌──────────────────────────────────────────────────────────────┐
│ Cabecera institucional  ·  verde-700, altura fija             │
├──────────────────────────────────────────────────────────────┤
│ Migas de pan / Convocatoria › Solicitud › Paso 2 de 5         │
├────────────────────────────────────┬─────────────────────────┤
│                                    │  ESTADO DEL EXPEDIENTE   │
│  Columna de trabajo (máx. 640 px)  │  ────────────────────    │
│  El formulario, la resolución,     │  ① Registrado    12 may  │
│  la documentación.                 │  ② En revisión   14 may  │
│                                    │  ③ Subsanación   ● hoy   │
│                                    │     quedan 7 días hábiles│
│                                    │  ④ Resolución    —       │
├────────────────────────────────────┴─────────────────────────┤
│ Pie: declaración de accesibilidad · reclamaciones · privacidad│
└──────────────────────────────────────────────────────────────┘
```

**Sobre el raíl numerado.** La numeración aquí está justificada porque el procedimiento
administrativo *es* una secuencia legal ordenada con fechas reales, no una decoración.
Si el contenido no fuera una secuencia, no llevaría números.

En móvil el raíl colapsa a una barra fija superior con dos datos: estado actual y días
hábiles restantes. Nada más.

## 7. Elemento firma: el justificante sellado

Este es el único sitio donde el diseño se permite ser espectacular. Todo lo demás calla
para que este momento suene.

Cuando la persona pulsa **Registrar solicitud** y la operación se confirma en servidor:

1. El formulario se retira (200 ms, `ease-out`).
2. Aparece el justificante: superficie `--papel-alto`, filete de 1 px, sin sombra.
3. El número de registro entra en Display 56 px, IBM Plex Mono, `tabular-nums`,
   con los grupos de dígitos espaciados para poder leerlos en voz alta.
4. El cuño de latón (`--sello`) se asienta sobre el papel: escala de 1.06 → 1.00 y
   opacidad 0 → 1 en 320 ms. Un solo movimiento, sin rebote.
5. Sello de tiempo con segundos, en monoespaciada, debajo.
6. Foco del navegador al botón **Descargar justificante**, y anuncio en
   `aria-live="polite"`: «Solicitud registrada con el número …».

Con `prefers-reduced-motion: reduce` el justificante aparece completo, sin transición.
El anuncio y el foco funcionan igual.

**Prohibido**: confeti, sonidos, iconos animados, marcas de verificación que se dibujan.
Esto es un acto administrativo, no un juego.

## 8. Movimiento

Solo existen **tres** momentos animados en todo el producto:

1. El sellado del justificante (arriba).
2. La transición entre pasos del formulario: desplazamiento de 8 px + opacidad, 180 ms.
3. La entrada de un mensaje de error de validación: opacidad + 4 px, 120 ms.

Todo lo demás es instantáneo. Curva por defecto `cubic-bezier(0.2, 0, 0, 1)`.
`prefers-reduced-motion: reduce` elimina las tres, no las acorta.

## 9. Voz

- **Segunda persona directa** en la interfaz: «Necesitas adjuntar el certificado».
  **Registro formal** solo en documentos con valor legal: justificantes, requerimientos,
  resoluciones.
- El botón dice exactamente lo que va a pasar, y el resultado usa el mismo verbo:
  «Registrar solicitud» → «Solicitud registrada». Nunca «Enviar» ni «Aceptar».
- Los errores dicen qué pasó y cómo arreglarlo, sin disculparse y sin vaguedad.
  Mal: «Ha ocurrido un error». Bien: «El NIF debe terminar en letra. Escríbelo sin
  espacios ni guiones».
- Las pantallas vacías son una invitación a actuar, no un lamento.
  «Todavía no tienes solicitudes. Consulta las convocatorias abiertas.»
- Nada de jerga de sistema. La persona gestiona *solicitudes* y *documentos*, no
  «registros» ni «entidades».
- Los textos legales obligatorios se muestran íntegros, pero acompañados del resumen en
  lenguaje claro generado por la funcionalidad de IA, claramente etiquetado como
  **resumen orientativo sin valor jurídico**.

## 10. Antipatrones que rechazar de plano

- Placeholder haciendo de etiqueta.
- Validación que solo aparece al enviar, cuando el campo ya no está en pantalla.
- `div` con `onClick` en lugar de `button`.
- Estado comunicado solo por color.
- Tabla de expedientes sin `<caption>`, `<th scope>` ni orden accesible.
- Modal que atrapa el foco mal o que no devuelve el foco al cerrarse.
- Iconos sin texto en acciones destructivas.
- Scroll infinito en un listado de expedientes. Paginación con números y total.
- Cuenta atrás que solo se ve, sin equivalente textual leíble.
