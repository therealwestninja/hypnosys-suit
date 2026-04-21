# · Hypnosis Script Generator ·

**A free, AI-driven, fully browser-native hypnosis experience generator.**

Generate personalised induction scripts with AI, hear them narrated by the browser's text-to-speech engine, and experience guided sessions with procedural soundscapes, 16 hypnotic visual effects, and real-time customisation — all running locally in your browser with zero server dependencies.

Built for [Perchance.org/hypnosis-script-generator](https://perchance.org/hypnosis-script-generator). No accounts. No downloads. No data leaves your device.

---

## What It Does

1. **Choose a Guide** — 25 built-in personas across four temperaments (contemplative, showman, somatic, authoritarian), each with a distinct voice, vocabulary, and therapeutic style. Create unlimited custom personas.

2. **Choose a Method** — 37 induction techniques across five categories: therapeutic (progressive relaxation, Ericksonian, fractionation), somatic (glove anesthesia, heartbeat sync, floating body), demonstration (magnetic hands, arm catalepsy, number amnesia), rapid (Dave Elman, shock induction, arm drop), and advanced (time distortion, positive/negative hallucination, eyes-open somnambulism).

3. **Set an Intention** — 20 intention types from emotional state shifts and post-hypnotic triggers through pain management, sleep induction, and behavioural conditioning. The AI crafts a technically-precise hypnotic suggestion from a rough description.

4. **Configure** — Session length (2–30 minutes), voice selection with quality scoring, 10 procedural soundscapes, 16 visual effects, pitch/volume/speed controls, and per-element HUD toggles.

5. **Generate & Begin** — One AI call produces a complete five-phase script (settling → induction → deepener → work → emergence). Edit the script, then begin a fully narrated, visually guided session with real-time controls.

---

## Features

### Audio
- **10 procedural soundscapes** — engine drone, rain (with Poisson-scheduled droplets), ocean waves, crackling fire, singing bowls (additive synthesis), heartbeat, binaural beats (theta 6Hz via ChannelMerger), pink/brown/white noise. All synthesised in real-time with the Web Audio API — no audio files, never loops audibly.
- **Narration ducking** — soundscape volume automatically dips to 20% during speech, ramps back between sentences.
- **Voice quality scoring** — voices sorted by neural/natural/enhanced quality with ★/● badges.
- **Ambience recording** — record the procedural soundscape to a downloadable `.webm` file via MediaRecorder.

### Visuals
- **16 Canvas 2D renderers** — spiral, breathing circle (4-7-8 pacer), tunnel, starfield warp, candle focal, moiré interference, vortex, mandala, pendulum harmonograph, lissajous, colour wash, flow-field particles, plasma, canvas pulse, and two kaleidoscope meta-effects wrapping flow field and plasma.
- **Phase-reactive speed** — visuals slow during settling (0.5×) and deepener (0.7×).
- **Live switching** — change the visual mid-session without interruption.
- **Reduced motion** — `prefers-reduced-motion` disables all animations; manual Off/Gentle/Full override.

### Session Player
- **Five-phase playback** with progress bar, phase dots, elapsed time, and current-sentence text display.
- **Keyboard controls** — Space pauses, Escape emergency-ends.
- **Fullscreen mode** with auto-hiding controls.
- **Screen Wake Lock** prevents display timeout during sessions.
- **Live settings panel** — slide-out panel for adjusting visual, audio, and HUD elements mid-session.
- **Script transcript** — toggle a full-text view of the script during playback.
- **Double induction** — the Double Induction method automatically picks a contrasting second voice and speaks both simultaneously.
- **Persona-matched emergency re-alerts** — 25 unique re-alert scripts, one per persona, each in that persona's voice.
- **Grounding exercise** — 5-4-3-2-1 senses exercise auto-displayed after emergency end.

### AI Generation
- **Suggestion crafter** — converts rough intentions into technically-precise hypnotic suggestions using Perchance's `ai-text-plugin`.
- **Full script generator** — produces complete five-phase scripts in the selected persona's voice with method-specific pacing rules.
- **20 intention types** with specific AI guidance for each (trigger-response links, pain-dial metaphors, conditioning protocols, amnesia framing, etc.).
- **Progress indicators** — animated spinner, elapsed-seconds counter, and indeterminate progress bar during generation.
- **Error handling** — `stopReason` checks, `hideStartWith`, and toast notifications on failure.

### Data & Tracking
- **IndexedDB storage** with automatic localStorage migration. `navigator.storage.persist()` prevents browser clearing.
- **Session history** — last 50 sessions tracked with date, guide, method, duration, completion status, and mood.
- **Streak tracking** — consecutive-day streak, total sessions, total minutes, completion rate, favourite guide.
- **Pre/post mood check-in** — 5-point emoji scale before and after sessions.
- **Script library** — save, name, load, and delete generated scripts.
- **Full backup/restore** — export all personas, scripts, and history as a single JSON file.

### Sharing & Export
- **Script export** — download any generated script as a `.txt` file with metadata header.
- **Config sharing** — compress session config to a URL fragment via `CompressionStream('gzip')`, share via Web Share API or clipboard.
- **Ambience recording** — download procedural soundscapes as `.webm` audio.

### Accessibility
- `aria-live` region announcing step changes.
- Focus management moving to step headings on navigation.
- `@media (prefers-reduced-motion)` with real alternatives (not just `animation: none`).
- Keyboard-first session controls.
- Content warnings before advanced/authoritarian sessions.
- Grounding exercise routing after emergency end.
- Voice quality badges in the dropdown.
- Category colours paired with labels (never colour alone).

### Assessment
- **Hypnotic Induction Profile quiz** — 3-question self-assessment inspired by the Spiegel Eye-Roll Sign and absorption scales. Maps to three suggestibility tiers with specific method and guide recommendations.

---

## Content Inventory

### 25 Guide Personas

| Category | Personas |
|---|---|
| **Contemplative** | The Driver, The Old Master, The Therapist, The Inner Voice, The Scientist, The Mystic, The Librarian, The Astronomer |
| **Showman** | The Showman, The Mentalist, The Bartender, The Carnival Barker, The Magician, The Late Night DJ |
| **Somatic** | The Bodyworker, The Yoga Teacher, The Dancer, The Martial Artist, The Midwife |
| **Authoritarian** | The Commander, The Headmistress, The Interrogator, The Drill Instructor, The Dominatrix, The Surgeon |

### 37 Induction Methods

| Category | Methods |
|---|---|
| **Therapeutic** | Highway, Spiral Fixation, Betty Erickson 3-2-1, Staircase Descent, Hand Heaviness, Confusion Method, Fractionation, Eye-Roll (Spiegel), 4-7-8 Breath, Progressive Relaxation, Ericksonian Conversational |
| **Somatic** | Body Awareness Scan, Floating Body, Glove Anesthesia, Heartbeat Synchronisation, Automatic Writing, Pain Dial |
| **Demonstration** | Magnetic Hands, Arm Catalepsy, Chevreul's Pendulum, Tipsy Without Drinking, Laughter Trigger, Number Amnesia, Hot Hand / Cold Hand, Stuck to Your Seat |
| **Rapid** | Handshake Interrupt, Arm Drop, Magnetic Gaze, Dave Elman Induction, Shock Induction |
| **Advanced** | Time Distortion, Age Regression, Positive Hallucination, Negative Hallucination, Eyes-Open Somnambulism, Double Induction, Full-Body Anesthesia |

### 20 Intention Types

**Foundation:** Emotional state shift · Post-hypnotic trigger · Behaviour change · Release / let go · Perception shift · Install an anchor · Just experience trance

**Performance:** Peak performance / flow · Deep focus / concentration · Creative flow / unblock

**Body:** Pain management / comfort · Sleep induction · Energy / vitality boost

**Demonstration:** Convincer demonstration · Novelty effect / party trick

**Advanced:** Time distortion · Suggested amnesia · Persona / identity play · Surrender / ego dissolution · Behavioural conditioning

---

## Technical Stack

- **Platform:** [Perchance.org](https://perchance.org) HTML panel — single-file, no build step.
- **AI:** Perchance `ai-text-plugin` for suggestion crafting and script generation.
- **Audio:** Web Audio API — `OscillatorNode`, `AudioBufferSourceNode`, `BiquadFilterNode`, `ChannelMergerNode`, `DynamicsCompressorNode`, `MediaRecorder`.
- **Visuals:** Canvas 2D with `requestAnimationFrame`, `ResizeObserver`, `OffscreenCanvas` for plasma, object pools for particles.
- **Storage:** IndexedDB via a lightweight async key-value wrapper. `navigator.storage.persist()`.
- **TTS:** Web Speech API `SpeechSynthesis` with Chrome 15-second watchdog, sentence-level chunking, and `onboundary` word highlighting.
- **Platform APIs:** Screen Wake Lock, Vibration API (Android), Web Share, Clipboard, CompressionStream/DecompressionStream, Fullscreen, Page Visibility.

---

## Browser Support

| Browser | Status | Notes |
|---|---|---|
| Chrome/Edge 90+ | ✅ Full | Best voice quality (Microsoft Neural voices on Edge). 15-second TTS bug mitigated by watchdog. |
| Safari 16.4+ | ✅ Full | Wake Lock, reduced-motion, fullscreen all supported. iOS may hide premium voices. |
| Firefox 124+ | ⚠️ Partial | `onboundary` word highlighting may not fire. Wake Lock supported. Needs `speech-dispatcher` on Linux. |
| Mobile Chrome/Safari | ✅ Full | Particle counts auto-reduced. Wake Lock re-acquires on tab visibility. |

---

## Installation

This is a Perchance generator — no installation required.

1. Visit the generator page at [Perchance.org/hypnosis-script-generator](https://perchance.org/hypnosis-script-generator)
2. The tool runs entirely in your browser
3. All data is stored locally in IndexedDB

---

## Privacy

- **Zero network requests** during sessions. All audio and visuals are synthesised locally.
- **No analytics, no tracking, no telemetry.**
- **AI calls** go through Perchance's `ai-text-plugin` — the same infrastructure as all Perchance generators.
- **All data** (personas, scripts, session history, mood data) stays in your browser's IndexedDB. Nothing is transmitted.
- **Backup files** are plain JSON on your local filesystem.
- **Shared config URLs** contain only the session configuration (persona ID, method ID, length, soundscape, intention text) — no personal data.

---

## Safety

- **Content warnings** shown before advanced/authoritarian sessions with confirmation required.
- **Emergency end** always available (button + Escape key) with persona-matched re-alert scripts.
- **Grounding exercise** (5-4-3-2-1 senses) automatically displayed after emergency end.
- **30-second cooldown** after emergency end before a new session can start.
- **Reduced motion** support via CSS media query and manual override.
- **Flicker cap** — all visual effects stay below 2Hz oscillation per WCAG 2.3.1.
- **Disclaimer** visible on Step 5: skip if you have epilepsy, dissociative disorders, or psychosis.
- **Sleep intention** suppresses the emergence phase — no unexpected re-alerting.

---

## License

MIT — see [LICENSE](LICENSE).

---

## Credits

**Created by** [therealwestninja](https://github.com/therealwestninja) · [DeviantArt](https://www.deviantart.com/west-ninja)

**Soundscape engine and visual renderers** adapted from [Adaptive Session Studio](https://github.com/therealwestninja) (MIT).

**Perchance platform** by [perchance.org](https://perchance.org).
