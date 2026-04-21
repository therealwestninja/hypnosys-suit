# Technical Architecture

Detailed technical reference for the Hypnosis Script Generator.

---

## File Structure

```
perchance_2.txt          # The complete generator (single file, ~3900 lines)
├─ <style>               # All CSS (~700 lines)
├─ HTML body             # Wizard, session, post-session, modals, live panel
└─ <script>              # All JavaScript (~2800 lines)
   ├─ IDB Storage        # IndexedDB wrapper
   ├─ Toast System       # Notification toasts
   ├─ Data Arrays        # METHODS, PERSONAS, INTENTIONS, PACING_RULES
   ├─ State              # Global state object
   ├─ Wizard Logic       # Step validation, navigation, rendering
   ├─ Soundscape Engine  # SoundscapeEngine class
   ├─ Visual Renderers   # 16 Canvas 2D render functions
   ├─ AI Integration     # craftSuggestion(), generateFullScript()
   ├─ Session Player     # startSession(), speakScript(), speak()
   ├─ Export/Share        # Script download, URL sharing, backup
   ├─ Event Handlers     # ~65 addEventListener calls
   └─ Init               # Async startup
```

---

## State Management

All application state lives in a single `state` object:

```js
const state = {
  // Wizard
  currentStep: 1,
  stepsCompleted: { 1: false, 2: false, 3: false, 4: false, 5: false },
  
  // Selections
  personas: [],                    // BUILTIN + custom
  selectedPersonaId: 'driver',
  selectedMethodId: 'highway',
  intentionType: 'trigger',
  
  // AI output
  craftedSuggestion: null,         // JSON from crafter
  fullScript: null,                // Raw script text
  parsedScript: null,              // { settling, induction, deepener, work, emergence }
  
  // Configure
  droneEnabled: true,
  soundscape: 'drone',
  length: 'medium',
  voice: null,                     // SpeechSynthesisVoice
  speechRate: 0.65,
  voicePitch: 0.85,
  voiceVolume: 1.0,
  droneVolume: 0.35,
  vizOverride: 'auto',
  hud: { fixationDot, textDisplay, phaseDots, phaseLabel, progressBar, sessionTime, labels },
  
  // Session runtime (set during playback)
  paused: false,
  abortFlag: false,
  _sessionStartTime: null,
  _currentPhase: null,
  _timeInterval: null,
  _wakeLock: null,
  _doubleInduction: false,
  _secondVoice: null,
  _preMood: null,
  _cooldownUntil: null,
  
  // UI
  editingPersonaId: null,
  personaFilter: 'all',
  methodFilter: 'all',
};
```

State is never persisted directly. Custom personas and the script library are stored in IndexedDB via `idbGet`/`idbSet`. Session history is stored separately under the key `stillness_history_v1`.

---

## IndexedDB Storage

A lightweight async wrapper around IndexedDB, adapted from the Adaptive Session Studio:

```
Database: 'stillness-db' (version 1)
Object Store: 'kv' (key-value)

Keys:
  'stillness_personas_v1'  → Array<Persona>    — custom personas
  'stillness_library_v1'   → Array<LibEntry>   — saved scripts
  'stillness_history_v1'   → Array<Session>    — session history (max 50)
```

On first load, `_initStorage()` checks IDB for each key. If empty, it migrates from `localStorage` (the v1 storage backend), then deletes the localStorage entry. It also calls `navigator.storage.persist()`.

---

## SoundscapeEngine

Replaces the original `AudioDrone`. Supports 10 procedural ambient types, all synthesised with the Web Audio API.

### Audio Graph

```
Source(s) → SceneGain → DuckGain → DynamicsCompressor → Destination
                          ↑
                    duck()/unduck()
                    (0.2 during speech,
                     1.0 between)
```

### Synthesis Recipes

| Type | Technique |
|---|---|
| `drone` | Bass sine 70Hz + harmonic 141Hz + brown noise through lowpass 280Hz + LFO on bass gain |
| `rain` | Pink noise bandpass 1.2kHz + brown rumble lowpass 200Hz + Poisson-scheduled sine droplets 2-6kHz |
| `ocean` | Bandpass noise 650Hz with 1/11 Hz sine LFO on gain + foam bandpass 3kHz |
| `fire` | Brown noise lowpass 400Hz + scheduled bandpass-filtered 8ms bursts at random intervals |
| `bowls` | Additive synthesis: partials at ratios 1:2.02:3.07:4.16:5.38 with 10-3s exponential decays, periodic re-strikes |
| `heartbeat` | Dual sine thumps at 55Hz 140ms apart through lowpass 200Hz, ~1.05s period |
| `binaural` | Two oscillators at carrier±3Hz via ChannelMergerNode (not StereoPanner) |
| `pink` | White noise through lowpass 800Hz |
| `brown` | Leaky-integrated white: `(last + 0.02*w) / 1.02` |
| `white` | 2-second AudioBuffer of `random()*2-1`, looped |

### Phase Modulation

For the `drone` type, `setPhaseFreq(phase)` ramps the bass frequency:
- settling: 70Hz
- induction: 67Hz  
- deepener: 63Hz
- work: 60Hz
- emergence: 70Hz

### Ducking

`duck()` ramps the duck gain to 0.2 over 400ms. `unduck()` ramps back to 1.0 over 800ms. Called from `speakScript()` around each spoken chunk.

---

## Visual Renderers

All renderers share the signature `(ctx, W, H, t, color)` where `t` is time in seconds multiplied by the phase speed.

### Renderer Registry

```js
const VIZ_RENDERERS = {
  tunnel, 'pendulum-viz', vortex, mandala, lissajous, colorwash,
  'canvas-pulse', breathing, moire, starfield, candle,
  flowfield, plasma, 'kaleidoscope-flow', 'kaleidoscope-plasma'
};
```

Plus three non-Canvas types handled by `setupVisual()`: `highway` (Canvas with custom loop), `spiral` (SVG), `pulse` (CSS animation).

### Kaleidoscope Meta-Effect

`makeKaleidoscope(subRenderer, folds)` returns a new renderer that:
1. Renders the sub-effect to an OffscreenCanvas
2. Draws N triangular clip-wedges with alternating `scale(-1,1)` mirrors
3. Composites into the main canvas

### Performance

- Canvas contexts created with `{alpha: false}` where applicable.
- `fillRect` with low alpha instead of `clearRect` for trail effects.
- Plasma renders at 192px width with precomputed 1024-entry sine LUT, then scales up.
- Flow-field uses object pools (250 mobile, 600 desktop) — no allocation during animation.
- Starfield uses `fillRect` for star points (4× faster than `arc`+`fill`).
- All renderers check `state.abortFlag` and bail out.
- `ResizeObserver` on all canvases handles viewport changes.

---

## TTS Architecture

### Sentence Chunking

`speakScript(text)` splits on `[PAUSE N]` markers, then splits text segments on sentence boundaries (`/(?<=[.!?])\s+/`). Each sentence is spoken individually with ducking around it.

### Chrome Watchdog

`speak(text)` checks if `speechSynthesis.speaking` and force-cancels before each utterance to reset Chrome's internal timer. `_doSpeak` adds two timeouts:

1. **Soft watchdog**: `text.length * 200ms + 8s` — if `!speaking`, resolve early.
2. **Hard ceiling**: `text.length * 250ms + 15s` — always resolve.

### Word Highlighting

`_doSpeak` attaches `utterance.onboundary` which updates `#textDisplay` with three spans: before (0.35 opacity), current word (full opacity), after (0.35 opacity). Falls back to sentence-level display on browsers where `onboundary` doesn't fire.

### Double Induction

When `state._doubleInduction` is true, `_doSpeak` fires a second `SpeechSynthesisUtterance` simultaneously with `state._secondVoice` at 0.92× rate and 0.75 pitch.

### Voice Quality Scoring

```
+100  neural|natural|online
+90   enhanced|premium
+85   siri
+50   google (remote)
+10   male (not female)
+5    uk|british|gb
+4    daniel|oliver|arthur|james|aaron
+2    localService
-50   espeak|compact
```

---

## AI Integration

### Perchance ai-text-plugin

Both AI calls use `root.aiTextPlugin()`:

```js
const data = await root.aiTextPlugin({
  instruction: '...',         // System prompt
  startWith: '...',           // Text the AI continues from
  stopSequences: [...],       // Stop generation triggers
  hideStartWith: true,        // Don't echo startWith in output
});
```

### Craft Suggestion

- Instruction: master hypnotherapist, intention type, universal rules, type-specific guidance.
- startWith: `{` (forces JSON output).
- Output: JSON with `craftedSuggestion`, `trigger`, `coreResponse`, `reinforcementPhrases`, `explanation`.

### Generate Full Script

- Instruction: persona voice + method detail + pacing rules + crafted suggestion + phase structure.
- startWith: `===SETTLING===`.
- Output: five-phase script with `===PHASE===` markers and `[PAUSE N]` timing markers.

### Error Handling

Both calls check `data.stopReason === 'error'` before using output. JSON parsing in the crafter is wrapped in try/catch with toast notifications on failure.

---

## Session Lifecycle

```
startSession()
  ├─ Cooldown check
  ├─ TTS availability check
  ├─ Parse script
  ├─ Set up double induction voices (if applicable)
  ├─ Switch to session screen
  ├─ Apply HUD toggles
  ├─ Set up visual (override or method default)
  ├─ Populate transcript
  ├─ Start soundscape
  ├─ Acquire Wake Lock
  ├─ Show content warning (if advanced/authoritarian)
  ├─ Start time tracker
  └─ For each phase:
       ├─ Update phase dots, progress bar, indicator
       ├─ Modulate drone frequency
       ├─ speakScript(phaseText)
       │    └─ For each sentence:
       │         ├─ duck()
       │         ├─ speak(chunk) → _doSpeak() → onboundary highlighting
       │         └─ unduck()
       └─ interruptibleSleep(pauseMs)
  
finishSession(wasEmergency)
  ├─ Close live panel
  ├─ Stop recording
  ├─ Stop drone, visuals, speechSynthesis
  ├─ Release Wake Lock
  ├─ Save session to history (with preMood)
  ├─ Build post-session summary
  ├─ Show trigger/suggestion reminder
  ├─ Show grounding exercise (if emergency)
  ├─ Set cooldown (if emergency)
  └─ Reset mood selection

emergencyEnd()
  ├─ Set abortFlag
  ├─ Cancel speechSynthesis
  ├─ Pick persona-matched re-alert script
  └─ speakScript(reAlert, bypassAbort=true) → finishSession(true)
```

---

## URL Config Sharing

Session configuration is serialised as a compact JSON object:

```js
{ p: personaId, m: methodId, i: intentionType, l: length, 
  s: soundscape, r: speechRate, t: intentionText }
```

Compressed via `CompressionStream('gzip')`, base64-encoded, and appended to the URL as `#cfg=...`. On load, `loadSharedConfig()` decompresses, validates all fields against known IDs, applies to state, and cleans the URL.

---

## Backup Format

```json
{
  "version": 2,
  "exportedAt": 1713700000000,
  "personas": [{ "id": "custom_123", "name": "My Guide", ... }],
  "library": [{ "id": "lib_456", "name": "Sleep session", "script": "...", ... }],
  "history": [{ "id": "sess_789", "date": 1713700000000, "guide": "The Therapist", ... }]
}
```

Import merges by ID — existing records are kept, new records are added. History is capped at 50 and sorted by date descending.
