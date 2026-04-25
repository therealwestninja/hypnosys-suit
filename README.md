# · Advanced Hypnosis Narration Engine ·

**A free, AI-driven, fully browser-native hypnosis experience generator.**

Generate personalized induction scripts with AI, hear them narrated by the browser's text-to-speech engine, and experience guided sessions with procedural soundscapes, hypnotic visual effects, and real-time customization — all running locally in your browser.

Live here:
[https://perchance.org/advanced-hypnosis-narration-engine](https://perchance.org/advanced-hypnosis-narration-engine)
No account required. No downloads.

---

## What's New

* **Chat with each guide before you commit** — open a short preview chat with any persona right from Step 1; their voice, vocabulary, and cadence come through immediately so you can pick the right one without running a full session
* **Tiered emergency exit** — first <kbd>Esc</kbd> opens a soft-pause overlay (you're safe, take a breath, continue or end); second <kbd>Esc</kbd> commits to a full counted re-alert. Pulled from clinical hypnosis literature on abreaction-vs-discomfort distinction
* **Interactive 5-4-3-2-1 grounding** — tap each step as you find the items; combined with a time/place reorientation card after every emergency exit
* **Eight session lengths, up to 90 minutes** — added Extended (~45m), Deep (~60m), and Marathon (~90m) for users who want longer arcs. Marathon doubles the descent with nested deepener+lightener pairs (LIFO unwind)
* **Local AI voice (offline, neural-quality)** — opt-in toggle loads Hugging Face SpeechT5 from CDN once (~80 MB cached), then runs entirely offline. Five speakers across male/female ranges. No cloud, no internet needed once cached
* **Persona-voiced affirmations** — AI-drafted affirmations now sound like the persona who'd deliver them: a Commander writes commands, the Inner Voice writes thoughts-just-below-words, the Old Master writes elemental fragments
* **Re-roll diversity** — when you retry a phase, the AI is told what the previous draft sounded like and instructed to vary opening, rhythm, and metaphor. Repeated retries no longer cluster around the same opening
* **Time-of-day signals** — small ✦ badges on the persona/method you usually pick at this hour, so your favorites surface automatically
* **Modality inference** — generation lightly steers toward visual/auditory/kinesthetic descriptions based on your settings + somatic-method usage patterns. Conservative — only kicks in when the signal is clear
* **AI background visibility fix** — the session stage no longer dims AI background images to ~9% opacity. Selected images now render at the intended ~30% (bug fix)
* **Hardened generation guards** — defensive `[[Speaker]]:` chat-format stop sequences merged into every AI call so chat-style bleed never reaches your script

---

## What It Does

### 6-Step Guided Workflow

1. **Choose a Guide**
   25 built-in personas across four temperaments, each with distinct tone and delivery. **Try-out chat** lets you have a short preview conversation with each persona before committing — get a feel for voice, vocabulary, and pacing in 1-2 minutes. Create unlimited custom personas. Import personas others have shared.

2. **Choose a Method**
   37 induction techniques across therapeutic, somatic, demonstration, rapid, and advanced categories.

3. **Set an Intention**
   Define what you want to achieve. The AI converts this into structured hypnotic suggestions.

4. **Affirmations**
   Add your own affirmations or have the AI draft them in the voice of your selected persona.
   These are no longer static — the AI:

   * weaves them into sessions
   * structures them across phases
   * adapts delivery style (woven / pulses / light touch)
   * **drafts in-persona** when you don't want to write your own

5. **Configure & Review**

   * **8 session lengths** — Micro (~2m), Quick (~5m), Short (~12m), Medium (~20m), Long (~30m), Extended (~45m), Deep (~60m), Marathon (~90m)
   * narrator voice — system voices, or **opt-in local AI voice** (offline, neural quality)
   * soundscape + visuals
   * edit and rearrange generated script
   * adjust phase structure

6. **Generate & Begin**
   Run a fully narrated session with real-time controls.

---

## Session Model

Sessions are no longer fixed to 5 phases.

They now include:

* settling
* induction
* deepener (or **deepener-1 → deepener-2** for marathon's nested descent)
* **multiple work blocks (up to 8 for marathon sessions)**
* affirmation phases (dynamic — paired with each work block)
* **lightener** (or **lightener-2 → lightener-1** mirrored unwind for marathon)
* wake
* **reintroduction**

This creates a full arc:
**descent → work → recovery → return**

Marathon sessions explicitly model double-depth descent, with each lightener handling one level back up — the same protocol used in clinical multi-layered fractionation.

---

## AI Generation

* **Streaming per-phase generation** — each phase renders live as it's written, with an inline preview you can read as it builds
* **Re-roll diversity** — retries are told what the previous draft opened with and instructed to vary rhythm, opening, and metaphor; repeated re-rolls produce meaningfully different output instead of converging
* **Persona-voiced output everywhere** — affirmations, scripts, and chat replies all use the persona's full role text (not just the tagline) so signature phrases and cadence come through
* **Modality-aware prompting** — when usage signals are clear, prompts include guidance like "lean into bodily sensation" or "lean into auditory descriptions" to match how you actually engage
* **Hardened stop sequences** — defensive guards against `[[Speaker]]:` chat-format bleed are merged into every AI call (Perchance API §14 best practice)
* **Semantic memory retrieval** — the generator embeds your past session digests and pulls in the most relevant ones when writing a new script (not just the most recent)
* **Token-aware prompt assembly** — profile, adaptive context, and memory are prioritized and trimmed as needed so long-term users don't silently blow past the context limit
* **Adaptive intelligence** — learns what works for you (personas, methods, lengths, depth preferences) and shifts recommendations over time
* **3-tier hierarchical memory** — per-session digests, rolling patterns (every 5 sessions), long-term profile (every 15)
* **Per-phase regeneration with diff** — unhappy with one phase? Retry just that one and compare old vs. new side-by-side
* **Multi-provider routing** — default free Perchance AI, or plug in your own Anthropic / OpenAI key (be aware your key lives in the browser)

---

## Script Editing (Step 6)

You can:

* reorder phases
* remove or add phases
* adjust structure
* edit full script text
* regenerate individual phases and compare outputs
* control session flow directly

This is a core feature, not just a preview step.

---

## Audio

* Procedural soundscapes (no audio files shipped — everything synthesized)
* narration ducking during speech
* **two narration paths**:
   - **system voices** — instant, integrates with mobile lock-screen controls, quality varies by OS
   - **local AI voice (opt-in)** — neural quality via Hugging Face SpeechT5 running entirely in your browser; ~80 MB one-time download, then cached and offline. Five speaker variants
* live adjustment during session (volume, pitch, rate, tempo)
* mobile lock-screen media controls

---

## Visuals

* multiple animated focus effects (spiral, tunnel, vortex, pendulum, mandala, candle, flow-field, aurora, and more)
* real-time tuning (speed, intensity, complexity)
* phase-reactive pacing
* reduced-motion support
* **background image generation** (AI, static URL, or YouTube) — properly visible during the session (the prior overlay bug that drowned them out is fixed)

---

## Session Player

* dynamic phase playback (not fixed count)
* progress tracking + transcript view
* live settings panel
* **tiered emergency exit** (see Safety below)
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
* time-of-day affordances — your usual picks for this hour are quietly highlighted with a ✦ badge

---

## Data & Systems

### History

Stores per session:

* full script
* phase structure
* affirmations + mode
* profile snapshot
* session metrics (depth, mood delta, completion state)

### Profiles

User profile is injected into AI generation:

* goals
* preferences
* traits
* appearance (for personalized imagery)
* **inferred sensory modality** (visual / auditory / kinesthetic) — derived conservatively from settings + usage; only surfaced when the signal is strong, otherwise omitted

### Adaptive System

Tracks and updates automatically:

* completion rates
* mood-delta variance
* method/persona preferences by reward signal
* novelty-seeking vs. repetition patterns
* depth tolerance
* recovery mode when sessions trend poorly
* **time-of-day patterns** — what you usually pick when, surfaced as ✦ badges on Step 1 / 2

### Tracking

Expanded analytics:

* session outcomes
* usage patterns
* feature interaction
* structural preferences
* hourly distribution

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

* **session configs** — share the persona, method, length, soundscape, affirmations, and optionally the generated script as a single URL
* **custom personas** — share a persona you've built; recipients can add it to their own library with one click
* **rich link previews** — shared URLs produce tailored titles and descriptions when pasted in chat apps
* **community persona packs** — opt-in third-party collections load lazily when selected

### Privacy-aware share flow

* recipients see an age-gate prompt before importing personas with potentially sensitive content
* URL is cleaned after import so a reload doesn't re-trigger the flow

---

## Privacy

* runs locally in browser
* no default data collection
* no telemetry
* **local AI voice** never sends audio to any server — synthesis runs entirely in-browser

### Optional (Opt-in Only)

Users may enable:

> **Remote backup + processing**

* requires explicit consent
* disabled by default
* clearly disclosed before activation

---

## Safety

The safety layer was reworked around clinical hypnosis literature on abreaction handling and post-emergency reorientation (Lynn et al., Kluft, Mott).

* **Tiered emergency exit**:
   - first <kbd>Esc</kbd> press → soft-pause overlay with current day/time. Speech and drone pause. You can resume (Continue / <kbd>Space</kbd> / <kbd>Enter</kbd>) or commit to ending
   - second <kbd>Esc</kbd> press while overlay is open → full counted re-alert in your guide's voice, then post-session screen
   - the dedicated "end & re-alert" button still goes straight to the hard end (deliberate clicks aren't intercepted)
* **Interactive 5-4-3-2-1 grounding** — tap each step as you find items; status persists until you start a new session
* **Reorientation card** — after emergency exit, an anchor card displays current day + time and "you're here, in your body, in this room"
* **Howard Alertness Scale check** — auto-suggests the grounding exercise if you self-rate low after a session
* contraindication screening
* session limits + cooldowns
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
* tiered <kbd>Esc</kbd> means accidental presses don't end the session abruptly — there's a forgiving middle layer

---

## Technical Stack

* Perchance (single-file app)
* `ai-text-plugin` for streaming generation
* `text-to-image-plugin` for backgrounds
* `upload-plugin` for shareable content
* `dynamic-import-plugin` for community packs
* Web Audio API (procedural synthesis + local AI voice playback)
* Canvas 2D rendering
* IndexedDB storage (persisted)
* Web Speech API (system TTS)
* **Hugging Face Transformers.js** (optional offline neural TTS via SpeechT5)
* Wake Lock, Media Session, Vibration, Clipboard APIs

---

## Browser Support

* Chrome / Edge: full support (including local AI voice)
* Safari: full support (local AI voice from Safari 16.4+)
* Firefox: partial (speech-synthesis differences; local AI voice supported)
* Mobile: fully supported (lock-screen controls on iOS 16+ / Android 10+; local AI voice has higher memory cost on phones)

Features that gracefully degrade when unavailable:

* semantic memory falls back to recency-based retrieval
* streaming falls back to full-phase render
* share links fall back to hash-URL encoding if upload service is unreachable
* haptics silently skip on desktop
* local AI voice falls back to system voice if model loading fails or browser lacks WASM/SIMD support

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
