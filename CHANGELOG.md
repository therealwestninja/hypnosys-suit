# Changelog

All notable changes to the Hypnosis Script Generator.

## [2.0.0] — 2026-04-21

### Major: Complete feature overhaul

**Content expansion**
- 25 guide personas (was 10) across 4 categories: contemplative, showman, somatic, authoritarian.
- 37 induction methods (was 20) across 5 categories: therapeutic, somatic, demonstration, rapid, advanced.
- 20 intention types (was 9) organised into Foundation, Performance, Body, Demonstration, and Advanced groups with `<optgroup>` navigation.
- 25 persona-matched emergency re-alert scripts (was 1 generic).
- Hypnotic Induction Profile self-assessment quiz with tiered method/guide recommendations.

**Procedural soundscape engine**
- Replaced the single-oscillator AudioDrone with a full SoundscapeEngine supporting 10 ambient types: drone, rain, ocean, fire, singing bowls, heartbeat, binaural beats, pink/brown/white noise.
- DynamicsCompressor on master chain (threshold −18, knee 24, ratio 3).
- Narration ducking (20% during speech, 100% between).
- Drone phase-frequency modulation (70Hz settling → 60Hz work → 70Hz emergence).
- Ambience-only audio recording via MediaRecorder.

**Visual effects**
- 16 Canvas 2D renderers (was 3): tunnel, pendulum, vortex, mandala, lissajous, colour wash, canvas pulse, breathing circle, moiré, starfield, candle, flow-field particles, plasma, kaleidoscope-flow, kaleidoscope-plasma. Plus original highway, spiral SVG, CSS pulse.
- Phase-reactive visual speed multipliers.
- ResizeObserver on all canvases.
- Live visual switching mid-session.
- Kaleidoscope meta-effect wrapping any sub-renderer with N-fold mirror symmetry.

**Session player**
- Progress bar, phase dots, elapsed time counter.
- Keyboard controls (Space pause, Escape end).
- Fullscreen toggle.
- Screen Wake Lock with visibility-change re-acquire.
- Double induction mode (two contrasting voices simultaneously).
- Word-level transcript highlighting via `onboundary`.
- Script transcript toggle during session.
- Live settings slide-out panel for mid-session adjustments.
- Content warnings before advanced/authoritarian sessions.
- Grounding exercise (5-4-3-2-1) after emergency end.
- 30-second emergency cooldown.

**Configure tab overhaul**
- Visual effect selector with all 16 renderers + "Auto" + "None".
- Soundscape type selector with 10 options.
- Voice pitch and narration volume sliders.
- 7 independent HUD element toggles (fixation dot, text, phase dots, phase label, progress bar, time, labels).
- Dynamic context panel showing guide/method with category badges.
- Contextual speed/length hints per method category.
- Combination warnings (e.g. advanced + short, sleep skipping emergence).
- Live value readouts on all sliders.

**Data & tracking**
- IndexedDB storage replacing localStorage, with automatic migration.
- Session history (last 50) with streaks, stats, favourite guide.
- Pre/post mood check-in (5-point emoji scale).
- Full backup/restore (JSON export/import with merge-on-conflict).
- Script export as .txt download.
- Config sharing via URL fragment + CompressionStream.
- `navigator.storage.persist()`.

**Accessibility**
- `aria-live` region for step announcements.
- Focus management on step transitions.
- `@media (prefers-reduced-motion)` with real visual alternatives.
- Content warnings with confirmation gates.
- iOS Safari viewport fix.

**Reliability**
- Toast notification system replacing all `alert()` calls.
- `stopReason === 'error'` checks on AI calls.
- `hideStartWith: true` on AI calls.
- Chrome TTS 15-second watchdog with cancel/re-speak.
- AI preload at page load (delayed on mobile).
- Craft progress indicator with elapsed timer.

### Bug fixes
- Fixed broken quote in magnetic-gaze method definition (`visual: 'candle,` → `visual: 'candle',`).
- Fixed duplicate `const elapsed` declaration in `finishSession`.
- Fixed typo: "narratied" → "narrated", double `</i>` tag.
- Fixed iOS Safari auto-zoom on input focus.

## [1.0.0] — 2026-04-20

### Initial release
- 5-step wizard: Guide → Method → Intention → Configure → Generate.
- 10 guide personas (contemplative + showman).
- 20 induction methods (therapeutic + demonstration).
- 9 intention types.
- AI-powered suggestion crafter and full script generator.
- Session player with TTS, fixation dot, highway/spiral/pulse visuals.
- Engine drone (Web Audio oscillator + brown noise).
- Script library with localStorage persistence.
- Custom persona editor.
- Post-session screen.
