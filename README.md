# · Advanced Hypnosis Narration Engine ·

**A free, AI-driven, fully browser-native hypnosis experience generator.**

Generate personalized induction scripts with AI, hear them narrated by the browser's text-to-speech engine, and experience guided sessions with procedural soundscapes, hypnotic visual effects, and real-time customization — all running locally in your browser.

Live here:
[https://perchance.org/advanced-hypnosis-narration-engine](https://perchance.org/advanced-hypnosis-narration-engine)
No account required. No downloads.

---

## What's New

* **Motif Engine** — every session picks ONE concrete sensory image from a curated 24-motif corpus (a brass key warm from being held, a lighthouse beam passing and returning, a steady ember beneath grey ash, …) and threads it across the whole session as a deliberate through-line:
  * **narration** — the AI is instructed to weave the motif image 2–4 times across the phases, lightly in early phases and with more weight at the deepest point
  * **visuals** — when the visualizer is on auto, the motif's preferred visualizer (mandala / candle / ripples / starfield, depending on the motif) is used, and the visualizer's color is tinted toward the motif's palette with a phase-weighted intensity curve
  * **audio cue** — an optional soft tone (bell / bowl / chime / drop, matched to the motif's sense) plays once at the deepest phase of the session
  * three independent toggles in Smart Director options let you keep the narrative motif while disabling the cue or visual side
  * rotation memory persists across sessions, so 12 consecutive sessions get 12 different motifs
* **Soundscape coherence layer** — every procedural soundscape (rain, ocean, fire, forest, wind, bowls, …) now has:
  * a **session breath** at 1/240 Hz (one ~4-minute cycle) gently modulating master gain by ±10% — soundscapes no longer hit a static steady state, they slowly swell and recede over a session's arc
  * a **motif warmth tilt** — warm motifs (ember, brass key, banked fire) shift the soundscape's tone filter toward warmer cutoffs; cool / neutral motifs leave it transparent
* **Public Perchance plugin exports** — other Perchance generators can now import HNE as a function library:
  * `root.hne.freshSeeds(opts)` returns evocative sensory anchor phrases from a 60+ phrase corpus tagged across 12 categories
  * `root.hne.freshMotifs(opts)` returns full motif objects (image, essence, sense, tags, palette, viz, cue) for use in meditation apps, tarot readers, dream journals, sleep aids, poetry generators
  * `root.hne.pickFromBank(opts)` — generic rotation utility that applies HNE's tag-filter + exclude-set + cold-start fallback + Fisher-Yates shuffle to **caller-provided** banks. Use when HNE's bundled meditation-register corpus doesn't fit your project's voice (carnival-mystic fortune teller, noir detective, clinical sleep app, …)
  * all three are tag-filtered and rotation-aware with a cold-start fallback
* **Community language packs** — translations live in separate Perchance generators that fork the data pack and translate any subset of its keys (missing keys fall through to English). One-line entry in the `hneLanguagePacks` list registers a translation with HNE's language picker
* **Community content packs** — third-party generators can publish persona / method / preset collections that HNE loads lazily on first use
* **Phase-locked entrainment** — binaural beats and isochronic tones ramp frequency per session phase across the α/θ/δ brain-wave map, with 6-second crossfades at each boundary
* **40 Hz gamma flicker visualizer** — alpha-grade entrainment surface with conservative implementation (5–15% alpha modulation, narrow centered pulse, blue palette, voice-coupled) and OS-aware reduced-motion fallback
* **Drag-and-drop phase reordering** — grab the ≡ handle on any phase row, with a live timeline view above the list showing proportional phase durations
* **Per-phase duration estimate + timeline** — every row shows ⏱ Xm Ys inline; click any timeline block to jump to that phase
* **2D Director calibration pad** — combined intensity × pace control on a drag-snap grid
* **A/B experiment mode** — blinded sessions on two random presets, rate 1–5, rolling per-preset averages
* **Conversational free-text check-in** — type how you feel ("tired but mind is racing") and a lexicon-based parser maps it to mood / tension / energy values
* **Phase audition** — 🔊 button on any phase row plays it aloud using current voice settings before committing to a full session
* **Annual review screen** — opens after 30+ days of practice; sessions, minutes, completion %, longest streak, mood arc, top guides / methods / contexts. Pure local aggregation, no AI
* **Audio/viz safety relaxation gate** — set your profile age range and the overcautious gates step back: gamma flicker confirmation skips, prefers-reduced-motion auto-degrade skips, speech rate caps widen, soundscape volume caps lift. Casual users keep all original safeguards
* **Shareable links** — post a persona or a full session config as a real URL with tailored link previews in Discord / Slack / iMessage
* **Streaming script generation** — each phase appears live as the AI writes it, instead of a spinner
* **Semantic memory retrieval** — the AI recalls past sessions that are *relevant* to your current intention, not just the most recent ones
* **Regeneration diffs** — retry a single phase and pick between the original and the new version side-by-side
* **Token-aware prompt assembly** — long profiles, memory, and affirmations never silently exceed the context window
* **Multi-provider AI** — use the free built-in Perchance model, or bring your own Anthropic / OpenAI key

---

## What It Does

### 6-Step Guided Workflow

1. **Choose a Guide**
   38 built-in personas across four temperaments, each with distinct tone and delivery. Create unlimited custom personas. Import personas others have shared.

2. **Choose a Method**
   37 induction techniques across therapeutic, somatic, demonstration, rapid, and advanced categories.

3. **Set an Intention**
   Define what you want to achieve. The AI converts this into structured hypnotic suggestions.

4. **Affirmations**
   Add your own affirmations. These are no longer static — the AI:
   * weaves them into sessions
   * structures them across phases
   * adapts delivery style (woven / pulses / light touch)

5. **Configure & Review**
   * session length
   * narrator voice
   * soundscape + visuals
   * edit and rearrange generated script
   * adjust phase structure

6. **Generate & Begin**
   Run a fully narrated session with real-time controls.

### Quick-Start Presets

31 one-click curated presets — Sleep, Quick Calm, Deep Focus, Pain Relief, Micro Reset, Anxiety Relief, Confidence, Peak Flow, Habit Change, Creative Flow, Energy Boost, Nightcap, Train Journey, Temple Sit, Mission Brief, Goal Lock-in, …

### Multi-Day Structured Programs

9 progressive programs that sequence sessions across days/weeks with a coherent arc:
* 7-Day Sleep
* 14-Day Calm
* 10-Day Confidence
* 6-Week Pain
* 7-Day Focus
* 7-Day Depth
* 5-Day Stress Reset
* 7-Day Sensuality
* 5-Day Surrender

---

## Session Model

Sessions are dynamic, not fixed to 5 phases. The full arc includes:

* settling
* induction
* deepener
* **multiple work blocks (up to 5 for deep sessions)**
* affirmation phases (dynamic)
* **lightener**
* **reintroduction**

**Shape:** descent → work → recovery → return.

The Motif Engine quietly anchors all of these to a single sensory image, so the session reads as one continuous thing rather than a list of phases.

---

## AI Generation

* **Streaming per-phase generation** — each phase renders live with an inline preview you can read as it builds
* **Motif through-line injection** — the chosen session motif is part of the shared prompt context for every phase, alongside profile, persona, method, freshness anchors, and memory
* **Fresh metaphor anchors** — 4 randomly-picked phrases per session from a 60+ phrase corpus, tag-matched to your profile, injected as "optional anchors" the AI may use to break mode-collapse
* **Semantic memory retrieval** — the generator embeds your past session digests and pulls in the most relevant ones when writing a new script (not just the most recent)
* **Token-aware prompt assembly** — profile, adaptive context, motif, freshness, and memory blocks are prioritized and trimmed against the context budget
* **Adaptive intelligence** — learns what works for you (personas, methods, lengths, depth preferences) and shifts recommendations over time
* **3-tier hierarchical memory** — per-session digests, rolling patterns (every 5 sessions), long-term profile (every 15)
* **Per-phase regeneration with diff** — unhappy with one phase? Retry just that one and compare old vs. new side-by-side
* **Multi-provider routing** — default free Perchance AI, or plug in your own Anthropic / OpenAI key (the key lives in the browser only)
* **AI scene directives** — the AI can emit `[BG:rain]` / `[VIZ:mandala]` sentinels mid-script to shift the soundscape or visualizer when the language calls for it (opt-out toggle)

---

## Script Editing (Step 5)

* drag-and-drop phases via the ≡ grab handle (or ↑/↓ buttons)
* see a proportional **timeline visualization** above the list — click any block to jump
* see **estimated duration** (⏱ Xm Ys) on every phase row, plus session total
* **audition** any phase aloud (🔊 button) before committing
* remove or add phases (induction, deepener, level, work, affirmation, lightener, wake, reintroduction, sounding, custom)
* edit full script text
* regenerate individual phases and compare outputs side-by-side
* author custom blocks, then ✨ Reimagine them in your guide's voice

This is a core feature, not just a preview step.

---

## Audio

* **27 procedural soundscapes** — fully synthesized via Web Audio (no audio files shipped): rain, ocean, fire, forest, stream, wind, thunder, storm, drone, bowls, heartbeat, binaural, isochronic, chimes, pink/brown/white noise, fan, cave, crickets, temple, spaceship, underwater, clock, train, city, owls, whales
* **Session breath layer** — slow 1/240 Hz modulation gives every soundscape gentle long-arc evolution instead of a static steady state
* **Motif warmth tilt** — soundscape tone subtly tracks the active motif's color temperature
* **Phase-locked entrainment** — binaural beats and isochronic tones ramp their frequency per session phase across the α/θ/δ map, with 6-second crossfades at each phase boundary
* **Narration ducking** — soundscape gain dips automatically during speech
* **Voice selection with quality sorting** — neural / premium voices ranked first
* **Live adjustment during session** — volume, pitch, rate, tempo all editable mid-session
* **Mobile lock-screen media controls** — pause / resume from the lock screen

---

## Visuals

* **30 animated focus effects** — spiral, tunnel, vortex, pendulum, mandala, candle, flowfield, plasma, aurora, lissajous, sacred-geo, kaleidoscope, waveform, fireflies, ripples, breathing, ink, smoke, starfield, colorwash, moire, highway, rain, pulse, pocket-watch, spinning-coin, hand-pendulum, canvas-pulse, gamma-flicker, and more
* **Motif palette tint** — visualizer color blends toward the motif's secondary palette tone, weighted by phase intensity (faint during settling/induction, peak at deepener/work, fading on lightener)
* **Motif viz biasing** — when the visualizer is on auto, the motif's preferred viz list is preferred over the method default
* **40 Hz gamma flicker** — alpha-grade, conservative implementation, voice-coupled
* **Real-time tuning** — speed, intensity, complexity all editable mid-session
* **Phase-reactive pacing** — visual rate shifts per phase
* **Reduced-motion support** — OS preference auto-degrades intense visualizers
* **Background image generation** — AI text-to-image, static URL, or YouTube video

---

## Session Player

* dynamic phase playback (not fixed count)
* progress tracking + transcript view
* live settings panel (audio + voice + visualizer + tempo all editable mid-session)
* emergency exit + grounding system
* fullscreen + wake lock
* optional ambient audio recording

---

## UI / UX

* mobile-first layout (portrait optimized)
* no horizontal overflow
* responsive grid-based UI
* improved readability (contrast + font sizing)
* clear navigation (back buttons, step clarity)
* onboarding tour for new users
* toast notifications (non-blocking, non-interrupting)
* command palette (Cmd/Ctrl-K)

---

## Data & Systems

### History

Stores per session:
* full script
* phase structure
* affirmations + mode
* profile snapshot
* session metrics (depth, mood delta, completion state)
* active motif id (so a session's through-line is searchable in history)

### Profiles

User profile is injected into AI generation:
* goals
* preferences
* traits
* appearance (for personalized imagery)
* category-scoped sharing controls (what the AI is allowed to see)

### Adaptive System

Tracks and updates automatically:
* completion rates
* mood-delta variance
* method/persona preferences by reward signal
* novelty-seeking vs. repetition patterns
* depth tolerance
* recovery mode when sessions trend poorly

### Tracking

Expanded analytics:
* session outcomes
* usage patterns
* feature interaction
* structural preferences

---

## Stats & Charts

* usage breakdowns (guides, methods, intentions)
* time-of-day patterns
* mood change tracking
* completion rates
* feature usage maps
* activity heatmap (GitHub-style)
* pre/post improvement sparkline

---

## Backup & Sharing

### Local

* full backup / restore (gzipped JSON)
* script export to .txt
* compatibility with older formats (automatic migration)

### Shareable Links

* **session configs** — share persona, method, length, soundscape, affirmations, and optionally the generated script as a single URL
* **custom personas** — share a persona you've built; recipients can add it to their library with one click
* **rich link previews** — shared URLs produce tailored titles and descriptions when pasted in chat apps
* **community persona packs** — opt-in third-party collections load lazily when selected

### Privacy-aware share flow

* recipients see an age-gate prompt before importing personas with potentially sensitive content
* URL is cleaned after import so a reload doesn't re-trigger the flow

---

## Use HNE as a Perchance Plugin

Other Perchance generators can import HNE's public functions:

```
hne = {import:advanced-hypnosis-narration-engine}
```

Then in the consuming generator:

```js
// Pull 3 evocative sensory anchors matching a profile tag set,
// avoiding any recently-used ones.
let anchors = root.hne.freshSeeds({
  profileTags: ["sleep", "release"],
  n: 3,
  excludeSet: ["the slow pull of evening tide"],
});

// Pull ONE recurring session motif — a structured image object with
// palette/viz/cue metadata for threading across narration, visuals, and audio.
let [motif] = root.hne.freshMotifs({
  profileTags: ["focus", "trance"],
  n: 1,
  excludeSet: previouslyUsedIds,
});
// motif = { id, image, essence, sense, tags, palette, viz, cue }
```

**Register caveat:** HNE's bundled corpus is meditation/hypnosis-coded — grounding, somatic, kinesthetic (*"a held shoulder finally dropping"*, *"a lighthouse beam, passing and returning"*). If your project's voice is different — carnival-mystic, clinical, noir, playful — using the curated phrases as primary content will dilute your tone.

For different voices, use `pickFromBank` instead: it exposes HNE's rotation machinery (tag-filter, exclude-set, cold-start fallback, Fisher-Yates shuffle) operating on **your own** data:

```js
const fortuneMotifs = [
  { card: "Queen of Cups", tags: "love|intuition",
    image: "tea leaves settling into the shape of a bird" },
  { card: "Ace of Cups",   tags: "love|beginning",
    image: "a chalice brimming, water trembling at the rim" },
  // …
];
const [draw] = root.hne.pickFromBank({
  bank:         fortuneMotifs,
  profileTags:  ["love"],
  n:            1,
  excludeSet:   recentlyDrawn,   // array of card names
  excludeField: "card",          // dedupe key on each item
});
// → { card: "Queen of Cups", tags: "love|intuition", image: "..." }
```

`pickFromBank` accepts plain string banks too — useful when you just want HNE's anti-repetition shuffle without object plumbing. See the top-editor docs block for the full signature.

Useful for any project that needs sensory imagery with rotation — meditation apps, tarot readers, dream journals, sleep aids, poetry generators, noir adventure games, anywhere you want variety-with-memory but with your own voice.

To embed the whole HNE UI in another page, use a standard iframe — session links and shared configurations work via URL hash params.

---

## Translation / Language Packs

The English data pack lives in a separate Perchance generator (`advanced-hypnosis-narration-engine-data`). To translate HNE:

1. Fork the data generator
2. Translate any subset of keys (BUILTIN_PERSONAS, METHODS, PRESETS, TOUR_STEPS, GLOSSARY, PROFILE_FIELD_SCHEMA, …); missing keys fall back to the English baseline
3. Keep the shape identical
4. Submit a one-line entry to `hneLanguagePacks` and your translation appears in HNE's language picker for everyone

AI prompt strings (systemPrompt, guidance, methodInstructions) determine how the model narrates — translate them carefully and test the output.

---

## Privacy

* runs locally in the browser
* no default data collection
* no telemetry

### Optional (Opt-in Only)

Users may enable:

> **Remote backup + processing**

* requires explicit consent
* disabled by default
* clearly disclosed before activation

---

## Safety

* contraindication screening
* session limits + cooldowns
* grounding system
* emergency exit
* alertness check
* crisis resources
* reduced-motion defaults for seizure-risk visuals

---

## Accessibility

* keyboard controls
* reduced motion support
* improved contrast + readability
* larger tap targets
* focus indicators
* ARIA labels on navigation

---

## Technical Stack

* Perchance (split across three generators: main app, data pack, optional community/language packs)
* `ai-text-plugin` for streaming generation
* `text-to-image-plugin` for backgrounds
* `upload-plugin` for shareable content
* `dynamic-import-plugin` for community / language packs
* `favicon-plugin`, `bug-report-plugin`, `comments-plugin`
* Web Audio API (procedural synthesis throughout — no audio files shipped)
* Canvas 2D rendering
* IndexedDB storage (persisted via `navigator.storage.persist()`)
* Web Speech API (TTS)
* Wake Lock, Media Session, Vibration, Clipboard APIs

---

## Browser Support

* Chrome / Edge: full support
* Safari: full support
* Firefox: partial (speech-synthesis differences)
* Mobile: fully supported (lock-screen controls on iOS 16+ / Android 10+)

Features that gracefully degrade when unavailable:

* semantic memory falls back to recency-based retrieval
* streaming falls back to full-phase render
* share links fall back to hash-URL encoding if upload service is unreachable
* haptics silently skip on desktop
* motif audio cue silently skips when WebAudio is blocked

---

## Installation

No installation required.
Runs entirely in browser.

---

## License

MIT

---

## Credits

Created by therealwestninja
Perchance platform: perchance.org
