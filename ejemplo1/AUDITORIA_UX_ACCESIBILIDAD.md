# Auditoría de UX y Accesibilidad — Proyecto ejemplo1

- **Fecha de auditoría:** 04 de septiembre de 2026
- **Archivos auditados:** `ejemplo1/index.html`, `ejemplo1/styles.css` (el CSS es un archivo minificado en una sola línea)
- **Ámbito:** navegación, responsive design, jerarquía visual, legibilidad, contraste WCAG, foco de teclado, semántica, textos alternativos, reducción de movimiento y experiencia móvil.
- **Método:** revisión manual de código y cálculo de contraste según algoritmo WCAG (fórmula relativa de luminancia + ratio). No se ejecutaron validadores automatizados ni pruebas en navegador (ver sección «No verificado»).

---

## 1. Resumen ejecutivo

La página presenta una base de accesibilidad sólida y superior al promedio: incluye *skip link* funcional, aterrizaje con `lang="es"`, landmarks correctos, jerarquía de encabezados impecable, textos alternativos apropiados, orden de foco lógico, `prefers-reduced-motion` y contraste correcto en los textos principales.

Sin embargo, los problemas más relevantes están concentrados en dos áreas:

1. **Contraste del color de acento azul (`--blue: #4aa6de`)** sobre fondos claros, que falla WCAG AA en texto pequeño (etiquetas de sección, años del palmarés, «10» de la marca).
2. **Indicador de foco de teclado casi invisible** en fondos claros: el anillo `:focus-visible` usa el mismo amarillo que el fondo de los botones y contrasta solo ~1.45:1 sobre blanco/`--paper`.

No se detectaron defectos bloqueantes de uso: la navegación es lineal y funcional, el layout se adapta a 1 columna en móvil, y los estados de foco visibles sí son informales pero legibles. Hay 2 hallazgos de severidad **alta**, 2 de severidad **media** y 4 de severidad **baja**, más refuerzos de mantenimiento.

| Severidad | Cantidad |
|---|---|
| Alta | 2 |
| Media | 2 |
| Baja | 4 |

---

## 2. Hallazgos por severidad

### 2.1 Severidad Alta

#### H-01 — Contraste insuficiente del color de acento azul sobre fondos claros (WCAG 1.4.3 AA)

- **Ubicación exacta:**
  - `styles.css` → selector `.section-label{color:var(--blue)}` (etiquetas «01 / Biografía», «02 / En cifras», «03 / Palmarés»). `color: #4aa6de` (var `--blue`).
  - `styles.css` → selector `.milestones span{color:var(--blue)}` (años «2004», «2021», «2022», «2023» en el palmarés).
  - `styles.css` → `.brand span{color:var(--blue)}` (el «10» de la marca).
  - Fondos implicados: `--paper (#f7f5ef)` en biografía y encabezado, y `#ffffff` en la sección Palmarés.
- **Contraste medido (WCAG):**
  - `#4aa6de` sobre `#f7f5ef` → **2,47:1**.
  - `#4aa6de` sobre `#ffffff` → **2,69:1**.
  - Umbral AA para texto pequeño (los labels miden `0.75rem`, uppercase): **4,5:1**. Umbral AA para texto grande: **3:1**.
- **Impacto para usuarios:** las etiquetas de sección, los años del palmarés y el «10» de la marca son prácticamente ilegibles o muy poco perceptibles para personas con baja visión, daltonismo o monitores de baja gama. Al tratarse de texto con significado (títulos numerados, fechas), falla el criterio AA.
- **Recomendación concreta:** oscurecer el azul de acento. Un tono como `#2b6e96` mantiene la identidad visual y supera 4,5:1 sobre blanco y `--paper`. Si se desea conservar `#4aa6de`, reservar ese tono solo para elementos no textuales (bordes, íconos) y usar el tono oscuro (`--blue-strong`) para todo el texto. Verificar el nuevo valor con la calculadora de contraste antes de aplicar.

#### H-02 — Indicador de foco de teclado invisible sobre fondos claros y sobre el botón (WCAG 2.4.7)

- **Ubicación exacta:**
  - `styles.css` → regla global `a:focus-visible{outline:3px solid var(--yellow);outline-offset:4px}`.
  - Afecta a: enlaces de `nav` en el encabezado, enlaces de la sección Palmarés, enlace del pie (`figcaption a`), enlace «Conocer su historia` (`.button`, fondo `--yellow`) y el propio `.skip-link` (fondo `--yellow`).
- **Problema:** el anillo de foco usa el mismo amarillo que los fondos sobre los que se dibuja.
  - Amarillo `#f8c742` sobre `--paper (#f7f5ef)` → **1,45:1** (prácticamente invisible).
  - Amarillo `#f8c742` sobre `#ffffff` → **1,59:1** (invisible).
  - Sobre el botón `.button` (fondo `--yellow`) el outline es amarillo sobre amarillo → sin distinción.
  - Sí funciona sobre el fondo oscuro del `.hero` (fondo `--ink`), única zona donde se percibe bien.
- **Impacto para usuarios:** para usuarios de teclado, comandos por voz o con movilidad reducida es imposible saber qué enlace tiene el foco al recorrer el encabezado, la navegación o el botón principal; pueden activar el elemento equivocado.
- **Recomendación concreta:** sustituir el único anillo amarillo por un indicador de doble tono con contraste en ambos fondos, por ejemplo:
  ```css
  a:focus-visible{outline:3px solid var(--ink);outline-offset:2px;box-shadow:0 0 0 6px var(--yellow)}
  ```
  Esto garantiza una corona oscura visible sobre fondos claros y una huella amarilla visible sobre fondos oscuros/amarillo. Ajustar también el `.button:focus-visible` y `.skip-link:focus` (su fondo es amarillo) para que hereden el mismo indicador. Verificar que el anillo tenga al menos 2 px y un contraste ≥ 3:1 contra el fondo adyacente.

---

### 2.2 Severidad Media

#### M-01 — Etiqueta de sección amarilla sobre el fondo azul de estadísticas (WCAG 1.4.3 AA)

- **Ubicación exacta:** `styles.css` → `.stats-section .section-label{color:var(--yellow)}`; fondo de la sección `.stats-section{background:#276c99}`. Uso en `index.html` (etiqueta «02 / En cifras»).
- **Contraste medido:** `#f8c742` sobre `#276c99` → **3,59:1**. Texto pequeño (`0.75rem`) → requiere **4,5:1**.
- **Impacto:** el rótulo de la sección «En cifras» es difícil de leer para personas con baja visión; al ser la única etiqueta de sección que no cumple, rompe la coherencia visual además de no pasar el criterio.
- **Recomendación concreta:** usar texto blanco (`#ffffff`, 5,69:1) para el label sobre el azul, o un amarillo más claro como `#ffe27a` (supera 4,5:1). El amarillo actual puede conservarse como acento decorativo del dígito del `:before`, si se decide mantener.

#### M-02 — Área táctil insuficiente de los enlaces de navegación en móvil (WCAG 2.5.8 / convención móvil 44×44 px)

- **Ubicación exacta:** `index.html` → `nav` interior (Biografía, Estadísticas, Palmarés); `styles.css` → `nav{...gap...}` sin `padding` vertical en `nav a`.
- **Problema:** los enlaces no tienen `padding`, por lo que su área de toque/hit box es la de la línea de texto (≈ 24–26 px de alto), muy por debajo del mínimo recomendado de **44×44 px** en táctil. A partir de `max-width:390px` el gap se reduce a `.6rem` y el riesgo de pulsar el enlace contiguo aumenta.
- **Impacto:** usuarios móviles con temblor de manos, baja motricidad fina o pantallas pequeñas activan el enlace erróneo o requieren varios intentos; también dificulta el uso con lápiz.
- **Recomendación concreta:** dar `padding:.55rem .6rem` a `nav a` (y, en general, un área mínima de 44×44, p. ej. con `min-height:44px; min-width:44px` centrado), y mantener al menos 8 px de separación entre enlaces. Esto además mejora la usabilidad general en escritorio.

---

### 2.3 Severidad Baja

#### B-01 — Enlaces que dependen únicamente del color (WCAG 1.4.1)

- **Ubicación exacta:**
  - `styles.css` → `nav a{color:var(--muted)}` sin subrayado (solo `text-underline-offset` preparado).
  - `index.html` / `styles.css` → `figcaption a{color:#fff}` en el pie de la imagen: el resto del caption es `#bec7d0`, por lo que el enlace solo se diferencia por el color blanco.
- **Impacto:** en el pie de foto, un usuario con daltonismo (deficiencias rojo-verde o de saturación) no distingue el enlace «Wikimedia Commons» del texto del caption.
- **Recomendación concreta:** aplicar `text-decoration:underline` (o `underline` permanente) a `figcaption a` y opcionalmente a `nav a`; el subrayado hace el enlace reconocible sin depender del color. En el nav el riesgo es bajo por estar en una barra de navegación, pero el subrayado en el figcaption es lo prioritario.

#### B-02 — Contenido decorativo emitido por CSS que algún lector de pantalla podría anunciar

- **Ubicación exacta:** `styles.css` → `.portrait:before{content:"10";...position:absolute;...}` (el «10» gigante decorativo del fondo del retrato).
- **Impacto:** algunos lectores de pantalla (con soporte de pseudoelementos) pueden anunciar el «10» fuera de contexto junto a la imagen, generando confusión; el número no tiene función informativa.
- **Recomendación concreta:** mover el «10» decorativo al HTML como `<span aria-hidden="true" class="portrait-badge">10</span>` dentro del `<figure>` y posicionarlo con CSS, o eliminar el `content` por CSS y usar `aria-hidden` explícito. Como mínimo, si se conserva el pseudoelemento, se debe confirmar con un lector real qué anuncia.

#### B-03 — Posible saturación del encabezado en pantallas de 320 px o menores

- **Ubicación exacta:** `index.html` → `.navigation` (marca «LM 10» + tres enlaces «Biografía | Estadísticas | Palmarés»); `styles.css` → `@media(max-width:390px){nav{gap:.6rem;font-size:.85rem}}`.
- **Problema:** con un ancho real de 320 px el contenido (marca ~45 px + ~245 px de enlaces + 32 px de márgenes del contenedor) queda al límite; el `flex-wrap:wrap` del `nav` hace que los enlaces bajen a una segunda línea, estirando el encabezado y reduciendo el área útil del hero.
- **Impacto:** en dispositivos muy pequeños el encabezado se ve apretado y puede provocar reflow del hero hacia abajo; es más un problema de experiencia visual que funcional.
- **Recomendación concreta:** probar en un viewport de 320 px; considerar reducir el `font-size` de la marca en ese rango, reducir el gap a `.4rem`, o agrupar los enlaces con `font-size:.82rem`. No se recomienda ocultar enlaces: el submenú hamburguesa añadiría complejidad innecesaria para una sola línea.

#### B-04 — Arquitectura tipográfica que no se renderiza como se proyectó (observación de fidelidad)

- **Ubicación exacta:** `styles.css` → `body{font-family:Inter,...}` y `font-weight:850` en `.stats dt`.
- **Problema:** la fuente **Inter no está cargada** (no hay `@font-face`, ni enlace a Google Fonts, ni `@import`), por lo que el sitio se verá siempre con la fuente del sistema (`Segoe UI`, etc.). El `font-weight:850` (valor intermedio no soportado por todas las familias) puede redondearse, cambiando la apariencia de las cifras.
- **Impacto:** no es un fallo de accesibilidad en sí mismo, pero la legibilidad prevista y la identidad visual dependen de un fallback que puede variar según plataforma; los pesos «fake» pueden producir texto más grueso o irregular.
- **Recomendación concreta:** cargar Inter con `@font-face`/Google Fonts (con `font-display:swap`) si el diseño lo requiere, o quitar «Inter» de la lista para confiar deliberadamente en las fuentes del sistema; y usar un peso estándar (800) en `.stats dt`.

---

### 2.4 Observación de mantenimiento (no aplica como fallo de accesibilidad)

- `ejemplo1/index backup.html` duplica el contenido de la página principal. Riesgo de que una revisión futura actualice solo uno de los archivos y se publiquen versiones divergentes. Se recomienda eliminar el respaldo o moverlo fuera de la raíz servida.

---

## 3. Aspectos verificados sin problemas

- **Idioma y metadatos:** `lang="es"`, `charset="utf-8"`, `viewport` con `width=device-width, initial-scale=1` (permite zoom), `title` descriptivo y `meta description` presente.
- **Inicio de contenido:** `skip-link` es el primer elemento enfocable, se oculta mediante desplazamiento fuera de pantalla (`top:-4rem`) y **se hace visible al recibir el foco** (`top:1rem`), con contraste de texto correcto (tinta sobre amarillo = 11,13:1). El destino `#contenido` corresponde al `<main>`.
- **Semántica y landmarks:** estructura correcta de `header > nav`, `main`, `footer`; el `nav` tiene `aria-label="Navegación principal"`; no hay landmarks duplicados ni contenido interactivo fuera del `main`.
- **Jerarquía de encabezados:** un único `h1`; jerarquía `h1 → h2 → h3` sin saltos de nivel; cada `section` está etiquetada con su `h2` mediante `aria-labelledby`. El listado de logros usa `ol/li` apropiado y el bloque de estadísticas usa `dl/dt/dd` bien formado.
- **Textos alternativos:** la imagen del retrato tiene `alt` descriptiva y precisa (jugador de la selección argentina, contexto del campo); la flecha «↓» del botón lleva `aria-hidden="true"` correctamente; la marca «LM 10» expone `aria-label` en el enlace raíz.
- **Renders que evitan cambios de layout (CLS):** la imagen declara `width="1200" height="800"` y `fetchpriority="high"` sobre el elemento LCP; el CSS fija el recorte con `object-fit`, por lo que el espacio está reservado desde el primer pintado.
- **Contraste correcto (WCAG AA, ≥ 4,5:1):**
  - Texto del botón (tinta `#121923` sobre amarillo `#f8c742`) → **11,13:1**.
  - Texto del hero (blanco sobre `#121923`) → **17,66:1**; eyebrow amarillo sobre tinta → **11,13:1**; intro `#d7dde2` sobre tinta → **12,89:1**; pie de imagen `#bec7d0` sobre tinta → **10,32:1**.
  - Enlaces del `nav` (`--muted` sobre `--paper`, fondo casi opaco `rgba(247,245,239,.94)`) → **5,13:1**.
  - Cifras del `.stats dt` (blanco sobre `#276c99`) → **5,69:1**; descripciones y nota de cifras (`#d9edf9`) → **4,72:1**.
  - Texto del pie de página (`--muted` sobre `--paper`) → **5,13:1**.
- **Reducción de movimiento:** `@media (prefers-reduced-motion: reduce){html{scroll-behavior:auto}}` implementado para el `scroll-behavior:smooth` de la base, que es la única animación del sitio.
- **Orden de foco:** el orden de tabulación coincide con el orden visual (skip-link → marca → nav → hero → secciones). No hay `tabindex` ni `aria-hidden` que rompan el flujo.
- **Responsive/móvil:** layout principal de dos columnas que conmuta a **una columna a ≤720 px**; el `stats` pasa a 2 columnas (≤720) y a 1 columna (≤390); la malla de logros reduce su columna de años a 320 px; los paddings/márgenes usan `clamp()` y unidades fluidas; el `overflow:hidden` del hero evita scroll horizontal por el gran «10» decorativo. El layout fluido debería soportar zoom al 200 % y viewports menores sin cortar contenido.
- **Interacción por teclado del botón:** `.button` es un `<a>` semánticamente correcto (navega a una sección), con área de pulsación ≈ 47 px y hover que mantiene contraste.

---

## 4. No verificado

No fue posible verificar empíricamente lo siguiente en esta auditoría; se requiere una segunda pasada con herramientas o navegador:

- **Validador automatizado:** no se ejecutó axe-core, WAVE ni Lighthouse. Los cálculos de contraste provienen de las fórmulas WCAG aplicadas a los valores CSS; **cualquier texto superpuesto a la imagen (color `#fff`/`#d7dde2` sobre fotografías) no fue verificado** porque la imagen proviene de una URL remota y su contenido visual no es analizable estáticamente.
- **Pruebas con lectores de pantalla reales** (NVDA, VoiceOver, TalkBack, Narrator) para confirmar el anuncio de pseudoelementos (B-02), el flujo de landmarks y el orden de lectura del `dl`.
- **Comportamiento en navegador:** rendimiento real (LCP/CLS), experiencia del scroll suave, estado del `skip-link` en foco y la carga/distponibilidad de la imagen remota de Wikimedia Commons (fuera de control del proyecto; en caso de caída, quedaría un hueco en el hero y el `figcaption` seguirá visible).
- **Zoom al 200 % y redimensión de ventana en dispositivos reales:** el análisis responsive se realizó sobre los media queries y el flex/grid declarados, no en un viewport ejecutado.
- **Fuente Inter:** al no estar cargada, el peso real de los textos y su legibilidad en pantalla dependen del sistema; se asumió la fuente del sistema a efectos de métricas.

---

*Informe generado el 04/09/2026. La corrección de los hallazgos H-01, H-02 y M-01 debe re-verificarse con la calculadora de contraste oficial (webaim.org/resources/contrastchecker o similar) una vez elegidos los nuevos colores.*