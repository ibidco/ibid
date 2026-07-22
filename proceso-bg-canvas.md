# Fondo animado de la sección Proceso
### Efecto "neurona / luciérnaga" — Canvas 2D puro

---

## Qué se ve

Un campo de 110 puntos pequeños distribuidos al azar sobre toda la sección. La mayoría están quietos y apenas visibles (gris muy tenue, con un pulso lento y personal). De vez en cuando uno "dispara" en amarillo, crece con un halo difuso y, al llegar a su punto máximo de brillo, puede contagiar el disparo a los puntos vecinos que tiene cerca. Las líneas que conectan puntos cercanos también se iluminan en dorado mientras dura el disparo. El efecto da la sensación de neuronas encendiéndose o luciérnagas parpadeando en la oscuridad.

---

## Cómo está montado

### HTML

```html
<!-- Dentro de la sección .proc-section, posición absolute -->
<canvas id="proc-canvas" aria-hidden="true"></canvas>
```

El canvas cubre toda la sección con `position: absolute; inset: 0`. El diagrama del proceso (el SVG de los círculos) va encima con `z-index: 1`.

### CSS

```css
#proc-canvas {
  position: absolute; inset: 0;
  width: 100%; height: 100%;
  pointer-events: none; /* no interfiere con clicks */
  z-index: 0;
}
```

---

## La lógica JS, paso a paso

### 1. Constantes que controlan el comportamiento

```js
const COUNT       = 110;    // número de puntos en el campo
const CONNECT     = 95;     // distancia máxima en px para dibujar una línea entre puntos
const FIRE_PROB   = 0.0012; // probabilidad por punto por frame de encenderse espontáneamente
const FIRE_FRAMES = 50;     // cuántos frames dura un disparo (a 60fps ≈ 0.8s)
```

### 2. Inicialización — `init()`

Se crean 110 partículas. Cada una guarda:

```js
{
  px:      Math.random(),          // posición horizontal como fracción 0-1
  py:      Math.random(),          // posición vertical como fracción 0-1
  phase:   Math.random() * Math.PI * 2,  // fase para el pulso personal (onda seno independiente)
  r:       Math.random() * 1.4 + 0.9,   // radio base entre 0.9 y 2.3 px
  fire:    0,      // contador de frames que le quedan de disparo (0 = apagado)
  fireMax: 0       // duración total del disparo, para calcular el progreso
}
```

Las posiciones son fracciones (0–1) y se multiplican por W/H en cada frame, así el efecto es completamente responsivo sin reiniciar.

### 3. El bucle principal — `loop()`

Corre con `requestAnimationFrame` (~60fps). En cada frame hace tres cosas en orden:

---

#### A. Actualizar estados de disparo

```
Para cada punto:
  Si está encendido (fire > 0):
    → Decrementar su contador (fire--)
    → Si llega al 60% de su duración (pico de brillo):
        mirar vecinos dentro de CONNECT px
        con 25% de probabilidad, encender a cada vecino cercano
            (duración aleatoria entre 40 y 65 frames)
  Si está apagado:
    → Con probabilidad 0.0012 por frame, encenderse espontáneamente
        (a 60fps esto ocurre estadísticamente cada ~14 segundos por partícula,
         pero con 110 partículas hay un disparo espontáneo cada ~0.1s en promedio)
```

Este es el mecanismo de **cascada**: un disparo se propaga a los vecinos cercanos, que a su vez se propagan a los suyos, creando ondas cortas de activación.

---

#### B. Dibujar las líneas de conexión

Para cada par de puntos dentro de `CONNECT` px:
- Si alguno de los dos está disparando → línea dorada `rgba(220,180,0,α)`, grosor 0.7px
- Si ambos están apagados → línea gris muy sutil `rgba(8,8,15,α)`, grosor 0.45px

La opacidad `α` disminuye con la distancia: los puntos muy cercanos tienen línea más visible, los que están al límite de `CONNECT` son casi invisibles.

---

#### C. Dibujar los puntos

**Punto encendido:**

Usa una curva de easing en tres fases según el progreso del disparo:
- Primeros 25%: fade in (0 → 1)
- Medio 40%: brillo máximo (1)
- Últimos 35%: fade out (1 → 0)

Dibuja dos capas:
1. **Núcleo**: círculo amarillo sólido `rgba(255,220,40,α)` al doble del radio base
2. **Halo**: gradiente radial difuso que se desvanece en 8× el radio base

**Punto apagado:**

```js
const pulse = 0.5 + 0.5 * Math.sin(tick * 0.018 + p.phase);
const a = 0.07 + pulse * 0.07;  // oscila entre 0.07 y 0.14 de opacidad
```

Cada punto tiene su propia velocidad de pulso (por `p.phase`) así que nunca parpadean todos al unísono. El resultado es un campo de puntos que "respira" de forma orgánica.

---

## Parámetros para ajustar el efecto

| Parámetro | Valor actual | Qué hace si lo subes | Qué hace si lo bajas |
|---|---|---|---|
| `COUNT` | 110 | Más denso, más conexiones | Más disperso |
| `CONNECT` | 95px | Líneas más largas, más conectado | Clusters más pequeños |
| `FIRE_PROB` | 0.0012 | Disparos más frecuentes | Efecto más tranquilo |
| `FIRE_FRAMES` | 50 | Destellos más largos (~0.8s) | Destellos más cortos |
| `0.25` (cascade) | 25% | Más propagación, ondas grandes | Disparos más aislados |
| `tick * 0.018` (pulse) | — | Pulso más rápido | Pulso más lento |

---

## Responsivo

Cuando la sección cambia de tamaño (resize del viewport o del contenedor), un `ResizeObserver` llama a `resize()` que actualiza `W` y `H`. Como las posiciones de los puntos son fracciones 0–1, los puntos se reescalan automáticamente sin reiniciar.

---

## Accesibilidad

- `aria-hidden="true"` en el canvas: los lectores de pantalla lo ignoran
- Si el sistema tiene `prefers-reduced-motion: reduce`, el loop no arranca (`if (!rm) loop()`)

---

## Dependencias

Ninguna. Canvas 2D nativo del navegador, sin librerías.
