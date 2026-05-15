# Cheatsheet Rápido de CSS

Referencia rápida en orden progresivo de necesidad. CSS moderno orientado a UI.

## 1. Selectores y Especificidad

- `*` → Selector universal (todos los elementos)
- `div p` → Descendiente (p dentro de div)
- `div > p` → Hijo directo
- `div + p` → Hermano adyacente
- `div ~ p` → Hermano general
- `.clase` → Clase
- `#id` → ID
- `[atributo=valor]` → Atributo exacto
- `:hover`, `:focus`, `:active` → Estados interactivos
- `:nth-child(n)`, `:first-child`, `:last-child` → Posición en el padre
- `:not(selector)` → Negación
- `:is(selector1, selector2)` → Agrupar selectores con misma especificidad
- `:has(selector)` → Padre que contiene un hijo (selector parental)
- `@media (hover: hover)` → Detecta si el dispositivo soporta hover

> **¿Cuándo usar qué?** `:has()` para estilos condicionales de un contenedor según su contenido. `:is()` para agrupar y reducir repetición. Preferir clases sobre IDs (menor especificidad, más reutilizable).

## 2. Box Model

- `box-sizing: border-box;` → `width` incluye padding y borde (poner en el reset global)
- `margin: auto;` → Centrar horizontalmente un bloque con ancho definido
- `padding: 8px 16px;` → Espaciado interno (vertical horizontal)
- `margin: 8px 0;` → Espaciado externo (vertical horizontal)
- `overflow: hidden / auto / scroll / clip` → Controlar desbordamiento

> **¿Cuándo usar qué?** `box-sizing: border-box` en el `*` siempre. `margin: auto` para centrar un bloque; `flexbox` para centrar múltiples elementos.

## 3. Colores y Fondos

- `background: linear-gradient(45deg, #f06, #4af);` → Degradado diagonal
- `background: radial-gradient(circle, #fff, #333);` → Degradado radial
- `color: oklch(70% 0.2 150);` → Espacio de color OKLCH (perceptual, moderno)
- `color: rgb(255 0 0 / .5);` → RGB con separación por espacios y opacidad
- `background-size: cover / contain;` → Ajustar imagen de fondo
- `background-attachment: fixed;` → Fondo fijo al hacer scroll

## 4. Tipografía Moderna

- `font-size: clamp(1rem, 2.5vw, 2rem);` → Tamaño fluido (mín, preferido, máx)
- `text-wrap: balance;` → Evita viudas en titulares
- `text-wrap: pretty;` → Similar a balance pero menos costo computacional
- `line-height: 1.5;` → Altura de línea (sin unidad = relativo al font-size)
- `letter-spacing: -0.02em;` → Tracking para titulares
- `text-overflow: ellipsis;` → Puntos suspensivos en texto truncado
- `white-space: nowrap; + overflow: hidden;` → Necesario para ellipsis
- `@font-face { font-family: 'X'; src: url('...'); }` → Fuente personalizada
- `font-display: swap;` → Mostrar texto con fuente del sistema mientras carga

> **¿Cuándo usar qué?** `clamp()` para tipografía responsive sin media queries. `text-wrap: balance` solo en titulares (no en párrafos largos).

## 5. Flexbox

- `display: flex;` → Contenedor flexible (1D: fila o columna)
- `flex-direction: row / column;` → Eje principal
- `justify-content: center / space-between / space-around / flex-start / flex-end;` → Eje principal
- `align-items: center / flex-start / stretch;` → Eje cruzado
- `flex-wrap: wrap;` → Permitir salto de línea
- `gap: 8px;` → Espaciado entre items (no confiar en margin)
- `flex: 1;` → Item crece y se encoge proporcionalmente
- `align-self: center;` → Alinear un item individual

> **¿Cuándo usar qué?** Flexbox para layouts unidimensionales (navbar, fila de cards, centrar un hijo). Para layouts de rejilla (2D), usar Grid.

<div style="display:flex;gap:8px;padding:12px;background:#f5f5f5;border-radius:8px;">
  <div style="flex:1;background:#4af;padding:16px;border-radius:4px;color:#fff;text-align:center;">1</div>
  <div style="flex:1;background:#4af;padding:16px;border-radius:4px;color:#fff;text-align:center;">2</div>
  <div style="flex:1;background:#4af;padding:16px;border-radius:4px;color:#fff;text-align:center;">3</div>
</div>

## 6. Grid

- `display: grid;` → Contenedor de rejilla (2D: filas y columnas)
- `grid-template-columns: 1fr 1fr 1fr;` → Tres columnas iguales
- `grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));` → Grid responsive sin media queries
- `grid-template-rows: auto 1fr auto;` → Filas: contenido, flexible, contenido
- `grid-column: span 2;` → Ocupar 2 columnas
- `gap: 16px;` → Espaciado entre celdas
- `place-items: center;` → Centrar items (align + justify)

> **¿Cuándo usar qué?** Grid para layouts bidimensionales (galerías, dashboard, layout de página). Si solo necesitas una fila o columna, usa Flexbox.

## 7. Posicionamiento

- `position: relative;` → Ancla para hijos absolutos / `top`/`left` desde su posición original
- `position: absolute;` → Fuera del flujo, relativo al ancestro posicionado más cercano
- `position: fixed;` → Fijo al viewport (no hace scroll)
- `position: sticky;` → Entre relative y fixed (se pega al tope al hacer scroll)
- `inset: 0;` → Atajo para `top:0; right:0; bottom:0; left:0`
- `z-index: 10;` → Capa de superposición (solo funciona en elementos posicionados)

> **¿Cuándo usar qué?** `sticky` para headers de tabla o navs. `absolute` para tooltips, modales o badges. `fixed` para overlays o barras que nunca se van. `relative` con `position: static;` no activa z-index.

## 8. Bordes y Sombras

- `border: 1px solid #ddd;` → Borde estándar
- `border-radius: 8px;` → Esquinas redondeadas
- `border-radius: 999px;` → Círculo/píldora (con width = height o con padding)
- `box-shadow: 0 2px 8px rgb(0 0 0 / .15);` → Sombra suave
- `box-shadow: 0 0 0 2px #4af;` → Anillo (outline con box-shadow)
- `outline: 2px solid #4af; outline-offset: 2px;` → Borde externo (no afecta layout)
- `filter: drop-shadow(0 4px 4px rgb(0 0 0 / .3));` → Sombra que respeta la forma del elemento (alpha)

> **¿Cuándo usar qué?** `box-shadow` para sombras de caja. `drop-shadow` para sombras que siguen bordes con transparencia (PNG, clip-path). `outline` para accesibilidad (focus visible).

## 9. Transiciones

- `transition: all 0.3s ease;` → Transición genérica (preferir propiedades específicas)
- `transition: transform 0.2s ease, opacity 0.2s ease;` → Transición específica (mejor rendimiento)
- `transition-timing-function: cubic-bezier(0.68, -0.55, 0.27, 1.55);` → Curva personalizada (efecto rebote)
- `transition: width 0.3s steps(4);` → Animación en pasos discretos

> **¿Cuándo usar qué?** Transicionar `opacity` y `transform` rinde mejor que `width`, `height` o `top`. Evitar `transition: all`. Preferir `ease` para UI general, `cubic-bezier` para rebotes.

<div style="transition: transform 0.2s ease, box-shadow 0.2s ease;display:inline-block;padding:8px 16px;background:#4af;color:#fff;border-radius:6px;cursor:pointer;" onmouseover="this.style.transform='scale(1.1)';this.style.boxShadow='0 4px 12px rgba(0,0,0,.3)'" onmouseout="this.style.transform='scale(1)';this.style.boxShadow='none'">Hover me</div>

## 10. Animaciones

- `@keyframes slide { from { transform: translateX(-100%); } to { transform: translateX(0); } }` → Animación reutilizable
- `animation: slide 0.5s ease forwards;` → Aplicar animación
- `animation-iteration-count: infinite;` → Bucle infinito
- `animation-direction: alternate;` → Reversa automática (rebote)
- `animation-timing-function: ease-in-out;` → Curva de la animación
- `prefers-reduced-motion: reduce` → Respetar preferencias de accesibilidad (`@media (prefers-reduced-motion: reduce) { animation: none; }`)

## 11. Transformaciones

- `transform: scale(1.1);` → Escalar
- `transform: rotate(45deg);` → Rotar
- `transform: translateX(20px);` → Mover en X
- `transform: translateY(-50%);` → Centrar verticalmente (combinado con absolute + top: 50%)
- `transform: skew(10deg);` → Inclinar
- `transform-origin: top left;` → Origen de la transformación

> **¿Cuándo usar qué?** Para centrar: `top: 50%; left: 50%; transform: translate(-50%, -50%);` sobre un absolute. Es más robusto que margin negativo. Preferir transform para animaciones de posición (mejor rendimiento que top/left).

## 12. Responsive

- `@media (max-width: 768px) { ... }` → Punto de quiebre tradicional
- `@media (prefers-color-scheme: dark) { ... }` → Modo oscuro del sistema
- `container-type: inline-size;` → Contenedor para container queries
- `@container (min-width: 400px) { ... }` → Consulta basada en el contenedor, no el viewport
- `width: clamp(300px, 80%, 1200px);` → Tamaño fluido
- `height: 100dvh;` → Altura dinámica del viewport (ideal para móviles sin el chrome del browser)

> **¿Cuándo usar qué?** `@media` para layout general (sidebar, header). `@container` para componentes reutilizables (cards, widgets) que deben adaptarse a su contenedor. `100dvh` para full-screen en móvil.

## 13. Efectos UI Modernos

- `filter: blur(4px);` → Desenfoque
- `filter: brightness(1.2);` → Brillo
- `backdrop-filter: blur(8px);` → Desenfoque del fondo (efecto vidrio / glassmorphism)
- `mix-blend-mode: multiply / overlay / difference;` → Mezcla de colores entre capas
- `clip-path: polygon(0 0, 100% 0, 100% 80%, 0 100%);` → Recortar elemento con forma
- `mask-image: linear-gradient(black, transparent);` → Degradado de opacidad (desvanecer)
- `-webkit-backdrop-filter: blur(8px);` → Safari requiere prefijo

> **¿Cuándo usar qué?** `backdrop-filter` para glassmorphism (modales, navs). `clip-path` para formas decorativas (triángulos, diagonales). `mask-image` para desvanecer bordes de imágenes. `mix-blend-mode: difference` es útil para crear esquemas de color que se adaptan al fondo.

<div style="backdrop-filter:blur(8px);-webkit-backdrop-filter:blur(8px);background:rgb(255 255 255 / .3);border:1px solid rgb(255 255 255 / .5);padding:16px;border-radius:12px;color:#333;max-width:300px;">
Glassmorphism effect
</div>

## 14. Variables CSS

- `:root { --primary: #4af; }` → Definir variable global
- `color: var(--primary);` → Usar variable
- `color: var(--primary, #08f);` → Variable con fallback
- `@property --angle { syntax: '<angle>'; inherits: false; initial-value: 0deg; }` → Variable tipada (para animar gradients)

> **¿Cuándo usar qué?** Usar `@property` cuando necesites animar propiedades que CSS no puede interpolar por defecto (como el ángulo de un gradient). Preferir variables para temas (colores, espaciado) y evitar valores mágicos repetidos.
