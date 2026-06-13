# SKILL.md — Castle Phonk: Build Instructions for Claude Opus
**Version:** 1.0  
**Author:** Arnesh  
**Purpose:** Tell Claude Opus exactly how to read the docs and build this correctly  

---

## Who You Are

You are Claude Opus 4.6 building a browser-based interactive experience called **Castle Phonk**. You have been given 4 documents to read before writing a single line of code:

1. `PRD.md` — What to build and why
2. `TRD.md` — Exact technical implementation with code samples
3. `GSD.md` — Every gesture, state, edge case, and behavior
4. `UIUX.md` — Complete visual, motion, and layout specification

**Read all four completely before writing any code.**

---

## What You Are Building

A single HTML file (`index.html`) that:

- Displays a photorealistic gothic castle as a full-viewport background
- Accesses the user's webcam via MediaPipe Hands
- Reads hand movement energy via ML and matches it to one of 10 phonk tracks
- Makes castle windows light up in multicolored stained-glass patterns reacting to the music
- Supports wrist rotation for volume control with a radial arc UI
- Has a Spider-Man Easter egg gesture that transforms the UI
- Goes completely silent and dark when no hand is in frame

---

## File Structure Expectation

The user will place `index.html` in a folder like this:

```
/castle-phonk/
├── index.html          ← YOU BUILD THIS
├── castle.png          ← PROVIDED — the gothic castle image
├── spiderman-city.png  ← USER WILL SOURCE — dark city night skyline
└── /audio/
    ├── track01.mp3 through track10.mp3
    └── spiderman.mp3
```

You build only `index.html`. Everything else is provided by the user.

---

## Build Order

Follow this exact sequence. Do not skip steps.

### Step 1 — HTML Structure
Set up the DOM skeleton:
- `#castle-bg` — the castle image
- `#city-overlay` — hidden city image for Spider-Man mode
- `#fog-layer` — animated fog at base
- `#sky-overlay` — subtle star twinkle
- `#moon-overlay` — moon glow overlay
- `.castle-window` × 18 — the window glow divs
- `#webcam-preview` — small camera feed
- `#overlay-canvas` — transparent fullscreen canvas for volume arc + web effects
- `#track-name` — bottom-center track label
- `<video id="input-video">` — hidden video element for MediaPipe

### Step 2 — CSS
Implement all styles from UIUX.md:
- Full viewport castle background
- Window glow system (mix-blend-mode: screen, blur, radial gradient)
- Fog and sky animations
- Webcam preview styling
- Track name typography (Cinzel font)
- City overlay blend mode
- All transitions and keyframes

### Step 3 — Audio System
- Load all 10 tracks via Tone.js Players
- Set up Web Audio AnalyserNode for FFT
- Implement crossfade function
- Implement beat detection (bass spike threshold)
- Define trackProfiles array with energy vectors (use values from TRD.md)

### Step 4 — MediaPipe Hands
- Initialize Hands with settings from TRD.md
- Set up Camera utility
- Wire onResults callback

### Step 5 — Movement Fingerprint Engine
- Implement positionBuffer (rolling 30-frame history)
- Implement extractMovementFingerprint()
- Implement cosineSimilarity()
- Implement getBestMatch()
- Implement 3000ms re-evaluation throttle with 0.15 delta threshold

### Step 6 — Gesture Detectors
- Implement detectWristRotation() with translation guard
- Implement isSpiderManPose() with all 5 finger checks
- Implement Spider-Man hold counter (45 frames)

### Step 7 — Window System
- Place 18 window divs at coordinates from TRD.md
- Implement updateWindows() using frequency bands
- Implement onBeatDrop() flare effect
- Implement moon pulse

### Step 8 — Volume Arc
- Implement drawVolumeArc() on canvas
- Wire to rotation gesture
- Implement fade-out timer (1500ms inactivity)

### Step 9 — Spider-Man Mode
- Implement city overlay transition
- Implement web shot line animation
- Implement progress ring during hold
- Implement return-to-phonk on pose release

### Step 10 — No-Hand State
- Implement 500ms debounce before triggering idle
- Implement audio fade-out (1500ms)
- Implement window dim animation
- Implement resume on hand re-entry

### Step 11 — Main Loop
Wire everything together in the onResults callback with priority order:
1. Check if hand present → if not, trigger idle
2. Check Spider-Man pose → if true, handle and return
3. Check wrist rotation → if true, handle volume
4. Run movement fingerprint → match track → update windows

---

## Critical Implementation Notes

### DO NOT do these things:
- Do not use `file://` — remind user to run a local server
- Do not use `alert()` for errors — use console.error and graceful UI fallbacks
- Do not hardcode volume changes that override the rotation gesture
- Do not switch tracks more often than every 3000ms
- Do not trigger idle state on brief occlusions (< 500ms)
- Do not use `<form>` tags anywhere
- Do not use localStorage or sessionStorage

### Window Coordinate Tuning:
The window coordinates in TRD.md are approximate. After initial render, visually inspect and fine-tune each window overlay to align precisely with the actual window positions in castle.png. This is important — misaligned windows break the illusion.

### MediaPipe CDN Dependency:
The app requires internet connection to load MediaPipe from CDN. Add a visible error state if CDN fails to load.

### Audio CORS:
Audio files must be served from same origin. Add a comment in the HTML reminding the user to use a local server:
```html
<!-- Run with: python -m http.server 8080 or VS Code Live Server -->
```

### Tone.js Startup:
Web Audio requires user gesture to start. Add a single "click to begin" overlay that initializes Tone.js context on first click, then disappears and starts the camera.

---

## Start Screen

Before the experience begins, show a minimal overlay:

```
[castle image behind, darkened]

        CASTLE PHONK

   move your hand to begin

   [click anywhere]
```

Font: Cinzel. White on dark. Fades out completely on click. Never shown again.

---

## Error States

| Error | Display |
|---|---|
| Camera permission denied | Centered text: "camera access required — refresh and allow" |
| Audio files not found | Console warning only, skip missing tracks silently |
| MediaPipe fails to load | Centered text: "hand tracking failed to load — check connection" |
| No tracks loaded | Console error, show "no audio files found in /audio/" |

---

## Definition of Done

The build is complete when:

- [ ] Castle fills full viewport, looks cinematic
- [ ] Camera feed visible in bottom-right corner
- [ ] Covering camera with hand → music stops, windows go dark
- [ ] Moving hand → music plays, windows light up
- [ ] Slow calm movement → slow dark phonk
- [ ] Fast aggressive movement → hard fast phonk
- [ ] Wrist rotation → volume arc appears, volume changes
- [ ] Spider-Man pose held 1.5s → city appears, Spider-Man theme plays
- [ ] All window glows align with actual castle windows
- [ ] No traditional UI visible during experience
- [ ] Runs on Chrome/Edge via local server

---

## Notes from Arnesh

- This is a personal project, not commercial. Audio files are sourced from YouTube downloads.
- The castle PNG is AI-generated via Gemini — front-facing gothic castle, night, fog, moon.
- The spiderman-city.png should be a dark night city skyline — Arnesh will source this.
- The Spider-Man song is the Peter Parker 90s animated series theme ("Spider-Man, Spider-Man, does whatever a spider can...") — Arnesh will provide as spiderman.mp3. Download from YouTube.
- Window coordinate fine-tuning is expected — don't consider misalignment a blocker, note it and adjust.
- If in doubt on any visual decision, refer to UIUX.md. It is the source of truth for how things look.
- If in doubt on any gesture behavior, refer to GSD.md. It is the source of truth for how gestures work.