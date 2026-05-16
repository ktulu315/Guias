# Cheatsheet Rápido de CSS

Referencia rápida en orden progresivo de necesidad. CSS moderno orientado a UI.

## 1. Selectores y Especificidad

Los selectores determinan a qué elementos se aplican los estilos. La especificidad es el sistema de puntuación que decide qué regla gana cuando hay conflictos: **ID > Clase > Etiqueta**. En producción, prefiere clases sobre IDs.

- `*` → Selector universal: aplica a **todos** los elementos. Útil para resets, pero evítalo en selectores complejos porque es lento.
- `div p` → Descendiente: cualquier `p` dentro de un `div`, sin importar la profundidad.
- `div > p` → Hijo directo: solo `p` que son hijos inmediatos de `div`.
- `div + p` → Hermano adyacente: el primer `p` que sigue inmediatamente a un `div`.
- `div ~ p` → Hermano general: todos los `p` que siguen a un `div` (mismo padre).
- `.clase` → Clase: reutilizable, especificidad baja (0,1,0). Es la forma recomendada para la mayoría de los casos.
- `#id` → ID: especificidad muy alta (1,0,0). Evítalo en CSS — usa clases. Los IDs son para JavaScript.
- `[type="text"]` → Atributo: selecciona por atributo HTML. Útil para formularios y data attributes.
- `:hover`, `:focus`, `:active` → Pseudo-clases de estado interactivo.
- `:nth-child(2n)`, `:first-child`, `:last-child` → Seleccionan por posición en el padre. `nth-child(2n)` = pares, `nth-child(3)` = tercero.
- `:not(.excluido)` → Selecciona todo **excepto** lo que coincida con el selector dentro del paréntesis.
- `:is(header, main, footer) p` → Agrupa selectores: equivalente a `header p, main p, footer p` pero con la especificidad del selector más específico dentro de `:is()`.
- `:has(selector)` → Selector parental: selecciona un elemento **si contiene** un hijo que coincida. Ej: `.card:has(img)` selecciona cards que contengan imagen.
- `@media (hover: hover)` → Media query que detecta si el dispositivo soporta hover táctil. Útil para diferenciar móvil de desktop.

> **Decisión:** `:has()` para estilos condicionales del contenedor según su contenido (ej: un grid donde las cards con imagen se ven diferente). `:is()` para agrupar sin aumentar especificidad. Preferir **clases sobre IDs** (menor especificidad, más reutilizable). `:not()` útil para bordes o estilos que se aplican a todos excepto el último.

## 2. Box Model

Todo elemento en CSS es una caja compuesta por: **content → padding → border → margin**. El padding separa el contenido del borde, el margin separa la caja de los elementos vecinos.

- `box-sizing: border-box;` → Cambia el cálculo: con `border-box`, el `width` incluye **padding + border**. Sin él (valor default `content-box`), si pones `width: 100px` y `padding: 20px`, el elemento mide 140px. **Ponlo en el reset global** (`*, *::before, *::after { box-sizing: border-box; }`).
- `margin: auto;` → Centra horizontalmente un bloque con ancho definido. Solo funciona si el elemento tiene un `width` explícito y está en bloque.
- `padding: 8px 16px;` → Atajo: `8px` arriba/abajo, `16px` izquierda/derecha. También: `padding: top right bottom left` (sentido horario).
- `margin: 8px 0;` → Margen vertical. El margen vertical entre dos elementos se **colapsa** (se usa el mayor). El horizontal no.
- `overflow: hidden / auto / scroll / clip` → Controla qué pasa cuando el contenido desborda la caja. `hidden` corta, `auto` muestra scrollbar solo si es necesario, `scroll` siempre muestra. `clip` es como `hidden` pero sin desplazamiento programático.

> **Decisión:** Siempre usar `box-sizing: border-box` en el reset universal. `margin: auto` para centrar un bloque, pero **Flexbox** es mejor para centrar múltiples elementos o centrar verticalmente.

## 3. Colores y Fondos

CSS moderno soporta múltiples espacios de color. Los degradados se tratan como imágenes de fondo.

- `background: linear-gradient(45deg, #f06, #4af);` → Degradado lineal en diagonal (45°). El primer color (#f06) empieza en una esquina, el segundo (#4af) termina en la opuesta.
- `background: radial-gradient(circle at top left, #fff, #333);` → Degradado radial desde la esquina superior izquierda. Útil para efectos de iluminación o viñetas.
- `color: oklch(70% 0.2 150);` → **OKLCH**: espacio de color perceptual y uniforme. Más preciso que HSL para gradientes suaves y accesibles. El tercer valor (150) es el tono en grados.
- `color: rgb(255 0 0 / .5);` → Notación moderna RGB con separación por espacios y canal alpha. Más limpia que `rgba()`.
- `background-size: cover / contain;` → `cover` escala la imagen para cubrir todo el contenedor (parte se corta). `contain` escala para que la imagen completa se vea (puede dejar bordes vacíos).
- `background-attachment: fixed;` → El fondo se queda fijo al hacer scroll. Crea el efecto parallax. Consume más recursos.

## 4. Tipografía Moderna

La tipografía responsive se logra con `clamp()` sin necesidad de media queries. El valor de `line-height` sin unidad (ej: `1.5`) es relativo al `font-size` y es la forma recomendada.

- `font-size: clamp(1rem, 2.5vw, 2rem);` → Tamaño fluido: **mínimo 1rem**, **preferido 2.5vw** (se ajusta al viewport), **máximo 2rem**. La función elige el valor medio. Ej: en un viewport de 800px, 2.5vw = 20px ≈ 1.25rem. Como está entre 1rem y 2rem, usa 1.25rem.
- `text-wrap: balance;` → Distribuye el texto de un **título** para evitar viudas (palabras solas al final). Solo funciona en bloques con pocas líneas. No usar en párrafos largos.
- `text-wrap: pretty;` → Similar a `balance` pero con menos costo computacional. Adecuado para párrafos. Evita que la última línea tenga una sola palabra.
- `line-height: 1.5;` → Altura de línea. **Sin unidad** es relativo al font-size (1.5 = 150% del font-size). Es la forma recomendada porque hereda correctamente.
- `letter-spacing: -0.02em;` → Ajuste de espaciado entre caracteres. Negativo para titulares (más compacto), positivo para mejorar legibilidad en mayúsculas.
- `text-overflow: ellipsis;` → Muestra `...` cuando el texto desborda. **Requiere** `white-space: nowrap` y `overflow: hidden` para funcionar.
- `@font-face { font-family: 'X'; src: url('...') format('woff2'); }` → Declara una fuente personalizada. Usar formatos modernos (woff2).
- `font-display: swap;` → Muestra el texto con una fuente del sistema mientras la fuente personalizada carga. Evita el flash de texto invisible (FOIT). Mejora la percepción de velocidad.

> **Decisión:** `clamp()` para tipografía responsive sin media queries — p. ej. `clamp(0.9rem, 2.5vw, 1.5rem)` para body text, `clamp(1.5rem, 4vw, 2.5rem)` para h1. `text-wrap: balance` solo en titulares. Preferir `font-display: swap` siempre.

## 5. Flexbox

Flexbox es un sistema de layout **unidimensional** (trabaja en una dirección: fila O columna). Ideal para distribuir espacio entre items, centrar contenido, y crear componentes como navbars.

- `display: flex;` → Activa flexbox en el contenedor. Sus hijos se convierten en **flex items** por defecto en fila.
- `flex-direction: row / column;` → Define el **eje principal**. `row` (default) = horizontal, `column` = vertical.
- `justify-content: center / space-between / space-around / flex-start / flex-end;` → Alinea en el **eje principal**. `space-between` separa items con el espacio sobrante entre ellos (sin espacio en los bordes). `center` los centra.
- `align-items: center / flex-start / stretch;` → Alinea en el **eje cruzado** (perpendicular al principal). `stretch` (default) estira los items para llenar la altura. `center` centra verticalmente.
- `flex-wrap: wrap;` → Permite que los items salten a la siguiente línea si no caben. Sin esto (default `nowrap`), los items se encogen.
- `gap: 8px;` → Espaciado entre items. Reemplaza la necesidad de `margin` en los hijos. No se colapsa. **Soporte completo en navegadores modernos.**
- `flex: 1;` → Atajo para `flex-grow: 1; flex-shrink: 1; flex-basis: 0;`. Hace que el item ocupe el espacio disponible proporcionalmente.
- `align-self: center;` → Permite que un item individual se alinee diferente al resto. Anula `align-items` del contenedor para ese item.

> **Decisión:** Flexbox para layouts **1D** (navbar, fila de cards, centrar un hijo). Para layouts de rejilla (filas Y columnas a la vez), usar Grid. `gap` reemplaza `margin` en los hijos — úsalo siempre que puedas.

<div style="display:flex;gap:8px;padding:12px;background:#f5f5f5;border-radius:8px;">
  <div style="flex:1;background:#4af;padding:16px;border-radius:4px;color:#fff;text-align:center;">1</div>
  <div style="flex:1;background:#4af;padding:16px;border-radius:4px;color:#fff;text-align:center;">2</div>
  <div style="flex:1;background:#4af;padding:16px;border-radius:4px;color:#fff;text-align:center;">3</div>
</div>

## 6. Grid

Grid es un sistema de layout **bidimensional** (filas Y columnas simultáneamente). Ideal para layouts de página completos, galerías, dashboards y cualquier estructura de rejilla.

- `display: grid;` → Activa grid en el contenedor. Sus hijos se colocan automáticamente en celdas.
- `grid-template-columns: 1fr 1fr 1fr;` → Tres columnas de igual ancho. `1fr` = una fracción del espacio disponible. También puedes mezclar: `200px 1fr 1fr`.
- `grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));` → Grid **responsive sin media queries**: crea tantas columnas de mínimo 250px como quepan, y cada una se estira hasta 1fr. `auto-fill` vs `auto-fit`: `auto-fit` colapsa columnas vacías, `auto-fill` las mantiene.
- `grid-template-rows: auto 1fr auto;` → Layout clásico de página: header (auto), contenido (flexible), footer (auto). La fila con `1fr` ocupa todo el espacio restante.
- `grid-column: span 2;` → El item ocupa 2 columnas. También puedes usar líneas: `grid-column: 1 / 3;` (de línea 1 a línea 3).
- `gap: 16px;` → Espaciado entre filas y columnas. No se necesita margin en los hijos.
- `place-items: center;` → Atajo para `align-items: center; justify-items: center;`. Centra todos los items en sus celdas.

> **Decisión:** Grid para layouts **2D** (galerías, dashboard, página completa). Si solo necesitas una fila o columna (navbar, fila de botones), usa Flexbox. `minmax()` + `auto-fill` es la técnica más potente para grids responsive.

## 7. Posicionamiento

El posicionamiento saca elementos del flujo normal del documento o los fija en relación a un contenedor. Solo los elementos posicionados (no `static`) aceptan `z-index`.

- `position: relative;` → El elemento mantiene su lugar en el flujo pero ahora es **punto de ancla** para hijos absolutos. También permite moverlo con `top`/`left` desde su posición original sin afectar a los vecinos.
- `position: absolute;` → **Fuera del flujo**: no ocupa espacio. Se posiciona relativo al **ancestro posicionado más cercano** (el primer `relative/absolute/fixed/sticky` que encuentre hacia arriba). Si no hay ninguno, usa el `<body>`.
- `position: fixed;` → Fijo al **viewport**: no se mueve al hacer scroll. Ideal para headers superiores, barras laterales fijas, o botones flotantes.
- `position: sticky;` → Híbrido entre `relative` y `fixed`: el elemento se comporta como `relative` hasta que al hacer scroll alcanza un límite definido (`top: 0`), entonces se pega como `fixed`. Requiere un `top`/`bottom` para activarse.
- `inset: 0;` → Atajo moderno para `top: 0; right: 0; bottom: 0; left: 0;`. Útil para que un absoluto cubra todo su contenedor padre. Soporte: Chrome 87+, Firefox 66+, Safari 14.1+, Edge 87+.
- `z-index: 10;` → Controla qué elemento se superpone a cuál. Solo funciona en **elementos posicionados** (no `static`). Mayor valor = más al frente. Sin z-index, el orden en el HTML decide.

> **Decisión:** `sticky` para headers de tabla, navs o índices laterales. `absolute` para tooltips, modales o badges. `fixed` para overlays, barras o botones flotantes. Siempre asegúrate de que el padre tenga `position: relative` cuando uses `absolute` en el hijo.

## 8. Bordes y Sombras

Los bordes y sombras son la base del diseño visual en CSS. Con `border-radius` puedes transformar rectángulos en círculos. `box-shadow` puede crear sombras, anillos y múltiples efectos superpuestos.

- `border: 1px solid #ddd;` → Borde estándar: **ancho estilo color**. `solid` es el más común. También `dashed`, `dotted`, `double`.
- `border-radius: 8px;` → Redondea las esquinas. A mayor valor, más redondeadas.
- `border-radius: 999px;` → Valor muy alto = forma de píldora. Para hacer un **círculo**, usa `border-radius: 50%` + `width` y `height` iguales, o simplemente `border-radius: 999px` con `padding` igual horizontal y vertical.
- `box-shadow: 0 2px 8px rgb(0 0 0 / .15);` → Sombra suave. El orden: **offset-x offset-y blur-radius color**. Para sombras múltiples, sepáralas con coma.
- `box-shadow: 0 0 0 2px #4af;` → **Anillo** sin offset ni blur. Útil para indicar foco o selección activa sin usar `outline`.
- `outline: 2px solid #4af; outline-offset: 2px;` → Borde externo que **no afecta el layout** (no suma al box model). Ideal para indicar foco en accesibilidad. `outline-offset` lo separa del elemento.
- `filter: drop-shadow(0 4px 4px rgb(0 0 0 / .3));` → Sombra que **respeta la forma** del elemento, incluyendo transparencias. Si tienes un PNG con alpha o un `clip-path`, `drop-shadow` sombrea el contorno real, no la caja.

> **Decisión:** `box-shadow` para sombras de caja estándar. `drop-shadow` cuando el elemento tiene transparencia (PNG, clip-path, mask). `outline` para accesibilidad (`:focus-visible`). `border-radius: 999px` para badges/píldoras.

## 9. Transiciones

Las transiciones permiten **cambiar suavemente** de un estado a otro (como hover). Solo funcionan cuando hay un cambio de estado (pseudo-clase, clase añadida, etc.). Prefiere transicionar `opacity` y `transform` por razones de rendimiento.

- `transition: all 0.3s ease;` → Transiciona **todas** las propiedades que cambien. **No recomendado** porque puede causar transiciones inesperadas y peor rendimiento.
- `transition: transform 0.2s ease, opacity 0.2s ease;` → Transiciona **propiedades específicas**. Más predecible y mejor rendimiento. El navegador solo calcula lo necesario.
- `transition-timing-function: cubic-bezier(0.68, -0.55, 0.27, 1.55);` → Curva personalizada. `ease`, `ease-in`, `ease-out`, `linear` son valores predefinidos. `cubic-bezier()` permite crear curvas con efecto **rebote** (cuando la curva supera 1 en el eje Y).
- `transition-duration: 0.3s;` → Duración en segundos. 200-300ms es el rango ideal para UI. Menos de 100ms es imperceptible, más de 500ms se siente lento.

> **Rendimiento:** `opacity` y `transform` se pueden transicionar en la GPU (no afectan el layout). `width`, `height`, `top`, `left`, `margin`, `padding` fuerzan recálculo de layout — evítalos. `transition: all` es cómodo pero malo para rendimiento y predecibilidad.

<div style="transition: transform 0.2s ease, box-shadow 0.2s ease;display:inline-block;padding:8px 16px;background:#4af;color:#fff;border-radius:6px;cursor:pointer;" onmouseover="this.style.transform='scale(1.1)';this.style.boxShadow='0 4px 12px rgba(0,0,0,.3)'" onmouseout="this.style.transform='scale(1)';this.style.boxShadow='none'">Hover me</div>

## 10. Animaciones

Las animaciones (`@keyframes`) permiten secuencias de estilos más complejas que las transiciones: múltiples pasos, loops, direcciones alternadas, y control independiente de cada propiedad.

- `@keyframes slide { 0% { opacity: 0; transform: translateX(-20px); } 100% { opacity: 1; transform: translateX(0); } }` → Define una animación reutilizable. Los nombres son arbitrarios. Puedes usar `from` (0%) y `to` (100%) como atajo.
- `animation: slide 0.5s ease forwards;` → Aplica la animación. Orden: **nombre duración timing-function fill-mode**. `forwards` mantiene el estado final después de la animación. Sin `forwards`, el elemento vuelve al estado inicial.
- `animation-iteration-count: infinite;` → Repite la animación sin fin. También puedes usar un número (2, 3, etc.).
- `animation-direction: alternate;` → Alterna dirección: va de ida y vuelta. Combinado con `infinite` crea un efecto de rebote suave.
- `animation-timing-function: ease-in-out;` → La curva de la animación. `ease-in-out` empieza lento, acelera, termina lento. Es la más natural para UI.
- `animation-delay: 0.5s;` → Espera antes de empezar. Útil para animaciones encadenadas (stagger).
- `@media (prefers-reduced-motion: reduce)` → Media query para respetar la preferencia del usuario por movimiento reducido. Dentro de ella: `*, *::before, *::after { animation: none; transition: none; }`. **Obligatorio para accesibilidad.**

> **Transiciones vs Animaciones:** Las transiciones son ideales para cambios de estado simples (hover, focus). Las animaciones son para secuencias más complejas (entrada de elementos, loops, múltiples pasos). Siempre incluir `prefers-reduced-motion`.

## 11. Transformaciones

`transform` modifica la **forma y posición** del elemento sin afectar el layout de los elementos vecinos. No saca el elemento del flujo (como sí hace `position: absolute`). Al operar en la GPU, es muy eficiente para animaciones.

- `transform: scale(1.1);` → Escala el elemento. `scale(1.5)` = 150% del tamaño original. `scale(0.5)` = 50%. Se puede escalar solo en X o Y: `scaleX(1.2)`.
- `transform: rotate(45deg);` → Rota el elemento. Acepta `deg` (grados), `turn` (1turn = 360°), `rad` (radianes).
- `transform: translateX(20px);` → Mueve horizontalmente. `translateY()` vertical. `translate(x, y)` ambas.
- `transform: translate(-50%, -50%);` → Técnica de centrado: combinado con `position: absolute; top: 50%; left: 50%;`, centra perfectamente el elemento sin importar su tamaño.
- `transform: skew(10deg);` → Inclina/deforma el elemento en el eje X. `skewY()` para el eje Y.
- `transform-origin: top left;` → Define el **punto de origen** de la transformación. Por defecto es `center center`. Cambiarlo a `top left` hace que `scale` y `rotate` partan desde la esquina.

> **Decisión:** Para centrar: `position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);`. Es más robusto que margen negativo. Preferir `transform` para animaciones de posición (mejor rendimiento que cambiar `top`/`left`).

## 12. Responsive

El diseño responsive se logra combinando media queries, unidades relativas y contenedores flexibles. Las **container queries** son el reemplazo moderno de las media queries para componentes reutilizables.

- `@media (max-width: 768px) { ... }` → Punto de quiebre tradicional. Se activa cuando el viewport mide 768px o menos. Flujo de trabajo: primero escribe el CSS para móvil (mobile-first), luego añades `@media (min-width: ...)` para pantallas grandes.
- `@media (prefers-color-scheme: dark) { ... }` → Detecta si el sistema operativo está en modo oscuro. Útil para temas: cambia colores, sombras y fondos sin JavaScript.
- `@media (hover: none) and (pointer: coarse) { ... }` → Detecta táctil (móvil/tablet). Útil para agrandar botones o eliminar hover effects en táctil.
- `container-type: inline-size;` → Declara un **contenedor** para container queries. El elemento ahora es un contexto de consulta.
- `@container (min-width: 400px) { ... }` → Se activa cuando el **contenedor** mide 400px o más, no el viewport. Ideal para componentes que deben adaptarse al espacio disponible, no al tamaño de la pantalla.
- `width: clamp(300px, 80%, 1200px);` → Ancho fluido: mínimo 300px, preferido 80% del contenedor, máximo 1200px. Sin media queries.
- `height: 100dvh;` → **Dynamic Viewport Height**. En móviles, `100vh` incluye la barra de navegación del navegador (que se oculta al hacer scroll). `100dvh` se ajusta dinámicamente. Soporte: Chrome 108+, Safari 15.4+.

> **Decisión:** `@media` para layout general (sidebar, header, número de columnas). `@container` para componentes reutilizables (cards, widgets) que deben adaptarse a su contenedor. `clamp()` para tamaños fluidos. `100dvh` para altura completa en móvil.

## 13. Efectos UI Modernos

Efectos visuales avanzados como glassmorphism (cristal) se logran con `backdrop-filter`. Las formas irregulares con `clip-path`. Los filtros afectan la representación visual del elemento como si fuera una imagen.

- `filter: blur(4px);` → Desenfoca el elemento. Útil para fondos de modales o estados "temporalmente deshabilitados". Valores típicos: 2-8px.
- `filter: brightness(1.2);` → Ajusta el brillo. `1.0` = normal, `0.5` = 50% oscuro, `1.5` = 150% brillante. Acepta también porcentajes: `brightness(150%)`.
- `backdrop-filter: blur(8px);` → Desenfoca lo que está **detrás** del elemento. Efecto vidrio/glassmorphism. Requiere que el fondo detrás tenga contenido visible. Combinar con fondo semitransparente: `background: rgb(255 255 255 / .2)`.
- `mix-blend-mode: multiply / overlay / difference;` → Mezcla los colores del elemento con los de detrás. `difference` invierte colores (útil para textos que se adaptan al fondo).
- `clip-path: polygon(50% 0%, 100% 100%, 0% 100%);` → Recorta el elemento en una forma arbitraria. Herramientas: https://bennettfeely.com/clippy/. También acepta `circle()`, `ellipse()`, `inset()`.
- `mask-image: linear-gradient(black, transparent);` → Máscara de opacidad: donde el gradiente es negro, el elemento se ve; donde es transparente, se oculta. Útil para desvanecer bordes de imágenes.
- `-webkit-backdrop-filter: blur(8px);` → Prefijo necesario para **Safari** (versiones antiguas). Ponlo siempre junto a `backdrop-filter` sin prefijo.

> **Decisión:** `backdrop-filter` para glassmorphism (modales, navs, cards). `clip-path` para formas decorativas (triángulos, hexágonos, diagonales en headers). `mask-image` para desvanecer bordes de imágenes. `mix-blend-mode: difference` para UI que se adapta dinámicamente al fondo.

<div style="backdrop-filter:blur(8px);-webkit-backdrop-filter:blur(8px);background:rgb(255 255 255 / .3);border:1px solid rgb(255 255 255 / .5);padding:16px;border-radius:12px;color:#333;max-width:300px;">
Glassmorphism effect
</div>

## 14. Variables CSS

Las **custom properties** (variables CSS) permiten reutilizar valores y crear temas dinámicos. A diferencia de las variables de preprocesadores (Sass/Less), las CSS nativas viven en el DOM y se pueden cambiar con JavaScript, media queries, o herencia.

- `:root { --primary: #4af; --spacing: 8px; }` → Define variables a nivel global (`:root` es equivalente a `<html>` pero con mayor especificidad). Los nombres deben empezar con `--`.
- `color: var(--primary);` → Usa una variable. El navegador busca el valor hacia arriba en el DOM (herencia). Si la variable no está definida, la propiedad se ignora (no cascada).
- `color: var(--primary, #08f);` → Variable con **fallback**: si `--primary` no está definida, usa `#08f`. Importante para componentes reutilizables.
- `@property --angle { syntax: '<angle>'; inherits: false; initial-value: 0deg; }` → **Variable tipada** (Chrome 85+). Declara el tipo de datos (`<angle>`, `<color>`, `<length>`, etc.). Permite que CSS **anime** la transición entre valores, algo que `var()` normal no puede hacer. Útil para animar el ángulo de un `conic-gradient`.

> **Decisión:** Usar variables para **temas** (colores primario/secundario, espaciado, radios de borde, sombras). Facilita cambiar la apariencia completa del sitio desde un solo lugar. Preferir `var(--color, fallback)` con fallback en componentes compartidos. `@property` para animar valores que CSS no puede interpolar por defecto.
