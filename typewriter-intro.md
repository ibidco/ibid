# Typewriter Intro — Documentación

Pantalla negra que aparece al cargar el sitio y escribe dos frases letra por letra antes de revelar el contenido.

---

## Cómo funciona

1. Un `<div>` cubre toda la pantalla con `position: fixed` y `z-index: 9999`.
2. JavaScript escribe la primera frase letra por letra usando `setTimeout` recursivo.
3. Al terminar la primera frase, el cursor se mueve a la segunda línea y repite el proceso.
4. Cuando termina la segunda frase, espera 2 segundos y hace fade-out con `opacity: 0`.
5. Si el usuario tiene activado "reducir movimiento" (`prefers-reduced-motion`), la pantalla se salta directamente.

---

## HTML

```html
<!-- Pantalla de intro — se oculta cuando termina la animación -->
<div id="intro" aria-hidden="true">
  <div>
    <!-- Línea 1: "Labnormally." — el cursor empieza aquí -->
    <p id="intro-line">
      <span id="intro-txt"></span>
      <span id="intro-cursor"></span>
    </p>
    <!-- Línea 2: "De las ideas a la realidad." — el cursor se mueve aquí al terminar línea 1 -->
    <p id="intro-sub">
      <span id="intro-sub-txt"></span>
    </p>
  </div>
</div>
```

**Notas:**
- `aria-hidden="true"` oculta el intro de lectores de pantalla; el contenido real del sitio ya está en el DOM.
- El cursor (`#intro-cursor`) empieza en `#intro-line` y se mueve físicamente al `<p>` de `#intro-sub` cuando arranca la segunda frase.

---

## CSS

```css
/* Fondo negro fijo que cubre toda la pantalla */
#intro {
  position: fixed;
  inset: 0;                              /* top/right/bottom/left: 0 */
  background: #08080F;                   /* negro casi puro */
  z-index: 9999;                         /* encima de todo */
  display: flex;
  align-items: center;                   /* texto centrado verticalmente */
  padding: 0 clamp(1.5rem, 6vw, 6rem);  /* padding responsivo */
  transition: opacity .75s ease;         /* fade-out suave al salir */
}

/* Estado de salida: se aplica con JS para disparar el fade */
#intro.out {
  opacity: 0;
  pointer-events: none; /* evita clicks durante el fade */
}

/* Línea 1 — "Labnormally." */
#intro-line {
  font-family: 'Barlow Condensed', sans-serif;
  font-size: clamp(3.5rem, 9vw, 9rem); /* responsivo: mínimo 3.5rem, máximo 9rem */
  font-weight: 900;
  letter-spacing: -.01em;
  text-transform: uppercase;
  color: #fff;
  line-height: 1;
}

/* Línea 2 — "De las ideas a la realidad." */
#intro-sub {
  font-family: 'Barlow Condensed', sans-serif;
  font-size: clamp(3.5rem, 9vw, 9rem);
  font-weight: 900;
  letter-spacing: -.01em;
  text-transform: uppercase;
  color: #fff;
  margin-top: .1em;
  line-height: 1;
  min-height: 1em; /* reserva espacio antes de que empiece a escribir */
}

/* Cursor parpadeante */
#intro-cursor {
  display: inline-block;
  width: .08em;
  height: .85em;
  background: #fff;
  margin-left: .06em;
  vertical-align: middle;
  animation: cur .65s step-end infinite; /* parpadeo con step-end = sin suavizado */
}

/* Animación del cursor: on/off sin transición suave */
@keyframes cur {
  0%, 100% { opacity: 1; }
  50%       { opacity: 0; }
}
```

---

## JavaScript

```js
// Detecta si el usuario prefiere reducir movimiento (accesibilidad)
const rm = window.matchMedia("(prefers-reduced-motion: reduce)").matches;

/* ── INTRO TYPEWRITER ── */
(function () {
  const overlay = document.getElementById('intro');
  const el      = document.getElementById('intro-txt');     // span línea 1
  const sub     = document.getElementById('intro-sub-txt'); // span línea 2
  const cursor  = document.getElementById('intro-cursor');
  if (!overlay || !el) return;

  // Bloquea el scroll mientras se muestra la intro
  document.body.style.overflow = 'hidden';

  const LINE1 = 'Labnormally.';
  const LINE2 = 'De las ideas a la realidad.';
  const SPEED = 72;    // ms base entre cada letra
  const HOLD  = 2000;  // ms de espera al terminar antes de cerrar

  // Cierra la pantalla con fade-out
  function dismiss() {
    overlay.classList.add('out');          // dispara la transición CSS opacity → 0
    document.body.style.overflow = '';     // restaura el scroll
    setTimeout(() => {
      overlay.style.display = 'none';      // retira del layout después del fade
    }, 820); // un poco más que la duración del transition (.75s)
  }

  // Escribe la segunda línea letra por letra
  function typeLine2(i) {
    if (i > LINE2.length) {
      setTimeout(dismiss, HOLD); // terminó → espera y cierra
      return;
    }
    sub.textContent = LINE2.slice(0, i);
    // Velocidad con variación aleatoria para simular escritura humana
    setTimeout(() => typeLine2(i + 1), SPEED + Math.random() * 35);
  }

  // Escribe la primera línea letra por letra
  function typeLine1(i) {
    if (i > LINE1.length) {
      // Terminó línea 1 → mueve el cursor al <p> de la línea 2 y arranca línea 2
      setTimeout(() => {
        document.getElementById('intro-sub').appendChild(cursor);
        typeLine2(0);
      }, 750); // pausa de 0.75s entre las dos frases
      return;
    }
    el.textContent = LINE1.slice(0, i);
    setTimeout(() => typeLine1(i + 1), SPEED + Math.random() * 40);
  }

  // Si prefers-reduced-motion está activo, salta directo al sitio
  if (rm) { dismiss(); return; }

  // Espera a que las fuentes carguen antes de arrancar
  // (evita que el tamaño de letra cambie a mitad de la animación)
  (document.fonts ? document.fonts.ready : Promise.resolve())
    .then(() => setTimeout(() => typeLine1(0), 200));
})();
```

---

## Personalización rápida

| Qué cambiar | Dónde |
|---|---|
| Textos de las frases | `LINE1` y `LINE2` en el JS |
| Velocidad de escritura | `SPEED` (ms por letra, más alto = más lento) |
| Pausa entre frases | El `750` del setTimeout en `typeLine1` |
| Pausa al final antes de cerrar | `HOLD` (ms) |
| Tamaño de letra | `clamp(3.5rem, 9vw, 9rem)` en el CSS |
| Color de fondo | `background: #08080F` en `#intro` |

---

## Dependencias

- **Fuente:** [Barlow Condensed 900](https://fonts.google.com/specimen/Barlow+Condensed) — debe cargarse antes de la animación (de ahí el `document.fonts.ready`).
- **Sin librerías externas.** Todo es CSS + JS vanilla.
