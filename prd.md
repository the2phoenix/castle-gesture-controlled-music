# PRD — Castle Phonk: Hand-Gesture Music Interface
**Version:** 1.0  
**Author:** Arnesh  
**Status:** Ready for Build  

---

## 1. Overview

Castle Phonk is a browser-based interactive experience where a photorealistic gothic castle serves as the UI canvas. The user controls music playback entirely through hand gestures captured via webcam. A machine learning model reads the energy, speed, and character of hand movements in real time and matches them to phonk tracks with corresponding energy profiles. The castle comes alive — its windows light up in multicolored stained-glass patterns that react to the music's frequency layers.

There are no buttons, no menus, no traditional UI. The hand IS the controller. The castle IS the interface.

---

## 2. Goals

- Build a fully gesture-driven music experience with zero traditional UI interaction
- Use ML to interpret hand movement energy and match it to phonk tracks dynamically
- Make a photorealistic castle feel alive and reactive to music
- Include one Easter egg gesture (Spider-Man pose) that transforms the entire experience
- Run entirely in the browser with no backend required

---

## 3. Users

Single user. Personal project. No authentication, no accounts, no multi-user support.

---

## 4. Core Features

### 4.1 Castle Background
- Photorealistic gothic castle image (provided PNG) rendered as full-viewport background
- 16-18 window positions manually mapped as overlay elements on top of the castle image
- Windows are invisible/dark by default when no music is playing

### 4.2 Hand Tracking
- Webcam access via browser MediaPipe Hands API
- Tracks 21 hand landmarks in real time at target 30fps
- Small webcam preview shown in bottom corner (dismissible)
- If no hand is detected in frame — music stops, all windows go dark immediately
- Hand re-enters frame — system resumes reading and matching

### 4.3 ML Energy Matching (Core Intelligence)
The system reads the following movement signals from hand landmarks:

| Signal | What It Captures |
|---|---|
| Movement speed | Velocity of wrist/palm across frames |
| Smoothness vs jerkiness | Variance in velocity vector |
| Hand height on screen | Y-position normalized 0–1 |
| Directional flow | Linear, circular, chaotic trajectory |
| Finger spread | Distance between fingertips (tension) |
| Stillness duration | How long hand hasn't moved significantly |

These signals combine into a real-time **movement fingerprint** vector. Each of the 10 phonk tracks is pre-tagged with a matching energy profile vector. The system computes cosine similarity and plays the closest matching track. As movement changes, the match is re-evaluated every 3 seconds to allow smooth transitions.

### 4.4 Volume Control Gesture
- Tight wrist rotation (hand spinning around its own axis, not translating across screen) = volume control
- Clockwise = volume up
- Counter-clockwise = volume down
- Speed of rotation maps to rate of volume change
- Visual feedback: a radial arc indicator appears near the hand only while rotating, fades out after 1.5s of no rotation
- Castle windows globally dim/intensify proportional to volume level

### 4.5 Multicolored Window Lighting
- Each window has its own color personality (deep purple, electric blue, blood red, molten gold, emerald, cyan)
- Windows pulse and shift colors based on frequency analysis of the playing track using Web Audio API
- Bass frequencies drive red/orange windows
- High frequencies drive blue/violet windows
- Mid frequencies drive gold/green windows
- At beat drops, all windows flare simultaneously
- Moon in the castle image pulses subtly on bass hits

### 4.6 Spider-Man Easter Egg
- Gesture: index finger + pinky + thumb extended, middle + ring fingers curled (classic Spider-Man pose)
- Detection: held for 1.5 seconds continuously
- Effect: entire UI transforms — castle image gets a dark city skyline overlay blending through, the 90s Peter Parker Spider-Man animated series theme plays ("Spider-Man, Spider-Man, does whatever a spider can"), web-shot animation radiates from hand position across the screen
- Holding pose keeps the mode active, dropping pose returns to phonk mode after 3s fade

### 4.7 No-Hand State
- When hand leaves frame: music fades out over 1.5s, all windows dim to black, castle returns to silent moonlit state
- No hand = no music. Always.

---

## 5. Audio System

- 10 user-provided phonk MP3 files loaded at startup
- Each track pre-tagged with energy profile (see TRD for schema)
- Web Audio API for frequency analysis and beat detection
- Smooth crossfade (2s) between tracks when match changes
- Volume range: 0–100%, default 60%

---

## 6. Out of Scope

- Mobile support (desktop only, needs webcam)
- Multi-user
- Saving/recording sessions
- Online streaming — all audio files are local
- Any backend, database, or authentication

---

## 7. Success Criteria

- Hand tracked smoothly at 30fps with no perceptible lag
- Correct track matched to movement energy within 1–2 seconds of gesture change
- Volume gesture distinguishable from general movement without false triggers
- Spider-Man pose triggers reliably after 1.5s hold
- Windows visually react to music in a way that feels alive and synchronized
- No hand = guaranteed silence, every time

---

## 8. Phonk Track Energy Profile Tags

Each of the 10 MP3s must be tagged before build. Use this schema:

```json
{
  "id": 1,
  "filename": "track01.mp3",
  "tempo": 0.3,         // 0 = very slow, 1 = very fast
  "aggression": 0.8,    // 0 = calm, 1 = intense
  "darkness": 0.9,      // 0 = melodic/light, 1 = dark/heavy
  "chaos": 0.4          // 0 = hypnotic/repetitive, 1 = unpredictable
}
```

These 4 axes form a vector. Hand movement is scored on the same 4 axes in real time. Cosine similarity finds the best match.