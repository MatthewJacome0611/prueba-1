# Reporte de auditoría de accesibilidad WCAG 2.2 AA

**Alcance:** `index.html` y `styles.css` de la carpeta `ejemplo2`
**Enfoque:** principios POUR: Perceptible, Operable, Comprensible y Robusto
**Tipo de revisión:** inspección estática del código; no sustituye pruebas con tecnologías de asistencia, teclado real ni herramientas automáticas de contraste.

## Resumen ejecutivo

La implementación presenta una base sólida de HTML5 semántico: utiliza landmarks, un único `h1`, `main`, `section`, `fieldset` y `legend`; los controles tienen etiquetas explícitas y se incluyen ayudas asociadas con `aria-describedby`. También incorpora un enlace para saltar al contenido, estilos `:focus-visible` y validación nativa.

Se identifican tres mejoras relevantes para alcanzar una implementación más robusta:

1. El texto de ayuda promete un límite de 5 MB para la imagen, pero HTML no aplica ese límite y el navegador no lo valida de forma nativa.
2. Los mensajes de error están presentes como `role="alert"` incluso cuando CSS los oculta; conviene controlar de forma explícita cuándo se anuncian para evitar alertas prematuras o repetidas.
3. Los grupos de radios y casillas usan `role="group"` aunque ya están contenidos en `fieldset` con `legend`; es una capa ARIA potencialmente redundante.

## 1. Perceptible

### Cumplimientos

- **Landmarks y jerarquía visual:** [index.html:10-24](index.html) contiene `header`, `main`, `section` y `footer`, con un único `h1` y un `h2` para la sección del formulario.
- **Texto alternativo y contenido multimedia:** no se incluyen imágenes ni otros elementos que requieran `alt`, `figure` o `figcaption`. Por tanto, no hay una omisión de texto alternativo en el alcance auditado.
- **Ayuda textual de los controles:** los textos de ayuda están vinculados mediante `aria-describedby`, por ejemplo en el campo de nombre y en los controles de preferencias ([index.html:36-42](index.html), [index.html:67-76](index.html)).
- **Indicadores no basados únicamente en color:** los errores incluyen texto explícito y los campos obligatorios muestran el símbolo junto con la leyenda “Campo obligatorio” ([index.html:31-33](index.html)).
- **Contraste declarado por estilos:** el texto principal oscuro sobre fondos claros, el texto claro sobre el encabezado oscuro y los botones con fondo azul oscuro están planteados para contraste alto. Debe confirmarse con un medidor WCAG sobre los colores finales y estados interactivos.

### Hallazgos

#### P-01 — Límite de archivo no implementado

- **Ubicación:** [index.html:113-115](index.html)
- **Severidad:** Media
- **Estado:** No cumple completamente con la expectativa comunicada.
- **Observación:** la ayuda indica “máximo 5 MB”, pero `accept` solo restringe tipos MIME y no limita el tamaño. La persona usuaria no recibe una validación equivalente si selecciona un archivo mayor.
- **Recomendación:** implementar una validación de tamaño que informe el error de forma accesible, o retirar la promesa de 5 MB y documentar el límite en el servidor.

## 2. Operable

### Cumplimientos

- **Salto al contenido principal:** [index.html:10](index.html) incluye `.skip-link` dirigido a `#contenido`; [styles.css:43-57](styles.css) lo muestra al recibir foco.
- **Foco visible:** [styles.css:59-63](styles.css) define `:focus-visible` para enlaces, campos, área de texto y botones, con contorno y separación visibles.
- **Controles nativos y teclado:** [index.html:27-134](index.html) utiliza controles HTML nativos (`input`, `textarea`, `button`) y no sustituye su interacción por componentes exclusivamente basados en scripts.
- **Orden de navegación:** el orden DOM sigue encabezado, formulario, campos, acciones y pie de página; no se observan `tabindex` positivos que alteren el orden natural.
- **Movimiento reducido:** [styles.css:299-304](styles.css) respeta `prefers-reduced-motion` para el desplazamiento suave.

### Hallazgos

#### O-01 — Dependencia de `:has()` para un estado de error

- **Ubicación:** [styles.css:269-271](styles.css)
- **Severidad:** Baja
- **Estado:** Riesgo de compatibilidad.
- **Observación:** la visibilidad del error de consentimiento depende de `:has()` en el selector original de la estructura. Aunque los navegadores modernos lo soportan, una estrategia basada en la validación nativa del propio control y/o en un estado explícito resulta más predecible para entornos antiguos o tecnologías integradas.
- **Recomendación:** mantener una presentación visible y consistente del mensaje cuando el formulario se intenta enviar, usando CSS compatible o una actualización de estado accesible cuidadosamente gestionada.

## 3. Comprensible

### Cumplimientos

- **Etiquetas sin placeholders:** [index.html:35-129](index.html) usa `label` visible y no emplea `placeholder` como sustituto de la etiqueta.
- **Agrupación comprensible:** [index.html:27-134](index.html) separa contacto, preferencias y detalles adicionales con `fieldset` y `legend`.
- **Reglas de validación comunicadas:** los campos incluyen `required`, `min`, `max`, `pattern` y rangos de fecha; las ayudas explican los valores esperados.
- **Mensajes vinculados:** los errores están asociados mediante `aria-describedby` y usan `role="alert"`; por ejemplo, correo, teléfono y fecha ([index.html:43-65](index.html)).
- **Patrón regla + corrección:** mensajes como “Selecciona una fecha dentro del periodo indicado” o “Introduce un teléfono válido” explican qué condición debe corregirse sin depender solo del color.

### Hallazgos

#### C-01 — Alertas de error declaradas desde la carga inicial

- **Ubicación:** [index.html:39-40](index.html), [index.html:47-48](index.html) y mensajes equivalentes del formulario; [styles.css:207-224](index.html)
- **Severidad:** Media
- **Estado:** Mejora necesaria.
- **Observación:** cada mensaje tiene `role="alert"` en el DOM desde el inicio, aunque CSS lo oculta. Dependiendo del navegador y lector de pantalla, esto puede generar anuncios al cargar, anuncios duplicados o una experiencia confusa antes de que la persona interactúe con el formulario.
- **Recomendación:** mantener el mensaje oculto semánticamente hasta que exista un error real (`hidden`/estado equivalente) y activarlo tras validación o envío, conservando `aria-describedby` en el campo.

#### C-02 — Estado de error no se comunica con un estilo de campo completo

- **Ubicación:** [styles.css:197-204](styles.css)
- **Severidad:** Baja
- **Estado:** Parcial.
- **Observación:** el borde rojo y el texto del error son adecuados como redundancia visual, pero la regla depende de `:not(:placeholder-shown)`. Como los campos no tienen `placeholder`, conviene verificar en los navegadores objetivo que los campos vacíos `required` muestren el estado esperado después del intento de envío.
- **Recomendación:** probar específicamente campos vacíos, valores inválidos y corrección posterior; preferiblemente aplicar una clase de estado tras el submit para una presentación determinista.

## 4. Robusto

### Cumplimientos

- **Semántica nativa:** [index.html:10-24](index.html) usa landmarks HTML5 y [index.html:27-134](index.html) usa controles nativos con sus capacidades de validación.
- **Asociación explícita de etiquetas:** los controles tienen `id` y sus etiquetas correspondientes tienen `for`, incluidos radios, casillas y consentimiento ([index.html:79-110](index.html), [index.html:124-129](index.html)).
- **Nombres accesibles:** los `legend`, `label` y textos de grupo proporcionan nombres comprensibles para los controles.
- **ARIA como complemento:** `aria-describedby` aporta ayuda y errores sin reemplazar `label`, `fieldset` o `legend`.

### Hallazgos

#### R-01 — Roles `group` potencialmente redundantes con `fieldset`

- **Ubicación:** [index.html:78](index.html) y [index.html:94](index.html)
- **Severidad:** Baja
- **Estado:** Requiere simplificación.
- **Observación:** “Modalidad preferida” y “Temas que te interesan” ya forman parte del `fieldset` “Preferencias de la jornada” y tienen texto de grupo. Añadir `role="group"` sobre `div` no aporta una relación más fuerte que la semántica nativa y puede producir anuncios redundantes en algunas combinaciones de navegador y lector de pantalla.
- **Recomendación:** conservar la agrupación nativa con `fieldset`/`legend`; si se necesita una subagrupación accesible, usar otro `fieldset` con `legend` en lugar de duplicar roles ARIA.

#### R-02 — Falta de validación accesible para el tamaño del archivo

- **Ubicación:** [index.html:113-115](index.html)
- **Severidad:** Media
- **Estado:** Repetido desde Perceptible por impacto en robustez.
- **Observación:** la interfaz comunica una regla que no está representada en la validación del control ni en un mecanismo de error anunciado. Esto puede producir comportamientos diferentes entre cliente y servidor.
- **Recomendación:** hacer que la regla exista en la validación efectiva y que el resultado se exponga mediante el mismo patrón `aria-describedby` + mensaje de error.

## Conclusión

La solución cumple ampliamente la estructura solicitada y aplica correctamente la mayoría de patrones POUR: semántica HTML5, etiquetas explícitas, ayuda contextual, foco visible, enlace de salto y validación nativa. Antes de considerarla una implementación WCAG AA plenamente robusta, se recomienda resolver el límite real del archivo, gestionar el ciclo de vida de los `role="alert"` y simplificar los roles ARIA redundantes. Después, deben ejecutarse pruebas con teclado, lector de pantalla y medición automatizada de contraste en los estados normal, foco, error y hover.
