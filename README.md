# 🏰 Castle Phonk: Gothic Hand-Gesture Music Interface

A browser-based interactive ritual space where a photorealistic gothic castle serves as your musical canvas. The user controls phonk music playback, tracks, and audio volume purely through webcam hand gestures—no buttons, no traditional UI. The hand is your controller; the castle is your interface.

---

## ⚡ Features

- **🏰 Alive Castle Canvas:** 18 stained-glass windows pulse, flare, and shift colors to real-time Web Audio frequency bands (bass drives reds/oranges, mids drive golds/emeralds, highs drive blues/purples).
- **🖐️ Real-Time 60 FPS Hand Tracking:** MediaPipe Hands tracks 21 hand landmarks directly in the browser with 100% offline, local WebAssembly neural network models.
- **🧠 ML Energy Matching:** Continuous movement vector engine analyzes speed, jerkiness/smoothness, hand height, and finger spread to match you with one of 10 phonk tracks using cosine similarity.
- **🎛️ Wrist-Rotation Volume Dial:** Tight rotation of your wrist acts as a physical knob—clockwise turns the volume up, counter-clockwise turns it down, complete with a glowing radial HUD arc.
- **🕷️ Spider-Man Easter Egg Pose:** Hold the classic Spider-Man web-shooter pose (🤘 index + pinky + thumb extended, middle + ring curled) for 1.5s to trigger a dark city overlay, radiating web shots, and the 90s Spider-Man theme.
- **🔇 No-Hand State:** When your hand leaves the camera frame, music smoothly fades out and windows dim to dark moonlight silence.
- **⚔️ Castle Tracks HUD:** Collapsible top-left panel displaying all 10 tracks, energy tags, and real-time active song indicator.
- **🚀 100% Local & Offline-Ready:** All audio files, MediaPipe neural network weights, and Tone.js libraries run locally without external CDN dependencies.

---

## 🎵 Tracklist

1. **Track 01 — Dark Ambience** *(Slow Dark)*
2. **Track 02 — Montagem Supersonic** *(Hard Phonk)*
3. **Track 03 — Montagem Bravo** *(Aggressive)*
4. **Track 04 — Midnight Slump** *(Deep Slow)*
5. **Track 05 — Hypnotic Phonk** *(Hypnotic)*
6. **Track 06 — Cala Boca** *(Brazilian)*
7. **Track 07 — Neon Velocity** *(Fast Beat)*
8. **Track 08 — Shadow Realm** *(Gothic Pace)*
9. **Track 09 — Cyber Mayhem** *(Chaotic)*
10. **Track 10 — Gothic Drop** *(Bass Heavy)*
- **Easter Egg:** *Spider-Man 90s Animated Series Theme*

---

## 🚀 Quick Start

Because modern browsers require same-origin serving for Web Audio and MediaPipe WASM assets, run via any local HTTP server:

### Using Python:
```bash
python -m http.server 8000
```

### Using Node.js:
```bash
npx serve .
```

Open **`http://localhost:8000`** in Google Chrome or Microsoft Edge, click **ENTER THE CASTLE**, and begin performing.

---

## 🛠️ Tech Stack

- **Vanilla HTML5 / Modern CSS3** (Gothic Cinzel typography, mix-blend-mode shaders, radial window glows)
- **MediaPipe Hands** (21 3D hand landmarks, local WebAssembly SIMD runtime)
- **Tone.js & Web Audio API** (Polyphonic audio players, AnalyserNode FFT, beat drop detection)
- **Cosine Similarity Engine** (Real-time 4D movement vector fingerprinting)

---

## 📜 Documentation

- [`prd.md`](prd.md) — Product Requirements Document
- [`trd.md`](trd.md) — Technical Requirements Document
- [`gsd.md`](gsd.md) — Gestural System Design
- [`uiux.md`](uiux.md) — Visual & Motion Design Specification
