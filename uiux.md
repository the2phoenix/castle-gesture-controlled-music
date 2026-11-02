# UI/UX PRO MAX — Castle Phonk
**Version:** 1.0  
**Author:** Arnesh  
**Purpose:** Complete visual, motion, and experience design spec for the builder  

---

## 1. Design Philosophy

This is not an app. It is a **ritual space**.

The user does not click, type, or tap. They perform. The castle watches and responds. Every visual decision should reinforce the feeling that the castle is alive, ancient, and reacting to the user's presence and energy.

**Core aesthetic:** Gothic dark luxury. Not horror. Not cartoon. Cinematic.  
**Tone:** Like a music visualizer crossed with a haunted mansion crossed with a music video.  
**Reference energy:** Travis Scott's Astronomical concert × Dark Souls × Miles Morales Spider-Man

---

## 2. Layout

```
┌────────────────────────────────────────────────────────┐
│                                                        │
│                                                        │
│              CASTLE IMAGE (full viewport)              │
│         windows glow overlaid on top of image          │
│                                                        │
│                                                        │
│                                                        │
│                                             ┌────────┐ │
│                                             │ webcam │ │
│                                             │preview │ │
│                                             └────────┘ │
│  ◉ volume arc (appears near hand, fades)               │
│  [track name — bottom center, subtle]                  │
└────────────────────────────────────────────────────────┘
```

- Castle image fills 100vw × 100vh, `object-fit: cover`
- No navbar, no header, no traditional UI chrome
- Webcam preview: bottom-right corner, 180×135px, rounded corners, 60% opacity, subtle border
- Track name: bottom-center, only visible while music plays, fades in/out with music
- Volume arc: renders on a transparent canvas overlay, appears near wrist position

---

## 3. Castle Image Rendering

```css
#castle-bg {
  position: fixed;
  inset: 0;
  width: 100vw;
  height: 100vh;
  object-fit: cover;
  object-position: center bottom; /* show base of castle */
  z-index: 0;
  filter: brightness(0.75) contrast(1.1) saturate(0.9);
}
```

The slight brightness reduction makes the window glows pop more dramatically.

---

## 4. Window Glow System

### 4.1 Window Element Styling (base — unlit)

```css
.castle-window {
  position: absolute;
  border-radius: 40% 40% 20% 20% / 50% 50% 20% 20%; /* gothic arch approximation */
  background: transparent;
  opacity: 0;
  mix-blend-mode: screen;
  filter: blur(5px);
  transition: opacity 0.3s ease;
  pointer-events: none;
  z-index: 2;
}
```

### 4.2 Window Lit State

When music is playing, each window animates based on its assigned frequency band:

```css
.castle-window.lit {
  opacity: var(--window-opacity); /* set by JS, 0.4–1.0 */
  background: radial-gradient(
    ellipse at center,
    var(--window-color) 0%,
    transparent 70%
  );
  animation: windowPulse var(--pulse-speed) ease-in-out infinite alternate;
}

@keyframes windowPulse {
  from { filter: blur(4px) brightness(0.8); }
  to   { filter: blur(6px) brightness(1.4); }
}
```

### 4.3 Window Color Personalities

Each window has a fixed color. The system modulates brightness/opacity, not color, based on audio.

| Window Zone | Colors Used |
|---|---|
| Ground floor left | Deep purple, blood red, molten gold |
| Ground floor right | Emerald, electric blue, violet |
| First floor left | Crimson, amber, indigo |
| First floor right | Cyan, forest green, orange |
| Tower windows | Hot pink, lavender, teal, cobalt |
| Upper center | Gold, scarlet |

### 4.4 Frequency → Window Mapping

```js
// Bass (0–300Hz) → drives warm windows (red, orange, gold)
// Mid (300Hz–2kHz) → drives neutral windows (purple, green, teal)  
// High (2kHz+) → drives cool windows (blue, cyan, violet)

function updateWindows(bass, mid, high) {
  WINDOWS.forEach(w => {
    let intensity;
    if (w.band === 'bass') intensity = bass / 255;
    if (w.band === 'mid')  intensity = mid / 255;
    if (w.band === 'high') intensity = high / 255;

    const el = document.getElementById(w.id);
    el.style.setProperty('--window-opacity', 0.3 + intensity * 0.7);
    el.style.setProperty('--pulse-speed', `${0.4 + (1 - intensity) * 0.8}s`);
  });
}
```

### 4.5 Beat Drop Effect

On detected beat drops (bass spike > 200/255):

```js
function onBeatDrop() {
  // All windows flare white-hot for 80ms then return
  document.querySelectorAll('.castle-window').forEach(w => {
    w.style.filter = 'blur(8px) brightness(2.5)';
    setTimeout(() => w.style.filter = '', 80);
  });

  // Moon pulse
  moonElement.style.filter = 'brightness(2) blur(2px)';
  setTimeout(() => moonElement.style.filter = '', 150);
}
```

---

## 5. Moon Element

The moon visible in the castle image gets a separate absolutely-positioned div overlaid on top of it (approximate position: top-right area of image).

```css
#moon-overlay {
  position: absolute;
  top: 6%;
  right: 18%;
  width: 8%;
  aspect-ratio: 1;
  border-radius: 50%;
  background: radial-gradient(circle, rgba(255,255,220,0.15) 0%, transparent 70%);
  mix-blend-mode: screen;
  transition: filter 0.1s ease;
  pointer-events: none;
  z-index: 2;
}
```

Pulses subtly on every bass hit.

---

## 6. Volume Arc UI

Rendered on a fullscreen transparent canvas (z-index 10). Appears only during rotation gesture. Positioned near wrist landmark coordinates.

```js
function drawVolumeArc(ctx, wristX, wristY, volumePercent) {
  const radius = 55;
  const startAngle = -Math.PI * 0.75;
  const endAngle = startAngle + (Math.PI * 1.5 * volumePercent);

  // Background track
  ctx.beginPath();
  ctx.arc(wristX, wristY, radius, -Math.PI * 0.75, Math.PI * 0.75);
  ctx.strokeStyle = 'rgba(255,255,255,0.15)';
  ctx.lineWidth = 4;
  ctx.lineCap = 'round';
  ctx.stroke();

  // Filled arc
  ctx.beginPath();
  ctx.arc(wristX, wristY, radius, startAngle, endAngle);
  ctx.strokeStyle = volumePercent > 0.7 ? '#EF4444' : '#A855F7';
  ctx.lineWidth = 4;
  ctx.stroke();

  // Volume number
  ctx.fillStyle = 'rgba(255,255,255,0.8)';
  ctx.font = 'bold 14px monospace';
  ctx.textAlign = 'center';
  ctx.fillText(`${Math.round(volumePercent * 100)}`, wristX, wristY + 5);
}
```

---

## 7. Track Name Display

```css
#track-name {
  position: fixed;
  bottom: 32px;
  left: 50%;
  transform: translateX(-50%);
  font-family: 'Cinzel', serif; /* gothic/elegant */
  font-size: 13px;
  letter-spacing: 0.25em;
  color: rgba(255, 255, 255, 0.5);
  text-transform: uppercase;
  opacity: 0;
  transition: opacity 1s ease;
  z-index: 20;
  pointer-events: none;
}

#track-name.visible {
  opacity: 1;
}
```

Fades in when a track starts playing. Fades out when music stops.

---

## 8. Spider-Man Mode

### 8.1 Web Ring Progress Indicator (during hold)

```js
// Draw growing web ring at hand position during pose hold
function drawSpiderProgress(ctx, x, y, progress) { // progress 0–1
  const maxRadius = 80;
  ctx.beginPath();
  ctx.arc(x, y, maxRadius * progress, 0, Math.PI * 2);
  ctx.strokeStyle = `rgba(220, 38, 38, ${0.3 + progress * 0.7})`;
  ctx.lineWidth = 2;
  ctx.setLineDash([4, 4]);
  ctx.stroke();
  ctx.setLineDash([]);
}
```

### 8.2 City Overlay

```css
#city-overlay {
  position: fixed;
  inset: 0;
  background-image: url('spiderman-city.png');
  background-size: cover;
  background-position: center;
  opacity: 0;
  mix-blend-mode: overlay;
  transition: opacity 2s ease;
  z-index: 1;
  pointer-events: none;
}

#city-overlay.active {
  opacity: 0.55;
}
```

### 8.3 Web Shot Animation

On trigger, 8 web lines shoot outward from hand position:

```js
function fireWebShot(originX, originY) {
  const angles = Array.from({length: 8}, (_, i) => (i / 8) * Math.PI * 2);
  angles.forEach(angle => {
    animateWebLine(ctx, originX, originY, angle, 600); // 600ms to extend
  });
}
```

---

## 9. Webcam Preview

```css
#webcam-preview {
  position: fixed;
  bottom: 24px;
  right: 24px;
  width: 180px;
  height: 135px;
  border-radius: 12px;
  opacity: 0.6;
  border: 1px solid rgba(255,255,255,0.15);
  object-fit: cover;
  transform: scaleX(-1); /* mirror */
  z-index: 30;
  transition: opacity 0.3s;
}

#webcam-preview:hover {
  opacity: 0.9;
}
```

---

## 10. Typography

| Element | Font | Size | Style |
|---|---|---|---|
| Track name | Cinzel (Google Fonts) | 13px | Uppercase, letter-spacing 0.25em |
| Volume number | Monospace | 14px | Bold |
| Any status text | Cinzel | 12px | Uppercase |

Load via: `@import url('https://fonts.googleapis.com/css2?family=Cinzel:wght@400;700&display=swap');`

---

## 11. Idle State Atmosphere

When no hand is detected, the castle should feel cinematic and alive even in silence:

```css
/* Subtle fog animation at base of castle */
#fog-layer {
  position: fixed;
  bottom: 0;
  width: 100%;
  height: 30%;
  background: linear-gradient(to top, rgba(180,200,220,0.12), transparent);
  animation: fogDrift 8s ease-in-out infinite alternate;
  pointer-events: none;
  z-index: 1;
}

@keyframes fogDrift {
  from { opacity: 0.6; transform: translateX(-1%); }
  to   { opacity: 1.0; transform: translateX(1%); }
}
```

Stars in the sky could subtly twinkle:
```css
/* Overlay a subtle noise/star texture on the sky portion */
#sky-overlay {
  position: fixed;
  top: 0; left: 0;
  width: 100%; height: 50%;
  background: url('data:image/svg+xml,...'); /* small dot pattern */
  opacity: 0.04;
  animation: starTwinkle 4s ease-in-out infinite alternate;
}
```

---

## 12. Color Palette Reference

| Name | Hex | Usage |
|---|---|---|
| Castle stone | #3D3D3D | Base tone reference |
| Night sky | #0A0A14 | Background fallback |
| Moon white | #FFFDE7 | Moon pulse color |
| Phonk purple | #7C3AED | Primary accent, window |
| Blood red | #DC2626 | Bass windows, high volume arc |
| Molten gold | #D97706 | Mid-energy windows |
| Electric blue | #2563EB | High frequency windows |
| Emerald | #059669 | Mid windows |
| Crimson | #EF4444 | Beat drop flare |
| Cyan | #06B6D4 | High freq windows |
| Spider red | #DC2626 | Spider-Man mode accent |

---

## 13. Final Checklist for Builder

Before shipping, verify:

- [ ] Castle image fills full viewport on all screen sizes
- [ ] Window overlays align with actual castle windows in the PNG
- [ ] No hand = guaranteed silence + dark windows (test by covering camera)
- [ ] Volume arc appears only during tight rotation, not general movement
- [ ] Spider-Man pose triggers only after full 1.5s hold
- [ ] Crossfade between tracks is smooth (no audio click)
- [ ] Beat drops cause visible window flare
- [ ] Moon pulses on bass hits
- [ ] Webcam preview is mirrored
- [ ] Track name fades in/out with music
- [ ] City overlay appears fully in Spider-Man mode
- [ ] App runs from local server (not file://) due to audio CORS