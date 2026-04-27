# Advanced Hypnosis Narration Engine — Roadmap

This file tracks improvements not yet shipped, ordered roughly by least to most effort. Items 1–6, 9, and 12 from the original review have already been implemented. The Script Block Editor Round 2 has shipped. Recent walkthrough patches and improvements are in §0.

---

## 0 · Recently shipped (this round)

| Item | Note |
|------|------|
| Webcam cleanup on content-warning bail | `stopWebcam()` + opt-in checkbox reset added to the rescue cleanup block in `startSession`; `stopAudioReactiveLoop()` also fires |
| Alertness + depth persistence | All three post-session sliders (mood, depth, alertness) now write to `history[0]` via the shared `persistPostCheckin()` helper; was previously discarding 2 of 3 signals |
| `moodFeedback` drowsy-message hygiene | Alertness handler now restores the mood-delta message (or clears) when alertness recovers, instead of leaving the stale "drowsy" warning |
| Journal save on screen change | `flushJournalSave()` runs before `restartBtn` / `returnToSetupBtn` / `postBackToScriptBtn` switches screens, plus the existing blur handler |
| Null-safe `restartBtn` | Matches adjacent `?.` pattern |
| Reduced-motion mid-session listener | `matchMedia('(prefers-reduced-motion: reduce)')` `addEventListener('change', …)` halves viz dynamics live and swaps intense visualizers to candle when the user toggles the OS preference mid-session |
| Crafted-suggestion voice preview | `craftedVoicePreviewBtn` / `craftedVoiceStopBtn` in the crafted-preview card; uses the same TTS path (system + local) as the main voice preview |
| Config presets save/load/delete | New "presets" bar at top of Step 5: snapshot length/voice/soundscape/viz/audio-reactive/HUD into named bundles. Stored in IDB under `stillness_config_presets_v1`. `getCurrentConfigSnapshot()`, `applyConfigSnapshot()`, `listConfigPresets()`, `saveConfigPreset()`, `deleteConfigPreset()`, `renderPresetDropdown()` |
| `promptAsync` modal helper | Companion to `confirmAsync` — text-input modal with Enter to submit / Esc to cancel. Used by the preset save flow; reusable for any future "name this thing" UX |
| **Safety: phase-navigation removed** | Reverted skip-phase button + `N` shortcut. Per the safety review, advancing through phases mid-session could let users skip past Lightener/Wake/Reintroduction, leaving them deep with no return path. Only allowed in-session actions: pause/resume, end & re-alert (full rescue), back to script (with confirm — ends session). `goToStep` also has a defensive guard that blocks while `_sessionStartTime && !_finishing` |
| Background-generation toasts | Existing `notify.success/warn` for script generation completion are auto-suppressed during session playback (via `_isInSessionPlayback`). The bg-image queue now also fires a one-shot completion toast once the active drain finishes |
| "Regenerate fresh" with explicit SD mode picker | New `regenFreshBtn` next to the existing regenerate buttons. When a user has pasted or manually edited a script and wants a fresh AI generation, this lets them choose Smart Director (profile + adaptive + memory) or Defaults (no personalization). Per-call override; persistent SD setting unchanged |
| Soft-pause overlay always-visible bug | Removed five orphan `-webkit-` CSS fragments left from prior cleanup. Critical one ate the `display: none` for `.soft-pause-overlay`; another ate `min-height: 0` for fullscreen nav buttons |
| `stopSequences` defensive `\n\n[[` on the two direct `aiTextPlugin` calls | Image-tagger + craftSuggestion now match the merge done in `aiGenerate`'s Perchance branch |
| `getAITier()` helper + `window.HNE` console diagnostic | Read-only shim: `HNE.tier()`, `HNE.profile()`, `HNE.adaptive()`, `HNE.history(n)`, `HNE.smartDirector()`, `HNE.bias()` — foundation for §4.2 |
| Director-calibration "what changes at this notch" hint | Live concrete-impact line under the slider: cold-start sessions, completion floor %, re-entry days, beginner cap state, tone — same math as `computeDirectorBrief` |
| Hide intense viz options for reduced-motion users | Sinks intense options to bottom of optgroups; sinks mostly-intense optgroups to end of select. Preserves choice without disabling |
| "Why this is greyed out" tooltip on advanced methods | Per-card lock state cached on `state._methodLockState` by `applyMethodGating`; locked cards get 🔒 badge, dimmed via `.card.locked`, click → `notify.info(reason)` instead of selecting |
| Director-brief reasoning shown post-session | New `#postDirectorBriefBox` card; renders `brief.reasoning[]`, `brief.warnings[]`, `brief.recommendations`, and bias note. Hidden on emergency exit, cold-start, or when SD was off |
| Per-persona/method completion-rate badges | New `computePickerStats(history)` populates `state._pickerStats`; persona + method cards get a bottom-right `N× · %compl · Δavg` badge with full-detail tooltip. Threshold 2+ uses |
| Privacy checkboxes per-field on profile | `pfShareAge` / `pfShareAppearance` / `pfShareGoals` — default-true so older profiles keep parity. `getProfileContext` gates AI exposure. Local Director still uses fields regardless |
| Pre-session readiness score | Derived 0-10 number from mood/tension(inv)/energy via `_updateReadinessScore()`. Live banner `#readinessRow` with band-shifted hint (depleted/low/balanced/resourced) |
| Why-this-session tag chips on Step 3 | 8 chips (anxious / can't sleep / overwhelmed / before event / after stressor / focus needed / processing emotion / just exploring); `state._sessionTags` Set persisted to `history[0].sessionTags`. Cleared on `resetWizardForNewSession` |
| Phase-aware base speech rate | `state._phaseRateMult` layered into `tickPlaybackDynamics`: `userBase × phaseMult × audioReactiveScale`. Map: settling 1.0, induction 0.92, deepener 0.85, work 0.95, lightener 1.0, wake 1.05, affirm 0.93 |
| Yes-set / pace-and-lead enforcement | Settling + induction prompts rewritten with explicit "three undeniable observations" + "TWO/THREE pace statements transition with 'and' or 'as' into ONE lead suggestion" |
| Embedded-command markup `[[…]]` + TTS dip | Pre-sanitization swap to `\u0001…\u0002` sentinels; per-sentence `_parseEmbeddedSegments()` splits into segments; `speak()` accepts `{volMult, pitchMult}`; embedded plays at vol×0.7 pitch×0.92. Markup primer added to AI prompt context (priority 2) |
| Pattern callouts on sidebar | `derivePatternCallouts(history)` with 6 ranked rules: repeat emergencies, sleep struggles, declining mood, improvement streak, gap return, method monotony. Top-2 surfaced in cyan-bordered banner above sidebar stats |
| Emergency-export 10s failsafe | Tightened from 30s, removed `remoteProcessingOptIn` gate so all stuck-boot users see it |
| Trigger-fired tap → adaptive reinforcement | New `TRIGGER_FIRES_KEY` IDB store; post-session button (debounced 1h, idempotent); `formatDirectorBriefForPrompt` reads `brief._triggerInfo` to emit "reinforce confidently" (3+ fires) / "consider refining" (0 fires + 7+ days) |
| Diff-aware AI prompting | `getProfileContext` now decay-weights recent changes by `exp(-days/14)`, drops below 0.1 weight, adds explicit "first session since user updated X and Y" callout for ≤7-day text/csv changes |
| Per-domain memory channels + mood-weighted retrieval | `findRelevantDigests(query, k, {domain, preMood})` — domain filters by `intentionType`; `preMood ≤ 4` reduces distance up to 30% for positive-delta digests. New digests tagged with intention/preMood/postMood |
| Token-budget-aware memory injection | Uses `aiTextPlugin({getMetaObject:true})` to size memoryCtx to 25% of `(idealMaxContextTokens − 800)` per perchance-skill §6.3. Falls back to current behavior if meta API absent |
| Weekly digest banner | Once-per-7-days `notify.info` on first load showing 7-day session count, completion %, mood Δ, top guide, top method. `localStorage.stillness_weekly_digest_last`; min 2 sessions in window |
| Voice journaling | `webkitSpeechRecognition` mic button next to journal textarea. Hidden when API absent. Continuous mode appends final transcripts; toggle-stop on second click |
| Diff visualization in profile panel | `#profileDiffsList` populated by `renderProfileDiffs()` from `_profileChangeLog`. Same source-label as AI sees; decay-weighted opacity; +/- color coding for csv diffs. Refreshed on `commitChangeLogEntry` and after profile load |
| AI-data audit log | `AI_AUDIT_KEY` IDB ring buffer, cap 100. `logAICall(provider, source, instruction)` records ts/provider/source/bytes (never content). Hooked at `aiGenerate` dispatch + the two direct `aiTextPlugin` sites. Read-only modal via `#viewAIAuditBtn`; also `window.HNE.audit(n)` |
| Stereo-pan plumbing for double-induction (local TTS) | `_playBuffer(audioData, sampleRate, myGen, opts)` accepts `opts.pan`. When `state._doubleInduction` active on local-TTS path, builds two parallel `StereoPannerNode` chains at ±0.4 with 1.5% rate offset on the second voice. Graceful skip on browsers lacking `createStereoPanner`. System-TTS path can't pan (Web Speech is OS-routed) — that's a known limitation |

---

## 1 · Analysis: AI steering controls (Profile Options + Debug)

### How it works today

Three separate layers steer the AI:

1. **Smart Director toggle** (`#smartDirectorEnabled`). One master switch. When off, **all** of the following stop:
   - `getProfileContext()` returns empty (no name/pronouns/goals/about etc. to AI)
   - `getAdaptive()` and `getAdaptivePromptBlock()` return nothing
   - Memory digest retrieval is skipped
   - `updateAdaptiveProfile()` and `updateSessionMemory()` no-op after sessions
   - Self-persona role-instruction skips its profile-merge step
2. **Director calibration slider** (`#directorBiasSlider`, range −3..+3, step 1). Shifts:
   - **Cold-start threshold** (3 ± bias completed sessions before personalization kicks in)
   - **Recovery-mode mood-delta limit** (−2 ± bias × 0.5)
   - **Completion-rate floor** (0.5 ± bias × 0.05)
   - **Days-since-last for re-entry treatment** (14 ± bias × 3)
   - **Positive-pacing floor** for "stay the course" praise (1.5 ± bias × 0.3)
   - **Emergency-exit cap** (2 normally, 3 at +3)
   - **Beginner cap** (softened but never removed)
   - **Tone directive** appended to the AI prompt: −2 → "grounded, careful, gently reassuring"; +2 → "forward-momentum, energizing"
3. **Debug consolidated toggle** (`#debugMode`). Single checkbox enables a bundle of bypasses: `verbose` SD logging, `skipRescue`, `skipSafetyGate`, `skipMethodGating`, `skipCooldown`. Surfaces a corner banner so the user can see they're in debug.

### Does it accomplish its goal?

**Yes, with caveats.** The system has clear separation of concerns: Smart Director is the on/off, calibration is the dial, debug is the dev escape hatch. The math behind calibration is conservative — every threshold moves by a "human-meaningful unit per notch" so the user observes a real shift without flipping the app's personality, and safety floors (beginner cap, emergency cap) can be softened but never disabled.

The tone directive (`_toneDirectiveFromBias`) is a small but important touch — it gives the AI a stylistic nudge even when no rule actually fires, so the calibration "feels" applied to every session and not just the rare edge cases.

### Where it could be improved

- **The slider has no in-context preview.** A user moving from 0 → +2 sees the label change to "lean forward" but doesn't see what concretely changes. A small explanatory line that updates with the slider would help: *"At +2: cold-start ends after 1 session, beginner cap softer, tone leans energizing."*
- **No A/B / experimentation hook.** The bias is global. Power users would benefit from "test a +1 bias for the next 3 sessions" temporary mode, with mood-delta tracking comparing to baseline.
- **No surfacing of what the Director actually did.** The director brief is logged to console under `sdLog`, but the user never sees the reasoning. A post-session "why this session was shaped this way" expandable card would build trust and let users adjust calibration consciously.
- **The bias is a single dial.** Real practitioners want more axes — "gentler tone but longer sessions", "more authoritarian but lighter pacing". A two-axis (intensity × pace) or three-axis (tone × depth × length) calibration would give finer control without overwhelming new users (who can leave it at "balanced" on all axes).
- **No per-domain calibration.** Someone might want forward bias for productivity sessions and cautious bias for sleep sessions. The current global bias mixes both.
- **Smart Director "off" is too binary.** Currently off means *no* personalization. An intermediate mode — pass profile but not memory/adaptive — would help users who want stable personalization without the system "learning" past sessions.

---

## 2 · Analysis: Trance induction effectiveness

### Does the app accomplish its goal?

**Yes for the core mechanism, with significant headroom on technique.** The script architecture (Settling → Induction → Deepener → Work × N → Lightener → Wake → Reintroduction) follows accepted clinical structure (Erickson, Kroger, Hammond). Key wins already shipped:

- **Per-phase TTS pacing** with directive-phrase pause expansion (`DIRECTIVE_RX`, `ELLIPSIS_RX`)
- **Deepener/Lightener pair tracking** with depth levels and LIFO unwinding
- **Pre-session safety gate** for missing wake/lightener/reintro phases
- **Soundscape + visualizer ducking** on speech onset (`drone.duck()`)
- **Phase-frequency drone shifts** that reinforce the depth arc subliminally
- **Audio-reactive visual modulation** that ties imagery to voice envelope
- **Wake Lock + lock-screen Media Session** so the device doesn't interrupt
- **Webcam-aware attention/eye-state sensing** (phase-aware: closed eyes are good during trance)
- **Adaptive personalization** of persona/method/length from history
- **Token resolution** (`{{user}}`, `{{nar}}`, `{{intention}}`, `{{affirmation}}`) baked into both AI prompts and pre-TTS

### Where it could be improved (technique)

These are levers research has shown matter for trance depth and durability of post-hypnotic effects, in roughly increasing-effort order:

1. **Embedded commands (lower volume / pitch shift).** Hypnotists drop voice volume on the command word inside a sentence (e.g., *"and you might find yourself **going deeper** and noticing how the room shifts"*). Could be implemented as `[[command]]` markup in the script + a per-utterance volume/pitch dip via SpeechSynthesisUtterance. The AI is good at producing these — we just need to ask for them and respect the markup at TTS time.

2. **Marked pauses with breath cues.** Most "[PAUSE 5]" markers are silent. A subtle breath-in-out cue at the start of a long pause is a powerful pacing tool. Could be a soft synthetic exhale, or even a single low note that fades.

3. **Confusion technique mid-induction.** Erickson's confusion-and-utilization patterns (the deliberate non-sequitur or paradox that the conscious mind tries to parse and gives up). The AI knows these patterns; the prompt could request them for `confusion` method specifically.

4. **Voice modulation across the arc.** Currently voice rate is a global slider. The phase has an obvious "right speed" — settling fast-then-slow, induction slow-and-slowing, deepener slowest, work moderate, lightener accelerating, wake brisk. We have `_baseSpeechRate` in state but only modulate via audio-reactive — a phase-aware base rate would be more durable than current behavior.

5. **Yes-set / pace-and-lead pattern.** Three observable truths followed by a suggestion: *"You're sitting here, you can hear my voice, you're breathing, **and** with each breath you find yourself going deeper..."* The AI is great at this; we just need a phase-specific prompt for "induction" to require N pace statements before each lead.

6. **Embedded fractionation across a session.** True fractionation = induction → wake to a partial trance → deeper induction → wake → deeper still. The current code blocks a second induction phase; the ROADMAP already lists this as deferred (§Script Editor Round 2). High value, high effort.

7. **Auditory drift / stereo separation for double-induction.** Two voices at slightly different pan positions create the dissociative effect of "being narrated to from both sides" — classical fractionation method. Currently `state._secondVoice` exists but plays through the same audio context. Routing each voice through a `StereoPannerNode` with opposite pan would dramatically improve depth.

8. **Posthypnotic trigger calibration over time.** When `craftedSuggestion.trigger` produces a real-world response, the user could tap a "trigger fired" button on the heatmap. Subsequent sessions reinforce the trigger (the AI has been told it's effective) or refine it (rephrase if not).

9. **Heart-rate variability biofeedback (webcam pulse).** rPPG (remote photoplethysmography) extracts pulse from face video. Already-shipped MediaPipe loader can be extended. HRV trending up during a session is a rough proxy for parasympathetic activation = trance going well; trending down can trigger an in-session adaptive softening.

10. **Mic-driven exhale detection (existing ROADMAP item #13).** Critical for breath-paced methods. Pacing the user's actual exhale instead of a fixed timer is a significant depth multiplier.

### Unexplored avenues / alpha-grade ideas

These are early/research-grade and would warrant flagging as alpha or beta features:

- **Binaural beat overlays driven by phase.** 4 Hz delta during deepener, 6 Hz theta during work, 10 Hz alpha during lightener. Already have `binaural` soundscape; expand to phase-locked frequency.
- **Isochronic tones beneath voice.** Less controversial than binaural (works without headphones) and there's modest evidence for entrainment. Ducked at −20 dB during voice, full during pauses.
- **40 Hz gamma flicker** (visualizer setting) — Iaccarino et al. showed entrainment effects on attention. Implementable as a 40 Hz strobe overlay at low intensity. Risk: motion-sensitive users; gate behind reduced-motion check.
- **Eye-state-conditioned intensity.** Webcam already tracks eye open/closed. With eyes open during settling, ramp visuals brighter; with eyes closed during deepener, drop visuals to near-black and shift all sensory weight to voice. Currently we just dim — switching the *primary modality* per eye-state is more significant.
- **Post-session intrusion-test prompt.** A common clinical assessment for depth: 30 seconds after wake, present a small unrelated cognitive task ("name three colors of fruit"). Reaction-time delay is a soft proxy for residual trance. Captured value goes into `depth` history.
- **Recall-quality probe.** Ask the user 24 hours later (push notification or banner on next visit) to recall the trigger phrase / one suggestion. Hit/miss feeds back into the adaptive system as "durability signal" separate from same-day mood delta.

### Safety stays primary

All of the above must remain inside the existing safety envelope:

- Pre-session safety gate (missing wake/lightener)
- `emergencyEnd` rescue sequence with persona-specific re-alert
- Beginner cap on depth (Director never overrides)
- Daily session cap
- Cooldown after emergency exit
- Reorientation card + 5-4-3-2-1 grounding on emergency end
- Reduced-motion preference auto-honored
- Phase navigation blocked during playback (this round)

These already work and are non-negotiable. Anything we add must layer on top, not bypass.

---

## 3 · Research pass: adjacent apps and what's worth borrowing

Survey of patterns from hypnosis, meditation, biofeedback, habit-tracking, and AI-assisted therapy apps.

### From hypnosis / meditation apps (Calm, Headspace, Reveri, Mindscape, Aura, Insight Timer)

- **Streak mechanics with grace days.** A 3-day-grace policy preserves long streaks across the inevitable bad day, which research suggests is more habit-supportive than rigid streaks. Calm + Headspace both use this pattern. We already have a heatmap; adding a streak counter with a 1-day grace would land naturally.
- **"Sleep stories" / wind-down vs activate distinction.** A clear binary at the top of the app: *Are we winding down or activating?* Routes the user past method-picker confusion. We already have `intentionType`; a one-question opener "is this for sleep tonight?" before Step 1 could shortcut for the most common use.
- **Background-only mode.** Calm has "stories that play in the background" while users are doing dishes etc. We don't, and probably shouldn't for trance work, but for the **affirmation interlude** specifically (no induction, just affirmations under soundscape), it's a worthwhile second-tier session shape.
- **Session bookmarks.** Insight Timer lets users bookmark specific timestamps in a session. For our model, a "bookmark this affirmation" while the transcript is open (and have it appear in the next session) is a reasonable analog.

### From biofeedback / body-aware apps (Muse, Whoop, Oura, Welltory, HRV4Training)

- **Pre-session readiness score.** Whoop/Oura compute a "you're depleted today" score from sleep/HRV. We have a pre-session 3-slider check (mood/tension/energy) but don't combine into a single number. A "today's readiness: 6/10" derived score with one-line interpretation ("rest preferred over peak work") is more actionable than three sliders to a beginner.
- **Strain × recovery framing.** Sessions tagged as "high strain" (advanced methods, deep depths) require recovery between them. We have a daily session cap; a more nuanced "you did a heavy session yesterday, suggest recovery today" would be more effective.
- **Breath coherence training.** Welltory explicitly trains 6-breath/min coherence breathing. Our breath-paced methods could expose this metric and show the user when they're hitting it.

### From habit-tracking apps (Streaks, Habitica, Way of Life, Daily)

- **Custom habit tags.** Users define their own labels (e.g. "before bed", "work break", "post-stressor") on each session. Aggregated, this gives them their own personal taxonomy beyond persona/method/intention. Useful for "what works when I'm stressed" queries.
- **Mood-trigger logging.** Why did you start this session? Pre-session "what's bringing you here today" with a 2-tap multi-select (anxious, can't sleep, before presentation, etc.). Routes back into the post-session "did this resolve" assessment.

### From user-profiling / journaling apps (Day One, Reflectly, Stoic, Daylio)

- **Pattern surfacing.** Daylio shows "you tend to feel low on Mondays" over time. We have heatmaps but no pattern callouts. A weekly digest "this week's pattern: shorter sessions trended better; you completed 5/7 days" would be valuable and is cheap to compute on existing data.
- **Photo + voice journal.** Day One supports voice-note journaling. Web Speech API → IDB blob is straightforward; pairs with the existing journal text field.
- **Timeline view.** Reflectly's chronological scroll of past entries with mood-color borders — much more affective than a list. We render a heatmap; a vertical timeline with persona portraits + mood deltas + first journal sentence would be richer.

### From AI-assisted tracking (Wysa, Woebot, Replika)

- **Conversational check-in instead of sliders.** "How was today?" with a free-text input the AI parses into structured sliders. Reduces friction; we already have an AI plugin and a profile system that could ingest this.
- **Pattern-based gentle nudges.** Wysa surfaces "I noticed you've been mentioning sleep a lot — would you like to try a sleep-focused session?" Our memory digests already capture themes; a once-a-week proactive surfacing would be powerful.
- **Confidence-bounded suggestions.** Woebot/Replika frame suggestions as *"based on N sessions"* — never over-claiming. Our adaptive system has the data to do this and currently doesn't surface confidence at all.

### From statistics / quantified-self (Exist, Gyroscope, Welltory)

- **Correlations across signals.** "You completed more sessions on weeks where you slept >7h." We have history; we don't show correlations. Easy first one: persona × completion rate × mood delta as a small grid.
- **Goal tracking with progress bars.** Set "complete 20 sessions in 30 days" and visualize progress. We have programs (structured day arcs) but not user-defined goals.
- **Annual / monthly review.** Year-in-review surfacing + most-used persona, biggest mood-improvement session, etc. Cheap, sticky, social-shareable.

---

## 4 · Improving our user-tracking & profiling tool — Perchance-stable, all-client-side, AI-tier-aware

Goal: tighten the feedback loop so the Director makes better recommendations and the user feels seen. Constraints:

- **Fully user-side / Perchance-stable.** No server. IDB + localStorage + (optionally) `_kv` for cross-device sync. Survives the Perchance sandbox.
- **AI-tier-aware.** Three tiers exist:
  - **Perchance free** (`aiTextPlugin`, no key, JSON-fragile, slow, capped context)
  - **OpenAI** (user key, fast, structured-outputs-capable, JSON-reliable)
  - **Anthropic** (user key, fast, longer context, very high quality, JSON-reliable)
- **Backwards compatible.** Older history records (without new fields) must keep working.

### 4.1 Signal layer (data we already collect, better-used)

Currently captured per session: `preMood, preTension, preEnergy, postMood, postTension, postEnergy, depth, alertness, journal, completed, durationSec, persona, method, intention, length, soundscape, voicePitch, voiceRate, vizSettings, profileSnapshot, workBlockCount, metrics, _startedUnsafe, ...`

**Underused:**
- `journal` text — never re-read by AI for trend extraction
- `metrics` (script analytics) — feeds adaptive only via reward, not as a feature
- `profileSnapshot` — stored but not diffed across sessions
- `_startedUnsafe` — stored but never visualized
- Eye-state / face-presence telemetry (when webcam is on) — captured live, never persisted

**Quick wins (low effort, no AI):**

- **Streak with grace days.** Pure JS over `history`; show in sidebar.
- **Per-persona/per-method completion rate badges.** Already have the data. Render in the picker cards.
- **Pattern callouts.** *"Your last 3 sleep sessions ended early — try a softer guide?"* Pure JS.
- **Pre-session readiness score.** Linear combo of (mood + energy − tension) + sleep gap heuristic. Surface as one number.
- **Why-this-session tag picker.** 8 chips on Step 3 ("anxious", "can't sleep", "before presentation", etc.). Stored alongside `intentionType`. Aggregates over time into themes.
- **Weekly digest.** Once-a-week banner on first load: "Your week: 5/7 sessions, avg mood +1.4, top guide Aria."

### 4.2 AI-tier-aware enrichment

Treat AI calls as two distinct populations:

- **Cheap/local-only path.** Stay client-side, no AI. Use heuristics + the existing memory digest system + simple cosine retrieval. This works on every tier.
- **Premium path.** When user has Anthropic or OpenAI key configured, opt-in to richer enrichment.

A single new helper, `getAITier()`, returns `'perchance' | 'openai' | 'anthropic'`. New AI features check this and pick their strategy:

```js
function getAITier() {
  if (state.extAiProvider === 'anthropic' && state._extAiKey) return 'anthropic';
  if (state.extAiProvider === 'openai'    && state._extAiKey) return 'openai';
  return 'perchance';
}
```

Per-feature behaviour tier-by-tier:

| Feature | Perchance free | OpenAI key | Anthropic key |
|---|---|---|---|
| Journal sentiment tagging | regex+lexicon score (no AI) | AI extracts 3 themes | AI extracts themes + valence + actionable next step |
| Trigger-fire tracking | manual user button only | AI confirms wording → cosine over journals | AI confirms wording → cosine over journals + suggests reinforcement |
| Pattern surfacing in weekly digest | rule-based ("3 sleep sessions ended early") | AI summarizes the week | AI writes a personal note from the Director |
| Director Brief | rule-based (current) | rule-based + 1 AI rationale line | rule-based + AI nuance + counterfactual ("had you chosen X instead, the brief would be Y") |
| Crafted-suggestion symbol image | n/a | n/a | textToImagePlugin (independent of text-AI tier) |
| Hierarchical memory summarization | every 3 sessions, 1500-char blocks (current) | every session | every session, with embedding update |

Detection of "do I have a key" must remain entirely client-side; we never tell a server about the key beyond the actual API request to the provider.

### 4.3 Profile diff log → AI as time-aware feature

We already maintain `_profileChangeLog` (ring buffer of profile/intention/affirmation changes). It's surfaced to the AI but never to the user.

Improvements:
- **Diff visualization in the profile panel.** "Last week you added 'public speaking' to goals." Stored, just not displayed.
- **Diff-aware AI prompting.** When a profile field changed in the last N days, mention the change explicitly to the AI. Currently just lists "Recent changes:" — we can be more explicit: "The user just added X to goals; this session is the first since that change."
- **Decay weighting.** Older diffs matter less. Weight by `exp(−ageDays / 14)` and drop entries below 0.1 weight from the AI context block.

### 4.4 Memory tier improvements

The existing 3-tier memory (rolling / long-term / digests + embeddings) follows the Perchance hierarchical-summarization pattern (see skill §6). What's missing:

- **Token-budget aware retrieval.** The skill recommends `idealMaxContextTokens − 800` to protect prefix cache. We could check via `root.aiTextPlugin({ getMetaObject: true })` and dynamically size the memory injection.
- **Prefix-cache protection.** The skill's batch-of-3 rule for summary writes. Our memory writes happen every session; batching to 3 would reduce prefix-cache invalidation when on Anthropic.
- **Mood-weighted retrieval.** When a user is mood-low pre-session, retrieve digests where the past delta was positive (recovery patterns). When high, retrieve novel sessions (broaden). Cheap re-rank on top of existing cosine.
- **Per-domain memory channels.** Sleep memory ≠ focus memory ≠ trauma memory. Tag each digest with its `intentionType` and retrieve from the matching channel. Big quality win and only requires a tag column.

### 4.5 Cross-device sync (ROADMAP item #10) revisited

The user wants this fully Perchance-stable. `root.kv.user.*` is the right path (per the perchance-api skill). Specifically:

- Each tracked dataset gets a single `_kv.user` key: `kv.user.history`, `kv.user.adaptive`, `kv.user.memory`, `kv.user.embed`, `kv.user.profile`, `kv.user.presets`.
- On every meaningful write to IDB, also push to `_kv.user` (debounced 5s).
- On load, pull each key and last-write-wins merge by per-record `_syncedAt` for histories; server-newer for singletons (adaptive, profile, presets); union-with-dedup for memory/embed.
- Embeddings can be large; spill oldest first to fit `_kv` per-value caps.
- Gate behind `state.remoteProcessingOptIn` with explicit listing of what syncs.

### 4.6 Privacy + safety guardrails

- **Profile fields stay opt-in for AI.** `useFirstName` already gates name. Do the same for `age`, `appearance`, `goals` — three more checkboxes, "share this with the AI". Without ticks, the AI never sees that field, but the local Director still uses it for filtering.
- **Journal text stays local by default.** Even when an external AI key is set, journals don't get sent unless the user explicitly enables a "let AI read my journals for pattern surfacing" toggle. That toggle is OFF by default.
- **Embedding store is local-only by default.** Embeddings can leak content via reconstruction. The cross-device sync above must not include embed unless the user opts in to that *separately* from history sync.
- **Audit log.** A hidden "what data has been sent to which AI" log that's user-readable. Builds trust.

---

## 5 · Backlog — sorted by effort

Effort scale: 1 (trivial, < 30min), 2 (small, < 2h), 3 (medium, half-day), 4 (large, 1–2d), 5 (multi-day).

### Effort 1

(All Effort 1 items shipped this round. See §0 above.)

| Originally listed | Status |
|---|---|
| Streak counter with grace day | Already implemented before this round (lines ~17128 — 2 single-day grace gaps, sidebar pill via `sidebarStreak`) |
| `stopSequences` defensive `\n\n[[` addition | ✅ shipped |
| Hide intense viz options for reduced-motion users | ✅ shipped |
| Show "why this is greyed out" tooltip on advanced methods | ✅ shipped |
| Director-brief reasoning shown post-session (read-only card) | ✅ shipped |
| `getAITier()` helper + console-visible diagnostic | ✅ shipped |
| Per-persona/method completion-rate badges in pickers | ✅ shipped |
| Privacy checkboxes per-field on profile | ✅ shipped |
| Director-calibration "what changes at this notch" hint | ✅ shipped |

### Effort 2

(All Effort 2 items shipped this round. See §0 above. Stereo-pan only applies on the local-TTS path; system TTS audio is OS-routed and can't be panned through Web Audio.)

| Originally listed | Status |
|---|---|
| Pre-session readiness score | ✅ shipped |
| Why-this-session tag chips on Step 3 | ✅ shipped |
| Pattern callouts on sidebar | ✅ shipped |
| Embedded-command markup `[[…]]` + TTS pitch/volume dip | ✅ shipped |
| Phase-aware base speech rate | ✅ shipped |
| Yes-set / pace-and-lead enforcement | ✅ shipped |
| Stereo-pan double-induction | ✅ shipped (local-TTS path; system TTS not pannable) |
| Trigger-fired tap → adaptive reinforcement | ✅ shipped |
| Mood-weighted memory retrieval | ✅ shipped |
| Per-domain memory channels | ✅ shipped |
| Diff-aware AI prompting | ✅ shipped |
| Diff visualization in profile panel | ✅ shipped |
| Weekly digest banner | ✅ shipped |
| Voice journaling via `webkitSpeechRecognition` | ✅ shipped |
| Token-budget-aware memory injection | ✅ shipped |
| `tryPersistBrowserStorageData` second-pass | Already implemented in earlier round |
| `exportRawDb` corrupt-record fallback | Already implemented (`_corruptItemReplacer` + per-slice `safe()`) |
| Emergency-export 10s failsafe timer | ✅ shipped (tightened from 30s, removed opt-in gate) |
| AI-data audit log | ✅ shipped |

### Effort 3

| Item | Notes |
|---|---|
| Two-axis Director calibration (intensity × pace) | More expressive than single slider. |
| Posthypnotic intrusion-test prompt 30s after wake | Reaction-time signal for depth quality. |
| 24-hour recall probe (banner on next visit) | Durability signal. Pairs with trigger-fire tap. |
| Pre-generate tomorrow's program session (existing #14) | Latency win for daily programs. |
| Eye-state-conditioned modality switching (visuals dark on closed eyes) | Bigger than current dim; more clinically meaningful. |
| Confidence-bounded recommendations ("based on 7 sessions") | Already computed; not surfaced. |
| Annual/monthly review screen | Year-in-review pattern; sticky. |
| Custom habit tags (user-defined labels per session) | Personal taxonomy beyond persona/method/intention. |
| AI-tier-aware journal sentiment tagging | Per the matrix in §4.2. |
| AI-aware Director Brief rationale line (Anthropic/OpenAI tiers) | Per matrix §4.2. |
| Phase-locked binaural frequency (delta/theta/alpha by phase) | Existing `binaural` soundscape; expand to phase-locked. |
| 40 Hz gamma flicker (alpha-grade, gated by reduced-motion) | Iaccarino-style. |
| Conversational check-in via free-text (parse to sliders via AI) | Reduces friction; uses existing AI. |
| Director-shows-its-work post-session card | Reasoning already in `brief.reasoning[]`. |
| Cross-device sync via `root.kv` (existing #10), AI-tier-aware | See §4.5. |
| Per-domain Director calibration (sleep/focus/etc.) | Solves "global bias mixes everything" problem. |

### Effort 4

| Item | Notes |
|---|---|
| Mic-driven exhale detection (existing #13) | Critical for breath-paced methods. |
| HRV via webcam rPPG | Real-time biofeedback signal. |
| Drag-and-drop phase reordering with constraints | From existing Round 2 deferred list. |
| Timeline view alongside phase list | Round 2 deferred. |
| Per-phase duration estimate (live) | Round 2 deferred. |
| `textToImagePlugin` persona portraits + suggestion symbols + program badges (existing #7) | Caching strategy from skill §3. |
| Browser API key exposure mitigations (existing #8) | Modal warning + sanitization audit + per-provider keys. |
| Multi-language narration | Locale detection + voice match + AI prompt translation. |
| Voice cloning via ElevenLabs/OpenAI TTS user-key | Same architecture as text-AI provider; different endpoint. |
| Offline session package (existing #11) | TTS pre-synth + zip + minimal player. |

### Effort 5

| Item | Notes |
|---|---|
| Fractionation (multi-induction cycles) | Round 2 deferred — needs first-class trance-cycle modeling. |
| A/B experiment mode | Random within shortlist + outcome reporting. |
| Community pack registry (existing) | Discovery UI for packs. |
| Live adaptive pacing across all methods | Generalization of mic-driven exhale. |

---

## 6 · Already shipped (running ledger)

| # | Item | Shipped change |
|---|------|---------------|
| 1 | `uploadPlugin` share links | `createShareUpload`, `shareCurrentConfig`, `shareCurrentPersona`, `loadUploadedSharePayload` |
| 2 | `$meta.dynamic` | Inline method/intention/preset maps in `perchance_1.txt` |
| 3 | `dynamicImport` community packs | `communityPacks` lane in DSL; `loadCommunityPackPersonas` |
| 4 | Semantic memory via `embedTexts` | `embedAndStoreDigest`, `findRelevantDigests`, `cosineDistance` |
| 5 | Token budgeting | `trimToTokenBudget` with priority-based dropping |
| 6 | Streaming phase generation | `onChunk` streaming; `setStreamingPreview` live-render |
| 9 | `stopReason === 'error'` audit | Hardened `aiGenerate` |
| 12 | Regeneration diff | `showPhaseDiffModal` + `retryPhase` selection |
| — | Script Block Editor Round 2 | Unlimited Work/Affirm; pair-aware add/remove; nested deepeners with depth levels; new default structure; role-tagged Work blocks; pre-session safety gate; visual pair coding |
| — | All §0 items above | This round |

---

Last updated: 2026-04-26
