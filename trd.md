# TRD — Castle Phonk: Technical Requirements Document
**Version:** 1.0  
**Author:** Arnesh  
**Stack:** Vanilla HTML/CSS/JS, Single File, No Build Step  

---

## 1. Tech Stack

| Layer | Technology | Reason |
|---|---|---|
| Hand Tracking | MediaPipe Hands (CDN) | 21-landmark real-time detection, browser-native |
| Audio Playback | Tone.js (CDN) | Precise scheduling, crossfading, Web Audio API wrapper |
| Frequency Analysis | Web Audio API (native) | AnalyserNode for FFT, beat detection |
| ML Matching | Pure JS (cosine similarity) | No model needed — vector math on 4-axis profiles |
| Rendering | HTML/CSS + Canvas overlay | Castle = img tag, windows = positioned divs, effects = canvas |
| Camera | MediaPipe Camera Utils | Handles stream + frame loop cleanly |
| Deployment | Single HTML file | Drop in folder, open in browser, done |

---

## 2. File Structure

```
/castle-phonk/
│
├── index.html              ← entire app lives here
├── castle.png              ← the gothic castle image (provided)
├── spiderman-city.png      ← city overlay for easter egg (sourced separately)
│
└── /audio/
    ├── track01.mp3
    ├── track02.mp3
    ├── track03.mp3
    ├── track04.mp3
    ├── track05.mp3
    ├── track06.mp3
    ├── track07.mp3
    ├── track08.mp3
    ├── track09.mp3
    ├── track10.mp3
    └── spiderman.mp3
```

---

## 3. MediaPipe Hands Setup

```js
// CDN imports required
// https://cdn.jsdelivr.net/npm/@mediapipe/hands/hands.js
// https://cdn.jsdelivr.net/npm/@mediapipe/camera_utils/camera_utils.js

const hands = new Hands({
  locateFile: (file) => `https://cdn.jsdelivr.net/npm/@mediapipe/hands/${file}`
});

hands.setOptions({
  maxNumHands: 1,
  modelComplexity: 1,
  minDetectionConfidence: 0.7,
  minTrackingConfidence: 0.6
});

hands.onResults(onHandResults);

const camera = new Camera(videoElement, {
  onFrame: async () => await hands.send({ image: videoElement }),
  width: 640,
  height: 480
});
camera.start();
```

### 3.1 Landmark Reference

Key landmarks used:

| Index | Landmark | Used For |
|---|---|---|
| 0 | WRIST | Base position, rotation detection |
| 4 | THUMB_TIP | Spider-Man pose |
| 8 | INDEX_TIP | Finger spread, Spider-Man pose |
| 12 | MIDDLE_TIP | Finger spread |
| 16 | RING_TIP | Spider-Man pose (curled check) |
| 20 | PINKY_TIP | Finger spread, Spider-Man pose |

---

## 4. Movement Fingerprint Engine

### 4.1 Signal Extraction (runs every frame ~33ms)

```js
// Maintain rolling buffer of last 30 frames of wrist position
const positionBuffer = []; // { x, y, timestamp }[]

function extractMovementFingerprint(landmarks) {
  const wrist = landmarks[0];
  const now = Date.now();

  positionBuffer.push({ x: wrist.x, y: wrist.y, t: now });
  if (positionBuffer.length > 30) positionBuffer.shift();

  const speed = computeSpeed(positionBuffer);          // 0–1
  const smoothness = computeSmoothness(positionBuffer); // 0–1
  const height = 1 - wrist.y;                          // 0=bottom 1=top
  const spread = computeFingerSpread(landmarks);        // 0–1

  return {
    tempo: speed,
    aggression: speed * (1 - smoothness),
    darkness: 1 - height,
    chaos: 1 - smoothness
  };
}
```

### 4.2 Cosine Similarity Matching

```js
const trackProfiles = [
  { id: 1, filename: "track01.mp3", tempo: 0.2, aggression: 0.3, darkness: 0.8, chaos: 0.2 },
  { id: 2, filename: "track02.mp3", tempo: 0.9, aggression: 0.9, darkness: 0.7, chaos: 0.8 },
  { id: 3, filename: "track03.mp3", tempo: 0.5, aggression: 0.5, darkness: 0.6, chaos: 0.4 },
  { id: 4, filename: "track04.mp3", tempo: 0.1, aggression: 0.2, darkness: 0.9, chaos: 0.1 },
  { id: 5, filename: "track05.mp3", tempo: 0.7, aggression: 0.6, darkness: 0.5, chaos: 0.6 },
  { id: 6, filename: "track06.mp3", tempo: 0.4, aggression: 0.7, darkness: 0.8, chaos: 0.5 },
  { id: 7, filename: "track07.mp3", tempo: 0.8, aggression: 0.8, darkness: 0.4, chaos: 0.7 },
  { id: 8, filename: "track08.mp3", tempo: 0.3, aggression: 0.4, darkness: 0.7, chaos: 0.3 },
  { id: 9, filename: "track09.mp3", tempo: 0.6, aggression: 0.5, darkness: 0.6, chaos: 0.9 },
  { id: 10, filename: "track10.mp3", tempo: 0.5, aggression: 0.6, darkness: 0.9, chaos: 0.5 }
];

function cosineSimilarity(a, b) {
  const keys = ['tempo', 'aggression', 'darkness', 'chaos'];
  const dot = keys.reduce((sum, k) => sum + a[k] * b[k], 0);
  const magA = Math.sqrt(keys.reduce((sum, k) => sum + a[k] ** 2, 0));
  const magB = Math.sqrt(keys.reduce((sum, k) => sum + b[k] ** 2, 0));
  return dot / (magA * magB);
}

function getBestMatch(fingerprint) {
  return trackProfiles.reduce((best, track) => {
    const score = cosineSimilarity(fingerprint, track);
    return score > best.score ? { track, score } : best;
  }, { track: null, score: -1 }).track;
}
```

### 4.3 Match Re-evaluation Throttle

- Re-evaluate best match every **3000ms** to avoid constant track switching
- If best match changes AND similarity delta > 0.15 → trigger crossfade to new track
- If similarity delta < 0.15 → stay on current track (prevents flickering)

---

## 5. Volume Rotation Gesture

### 5.1 Detection Logic

```js
// Track wrist angle relative to palm center over time
function detectWristRotation(landmarks, prevLandmarks) {
  if (!prevLandmarks) return 0;

  const wrist = landmarks[0];
  const middleMCP = landmarks[9]; // palm center reference

  // Compute angle of wrist→middleMCP vector
  const angle = Math.atan2(
    middleMCP.y - wrist.y,
    middleMCP.x - wrist.x
  );
  const prevAngle = Math.atan2(
    prevLandmarks[9].y - prevLandmarks[0].y,
    prevLandmarks[9].x - prevLandmarks[0].x
  );

  let delta = angle - prevAngle;
  // Normalize to -PI..PI
  if (delta > Math.PI) delta -= 2 * Math.PI;
  if (delta < -Math.PI) delta += 2 * Math.PI;

  return delta; // positive = clockwise, negative = counter-clockwise
}
```

### 5.2 Distinguishing Rotation from Translation

- Rotation gesture is ONLY triggered when:
  - Wrist translation (x,y movement) is **below threshold** (< 0.03 normalized units/frame)
  - Angle delta is **above threshold** (> 0.04 radians/frame)
- This ensures moving the hand around doesn't accidentally trigger volume changes

---

## 6. Spider-Man Gesture Detection

```js
function isSpiderManPose(landmarks) {
  // Thumb extended: tip far from wrist
  const thumbExtended = dist(landmarks[4], landmarks[0]) > 0.35;

  // Index extended: tip far above MCP
  const indexExtended = landmarks[8].y < landmarks[5].y - 0.06;

  // Middle curled: tip close to palm
  const middleCurled = landmarks[12].y > landmarks[9].y - 0.02;

  // Ring curled: tip close to palm
  const ringCurled = landmarks[16].y > landmarks[13].y - 0.02;

  // Pinky extended: tip far above MCP
  const pinkyExtended = landmarks[20].y < landmarks[17].y - 0.06;

  return thumbExtended && indexExtended && middleCurled && ringCurled && pinkyExtended;
}

// Hold detection: must be true for 45 consecutive frames (~1.5s at 30fps)
let spiderManCounter = 0;
const SPIDER_MAN_THRESHOLD = 45;
```

---

## 7. Audio System

### 7.1 Tone.js Setup

```js
const players = {};
trackProfiles.forEach(track => {
  players[track.id] = new Tone.Player(`audio/${track.filename}`).toDestination();
});

// Spider-Man track
const spiderPlayer = new Tone.Player("audio/spiderman.mp3").toDestination();

// Master volume
const masterVol = new Tone.Volume(-6).toDestination();
// All players connect through masterVol

// Crossfade between tracks
function crossfadeTo(newTrackId) {
  if (currentTrackId === newTrackId) return;
  const fadeOut = players[currentTrackId];
  const fadeIn = players[newTrackId];
  // Ramp volume down on old, up on new over 2s
  fadeOut?.volume.rampTo(-60, 2);
  setTimeout(() => fadeOut?.stop(), 2000);
  fadeIn.volume.value = -60;
  fadeIn.start();
  fadeIn.volume.rampTo(0, 2);
  currentTrackId = newTrackId;
}
```

### 7.2 Frequency Analysis

```js
const analyser = Tone.context.createAnalyser();
analyser.fftSize = 256;
Tone.Destination.connect(analyser);

const dataArray = new Uint8Array(analyser.frequencyBinCount);

function getFrequencyBands() {
  analyser.getByteFrequencyData(dataArray);
  const bass = average(dataArray.slice(0, 10));     // 0–300Hz
  const mid = average(dataArray.slice(10, 40));      // 300Hz–2kHz
  const high = average(dataArray.slice(40, 128));    // 2kHz+
  return { bass, mid, high }; // each 0–255
}
```

---

## 8. Window Mapping

18 window positions are hardcoded as percentage coordinates relative to castle image dimensions. Each window element is an absolutely positioned div with:
- `border-radius` matching the gothic arch shape
- `mix-blend-mode: screen` for a natural glow effect over the stone texture
- `filter: blur(4px)` for soft light bleed

```js
const WINDOWS = [
  // Ground floor - left wing
  { id: 'w1', x: 8.5, y: 52, w: 5.5, h: 10, color: '#7C3AED' },
  { id: 'w2', x: 15.0, y: 52, w: 5.5, h: 10, color: '#DC2626' },
  { id: 'w3', x: 21.5, y: 52, w: 5.5, h: 10, color: '#D97706' },
  // Ground floor - right wing
  { id: 'w4', x: 63.5, y: 52, w: 5.5, h: 10, color: '#059669' },
  { id: 'w5', x: 70.0, y: 52, w: 5.5, h: 10, color: '#2563EB' },
  { id: 'w6', x: 76.5, y: 52, w: 5.5, h: 10, color: '#9333EA' },
  // First floor - left wing
  { id: 'w7', x: 8.5, y: 36, w: 5.5, h: 11, color: '#EF4444' },
  { id: 'w8', x: 15.0, y: 36, w: 5.5, h: 11, color: '#F59E0B' },
  { id: 'w9', x: 21.5, y: 36, w: 5.5, h: 11, color: '#8B5CF6' },
  // First floor - right wing
  { id: 'w10', x: 63.5, y: 36, w: 5.5, h: 11, color: '#06B6D4' },
  { id: 'w11', x: 70.0, y: 36, w: 5.5, h: 11, color: '#10B981' },
  { id: 'w12', x: 76.5, y: 36, w: 5.5, h: 11, color: '#F97316' },
  // Tower windows - left tower
  { id: 'w13', x: 4.0, y: 24, w: 4.0, h: 7, color: '#A855F7' },
  { id: 'w14', x: 9.5, y: 24, w: 4.0, h: 7, color: '#EC4899' },
  // Tower windows - right tower
  { id: 'w15', x: 76.5, y: 24, w: 4.0, h: 7, color: '#14B8A6' },
  { id: 'w16', x: 82.0, y: 24, w: 4.0, h: 7, color: '#6366F1' },
  // Upper center windows
  { id: 'w17', x: 44.0, y: 38, w: 4.5, h: 8, color: '#F59E0B' },
  { id: 'w18', x: 52.0, y: 38, w: 4.5, h: 8, color: '#EF4444' },
];
```

> **Note:** These coordinates are approximate. Claude building the app should fine-tune these by rendering the castle image and visually adjusting overlays until they align perfectly with the actual window positions in the PNG.

---

## 9. Performance Targets

| Metric | Target |
|---|---|
| Hand tracking framerate | 30fps |
| Gesture → audio response | < 100ms |
| Track match re-evaluation | Every 3000ms |
| Crossfade duration | 2000ms |
| Spider-Man hold duration | 1500ms (45 frames) |
| Window animation framerate | 60fps (requestAnimationFrame) |

---

## 10. Browser Requirements

- Chrome or Edge (MediaPipe works best here)
- Webcam permission required
- Web Audio API support (all modern browsers)
- CORS: audio files must be served from same origin — run via local server (`python -m http.server` or Live Server extension), not file://