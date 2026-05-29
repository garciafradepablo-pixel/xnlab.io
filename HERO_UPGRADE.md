# XNLAB — Hero upgrade (apply to the `garciafradepablo-pixel/xnlab` Next.js source)

> This file lives in the `xnlab.io` repo (the saved HTML snapshot). The **live site**
> is built from the **`garciafradepablo-pixel/xnlab`** TypeScript/Next.js repo, so the
> hero must be changed there. Everything below is copy‑paste ready and was validated
> against the rendered markup. Apply it in the Next.js hero component (e.g.
> `app/page.tsx` or `components/Hero`).

## 1. Value changes (find → replace in the hero component)

### Title `XNLAB` (`<h1>`)
| property | before | after |
|---|---|---|
| `font-size` | `clamp(88px, 16vw, 220px)` | `clamp(140px, 27vw, 420px)` |
| `letter-spacing` | `-0.04em` | `-0.045em` |
| `line-height` | `0.86` | `0.82` |

### Opacity of the layers behind the title (raise presence)
| layer | before | after |
|---|---|---|
| background image (`01_background_*`) | `0.55` | `0.72` |
| orbit ring (`03_back_orbits`) | `0.28` | `0.5` |
| base symbol (`05_main_bottom_symbol`) | `0.7` | `0.88` (glow `18px/0.5` → `22px/0.6`) |
| haze overlay (`02_haze_overlay`) | `0.08` | `0.14` |
| central radial veil | `rgba(3,2,1,0.3) 0% … 0.92 100%` | `rgba(3,2,1,0.1) 0% … 0.88 100%` |
| warm glow blob | `opacity: 0` | `opacity: 0.85` |
| cool glow blob | `opacity: 0` | `opacity: 0.7` |

### Flourish sizes (bigger orb / ring / symbol)
| element | before (width & height) | after |
|---|---|---|
| orbit ring wrapper | `clamp(320px, 55vw, 680px)` | `clamp(420px, 70vw, 940px)` |
| core orb wrapper | `clamp(70px, 8vw, 110px)` | `clamp(96px, 11vw, 150px)` |
| base symbol wrapper | `clamp(180px, 32vw, 380px)` | `clamp(240px, 40vw, 520px)` |

## 2. Per-element entrance + continuous motion (CSS — drop in a global stylesheet)

```css
@media (prefers-reduced-motion: no-preference){
  @keyframes xn-float{0%,100%{transform:translateY(0)}50%{transform:translateY(-10px)}}
  @keyframes xn-orbit{from{transform:rotate(0deg)}to{transform:rotate(360deg)}}
  @keyframes xn-breathe{0%,100%{transform:scale(1)}50%{transform:scale(1.03)}}
  @keyframes xn-rise{0%{opacity:0;transform:translateY(26px);filter:blur(8px)}100%{opacity:1;transform:none;filter:none}}
  @keyframes xn-rise-sm{0%{opacity:0;transform:translateY(14px)}100%{opacity:1;transform:none}}
  @keyframes xn-fade{0%{opacity:0}100%{opacity:1}}
  @keyframes xn-orb-in{0%{opacity:0;transform:scale(0.82)}100%{opacity:1;transform:scale(1)}}

  /* entrance — stagger: eyebrow → wordmark → subcopy */
  .hero-eyebrow{animation:xn-rise-sm 1.2s cubic-bezier(.22,1,.36,1) .1s both}
  .hero-title{animation:xn-rise 1.6s cubic-bezier(.22,1,.36,1) .25s both}
  .hero-subcopy{animation:xn-rise-sm 1.2s cubic-bezier(.22,1,.36,1) .9s both}

  /* core objects: entrance, then settle into slow life */
  .hero-orb{animation:xn-orb-in 1.6s cubic-bezier(.22,1,.36,1) .15s both, xn-float 11s ease-in-out 1.9s infinite}
  .hero-ring{animation:xn-fade 2s ease .2s both, xn-orbit 165s linear 2.2s infinite}
  .hero-symbol{animation:xn-fade 2s ease .5s both, xn-breathe 17s ease-in-out 2.5s infinite}

  /* cooling: pause off-screen */
  .xn-paused, .xn-paused *{animation-play-state:paused!important}
}
```
Add the classes (`hero-eyebrow`, `hero-title`, `hero-orb`, `hero-ring`, `hero-symbol`,
`hero-subcopy`) to the matching elements. Animations override inline opacity, and
under reduced-motion everything stays visible (no animation).

## 3. Pointer parallax — 3D depth (Next.js client component)

```tsx
"use client";
import { useEffect, useRef } from "react";

/** Attach refs to the three hero layers (ring, orb, symbol). Each keeps its own
 *  `translate(-50%,-50%)` centering; this only adds an eased cursor offset. */
export function useHeroParallax(
  ring: React.RefObject<HTMLElement>,
  orb: React.RefObject<HTMLElement>,
  symbol: React.RefObject<HTMLElement>,
) {
  useEffect(() => {
    if (typeof window === "undefined") return;
    const mm = window.matchMedia;
    if (mm("(prefers-reduced-motion: reduce)").matches) return;
    if (!mm("(pointer: fine)").matches) return;
    const layers: [HTMLElement | null, number][] = [
      [ring.current, 18], [orb.current, 30], [symbol.current, 44],
    ];
    let tx = 0, ty = 0, cx = 0, cy = 0, raf = 0;
    const loop = () => {
      cx += (tx - cx) * 0.08; cy += (ty - cy) * 0.08;
      for (const [el, d] of layers) {
        if (el) el.style.transform =
          `translate(${(cx * d).toFixed(2)}px,${(cy * d).toFixed(2)}px) translate(-50%,-50%)`;
      }
      raf = Math.abs(tx - cx) > 0.001 || Math.abs(ty - cy) > 0.001
        ? requestAnimationFrame(loop) : 0;
    };
    const onMove = (e: PointerEvent) => {
      tx = (e.clientX / window.innerWidth - 0.5) * 2;
      ty = (e.clientY / window.innerHeight - 0.5) * 2;
      if (!raf) raf = requestAnimationFrame(loop);
    };
    window.addEventListener("pointermove", onMove, { passive: true });
    return () => { window.removeEventListener("pointermove", onMove); if (raf) cancelAnimationFrame(raf); };
  }, [ring, orb, symbol]);
}
```

## 4. Notes
- Keep transforms/opacity/filter only; the off-screen pause keeps GPU/heat low
  ("refrigeración"). Infinite loops are transform-only.
- No new dependencies. Works with `next/font`, `public/` assets and `"use client"`.
- The full reference (sections, sectors, contact form, SEO/JSON-LD, footer, motion)
  is in `XNLAB.html` on the `xnlab.io` PR — use it as the design source of truth.
