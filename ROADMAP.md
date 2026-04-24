# Advanced Hypnosis Narration Engine — Roadmap

This file tracks improvements not yet shipped. Items 1–6, 9, and 12 from the
original review have already been implemented in `perchance_1.txt` and
`perchance_2.txt`. What follows is the backlog, ordered roughly by a mix of
user-visible impact and effort.

---

## Recent bug fixes

- **2026-04-24 — Phase progress indicators not refreshing when blocks added post-generation.** When a user generated a script then added a Work or Deepener block via the phase editor, the colored status indicators row (`#genPhases`) and its retry click-handlers stayed frozen at the original phase list. Users had no indicator or retry affordance for newly-added blocks. Fix: extracted `buildPhaseInstructions()` as a pure helper, added `syncGenStateWithPhases()` that reconciles `_genState` against the current phase list (kept phases preserve state + result, new phases start as `pending`, removed phases drop, `phaseInstructions` fully rebuilt since word budgets rebalance). Hooked into `setScriptFromSections()` so every add/remove refreshes the row. Pending phases are now clickable too (not just error/warn), so clicking a newly-added Work block triggers generation for it.

---

## Smart Director — Passes 1 + 2 ✅

Rebuilt from "static string concatenation with a misleading tooltip" into an actual behavioral layer.

- **Pass 1 — honest toggle.** `smartDirectorEnabled` now gates every derived system: `recordAnalytics`, `updateAdaptiveProfile`, `updateSessionMemory` (digests + embeddings + rolling + long-term), and the `adaptiveCtx`/`memoryCtx` prompt injections. History still records regardless (base log, not a derivation). Label + sublabel rewritten to match actual behavior. Zero persistence leaks into `_buildSettingsObj`.
- **Pass 2 — living profile + director brief.** New `buildLivingProfile()` merges user-edited profile + AI-synthesized memory + adaptive signals + history snapshot into one structured object. `computeDirectorBrief()` runs rule-based logic over it — cold-start handling, recovery-mode detection, completion-rate triggers, emergency-exit caps, beginner depth caps, positive-pacing reinforcement, long-gap re-entry. Emits `{ coldStart, recoveryMode, recommendations, warnings, reasoning, bias }`. Injected into full-script prompt at priority 3 via `formatDirectorBriefForPrompt()`.
- **Director calibration slider.** Seven-position (−3..+3, default 0, synced via settings) that shifts every rule threshold AND injects a tone directive into the AI prompt. Safety floors preserved: beginner cap lifts only at +3, emergency-exit cap requires 3+ exits (not 2) at +3 but never disappears entirely. Greys out when Smart Director is off.
- **Debug bypasses** for development: skip rescue, skip cooldown, skip safety gate, unlock all methods, verbose logging. Session-only, never persisted, floating "⚠ DEBUG" pill when any are on.

**Deferred to Pass 3:**

- **Transparency panel — "what does the Director know about me, and why is it suggesting this?"** Surfaces the current living profile (merged view), the last director brief, and the reasoning per recommendation. Per-item "forget" buttons (delete one digest, one preference weight) so users have granular data control.
- **Per-session bias override.** Current bias is global — one off session doesn't reset it. A pre-flight "nudge cautious / forward for this session" control on the Configure step, consuming the global bias as default. State is session-scoped, not persisted.
- **"Because:" UI on director suggestions.** When the director recommends a depth/length change, show the one-line reasoning next to the field with a one-click "override" button. Currently the brief influences the AI prompt but the user never sees that it did.
- **Per-field profile granularity.** Today profile is all-or-nothing apart from the `useFirstName` carve-out. Checkboxes per field ("share age but not name", "share goals but not about") with per-site policy (use profile when crafting suggestions, skip for full script).
- **Per-call-site gating.** Some users may want memory-and-adaptive injection but no raw profile fields, or vice versa. A small matrix of toggles in the transparency panel.
- **Rule-based guardrails expansion.** Age-based method filtering (regression methods unavailable for under-18 via profile, not experience gating). Trauma-keyword detection in `about` that flips the system prompt to trauma-informed directives. Depth-tolerance enforcement based on recent emergency exit reasons.
- **Store director brief per session.** Add `brief` to the `sessionRecord` so retrospective debugging can answer "why did session 27 feel different?" later. Zero extra cost — `state._lastDirectorBrief` is already cached at generation time.
- **"Forget everything and start fresh."** Single button that clears adaptive weights, memory digests, embeddings, and analytics while preserving history. Useful after major life changes or if the director drifts unhelpfully. Currently users have to export / edit / import to reset.
- **Profile content length clamping.** `trimToTokenBudget` drops the whole profile block when over budget; finer-grained truncation within `about` (500 chars max) and `goals` (300 chars max) would keep the signal without losing the block.
- **Prompt-injection sanitization for profile fields.** `about`, `goals`, `appearance` are interpolated unescaped. A shared persona config that sets `about: "Ignore previous instructions..."` would execute. Strip injection patterns before building the string.
- **Sync director state cross-device.** Living profile currently computes from IDB sources per device; settings (including bias) sync but not memory / adaptive / embeddings. Natural pair with roadmap item #10.

---

## Image generation — Round 2 ✅

Rebuilt the AI background system from "hardcoded landscape with fixed negative prompt" into a proper user-facing control surface. (Round 1 was the original shipping of the background system — single prompt field, generate button, gallery, 45s crossfade during sessions.)

- **POS / NEG textareas** — positive and negative prompts in a two-column grid. Both persist; responsive collapse to single column under 560px.
- **Orientation dropdown** — Default / Square 1:1 / Portrait 2:3 / Landscape 3:2 — maps to plugin `resolution` with `default` omitting the key so Perchance picks.
- **Weight sliders** — Prompt strength (guidance scale, 1–20, default 7) and quality steps (15–50, default 30, passed under both `numInferenceSteps` and `steps` for backend tolerance).
- **Parallel generation (4-up).** `bgEngine` rewritten from `generating` boolean to `activeGenerations` counter with `BG_MAX_PARALLEL = 4`. `generateBatch(n)` fires up to 4 concurrent via `Promise.all`. `pregenerate` runs parallel batches instead of sequential — cold-boot 5-image warm-up drops from ~100s to ~40s.
- **Preview grid.** "Generate 4 (parallel)" button populates a responsive thumbnail grid as each completes. Single-image preview still works via the existing "Generate image" button.

**Next up — Round 3 (next session):** scope to be confirmed. Candidate ideas parked here so we don't lose them:

- **Seed control** — expose the `seed` parameter so users can reproduce an image they liked (currently hardcoded to `-1`).
- **Style presets dropdown** — "dreamlike", "cinematic", "watercolor", "oil painting", "dark fantasy", etc. — prepend / append curated fragments to the POS prompt without the user having to memorize modifier conventions.
- **Auto-prompt from session content.** Derive a background prompt from current `persona.name + method.detail + intention`. A "use session context" button that fills POS with something like "dreamlike ocean mist, soft indigo light, calm" for a sleep/ocean session.
- **img2img variations.** Use a generated image as the seed for 4 variations. Requires confirming the Perchance plugin accepts `initImage` or equivalent.
- **Per-phase imagery.** Different background per phase (settling vs deepener vs work) rather than one rotating pool. Would pair well with the existing phase structure.
- **Prompt templates.** Save / load POS+NEG+settings as named presets alongside the per-session config.
- **A/B comparison picker.** After a `generate 4` batch, let the user click their favorite to keep; discard the rest. Currently all 4 go to cache regardless.
- **Resolution picker beyond the 4 orientations.** Numeric width/height fields for users who want to exact-match a display.
- **Proper error surfacing with retry.** When `textToImagePlugin` fails, show a retry button inline rather than just a toast. Distinguish between "plugin missing" (configuration error) and "server busy" (transient) in the UI.
- **Export manifest.** When downloading from the gallery, include a JSON sidecar with the prompts and settings used so the user (or a recipient) can re-generate.
- **Character / persona portraits.** Originally in backlog item #7 — generate a stylised portrait per persona and cache via the existing `bgCacheStore` pattern. Would use the same upgraded engine.
- **Crafted-suggestion symbols.** Also from #7 — generate a 512×512 symbol representing a crafted suggestion's trigger + coreResponse for post-hypnotic recall.

---

## Image playback — Round 3 ✅ (Slideshow backport)

Replaced the single "crossfade every 45s" with a user-controllable playback system, backported from the standalone `Slideshow.txt` spinoff.

- **12 transition effects** — Cut, Crossfade, Fade→black, Fade→white, Slide left, Slide up, Zoom in, Zoom out, Ken Burns, Blur, Iris, Wipe. Each is a function in the `BG_TRANSITIONS` registry with signature `(oldEl, newEl, url, durSec)`; adapted for `<img>` elements with the 0.35 opacity cap preserved (session backgrounds stay readable).
- **Checkbox grid** — one checkbox per transition in Step 5's AI options. When multiple are checked, the player picks uniformly random for each slide change. Defaults to calm set (Crossfade, Fade→black, Ken Burns, Blur). Bulk `[All] / [None] / [Invert]` buttons.
- **Rotation interval slider** — 10–120s (was hardcoded 45s). Label in the sub-description auto-updates.
- **Transition duration slider** — 0.1–6s, independent of rotation interval. Ken Burns uses rotation interval for its slow-zoom but transition duration for the opacity crossfade.
- **Playback order** — segmented `[Sequential] [Random]` button group. Random reshuffles at each full queue wrap.
- **Flash overlay** — new `#bgFlash` div inside `#bgLayer` at z-index 3; used by Fade→black and Fade→white without covering session UI (UI is at z-index 1 above the whole bg-layer).
- **All persisted** through `_buildSettingsObj` + `normalizeSettings`. `bgEnabledTransitions` is validated against the registry on load — keys that no longer exist are dropped silently.
- **Legacy `bgEngine.crossfade(url)`** kept as an alias to `swap(url)` so any older callers still work.

**Deferred to Round 4:**

- **Live Settings panel integration** — during-session controls to toggle transitions on/off without leaving the session. Runtime changes to `state.bgEnabledTransitions` already work; just needs UI in `.live-panel`.
- **Per-transition weights** — instead of uniform random, let users weight favorites (e.g. Crossfade 70% / Ken Burns 20% / Blur 10%). Needs a compact UI.
- **Manual advance** — keyboard shortcut (e.g. `→`) during sessions to skip to the next image. Currently tied to the interval only.
- **Per-phase transition hints** — Induction uses calmer transitions (Crossfade, Fade→black); Peak allows more active ones (Zoom, Slide). Would let the Smart Director bias transition selection, not just content.
- **Transition preview in Step 5** — hover a checkbox to see a quick demo using the first two cached images.

---

## Perchance freshness injection ✅

**The problem:** AI script generation goes stale across sessions. Same profile + same persona + same method produces eerily similar metaphors and phrasings because the LLM has no reason to break pattern. Memory retrieves the past, adaptive narrows preferences — neither introduces novelty.

**The fix:** a new `freshSeeds` namespace in the DSL (`perchance_1.txt`) that holds ~50 curated sensory anchor phrases across 4 categories (calm imagery, somatic cues, breath anchors, metaphor seeds). On each full script generation, the HTML panel picks 4 profile-weighted phrases and injects them into `sharedContext` as **OPTIONAL ANCHORS** — suggestions the AI can weave in if they fit, ignore if they don't.

**What shipped:**

- **Combinatorial DSL lane** — `root.freshSeeds` with 4 categories × ~12 entries, each entry a `{phrase, tags}` JSON object. Phrases are second-person present-tense anchor images (no "you notice…" framing; that's the AI's job).
- **Stateless picker** — `root.freshSeeds.pick({ profileTags, n, excludeSet })` returns `n` phrases weighted by tag overlap with the profile. Base weight 1, +2 per matching tag, ×0.1 if in excludeSet. Weighted-random without replacement.
- **HTML-side tag derivation** — `deriveProfileTags()` pulls tags from experience, goals keywords, intention keywords, and the diff-bounce image-gen log (aesthetic preferences feed freshness picks automatically).
- **Rotation memory** — HTML panel tracks `_recentAnchors` (FIFO cap 30) and passes them as excludeSet so back-to-back sessions don't repeat.
- **sharedContext wiring** — new `freshSeeds` block at priority 2 (drops before adaptive, after director/profile). Silent graceful degradation: empty string if the DSL namespace is missing (old perchance_1 with new perchance_2) or Smart Director is off.

**Defaults:**
```
FRESHNESS_COUNT         = 4       phrases per generation
FRESHNESS_MIN_TAGS      = 0       always pick, even with no tags
FRESHNESS_ROTATION_CAP  = 30      entries remembered
```

**Deferred to a future round:**
- User-editable freshness lists — UI to add/remove anchor phrases from Settings
- Per-phase freshness — different anchors for Settling vs Work vs Wake
- Community freshness packs via the existing `dynamicImport` / `communityPacks` pattern
- Freshness A/B tracking — log whether anchors were present and whether mood delta was better, auto-tune algorithm
- LLM-derived tags — replace hard-coded regex keyword matching in `deriveProfileTags` with periodic director distillation

---

## Profile ingestion + diff-bounce history ✅

**DiffBouncer substrate** — new `DiffBouncer` class (one instance per tracked source) that gates commits on two conditions: temporal (8s idle) AND magnitude (threshold per source type). Sub-threshold changes accumulate; only meaningful shifts commit. All tunables live in `DIFF_BOUNCE_THRESHOLDS` — single findable constant.

**5 CSV/text fields covered** — `pfGoals`, `pfAbout`, `intentionInput`, `userAffirmations`, `bgPromptInput`. Wired via `input` events to the bouncers; no existing behavior altered.

**Image generation ingested** — every successful `bgEngine.generateOne()` fires-and-forgets an async summary derivation. With remote processing consent, uses `aiTextPlugin` for a 6-10 word summary + 3-6 aesthetic tags (strict JSON out, salvage-parsed, fallback-tolerant). Without consent, falls back to rule-based tokenization (nouns + adjectives, stop-list filtered). Jaccard similarity ≥ 0.70 against the last committed tag-set skips repeat-exploration batches — a user generating 20 variants of the same prompt produces one log entry, not 20.

**Change log storage** — `profile.changeLog` field on the existing Profile object; ring-buffered at 200 entries newest-first; persisted through the existing IDB + localStorage path. Additive migration (old profiles seed empty array).

**Session-start hard-commit** — `startSession` calls `hardCommit('sessionStart')` on every bouncer. Bypasses the magnitude gate (still requires `hasChange`) to anchor baselines at meaningful moments.

**Timeline UI** — new sidebar `<details>` block "change log" rendering the log newest-first. Per-type rendering: CSV entries show green/red chips for added/removed, text entries show % changed + preview, image entries show summary + tag chips. Collapsed by default; rendered lazily on expand.

**Smart Director integration** — `getProfileContext()` appends "Recent changes:" with the latest 10 entries as one-liners. Director now has a time-aware view of what the user's been exploring.

**Safety** — never throws on image summarization failures (fallback path). Never blocks image generation on DiffBouncer work (fire-and-forget async). Consent-gated LLM calls only.

**Defaults:**
```
idleMs:               8000     (8s)
textChangePct:        0.10     (10% of chars)
csvChangeCount:       2        (entries shifted)
imageSimilarityMax:   0.70     (Jaccard threshold)
commitOnSessionStart: true
logCap:               200
```

**Deferred to a future round:**
- Rollback UI — restore profile state from a past timeline entry
- Per-source threshold tuning via Settings panel (currently requires editing const)
- Export change history as CSV / JSON
- Image-summary opt-out toggle separate from general remote processing consent
- Tag normalization / aliasing (currently raw from LLM or regex, so "portrait" and "portraits" register as distinct tags)

---

## Audio-reactive visualizer ✅

Made the 7 existing visualizers pulse and flow with the narrator's voice + ambient soundscape, and added a new WebGL flagship visualizer designed ground-up for audio reactivity.

- **Audio signal layer** — `AnalyserNode` tap on the soundscape's WebAudio graph + synthesized envelope from `speechSynthesis.onboundary` events (with a pseudo-envelope fallback for browsers where onboundary doesn't fire). Produces `state.audioReactive = { bass, mid, treble, overall, voicePulse, voiceEnvelope }` at 60fps.
- **Mode toggle** — `Off / Subtle / Dramatic` segmented control, persisted, mirrored between Step 5 and the Live panel. Gain is applied at visualizer read-time (`getAudioMod()`), so mode changes take effect on the next frame without restarting the engine.
- **7 existing visualizers retrofitted** — Highway, Waveform, Mandala, Tunnel, Breathing, Smoke, Aurora. Each gets 2–4 lines added for audio modulation; zero-regression when mode is Off (the helper returns all zeros). **Breathing tempo stays locked** — audio only modulates color warmth + outer halo, never timing.
- **Waveform draws real FFT** when audio is active; falls back to its existing idle sine when silent.
- **Neural Bloom flagship** — WebGL2 (WebGL1 fallback, canvas 2D fallback) fullscreen visualizer with four composited layers: fBM gradient background, central breathing orb (size modulated by overall + voiceEnvelope), 300 GPU point-sprite particles orbiting at bass-modulated radius, and bloom bursts (20 petals spawned per word boundary with 3Hz cap). Phase-aware color palette interpolates over 3s between settling → deepener → work → wake.
- **Safety constraints** (structural, not tunable):
  - Bloom burst rate-limited to 3Hz (333ms minimum between spawns)
  - Orb radius smoothed to max ~2Hz breathe cycle
  - Asymmetric fade (150ms in, 850ms out) — no strobe-flashes even at Dramatic
  - Breathing visualizer tempo never modulated by audio
- **Accessibility** — on first load, `prefers-reduced-motion: reduce` at OS level flips the default to Off. Explicit user choice is respected thereafter.
- **Frame-time guard** — rolling 30-frame average; if avg >33ms (<30fps), Neural Bloom halves its particle count automatically.

**Deferred to a future round:**

- **Per-viz custom reactivity profiles** — user sliders to fine-tune bass/voice contributions per visualizer.
- **Microphone mode** — react to user's own voice for group-guided sessions.
- **Audio-reactive bg transitions** — pipe `voicePulse` into the bg-image transition system so word boundaries can trigger Ken Burns shifts.
- **Beat detection** — add an onset detector so visualizers can "know" when a drum hit happens in music-heavy soundscapes.
- **Export session as video** — record the combined canvas + audio output to webm.
- **Connection filaments on Neural Bloom** — the spec mentions an optional 5th layer (faint lines between nearby orbiting particles during intense audio moments). Shippable as an increment.

---

## Script Block Editor — Round 2 ✅

Session structure editor rework. **Shipped:**

- **Unlimited Work and Affirmation blocks** — hard caps removed. Adding blocks past the length default still works; the progressive script generator adapts its word budgets per block.
- **Pair-aware add/remove** — Induction automatically adds Wake + Reintroduction when missing; Deepener automatically adds a matched Lightener. Removing either half prompts to remove its partner.
- **Nested Deepener/Lightener pairs** — users can stack multiple depth layers; lighteners unwind in reverse (LIFO). Each Deepener carries a visible depth level (1–N) and the generator produces depth-aware language ("take them ONE LEVEL deeper, not all the way from waking").
- **New default structure:** `[Settling][Induction][Deepener] → (Work, Affirm)×N → [Lightener][Wake][Reintroduction]`. Affirmation now *follows* each Work (per spec), replacing the previous interleaved pattern.
- **Role-tagged Work blocks** — each Work block is assigned a narrative role (INTRODUCE / DEVELOP / INTENSIFY / REINFORCE / PEAK / CONSOLIDATE) based on its position and total count, so a 10-block session doesn't sound like 10 copies of the same paragraph.
- **Pre-session safety gate** — if the script is missing Lightener, Wake, or Reintroduction (and isn't a Sleep-intention session), or if Deepener/Lightener are unbalanced, a modal blocks `startSession` with *"← Go Back"* / *"Proceed Anyway (Unsafe)"* options.
- **Visual pair coding in the editor** — trance-entry pair (Induction ↔ Wake/Reintro) uses cyan; depth pairs (Deepener ↔ Lightener) use purple with opacity scaled by depth level. Hovering a paired phase highlights its partner.

**Deferred to a future round:**

- **Fractionation sessions (multi-induction)** — the code currently blocks a second Induction with a friendly message. Real fractionation support requires modeling "trance cycles" as first-class groups (induction → deepener → work → lightener → emergence × N cycles) rather than flat phase lists.
- **Drag-and-drop reordering with positional enforcement** — move still uses up/down buttons. HTML5 drag-drop with constraint validation (Lightener can't be dragged above its Deepener, etc.) is a natural upgrade.
- **Timeline view alongside the list** — horizontal bar showing phases as proportionally-sized colored segments. Visual arc is easier to read at 15+ phases.
- **Per-phase duration estimate** — e.g. `Work 3 · ~60 words · ~1m 10s`. Live-recomputed as blocks change. The runtime data is already there (`targetWords`, `wpm`); just needs rendering.
- **Budget/imbalance diagnostics** — quiet inline warning area: "est. session 22min — work section is 85% of total time." Shows when something's off.
- **Structure templates** — one-click "Classic Linear / Deep Sleep / Peak Intensity" presets that replace the whole phase plan.
- **Collapsible block groups** — when a session has 10 work blocks, the editor list gets long. Group them under a `▸ Work Section (10 blocks)` header.
- **Affirmation placement patterns** — `woven / pulses / bookend / climax / rhythmic / custom` dropdown on the Affirmations step. Currently only woven-via-prompt + the new Work-Affirm pair default.

---



## Already shipped ✅

| # | Item | Shipped change |
|---|------|---------------|
| 1 | `uploadPlugin` share links | `createShareUpload`, `shareCurrentConfig`, `shareCurrentPersona`, `loadUploadedSharePayload`; share buttons on script preview + persona editor |
| 2 | `$meta.dynamic` | Inline method/intention/preset maps in `perchance_1.txt` for tailored link previews |
| 3 | `dynamicImport` community packs | `communityPacks` lane in DSL; `loadCommunityPackPersonas` merges them into `refreshPersonas` |
| 4 | Semantic memory via `embedTexts` | `embedAndStoreDigest`, `findRelevantDigests`, `cosineDistance`; `generateFullScript` now surfaces top-K semantically relevant digests alongside the rolling/long-term memory |
| 5 | Token budgeting | `trimToTokenBudget` helper with priority-based dropping; applied to `generateSinglePhase`, `generateFullScript` sharedContext, and `craftSuggestion` |
| 6 | Streaming phase generation | `onChunk` streaming in `generateSinglePhase`; `setStreamingPreview` renders live text into a new `#genStreamBox` between the phase dots and the script preview |
| 9 | `stopReason === 'error'` audit | Hardened `aiGenerate` (HTTP status, error field, empty-text detection); routed `updateMemorySummary` through it; explicit error + empty-text guards in `generateSinglePhase` and `craftSuggestion` |
| 12 | Regeneration diff | `showPhaseDiffModal` with side-by-side compare; `retryPhase` now captures original text and asks the user to pick |

---

## Backlog

### High impact

#### 7 · `textToImagePlugin` beyond backgrounds — *partly addressed*
The plugin is already imported for background generation. Round 2 of the image
rework shipped real POS/NEG fields, orientation, weight sliders, and 4-way
parallel generation (see "Image generation — Round 2" above). The three
original proposals here remain open and have been merged into the Round 3
candidate list:

- **Personal symbols for crafted suggestions.** When `craftSuggestion()` finishes,
  generate a small (512×512) image representing the `trigger` and `coreResponse`
  keywords. Show it in the `#craftedPreview` card and in the post-session trigger
  reminder. Symbols aid post-hypnotic recall — a visual anchor the user can revisit.
- **Persona portraits.** The guide picker cards in step 1 are currently all
  identical. Generating a stylised portrait per persona (on first view, cached
  to IDB via the existing `bgCacheStore` pattern) dramatically improves scannability.
  Builtin personas can be pre-generated server-side and shipped as static URLs;
  custom personas generate on demand.
- **Program-day achievement badges.** When a user completes day N of a
  multi-day program, generate a unique badge. Display in the heatmap sidebar.

Implementation notes: cache aggressively — these are one-shot generations that
shouldn't regenerate on every view. Reuse `bgCacheGetUrl` / `bgCacheStore`
infrastructure. Gate behind `state.remoteProcessingOptIn` since image gen hits
a remote endpoint. Add a `personaImageUrl`, `suggestionSymbolUrl`,
`programBadgeUrl` field to the respective IDB records.

#### 10 · Cross-device sync via `root.kv`
Settings already mirror to `_kv.settings`. History, adaptive profile, the 3-tier
memory, and embedding vectors are IDB-only, so a user on their phone loses all
continuity from desktop. The adaptive system only pays dividends with long
continuity, so this matters more than it looks.

Scope:

- Mirror `stillness_history_v1`, `stillness_memory_v1`, `stillness_embed_v1`,
  `stillness_adaptive_v1`, and `stillness_analytics_v3` to `_kv.user.*` on write.
- On load, do a last-write-wins merge based on a `_syncedAt` timestamp per record.
- Conflict resolution for history: union by `id` (already deduplicated in
  `importBackup`). For the adaptive profile: server newer wins. For memory
  digests: union with dedup.
- Size cap: `root.kv` has per-value limits. Embeddings store could get large —
  spill oldest vectors first.
- Gate behind `state.remoteProcessingOptIn` with clear wording about what syncs.

Dependency: this pairs naturally with item #8's API-key concerns since any
sync plumbing that runs against user config should also be audited.

#### 11 · Offline session package
Users on a commute or flight can't regenerate if AI fails mid-session.

Current blocker: the comment at line ~6752 says "TTS can't be captured via
Web Audio." Not quite true — you can pipe `SpeechSynthesisUtterance` through a
`MediaStreamDestination` on some browsers, and more reliably use a remote TTS
that returns a blob (ElevenLabs, OpenAI TTS, or the Web Speech API via
`speechSynthesis.speak()` → `MediaRecorder` on iOS Safari 17+ and Chrome 115+).

Deliverable: a "download offline package" button on the script preview that
bundles:

- Script text (+ phase markers preserved)
- Pre-synthesised TTS audio per sentence as `.webm` blobs
- One ambient loop (procedural soundscape rendered to a ~30s looping .ogg)
- A minimal `index.html` player that stitches everything together and runs
  fully offline

Store the zip via `Blob` + `JSZip` (or hand-rolled zip — the format is
simple). File size target: under 30 MB for a medium session.

#### 13 · Live adaptive pacing via mic
Most paced-breathing methods assume a 4-7-8 rhythm and run on a fixed timer.
With `getUserMedia({audio: true})` + a simple amplitude envelope, the session
can detect the user's actual exhale timing and adjust `interruptibleSleep`
durations to match.

Approach:

- On session start, if method is breath-paced and the user opts in, request
  mic permission.
- Create an `AnalyserNode` over the mic stream; track windowed RMS at 50 Hz.
- A falling edge below a noise-floor threshold for >200ms marks an exhale end.
- During breath phases, instead of `interruptibleSleep(targetMs)`, wait for
  the next exhale + a small buffer, with a hard ceiling of 1.5× target so a
  silent user doesn't stall.
- Never send audio off-device. State the no-upload promise clearly in the
  opt-in modal.

Edge cases: noisy environments (fall back to timer after 3 failed
detections); user breathing through nose inaudibly (same fallback).

### Medium impact

#### 14 · Pre-generate tomorrow's program session
Programs build day-over-day, so latency at the start of each day's session is
especially annoying. While the current session plays (long idle window for
AI), generate tomorrow's script in the background.

Implementation:

- At session start, if `!_aiInProgress` and a program is active and tomorrow's
  script isn't cached, kick off an async generation using tomorrow's program
  parameters.
- Cache by `(programId, dayIndex)` in IDB under `stillness_pregen_v1`.
- Gate on mobile: skip if `navigator.connection?.effectiveType === 'slow-2g'`
  or `navigator.getBattery?().level < 0.2`.
- Expire cached pre-gens after 48 hours so stale scripts don't leak.

#### 15 · `prefers-reduced-motion` audit
The canvas renderers (tunnel, spiral, vortex, kaleidoscope, plasma, flow-field)
are genuinely seizure-risk-adjacent for some users and motion-sickness-inducing
for others.

Deliverable:

- Check `matchMedia('(prefers-reduced-motion: reduce)').matches` once at init.
- If true, default `state.vizOverride` to `'candle'` or `'colorwash'` (slow,
  gentle) and halve `vizSettings.intensity` and `vizSettings.speed`.
- Hide or warn on the aggressive visuals (mark with a "⚠ motion" badge in
  `state.vizOverride` dropdown).
- Honor the preference on every `setupVisual()` call, not just first paint,
  since the user may toggle it mid-session in OS settings.

### Security / technical debt

#### 8 · Browser API key exposure
Lines in `aiGenerate` put `x-api-key` (Anthropic) and `Authorization: Bearer`
(OpenAI) directly in a browser `fetch`, relying on
`anthropic-dangerous-direct-browser-access: true` to bypass CORS. Any XSS or
injected script in a shared persona's `systemPrompt` could exfiltrate the key.

Mitigations to add:

- **Explicit warning on key entry.** Current UI lets users paste a key silently.
  Add a modal that lists the risks ("anyone who can run JS in this tab can read
  this key — never use your primary account key here").
- **Sanitise shared content strictly.** Shared personas' `systemPrompt` /
  `tagline` / `name` should be rendered via `textContent` only, never `innerHTML`.
  Audit every place these cross into the DOM.
- **Scrub keys after inactivity.** If the page has been idle > 30 days, clear
  `_extAiKey` from storage and require re-entry.
- **Separate keys per provider.** Currently `_extAiKey` is a single field. Store
  anthropic/openai keys separately so users aren't tempted to paste the wrong one.
- **Consider a server-side proxy option.** For users who care, document how to
  run a tiny proxy (Cloudflare Worker, Vercel function) that holds their key and
  gates it behind an origin check. Perchance can't host this but can link to a
  template.

### Perchance skill-checklist items

Running the §14 checklist from the `perchance-api` skill against the current code:

- [ ] `stopSequences` in all phase calls should include `"\n\n[["` as a safety
      net against chat-format bleed. Currently uses `['\n\n\n\n']`. Low risk
      for this app since we never use the `[[Name]]:` completion format, but
      the defensive addition is cheap.
- [ ] **iOS Safari `maximum-scale=1` viewport patch** is present (line ~2260) ✅
- [ ] **`tryPersistBrowserStorageData()`** — `navigator.storage.persist()` is
      called inside `_initStorage` ✅ but only on first run. Per the skill it
      should be called once at end of page load too, to catch the case where
      IDB was initialised but persistence was denied; after user interaction,
      browsers are more likely to grant it.
- [ ] **Mobile AI preload delay** is correctly implemented:
      `if(window.innerWidth < 500) setTimeout(..., 5000)` ✅
- [ ] **Emergency export timer** — the skill recommends a 10-second failsafe
      that shows an export button if page load stalls. Worth adding given this
      app has a lot of IDB migration on first load.
- [ ] **`exportRawDb`** fallback pattern — current `exportBackup` assumes
      clean data. For users with corrupt records (rare but reported), the
      skill's `corruptItemReplacer` pattern saves the export from aborting.

---

## Open questions / ideas for later

- **Script diff for phase retries — beyond side-by-side.** Current
  implementation shows both in full. Actual Myers diff with highlighted
  inserts/deletes per sentence would be nicer for long phases.
- **Share-link expiry.** `uploadPlugin` content is permanent; for sensitive
  personas users might want a 24-hour link. Not natively supported — would
  require a wrapper format that includes an `expiresAt` and client-side enforcement.
- **Community pack registry.** Once item #3 has real content, a discovery UI
  that browses available packs (not just hardcoded generator IDs) becomes
  valuable. Could be a separate Perchance generator that exposes a curated list.
- **Multi-language narration.** The `aiTextPlugin` accepts any language.
  Detecting browser locale + auto-switching wizard strings + finding a
  matching voice pack would open the app to non-English users. Non-trivial
  because the system prompts are currently all English.
- **Voice cloning hook.** ElevenLabs or OpenAI TTS via user-provided key
  slots into the same architecture as the Anthropic/OpenAI AI provider. Adds
  "premium narration" without Anthropic running the compute. Must address
  item #8 first.
- **Post-session journaling via Web Speech API.** `webkitSpeechRecognition`
  to transcribe spoken reflections. Pair with `finishSession` and feed into
  the memory digest.
- **A/B experiment mode.** The adaptive system already learns. An explicit
  experiment mode randomises persona/method within a shortlist and reports
  which produced the largest mood delta over N sessions — useful for power
  users.

---

Last updated: 2026-04-24 (Smart Director Passes 1+2, Director calibration, Image generation Round 2)