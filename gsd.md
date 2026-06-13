# GSD — Castle Phonk: Gestural System Design
**Version:** 1.0  
**Author:** Arnesh  
**Purpose:** Complete specification of all gesture behaviors, states, and edge cases  

---

## 1. Gesture System Philosophy

There are two types of gestures in this system:

| Type | Description | Examples |
|---|---|---|
| **Continuous / Expressive** | No fixed shape. The ML reads movement energy over time and responds fluidly | Phonk matching, Volume rotation |
| **Discrete / Pose** | A specific hand shape held for a duration triggers a one-time event | Spider-Man Easter egg |

The phonk selection is **never triggered by a specific gesture**. It is always a continuous ML interpretation of how the hand is moving. This means the user never has to learn anything — they just move naturally and the system responds.

---

## 2. System States

```
┌─────────────────────────────────────────────────────────┐
│                    SYSTEM STATES                         │
│                                                          │
│  [IDLE]          No hand in frame                        │
│     │                                                    │
│     ▼  hand enters frame                                 │
│  [READING]       Hand present, analyzing movement        │
│     │                                                    │
│     ├──► [PLAYING]     Match found, phonk playing        │
│     │         │                                          │
│     │         ├──► [ROTATING]   Volume gesture active    │
│     │         │         │                                │
│     │         │         └──► [PLAYING]  rotation ends    │
│     │         │                                          │
│     │         └──► [SPIDER-MAN] Pose held 1.5s           │
│     │                   │                                │
│     │                   └──► [PLAYING]  pose dropped     │
│     │                                                    │
│     └──► [IDLE]  hand leaves frame                       │
└─────────────────────────────────────────────────────────┘
```

---

## 3. Gesture Specifications

---

### 3.1 No Hand — IDLE State

| Property | Value |
|---|---|
| Trigger | Hand landmarks not detected for > 500ms |
| Audio | Fade out over 1500ms → silence |
| Windows | Dim to black over 1500ms |
| Castle | Returns to default moonlit state |
| Resume | Instant on hand re-entry — re-reads movement, re-matches track |
| Edge case | Brief occlusions (< 500ms) should NOT trigger idle to prevent flicker |

---

### 3.2 Continuous Movement → Phonk Matching

| Property | Value |
|---|---|
| Trigger | Hand present and moving |
| Input | 30 frames of position/landmark history |
| Output | 4-axis energy vector → cosine similarity match |
| Re-evaluation | Every 3000ms |
| Switch threshold | Similarity delta must be > 0.15 to switch tracks |
| Crossfade | 2000ms smooth crossfade between tracks |
| No fixed gesture | ANY hand shape, any movement style |

**Movement → Energy Axis Mapping:**

| Movement Character | tempo | aggression | darkness | chaos |
|---|---|---|---|---|
| Slow, smooth, low position | low | low | high | low |
| Fast, jerky, high position | high | high | low | high |
| Fast, smooth, high | high | low | low | medium |
| Slow, jerky, low | low | medium | high | medium |
| Circular, medium pace | medium | low | medium | low |
| Rapid side-to-side | high | high | medium | high |
| Fingers spread wide, still | low | low | medium | low |
| Fingers tight, fast | high | high | high | medium |

---

### 3.3 Volume Rotation Gesture

| Property | Value |
|---|---|
| Gesture | Wrist rotating around its own axis (like adjusting a dial) |
| Detection | Angle delta between wrist→palm vector across consecutive frames |
| Direction | Clockwise = volume UP / Counter-clockwise = volume DOWN |
| Activation condition | Translation < 0.03 units/frame AND angle delta > 0.04 rad/frame |
| Rate | Fast rotation = fast volume change. Slow = fine adjustment |
| Range | 0% to 100% volume |
| Deactivation | When rotation stops (delta falls below threshold for 10 frames) |
| Visual feedback | Radial arc appears near hand, fades after 1500ms of inactivity |
| Window feedback | All windows scale brightness proportional to volume |
| Conflict with phonk | Volume rotation does NOT change track matching — runs in parallel |

**Rotation Detection Guard:**
The key distinction from general movement — if the wrist is ALSO translating across the screen, it's expressive movement, not a volume gesture. Only a wrist spinning in relative place counts.

---

### 3.4 Spider-Man Easter Egg

| Property | Value |
|---|---|
| Gesture | Thumb + index + pinky extended. Middle + ring fingers fully curled |
| Hold duration | 1500ms (45 frames at 30fps) |
| Confidence required | All 5 finger conditions true for every frame of hold |
| Visual progress | Faint web-like ring grows around hand position during hold |
| Trigger | On 45th consecutive positive frame |
| Audio | Crossfades immediately to spiderman.mp3 |
| Visual | City skyline overlay fades in over castle (opacity 0 → 0.6 over 2s) |
| Web animation | Web shot lines radiate from hand position outward across screen |
| Deactivation | Drop pose → after 3000ms fade → returns to phonk mode |
| Re-trigger | Must drop and re-hold to trigger again |

**Finger Extension Logic:**

```
Thumb extended:   dist(TIP[4], WRIST[0]) > 0.35
Index extended:   TIP[8].y < MCP[5].y - 0.06
Middle curled:    TIP[12].y > MCP[9].y - 0.02
Ring curled:      TIP[16].y > MCP[13].y - 0.02
Pinky extended:   TIP[20].y < MCP[17].y - 0.06
```

---

## 4. Gesture Priority & Conflict Resolution

When multiple gesture conditions could be true simultaneously:

| Priority | Gesture | Overrides |
|---|---|---|
| 1 (highest) | Spider-Man pose | Everything |
| 2 | Volume rotation | Phonk matching (runs parallel, doesn't change track) |
| 3 | Phonk movement matching | Default always-on |
| — | No hand | Kills everything |

Spider-Man check runs first every frame. If true, skip all other gesture processing.

---

## 5. Gesture Feedback Summary

| Gesture | Audio Feedback | Visual Feedback |
|---|---|---|
| Hand enters frame | Track begins fading in | Windows begin warming up |
| Hand exits frame | Track fades to silence | Windows dim to black |
| Movement energy changes | Track crossfades over 2s | Windows shift color intensity |
| Volume rotate clockwise | Volume increases | Arc grows, windows brighten |
| Volume rotate counter-CW | Volume decreases | Arc shrinks, windows dim |
| Spider-Man hold (progress) | None | Web ring grows at hand |
| Spider-Man triggered | 90s Peter Parker animated series theme plays | City overlay + web shot |
| Spider-Man released | Phonk fades back in | City overlay fades out |

---

## 6. Edge Cases & Failure Handling

| Scenario | Behavior |
|---|---|
| Webcam permission denied | Full-screen prompt explaining camera is required, no castle shown |
| Audio files not found | Console error, track skipped, system continues with available tracks |
| Only 1 hand tracked (correct) | Normal operation |
| Hand partially out of frame | Continue tracking with available landmarks, may reduce confidence |
| Very low light / poor detection | Falls back to IDLE state after 500ms of failed detection |
| Rapid hand re-entry/exit | Debounce 500ms before triggering idle state changes |
| Spider-Man pose breaks mid-hold | Counter resets to 0, must start hold again |
| All tracks identical energy | System picks track 1 as tiebreaker |