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

### Round 3 — Effort 3 + adaptive layer

| Item | Note |
|------|------|
| Custom habit tags on Step 3 | User-defined tags extending the why-this-session chips. `CUSTOM_TAGS_KEY='stillness_custom_tags_v1'`, cap 20. Add field + dynamic chips with × delete |
| Confidence-bounded adaptive recommendations | `getRecommendation` returns per-field confidence (top-vs-second separation, normalized). Suppresses picks below 0.10–0.15 separation or <3 samples. Stashed at `state._lastAdaptiveRec` |
| Director-shows-its-work card on script preview | `#directorWorkCard` above script textarea. Adaptive picks with strong/moderate/tentative labels, recovery flag, calibration bias notch, active session tags |
| 24-hour passive recall probe | Stashed at `state._pendingRecallProbe`, surfaces on post-session trigger card next time user reaches it organically. One-shot |
| Posthypnotic intrusion-test prompt | 30s timer in `finishSession` surfaces "If you'd like to test X — now's a natural moment" notify. Skipped on emergency exit and ongoing-state |
| Per-domain Director calibration | `state.directorBiasByDomain` map (intentionType → -3..+3). Layered on global bias inside `computeDirectorBrief`, clamped after combining. Compact details disclosure under global slider |
| Two-axis Director calibration (intensity + pace) | `state.directorPace` -3..+3 slider. Multiplies into `tickPlaybackDynamics` as `paceMult = 1 + paceBias × 0.04`. Each notch ≈4% rate change |
| AI-aware Director Brief rationale | `formatDirectorBriefForPrompt` branches on `getAITier()`. Anthropic/OpenAI emits structured Rationale + Warnings sections; Perchance keeps compact form for prefix-cache protection |
| AI-tier-aware journal sentiment classification | Paid-tier-only on journal blur. Calls `aiGenerate` with valence/intensity/integration classification on entries ≥25 chars. Stashed on `history[0].journalSentiment`. Idempotent |
| Annual review screen | `#openAnnualReviewBtn` in sidebar stats. Hidden until ≥30 days × ≥5 sessions. Pure-aggregation modal: trailing 365 days, sessions/minutes/completion %, longest streak (2-day grace), mood arc, top guides/methods/contexts. No AI calls |

### Round 3b — UX softening (returning users seamless)

| Item | Note |
|------|------|
| Weekly digest popup → silent compute | No notify on init. Stashed at `state._weeklyDigest`, surfaced as quiet amber-bordered tile in sidebar stats |
| 24-hour recall probe popup → passive surface | Moved to post-session trigger card |
| Returning-user "welcome back" callouts removed | Day-count language gone; only 60+ day re-orientation hint kept |
| Posthypnotic intrusion-test wording softened | "If you'd like to test" instead of "Posthypnotic test: try X right now". 9s instead of 11s |

### Round 3c — Audio + Viz priorities

| Item | Note |
|------|------|
| Phase-locked binaural beats | Existing fixed 6Hz θ replaced with per-phase brain-wave map. `setPhaseFreq` ramps L/R oscillators ±0.4 with 6s linearRamp. Map: settling 8Hz α, induction 6Hz θ, deepener 3.5Hz θ-δ, level 4Hz, work 5Hz, affirm 5Hz, lightener 7Hz, wake 12Hz α, reintroduction 14Hz β. Refs at `this._binauralRefs`. Label "phase-locked α/θ/δ" |
| Isochronic tones soundscape | NEW recipe. Single 220Hz carrier with amplitude-modulating LFO at entrainment frequency. Bias source (ConstantSourceNode) keeps gain swing 0..0.4. `setPhaseFreq` ramps LFO over 6s. Refs at `this._isochronicRefs`. Works on any speaker, no headphones needed |
| Soundscape continuity on mid-session change | `drone.setPhaseFreq(state._currentPhase)` called immediately after `drone.start` in `liveSoundscapeSelect` handler — phase-locked behavior catches up without waiting for next phase boundary |
| 40Hz gamma flicker visualizer (alpha-grade) | `renderGammaFlicker` registered in `VIZ_RENDERERS`. 5–15% alpha modulation only, 45% radius radial pulse, cyan/blue palette, fixation dot center, voice envelope adds max +5%. Engaged users get full 40Hz; anonymous fall back to 0.2Hz under prefers-reduced-motion. Added to `INTENSE_VIZUALS` |

### Round 3d — Audio/viz safety relaxation gate

| Item | Note |
|------|------|
| `_audioVizSafetiesRelaxed()` gate | Returns true when profile age is set. When relaxed: gamma flicker confirmation modal skipped; reduced-motion auto-degrade on boot (dropdown reordering, ⚠ tagging, "gentler defaults applied" notify) skipped; mid-session reduced-motion swap doesn't force-swap user's choice; speech rate caps widen 0.4×–2.0× (silent clamp); soundscape volume cap opens to 0..1 (silent clamp); soundscape labels soften ("⚠ Binaural beats require..." → "Binaural beats — best with stereo headphones"); optgroup label "Experimental ⚠" → "Experimental". Casual users (no age set) keep all original safeties |

### Round 4 — Effort 4 quick wins

| Item | Note |
|------|------|
| Drag-drop phase reorder | Rows `draggable=true` with ≡ grab handle. Native HTML5 dragstart/dragover/drop. Source dims to 0.4 opacity, drop target gets cyan inset shadow on top/bottom edge per cursor half. Mousedown on non-handle elements cancels drag (preserves text selection). New `movePhaseSectionTo(srcIdx, dstIdx)`. Existing ↑/↓ buttons stay as fallback. Goes through `setScriptFromSections` + `history.beginChange` |
| Per-phase duration estimate | ⏱ Xs / Xm Ys tag inline on every phase row. Same words/50 wpm + pauseSecs formula as global metrics. Tooltip exposes word count + pause seconds |
| Persona portrait grid hydration | Cached portraits hydrate as 18% opacity thumbnails behind text on persona grid cards. Async lookup per card with isConnected bail. Card text lifted to z-index 1. Skipped on isSelf. Generation flow unchanged — gates on `remoteProcessingOptIn` (legitimate external network call) |
| Browser API key mitigations | Per-session call counter in Options modal next to API key field, reads from `_aiAuditLog` filtering anthropic/openai. Color-shifts faint→dim→amber at 50/100. Auto-refreshes every 4s while row visible. Soft notify at 100 calls (one-time per session via sessionStorage). One-click "clear & revert to Perchance" button — wipes active key + per-provider cache + dropdown. Preserves audit log |

### Round 5 — UI cleanup + Effort 5 starters

| Item | Note |
|------|------|
| Universal "use profile details in script" toggle | Replaced three separate Share Age / Appearance / Goals checkboxes with one toggle below "Use first name in scripts?". Hidden mirrors preserve existing per-field read paths. `pfShareAll` change fans out to all three; load-time sync respects legacy per-field opt-outs (rolls into off if any was explicitly false) |
| Profile internals relocated | "Recent edits the AI is aware of" + "AI activity log" moved from Profile area into Session History details (with border-top dividers between blocks). Empty parent details collapses removed. IDs preserved so existing handlers (`profileDiffsList`, `viewAIAuditBtn`) work unchanged |
| Quick-Start Presets / Structured Programs as boxed expandable interfaces | `details.presets-collapsible` boxed containers. ▸/▾ chevron, bordered card, hover lift, "tap to expand" hint that hides on open. Both default-collapsed for new users; per-section state persists to `stillness_presets_open_v1` and `stillness_programs_open_v1` on toggle |
| 2D Director calibration pad | Combined intensity × pace control. Snap-to-integer grid on both axes. Single drag captures both values; sliders remain canonical inputs. Click + drag, with pointer capture for smooth tracking. Center cross marks balanced (0,0) origin. Persists once on pointerup |
| A/B experiment mode | Compare bar below presets (toggleable). Two preset dropdowns + "run blinded" button picks one at random, applies it, marks the session. Post-session: 1–5 rating widget reveals which side after submit. Per-preset rolling tally in `stillness_ab_results_v1`; "stats" button shows averages. `state.abModeRevealed` persists across sessions |
| Timeline view alongside phase list | `#phaseTimelineWrap` above `#phaseEditorList`. Each section a proportional flex block with width = duration / total. Category-colored backgrounds (tl-settling/induction/deepener/level/work/affirm/lightener/wake/reintroduction/emergence/sounding/custom). Inline label on blocks ≥7% of width. Click → smooth-scroll to + flash matching list row. Phase-meta footer: total time + phase count |
| Conversational free-text check-in | "type it instead" toggle below "how are you right now?" reveals textarea. Apply parses text via weighted lexicon (40+ phrases per axis, signed weights -4..+4) and pushes inferred values onto sliders + emoji selector. Per-category strongest-absolute-weight wins. Categories with no matches left untouched. Hint surfaces parsed values or warns if no signal |
| Phase audition | 🔊 audition button on every phase row. Speaks the phase aloud using current voice settings (system or local TTS). Single global slot — clicking another phase cancels the previous; clicking same button stops. Strips PAUSE markers + leading phase header. Disabled during live sessions. Active button gets cyan border + tinted background |
| Adaptive playback dynamics default-on for engaged users | When profile age is set AND no saved settings exist, `playbackDynamics.enabled` defaults to true. Anonymous users keep conservative off-default. One-time flip; persisted value wins on subsequent loads |

### Round 6 — Step 5 layout polish

| Item | Note |
|------|------|
| Slider rows refactored | Slider sits on top, `.slider-meta` line beneath. Canonical `.toggle-row-cb` pattern (checkbox-left). Cleans up the muddled grid that ran labels next to controls inconsistently |
| Local-AI-voice block loosened | Toast notifications + retry button replace the inline error UI. Block gets graceful fallback when initialization fails |
| Style preset chips active state strengthened | ✓ checkmark + 2px neon border on active chips, replacing the prior subtle background-only treatment that was easy to miss |
| Generate buttons relabeled | "+1 image" / "+5 images" / "-1" / "clear queue" replace the prior generic "generate" / "stop" verbs that didn't tell the user how many came out |
| Hide Gallery bug | Was using truthy check on `style.display` (always truthy unless empty string). Now uses `_galleryOpen` boolean. Closed gallery actually closes |

### Round 7 — Image gen enhance + Guide Builder

| Item | Note |
|------|------|
| ✨ enhance button on Step 5 image gen | Length picker (short / medium / long) using `BG_ENHANCE_INSTRUCTIONS` adapted from promptyx PROMPT_INSTRUCTIONS. Engineer pass tightens the prompt without mutating the textarea until user accepts |
| Guide Builder modal | New "✨ Build a Guide" entry on Step 1. Multi-step AI pipeline mirroring character-sfx's pattern: Identity (name+category+tagline) → Voice (rich systemPrompt) → Reminder (derived from Voice) → Image accents (prefix+suffix). Each step rendered as a `.gb-step` row with status icon + per-step ↻ regenerate. Smart cascade: regenerating Identity invalidates Voice + Reminder + Image. Cancel via shared `_gbToken`. Save gate: Identity + Voice required. Stored as custom persona via existing `saveCustomPersonas` |

### Round 8 — Post-session check-in parity + bug-hunt pass 1

| Item | Note |
|------|------|
| Post-session "type it instead" toggle | Mirrors the pre-session pattern. POST_CHECKIN_LEXICON + `parsePostCheckinText` parse mood / alertness / depth from free text and push values onto the three sliders. Same lexicon shape as pre-session, tuned for post-session vocabulary ("calmer", "drowsy", "went deep") |
| applyConfigSnapshot wrong DOM IDs | Two cases where the snapshot loader referenced IDs that don't exist: `voiceVolumeVal` → `voiceVolVal`, `useLocalTTS` (a state property) → `localVoiceToggle` (the actual checkbox). Fix means loading a saved preset now updates these two visible elements correctly instead of leaving them stuck on prior values |
| autoSelectVoiceForPersona orphan wired | Function existed with VOICE_PACKS table mapping persona category → preferred voice, but was never called. Now fires on persona-click in renderPersonaGrid; internal `_savedVoiceName` guard prevents override of user choice |

### Round 9 — Pre-gen for tomorrow's program

| Item | Note |
|------|------|
| Consent-based pre-gen offer | After completing a program day, post-session screen surfaces a card: "Want me to pre-generate tomorrow's session in the background?" Yes → schedules via `schedulePreGenForProgramDay`; Skip → dismisses. User chose to be in control of when AI calls fire — no silent quota burn. Toast on completion: "✨ Tomorrow's session is ready" or "Couldn't pre-generate — you can generate when you start the day" |
| Stealth pre-gen runner | `runPreGenForProgramDay` snapshots state, mutates to mirror tomorrow's settings (the same logic startProgramDay would set up), runs `generateFullScript` in headless mode (DOM writes guarded by `state._preGenMode` flag), restores state, caches result keyed by program+day |
| Cache + 26h expiry | `PROGRAM_PREGEN_KEY = 'stillness_program_pregen_v1'`. Most-recent 3 entries kept. `getCachedPreGen` checks freshness; `clearPreGenCache` removes a specific entry; `cancelActivePreGen` aborts in-flight generation |
| Pre-gen badge on Step 6 | When startProgramDay loads a cached script, a small `#preGenBadge` mounts above `#scriptText`: "✨ Pre-generated for you 6 hours ago — ready to play, or **regenerate fresh**." Click regenerate-fresh clears the cache and triggers a fresh generation. Click × dismisses the badge |
| Conflict resolution | When user starts a session OR clicks generate-fresh during in-flight pre-gen, the conflict path cancels pre-gen first (sets `state._preGenCanceled`, polls for lock-clear up to 4s). Pre-gen runner's `finally` block fires the appropriate toast outside `_preGenMode` so the notify gate doesn't suppress it |

### Round 10 — Saved Scripts library on Step 6

| Item | Note |
|------|------|
| Saved Scripts panel | New collapsible `<details class="presets-collapsible" id="savedScriptsDetails">` above `<h3>session script</h3>`. Live count badge in summary line. Empty state for new users. Newest-first ordering |
| Save current script | 💾 button in script-actions row. Defaults name to `${persona.name} · ${intentionText}` (or persona+method if no intention). Uses `promptAsync` for naming. Auto-opens the details panel after save so the new entry is visible |
| Load + rename + delete | Load confirms before clobbering unsaved edits; load path writes through `state.fullScript`, `state.parsedScript`, `state.generatedPhaseOrder`, and `$('scriptText').value` so phase metrics + timeline + Director card all refresh. Rename inline via `promptAsync`. Delete confirms first |
| `SAVED_SCRIPTS_KEY` IDB store | `stillness_saved_scripts_v1`, capped at 30 entries (newest kept on overflow). Schema: `{id, name, script, savedAt, sourceMeta:{persona,method,length,intention}}` |

### Round 11 — Themed app-intro + tour refresh

| Item | Note |
|------|------|
| App-intro panel | Replaced hastily-tacked `<div class="description">` with a properly themed `<details class="app-intro">`. Closed: one-line pitch + "tap for what this is" hint with chevron rotation. Open: italic blockquote (the "single-most complete" line treated as design rather than rendered noise), one-paragraph lede, 8-tile use-case grid (sleep / focus / decompression / processing / mindfulness / visualizations / habit reinforcement / creative-state), three numbered onboarding paths, foot-tags. Cyan accent border-left ties to active-state language used elsewhere |
| Smart open/close | First-time visitors (no session history) → open by default. Returning users (any completed sessions) → collapsed. Manual toggle persists separately to `stillness_intro_open_v1` |
| Onboarding tour refresh | Tour grew 12 → 14 steps. Adds Guide Builder coverage (Step 3), step dedicated to image-gen ✨ enhance + style chips (new step), Saved Scripts library + 💾 save mention (Step 6 step), pre-gen for tomorrow's program (new step). Tightened existing prose, updated method/persona counts. Same TOUR_KEY so existing users aren't re-prompted |

### Round 12 — Bug-hunt passes 2 + 3

| Item | Note |
|------|------|
| `state._startedUnsafe` wired | Previously set in startSession on safety-gate bypass but never read. Now propagates to `history[0].startedUnsafe` in the session record AND surfaces inline in the post-session reflection card as a quiet amber-bordered note ("this session ran with the pre-session safety check bypassed…") |
| `_rulesPreDuckVolume` removed | Vestigial cleanup from pre-stack-based duck implementation. Cleared in two places (line 21381 + 23927) but never set/read. Both lines deleted; comment at second site softened to drop the duck-snapshot reference |
| `getEmergencyCooldownRemaining` DRY | Helper existed but startSession had inlined the same calculation. Replaced inline with helper |
| `HNE.storage()` + `HNE.clearMetrics(true)` | Console diagnostic shim extended. `storage()` exposes `checkStorageBudget` (orphan utility); `clearMetrics(true)` exposes `clearMetricsHistory` (orphan with broken doc claim). The clear method requires explicit `true` arg so accidental autocomplete can't fire it |
| Dead-code removal | Deleted `getPreferred` (generic top-N utility, no caller) and `getSpokenScriptPreviewText` (TTS preview builder, no caller — probably remnant of an unshipped "preview entire script aloud" feature) |
| `stepsCompleted` shape consistency | Init had keys 1-5 but every reassignment had keys 1-6. Added `6: false` to init so the shape matches everywhere |
| `.app-intro-paths-label` orphan CSS | Class rule existed but markup never had a label element. Added "Three ways to start" label markup paralleling the "Try it for" label above the uses grid |
| `idbSet` / `idbDel` silent error swallowing | Both used empty `catch {}`. Storage-budget exceeded, permission errors, IDB version conflicts all failed silently with no developer signal. Now log via `console.warn` while preserving the void-return caller contract — affects ~40 IDB write sites |
| `backToEditBtn` cleanup gap | When user aborted a session via "back to script", three things weren't reset that finishSession does: webcam stream kept running until navigation elsewhere, playback dynamics state leaked into next session, drone.stop was fire-and-forget instead of awaited. All three now mirror finishSession's contract |

### Round 13 — Conversational check-in + phase-editor generate + regen-button extension

| Item | Note |
|------|------|
| Text-first check-in (Step 5 + post-session) | Conversational text input is the default; sliders are the alternative behind a `use sliders instead` toggle. Both check-ins get a chat-style header (avatar circle 💬 + framed question + toggle) and the textarea below. Pre-session: "how are you arriving?". Post-session: "how did that land?" — visually elevated with `.post-checkin-elevated` cyan-border-left card. Auto-apply on textarea blur so users who type and click another button (next →, restart, etc.) don't lose their signal — silent on auto-apply, no nag if nothing parses |
| Post-session feedback echo | New `#postCheckinFeedbackEcho` block below the input — a quiet quote-block-styled acknowledgment that surfaces parsed signals as natural language rather than raw numbers. "So: **better than before** · **a little drowsy** · **went very deep**. Got it." 4-5 phrase bands per dimension tuned to score ranges |
| In-row "✨ generate" button on phase editor | Every non-custom phase row now has a generate button (custom blocks keep their existing ✨ reimagine). Two visual states: idle (subtle, like reimagine) and pending (cyan-tinted with 2.4s soft pulse animation) — pending fires when section text matches `scaffoldForPhase` output. Hot path uses `_genStateRetryHandler`; cold path falls back to `generateFullScript` so first-time generation has proper shared context |
| Content-aware `syncGenStateWithPhases` | Detects scaffold-only sections via comparison with `scaffoldForPhase` output. Renamed phases (work → work-1 / work-2 promotion) carrying real content keep 'done' status; freshly-added scaffold-only phases correctly read as 'pending'. Also fills `gs.phaseResults` for done renamed phases from `state.scriptSections` so re-rolls can show proper diff. Fixes the bug where adding a 2nd work block reverted the existing work block to pending |
| Regenerate button now picks up pending phases | Step-6 `regenScriptBtn` previously only retried `error` / `warn` phases. Now also picks up `pending` (user added a phase via editor that hasn't been generated). Calls `syncGenStateWithPhases` first to ensure stateMask reflects current sections. Messaging picks language: "filled in" if mostly pending, "regenerated" if mostly failed, neutral for mixed. Tooltip updated to explain the new behavior |

### Round 14 — Four new phase types

| Item | Note |
|------|------|
| **Teaching** phase type | Psychoeducation primer — explains the method in plain language, normalizes the experience, sets expectations. Slot: between settling and induction (early framing). Useful for first-timers or when introducing a new technique. AI instruction enforces no-jargon, no-mystical-framing tone. Scaffold provides a clinical default that can be regenerated |
| **Training** phase type | Skill rehearsal in trance — narrator leads user through 3-5 repetitions of one specific practice (mental, sensory, or somatic). Slot: in the work zone (between deepener and lightener). Each rep slightly easier than the last (Erickson "ratification through repetition"). Stackable with numbering for multi-skill sessions |
| **Mission** phase type | Posthypnotic homework — ONE concrete, doable, waking-life task tied to the user's intention. Slot: between wake and reintroduction (delivered as user surfaces). AI instruction enforces specificity (named trigger, single small action) and positive framing (what to do, not what to avoid) |
| **Goal** phase type | Outcome anchor / future-pacing — sensory rehearsal of what success FEELS like (somatic, not cognitive). NOT a goal-setting exercise. Slot: between settling/teaching and induction (intention before going under). Vivid present-tense imagery, 3-4 sensory details only the user would notice |
| Eleven-place ripple | All four new types wired through: PHASE_DURATIONS (8 length buckets), getPhaseLabel, scaffoldForPhase, buildPhaseInstructions (persona-aware role variants when stacked), findPhaseInsertionIdx (canonical positions), addPhaseSection (numbered-stackable list + explicit branch), HTML "+ add" buttons (4 new with clinical-purpose tooltips), click handlers, editor row pair-tags (📖 teaching, ⚙ training, ◎ goal, ➤ mission with color-coded tints), timeline + row CSS (.tl-teaching/training/goal/mission, .is-teaching/training/goal/mission), and getSessionPhaseBlueprint via _applyPhaseExtras() |
| `state._phaseExtras` + `_applyPhaseExtras` helper | New state field initialized as `[]`. When a preset/program declares `phaseExtras`, applyPreset/startProgramDay sets the array. getSessionPhaseBlueprint runs through _applyPhaseExtras after building the default phases — for each extra, computes canonical insertion position via findPhaseInsertionIdx, splices in, then renormalizes IDs (work→work-1/work-2 numbering pattern) |
| Four new opt-in presets | Method Intro (📖 teaching), Skill Builder (⚙ training), Mission Brief (➤ mission), Goal Lock-in (◎ goal). Each combines an appropriate persona/method/intention with the matching phaseExtras to demonstrate the new types in a one-click bundle |
| Existing presets gain extras | Habit Change preset adds `phaseExtras: ['mission']` (posthypnotic homework reinforcing the new habit). Peak Flow preset adds `phaseExtras: ['goal']` (somatic rehearsal of performance state) |
| Programs progressively layer phase types | Confidence-10: teaching primer day 1, training days 6-8, goal day 9, goal+mission day 10. Anxiety-14: daily mission days 7-12, goal+mission days 13-14. Pain-6: training every day (clinical pain-dial protocol). Focus-7: goal day 7. Depth-7: teaching day 1, training days 6-7. Stress-5: goal day 5. Per-day overrides specify `phaseExtras` and trump program-wide settings |

### Round 14b — Perchance parser hotfix (chained root access) + freshSeeds list

| Item | Note |
|------|------|
| Diagnosed `root.X.Y` static-analysis trap | Perchance's parser scans source for `root.X.Y` patterns and treats them as list paths — interpreting `root.freshSeeds.pick` as a top-level list literally named `freshSeeds.pick` (a name with a dot in it, which is invalid). This triggers a cascade: once the parser is confused about a list declaration, it interprets surrounding lines as DSL items and warns about every `=` and `{` it sees, even unrelated CSS in the embedded `<style>` block. Single root cause → many secondary warnings at once |
| Fix: alias before chained access | `buildFreshnessBlock()` now does `const seedsList = (typeof root !== 'undefined') ? root.freshSeeds : null;` then calls `seedsList.pick(...)` on the local. Only `root.freshSeeds` (single dot) appears in source — Perchance treats it as one optional top-level list reference and gracefully no-ops when undefined. Runtime behavior is identical |
| Comment scrubbed too | The explanatory comment about the fix originally contained the literal `root.X.Y` pattern. Replaced with parser-safe phrasing in case Perchance scans comments raw before stripping them. Final sweep: zero `root.\w+\.\w+` hits across both perchance_1.txt and perchance_2.txt |
| Confirmed upstream pattern | Verified `ai-character-chat` upstream has zero `root.X.Y` chained access anywhere in its 690-line DSL or HTML body. This is the canonical Perchance idiom — chained access through `root` is forbidden by the parser even though both files have plenty of `root.X` single-dot calls (`root.aiTextPlugin`, `root.textToImagePlugin`, `root.uploadPlugin`, `root.communityPacks`, `root.kv`, `root.superFetch`, `root.knownProject`, `root.freshSeeds` are all fine) |
| Defined `freshSeeds` list in perchance_1.txt | The HNE Smart Director was calling `root.freshSeeds.pick(...)` to draw fresh metaphor anchors, but the list itself was never defined. Now perchance_1.txt declares a `freshSeeds` Perchance list with a `pick(opts)` function that filters by tag overlap, falls back to any phrase if filtered set is too small, Fisher-Yates shuffles, and returns up to n. Initial corpus: 60+ tagged anchor phrases across 12 categories (sleep/anxiety/performance/focus/creativity/pain/trance/confidence/behavior/grounding/release/emotional). Now Smart Director gets fresh imagery per session, reducing AI's tendency to converge on the same metaphors (staircase, warm light, deep ocean) for similar requests |

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
| Phase-locked binaural frequency (delta/theta/alpha by phase) | ✅ shipped — `setPhaseFreq` ramps L/R oscillators per phase across α/θ/δ map |
| 40 Hz gamma flicker (alpha-grade, gated by reduced-motion) | ✅ shipped — `renderGammaFlicker` with 5-15% alpha modulation, blue/cyan palette, voice-coupled, prefers-reduced-motion fallback |
| Conversational check-in via free-text (parse to sliders via AI) | ✅ shipped — lexicon-based parser (no AI call), pushes inferred mood/tension/energy onto existing sliders |
| Director-shows-its-work post-session card | ✅ shipped — `#directorWorkCard` on script preview surface |
| Cross-device sync via `root.kv` (existing #10), AI-tier-aware | Pushed to back-of-roadmap (external sync) |
| Per-domain Director calibration (sleep/focus/etc.) | ✅ shipped — `directorBiasByDomain` map, layered onto global bias inside `computeDirectorBrief` |

### Effort 4

| Originally listed | Status |
|---|---|
| Drag-and-drop phase reordering with constraints | ✅ shipped (HTML5 native drag with grab handle, drop indicators, falls back to existing ↑/↓ buttons) |
| Per-phase duration estimate (live) | ✅ shipped (⏱ tag on every phase row, words/50 wpm + pauseSecs formula) |
| `textToImagePlugin` persona portraits | ✅ shipped — already core. Extended: cached portraits hydrate as soft thumbnails on persona grid cards |
| Browser API key exposure mitigations | ✅ shipped (per-session call counter visible in Options, "clear & revert to Perchance" one-click button, soft notify at 100 calls per session) |
| Timeline view alongside phase list | ✅ shipped — proportional category-colored blocks, click-to-jump |
| Two-axis Director calibration UI revamp (2D pad) | ✅ shipped — combined intensity × pace control with snap-to-integer |
| Phase audition (preview each phase aloud) | ✅ shipped — 🔊 button per phase row with single-slot enforcement |
| Adaptive playback pacing default-on for engaged users | ✅ shipped — auto-enabled when profile age set on fresh slate |

### Effort 5

| Item | Notes |
|---|---|
| A/B experiment mode | ✅ shipped — preset compare bar, blinded run, post-session 1-5 rating, per-preset rolling tally |
| Fractionation (multi-induction cycles) | Open — needs first-class trance-cycle modeling. Big architectural lift |
| Pre-generate tomorrow program session | Open — needs full state-clone harness |
| Live adaptive pacing across all methods | Partially shipped via adaptive playback dynamics default-on. Generalization to per-method profile still open |

### Deferred — back of roadmap

(Per direction, items dealing with external API calls / packages, webcam, multi-language, offline support, classes / training, and any non-app-feature integrations are deferred to here.)

| Item | Notes |
|---|---|
| Eye-state modality switching | Webcam-driven |
| HRV via webcam rPPG | Webcam-driven biofeedback |
| Mic-driven exhale detection | Local mic input — borderline; deferred until a clear method needs it |
| Multi-language narration | Locale detection + voice match + AI prompt translation |
| Voice cloning via ElevenLabs/OpenAI TTS user-key | External API |
| Offline session package | TTS pre-synth + zip + minimal player |
| Cross-device sync via `root.kv` | External sync infra |
| Community pack registry | Discovery UI for external content packs |
| Training / classes integration | Out of scope per direction |

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
| — | Round 3 / 3b / 3c / 3d / 4 / 5 batches | See §0 sub-sections |

---

Last updated: 2026-04-27
