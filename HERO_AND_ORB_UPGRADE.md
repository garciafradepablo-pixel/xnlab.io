# XNLAB — Hero upgrade + orbe esmeralda (aplicar en el Next.js real)

> **IMPORTANTE — repos:** El sitio en vivo **xnlab.io** se construye desde el repo
> **`garciafradepablo-pixel/xnlab`** (TypeScript / Next.js). **Este** repo
> (`garciafradepablo-pixel/xnlab.io`) solo contiene `XNLAB.html`, un *snapshot HTML
> renderizado* que **no se despliega**. Por eso editar `XNLAB.html` no cambia nada en
> producción. Todo lo de abajo debe aplicarse en el hero y en el array de sistemas del
> repo `garciafradepablo-pixel/xnlab` (p. ej. `app/page.tsx` o `components/Hero`,
> `components/Systems`). Los valores están validados contra el markup renderizado.

---

## TAREA 1 — Orbe del sistema 04 "Operaciones de Cliente" → VERDE ESMERALDA

En la sección "El universo / Seis sistemas" hay un array de sistemas. El sistema **04
"Operaciones de Cliente"** usa el mismo naranja/ámbar que el **01 "Producto"** (color
duplicado). Cámbialo a **verde esmeralda**, que no choca con ninguno de los demás
(02 violeta · 03 azul acero · 05 plata · 06 azul).

### Si el color es un campo del array
Busca la entrada de "Operaciones de Cliente" y cambia **solo su color/acento**:

```ts
// ANTES (duplica el naranja del 01 Producto)
{ id: '04', title: 'Operaciones de Cliente', color: '#E8923A' /* o el ámbar que use */, ... }

// DESPUÉS — verde esmeralda
{ id: '04', title: 'Operaciones de Cliente', color: '#10B981', ... }
```

Paleta esmeralda recomendada (elige según cómo se use el color):
- **Acento sólido:** `#10B981` (emerald‑500)
- **Más profundo / cinematográfico:** `#059669` (emerald‑600)
- **Glow / sombra:** `rgba(16, 185, 129, 0.55)`
- **Gradiente de orbe:** `radial-gradient(circle at 35% 30%, #6EE7B7 0%, #10B981 45%, #065F46 100%)`

### Si cada orbe es una imagen (`orbImage` / `src` por entrada)
Sustituye solo la imagen del orbe 04 por una versión esmeralda (mismo tamaño y estilo
de render que las demás), y actualiza el glow/sombra asociado a `rgba(16,185,129,0.55)`.
No toques las otras 5 entradas.

> Verifica que el verde elegido tenga contraste suficiente sobre el fondo oscuro y que
> el glow no se confunda con el azul del 06.

---

## TAREA 2 — Upgrade del hero

### 2.1 Valores (find → replace en el componente del hero)

**Título `XNLAB` (`<h1>`):**

| propiedad | antes | después |
|---|---|---|
| `font-size` | `clamp(88px, 16vw, 220px)` | `clamp(140px, 27vw, 420px)` |
| `letter-spacing` | `-0.04em` | `-0.045em` |
| `line-height` | `0.86` | `0.82` |

**Presencia de las capas detrás del título (subir opacidad):**

| capa | antes | después |
|---|---|---|
| imagen de fondo (`01_background_*`) | `0.55` | `0.72` |
| anillo orbital (`03_back_orbits`) | `0.28` | `0.5` |
| símbolo base (`05_main_bottom_symbol`) | `0.7` | `0.88` (glow más fuerte: `drop-shadow(0 0 18px rgba(180,150,120,0.5))` → `drop-shadow(0 0 30px rgba(190,160,130,0.78))`) |
| haze (`02_haze_overlay`) | `0.08` | `0.14` |
| velo radial central | `radial-gradient(ellipse at 50% 40%, rgba(3,2,1,0.3) 0%, rgba(3,2,1,0.92) 100%)` | `radial-gradient(ellipse at 50% 40%, rgba(3,2,1,0.1) 0%, rgba(3,2,1,0.88) 100%)` |
| halo cálido (arriba‑dcha, `blur(40px)`) | `opacity: 0` | `opacity: 0.85` |
| halo frío (abajo‑izq, `blur(44px)`) | `opacity: 0` | `opacity: 0.7` |

**Agrandar la floritura (ancho y alto):**

| elemento | antes | después |
|---|---|---|
| anillo orbital (wrapper) | `clamp(320px, 55vw, 680px)` | `clamp(420px, 70vw, 940px)` |
| orbe central (wrapper) | `clamp(70px, 8vw, 110px)` | `clamp(96px, 11vw, 150px)` |
| símbolo base (wrapper) | `clamp(180px, 32vw, 380px)` | `clamp(240px, 40vw, 520px)` |

### 2.2 Animación de entrada escalonada + vida lenta (CSS global)

Añade clases `hero-eyebrow`, `hero-title`, `hero-subcopy`, `hero-orb`, `hero-ring`,
`hero-symbol` a los elementos correspondientes y pega este CSS:

```css
@media (prefers-reduced-motion: no-preference){
  @keyframes xn-rise    {0%{opacity:0;transform:translateY(26px);filter:blur(10px)}100%{opacity:1;transform:none;filter:none}}
  @keyframes xn-rise-sm {0%{opacity:0;transform:translateY(14px);filter:blur(6px)} 100%{opacity:1;transform:none;filter:none}}
  @keyframes xn-orb-in  {0%{opacity:0;transform:scale(.82)}100%{opacity:1;transform:scale(1)}}
  @keyframes xn-fade    {0%{opacity:0}100%{opacity:1}}
  @keyframes xn-float   {0%,100%{transform:translateY(0)}50%{transform:translateY(-10px)}}
  @keyframes xn-orbit   {from{transform:rotate(0)}to{transform:rotate(360deg)}}
  @keyframes xn-breathe {0%,100%{transform:scale(1)}50%{transform:scale(1.03)}}

  /* entrada: eyebrow → título → subcopy → orbes */
  .hero-eyebrow{animation:xn-rise-sm 1.1s cubic-bezier(.22,1,.36,1) .15s both}
  .hero-title  {animation:xn-rise    1.6s cubic-bezier(.22,1,.36,1) .45s both}
  .hero-subcopy{animation:xn-rise-sm 1.1s cubic-bezier(.22,1,.36,1) .80s both}

  /* orbes: entran y luego mantienen vida lenta */
  .hero-orb   {animation:xn-orb-in 1.4s cubic-bezier(.22,1,.36,1) 1.05s both, xn-float   11s ease-in-out 2.4s infinite}
  .hero-ring  {animation:xn-fade   1.8s ease 1.00s both,                      xn-orbit  165s linear      2.6s infinite}
  .hero-symbol{animation:xn-fade   1.8s ease 1.18s both,                      xn-breathe 17s ease-in-out 2.7s infinite}

  /* refrigeración: pausa cuando la sección no está en viewport */
  .xn-paused, .xn-paused *{animation-play-state:paused !important}
}
/* prefers-reduced-motion: sin animación; los elementos quedan en su estado final */
```

> Solo se animan `transform / opacity / filter` (compositor‑friendly).

### 2.3 Parallax 3D de puntero (solo `pointer:fine`, off en reduced‑motion)

Hook React reutilizable. Asigna a cada capa un `data-depth` (más alto = se mueve más) y
mantén el centrado con `translate(-50%,-50%)` para las capas centradas.

```tsx
import { useEffect, useRef } from 'react';

export function useHeroParallax<T extends HTMLElement>() {
  const ref = useRef<T>(null);
  useEffect(() => {
    const root = ref.current;
    if (!root) return;
    const reduce = matchMedia('(prefers-reduced-motion: reduce)').matches;
    const fine   = matchMedia('(pointer: fine)').matches;
    if (reduce || !fine) return; // accesibilidad + solo punteros finos

    const layers = Array.from(root.querySelectorAll<HTMLElement>('[data-depth]'));
    let tx = 0, ty = 0, px = 0, py = 0, raf = 0, visible = true;

    const onMove = (e: PointerEvent) => {
      tx = (e.clientX / innerWidth  - 0.5) * 2;
      ty = (e.clientY / innerHeight - 0.5) * 2;
    };
    const tick = () => {
      px += (tx - px) * 0.06; py += (ty - py) * 0.06;
      for (const el of layers) {
        const d = parseFloat(el.dataset.depth || '0');
        const centered = el.dataset.centered === 'true';
        const dx = (px * d).toFixed(2), dy = (py * d).toFixed(2);
        el.style.transform = centered
          ? `translate(calc(-50% + ${dx}px), calc(-50% + ${dy}px))`
          : `translate(${dx}px, ${dy}px)`;
      }
      if (visible && !document.hidden) raf = requestAnimationFrame(tick);
    };
    const io = new IntersectionObserver(([en]) => {
      visible = en.isIntersecting;
      root.classList.toggle('xn-paused', !visible); // pausa animaciones CSS off‑screen
      if (visible && !raf) raf = requestAnimationFrame(tick);
      if (!visible && raf) { cancelAnimationFrame(raf); raf = 0; }
    }, { threshold: 0 });

    addEventListener('pointermove', onMove, { passive: true });
    addEventListener('pointerleave', () => { tx = 0; ty = 0; }, { passive: true });
    document.addEventListener('visibilitychange', () => {
      if (!document.hidden && visible && !raf) raf = requestAnimationFrame(tick);
    });
    io.observe(root);
    raf = requestAnimationFrame(tick);
    return () => { cancelAnimationFrame(raf); io.disconnect(); removeEventListener('pointermove', onMove); };
  }, []);
  return ref;
}
```

Uso (profundidades sugeridas; mantén el centrado donde ya lo usabas):

```tsx
const heroRef = useHeroParallax<HTMLElement>();
// <section ref={heroRef}>
//   <div data-depth="7"  ...background... />
//   <div data-depth="30" ...halo cálido... />
//   <div data-depth="30" ...halo frío... />
//   <div data-depth="16" data-centered="true" className="hero-ring"   ...anillo... />
//   <div data-depth="26" data-centered="true" className="hero-orb"    ...orbe... />
//   <div data-depth="12" data-centered="true" className="hero-symbol" ...símbolo... />
//   <div data-depth="6"  ...grupo de texto (eyebrow/título/subcopy)... />
//   <div data-depth="18" ...haze... />
```

---

## Rendimiento / accesibilidad ("refrigeración")
- Solo `transform / opacity / filter`; nada de animar `width/top/left/box-shadow`.
- `prefers-reduced-motion: reduce` desactiva entrada, vida lenta y parallax (estado final estático).
- Parallax solo en `pointer: fine` (no en táctil).
- Pausa de animaciones cuando la sección sale del viewport (clase `.xn-paused`) y cuando la pestaña está oculta.
- Centrado preservado con `translate(-50%,-50%)` vía `calc()`.

## Checklist al aplicar en `garciafradepablo-pixel/xnlab`
- [ ] Tarea 1: orbe 04 "Operaciones de Cliente" en esmeralda (`#10B981` / gradiente), resto intacto.
- [ ] Tarea 2: valores del hero (título, opacidades, halos, velo, tamaños).
- [ ] CSS de entrada + vida lenta con las 6 clases.
- [ ] Hook de parallax montado en el `<section>` del hero.
- [ ] `npm run lint && npm run build` en verde; arranque OK.
- [ ] QA: desktop/móvil, con y sin reduced‑motion, puntero fino y táctil.
