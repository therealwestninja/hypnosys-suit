# Advanced Hypnosis Narration Engine — Roadmap

This file tracks improvements not yet shipped. Items 1–6, 9, and 12 from the
original review have already been implemented in `perchance_1.txt` and
`perchance_2.txt`. What follows is the backlog, ordered roughly by a mix of
user-visible impact and effort.

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

### §0 Recent infrastructure rounds

#### Round 14b — Backup completeness audit
- `exportBackup` now packages 6 previously-missing slices: `memory`, `configPresets`, `savedScripts`, `triggerFires`, `customTags`, `abResults`. Backup `version` bumped to 5 then 6.
- Round 14 also added 4 opt-in phase types — `teaching` / `training` / `mission` / `goal` — wired through `_phaseExtras` state, with eleven-place ripple to phase resolution, `buildPhaseInstructions`, presets, programs, and tour copy.

#### Round 15 — Cloud + auto-tempo + free voices
- **Auto-tempo bug-hunt.** Six bugs in `_stripScriptChars` / `_stripOrphanPunctuation` / `fixScriptTimings`: missing whitespace collapse after char strip (caused TTS pacing artifacts), no ellipsis support, ellipsis orphans not caught, wrong pipeline order, defensive whitespace scrub missing, scattered hardcoded defaults. Fixed via new `_normalizeEllipsis`, `collapseSpaces` parameter, `_autoTempoDefaults()` / `_autoTempoOpts()` helpers. Settings `version` bumped 5→6. UI got two new controls (3-way ellipsis mode + collapse-spaces toggle) in the auto-tempo cleanup options modal. 21 unit tests pass.
- **Cloud accounts via PerDB.** Backend at `https://perdb.koyeb.app/api`, three collections (`hne_users` / `hne_user_content` / `hne_presence`), shared community API key with deploy-your-own swap path documented inline. PBKDF2-HMAC-SHA256 100k-iter password hashing, per-user salt, hex-encoded. State at `state.cloudUser` mirrored to `localStorage CLOUD_USER_KEY`. Sidebar slug at top of sidebar (with separator below) shows three states: signed-out / signed-in card / loading. Sign-in modal with sign-in + sign-up tabs. Cloud upload buttons added to: per-script row in saved-scripts (`☁`), persona editor (`☁ cloud`), backup section (`☁ upload settings` + `☁ my cloud`).
- **Comments + presence.** `comments-plugin` imported in perchance_1.txt, channel `hne-public`, lazy-embedded only when modal opens. Live presence beacon: 30s heartbeat to `hne_presence`, count = distinct user IDs with `lastSeen` in last 90s, polled every 60s. `document.hidden` guard so backgrounded tabs fall out of count automatically.
- **Unified Activity Timeline** (replaced 3 separate panels: change log + "what AI sees" + AI activity log) — single chronological feed merging `_profileChangeLog` + `_aiAuditLog` + last 30 sessions, with filter chips (All / Profile edits / AI calls / Sessions) and a "view full →" expand modal. `renderProfileDiffs()` and `renderChangeLog()` kept as backward-compat shims.
- **Charts modernized** — added 4 new chart tiles (visuals / voices / length / affirmation modes — these were tracked in analytics for ages but never charted), fixed `drawFeatureGrid` missing `PRE_GEN` (bit 256).
- **Button width fixes** — base `.btn` got `box-sizing: border-box; min-width: 0; overflow-wrap: anywhere;` so flex/grid kids can shrink and long labels wrap inside the button. New `.btn.icon` and `.btn.icon.sm` modifiers for single-glyph buttons. `.script-actions` switched grid → flex-wrap.
- **Local AI voice 404 fixed** — `SPEAKER_BASE` was pointing at the dataset root which doesn't host individual files. Switched to the demo Space at `/spaces/Matthijs/speecht5-tts-demo/resolve/main/spkemb/` with curated `.npy` filenames per speaker. Added `_parseNpy()` helper for the NumPy header (4 unit tests pass).
- **Free narrator voices guide modal** — OS-detected priority sections for iOS / iPadOS / macOS (version-aware path for Sequoia 15+) / Windows 10/11 / Android / Linux + universal sections (HNE's offline AI voice / Microsoft Edge for cloud-neural voices / cloud TTS services with free tiers). All links verified before shipping.

#### Round 16 — Cross-system audit + ripples
- **`importBackup` rebuilt** to restore the 6 slices `exportBackup` was already packaging (memory, configPresets, savedScripts, triggerFires, customTags, abResults). Each slice gets a slice-appropriate merge strategy (union by ID for arrays, first-write-wins for memory). Per-slice counts in the success notification.
- **Share-config payload v1 → v2** to include `_phaseExtras`, `bgPrompt`, `bgNegativePrompt`, `bgOrientation`, `bgActiveStylePresetIds`, `droneEnabled`, `droneVolume`. v1 payloads still work — v2 fields are conditionally restored.
- **Comments-plugin embed flow corrected** — was using wrong protocol (the plugin returns a String marker for the host to insert, not auto-injecting). Switched to documented-only options (channel, channelLabel, width, height numeric, newestCommentsAtTop). Removed the dead `generateAvatarDataUri` reference (now resolved — see avatars below).
- **Presence beacon visibility lifecycle** — `_sendPresence` early-returns on `document.hidden`; immediate heartbeat fires in the `visibilitychange` handler when tab becomes visible.
- **Post-session refresh** — `renderSessionHistory` and `renderTimeline` now called after `recordSessionInMetricsHistory` so an open Session History details shows new entries immediately.
- **Deterministic SVG avatars** — `_hashSeed` (FNV-1a 32-bit) + xorshift stream → `generateAvatarSvg(seed)` returns a 5×5 mirrored identicon SVG with HSL-derived colors. Same seed → same avatar across devices and refreshes. Used in: cloud sidebar slug, my-cloud modal header, comments avatar (via `setAvatarUrlForNextComment` data URI). 9 unit tests pass.
- **Cloud round-trip closed** — my-cloud modal now has `↓ import` per item that re-attaches the content to the local library (script → saved-scripts library; persona → custom personas; settings → applied with confirmation). Old `download` button kept as `file` for portability.
- **Saved-scripts re-renders on cloud sign-in/out** so per-row `☁` upload button enables/disables live.
- **FEAT bitmask extended** — `CLOUD_UPLOAD: 512`, `COMMENTS: 1024`. Both tracked in `cloudUploadContent` and `openCommentsModal`. `drawFeatureGrid` updated to match.
- **Sidebar display name fallback** — `profile.name` → `cloudUser.displayName` → `'Practitioner'`. Refreshed on cloud sign-in/out via `_refreshSidebarDisplayName`. Cloud handle is community identity, NOT used in AI prompts.
- **Tour expanded** to 16 steps adding Cloud + Free Voice Options coverage.

#### Round 17 — App↔Chat firewall
- **Privacy invariant declaration** added at the top of the cloud module + over `getProfileContext`: *"App-domain user data NEVER bleeds into chat-domain infrastructure (PerDB, comments, presence). Chat-domain identity NEVER bleeds into AI prompt assembly."*
- **Real leak fixed** — cloud upload was including profile data in the payload. Stripped to community-relevant fields only. Sidebar display-name fallback that read `cloudUser.displayName` into AI prompts was reverted (the previous Round 16 change incorrectly conflated the two identity sources).
- **`getProfileContext` firewall comment** documents that it MUST read only from `getProfile()` and never reference `state.cloudUser`, comments, presence, or any chat-domain source.

#### Round 18 — Community gallery for shared content
- **Allowlist filter** — `COMMUNITY_KINDS_ALLOWED` Set restricts gallery to `script` and `persona` kinds; settings/backup uploads stay private to the user even when shared via PerDB.
- **`renderCommunityBrowser`** modal — paginated browse with author SVG avatar, preview / import / report buttons. Reports stored in `hne_reports` collection.
- **Per-item delete** for own content with confirmation modal.
- **Sign-in gating** — `body.cloud-signed-out .cloud-upload-affordance{display:none!important}` CSS hides upload buttons when not signed in. `_refreshCommunityNavVisibility` called from `_setCloudUser` / `_clearCloudUser` / boot.

#### Round 19 — 11-bug megabatch
- **Bug 1 — Cramped action-row buttons** (round 3): switched `.phase-action-row` to `flex-wrap` with `flex:0 0 auto` per button.
- **Bug 2 — Local folder slideshow**: `bgEngine.startFolderSlideshow` + `_nextFolderIdx` + `_displayFolderFrame` with three order modes (sequential, random no-repeat, random with repeats). File System Access API on Chromium with `<input type=file webkitdirectory>` fallback. 11 unit tests pass.
- **Bug 3 — Auto-tempo PAUSE artifacts** (`[PAUSE 6].[PAUSE 2]` → narrator says "Dot"): three new regex passes anchored to PAUSE markers catch orphan punct between markers (existing pattern needed whitespace). Adjacent-marker collapse with summed durations capped at 12s. Playback noop-segment guard at segment + per-sentence loop levels. 15 unit tests pass.
- **Bug 4 — Steps 3+4 generation depth**: Step 3 craftedSuggestion 3-5 → 6-9 sentences, reinforcementPhrases 3-4 → 6-8, NEW `somaticAnchors[4-6]`, NEW `counterIntuitive[2-3 optional]`, token budget 600→900. craftedBlock in script generation now ships ALL fields with explicit weaving instructions. showCrafted preview renders new fields. Step 4 affirmations 3-5 → 8-12, `.slice(5)` → `.slice(12)`, token budget 600→900, textarea rows 5→9.
- **Bug 5 — Begin Session distinct + Share to Community**: `.btn-primary-action` CSS (240px min-width, 12px×28px padding, gradient + glow). `.begin-session-row` (centered, dashed top border). `🌐 share to community` button calls `saveCurrentScript` → `_uploadSavedScriptToCloud`.
- **Bug 6 — Sign-in gating** for community uploads — see Round 18 (this bug originally described the gap, fix shipped under R18 banner).
- **Bug 7 — Gallery downloads**: `_bgDownloadExtFromMime` detects jpg/png/webp from blob.type. `_bgDownloadInflight` Set per-id 1.5s debounce. Tooltip "all as zip" → "Download all images individually". `bgDownloadAllBtn` disables while running with progress text.
- **Bug 8 — Methods unlock toast + preset nesting**: `applyMethodGating` compares prevLockedCats to newLockedCats; newly-unlocked → 8s `notify.success`. Sleep `long → extended`, Temple Sit `long → extended`, **Deep Dive `long → marathon`** (triggers proper multi-deepener nesting that the preset name promised).
- **Bug 9 — `sessionBus`** universal generator backend: pub/sub at `~20420`. `signature()` = length::persona::method::intention::phaseOrder::scriptHash::phaseExtras. `notifyChange(reason)` only fires on signature change. Built-in subscribers handle `genProgress` hide + `renderPhaseEditor` + `renderTimeline`. Wired `hideScript`, `lengthSelect.change`, `scriptText.input`, `applyPreset`, `autoTempoBtn`. 10 unit tests pass.
- **Bug 10 — Step-3 "why now?" pre-selects**: `.session-tag-chip.selected` made dramatically more obvious (rgba 0.22 bg, 1.5px border, glow shadow, font-weight:600, ✓ checkmark prefix). Defensive `state._sessionTags ||= new Set()` in handler.
- **Bug 11 — Comments postMessage error**: `_deferUntilCommentsIframeReady` polls iframe + `load` event with 5s deadline. Global `unhandledrejection` swallow for postMessage races.

#### Round 20 — Skill-checklist + reduced-motion + suggestion symbols
- **Perchance skill-checklist verified all complete**: `stopSequences ['\n\n\n\n', '\n\n[[', '\n[[']` at line ~18979, iOS viewport patch, two `navigator.storage.persist()` calls (init + 5s post-init), mobile preload delay, 10s emergency export failsafe (~30586), `_corruptItemReplacer()` in exportBackup (~21577).
- **Backlog #15 — `prefers-reduced-motion` audit**: ⚠ motion badges added to 7 intense viz options (tunnel, starfield, vortex, flowfield, plasma, kaleidoscope-flow, kaleidoscope-plasma). `INTENSE_VIZ_LIST` Set centralized. `setupVisual` runtime gate re-evaluates on every call (init + manual + preset apply paths). 60s toast debounce. Init-time dampening for first-run users (halves intensity/speed, defaults to candle, drops audio-reactive dramatic→subtle, persists once).
- **Backlog #7 — suggestion symbols** (textToImagePlugin extension): `_suggestionHash` via FNV-1a on `trigger|coreResponse`. `SYMBOL_KEY(hash)` IDB lookup. `_buildSymbolPrompt` favors abstract metaphor. `generateSuggestionSymbol(suggestion)` — manual trigger, dedup map, opt-in gated, plugin-availability gated. UI: `🎨 symbol` button in crafted-preview action row, image displays inline above actions when generated. Hash-based caching across re-crafts. 9 unit tests pass.

#### Round 21a — Mic + webcam unified live sensing
- **Backlog #13 — Live adaptive pacing via mic**: Full mic module at `~7787` — `startMic()` / `stopMic()` / `_tickMic()` (50Hz) / `_onExhaleEnd()` / `waitForExhale(targetMs, maxMs)` / `_renderMicStatus()`. Constants: `MIC_TICK_MS=20`, `MIC_FALLING_EDGE_DEBOUNCE_MS=200`, `MIC_NOISE_HEADROOM=0.02`, `MIC_INHALE_HEADROOM=0.05`, `MIC_CALIBRATION_MS=2000`, `MIC_RECENT_BREATHS_MAX=5`, `MIC_FALLBACK_AFTER_SILENT=3`. State machine: rising edge above floor + 0.05 → inhale; EMA below floor for >200ms → exhale-end. Calibration: max EMA over first 2s + 0.02 headroom = noise floor. echoCancellation/noiseSuppression/autoGainControl all OFF. Separate AudioContext from drone. `CustomEvent('hne:exhale-end')` dispatched. 8 unit tests pass.
- **Unified Live Sensing panel in Step 5** — replaced standalone webcam opt-in with `.live-sensing-panel` containing both webcam + mic toggles, shared "All processing local" privacy framing, mic status row with dot+text indicator.
- **state._mic shape** added at `~6376`: enabled, status, stream, audioCtx, analyser, sourceNode, _fftBuffer, ema, noiseFloor, inhaling, lastExhaleAt, consecutiveSilent, recentBreaths[5], tickHandle, errorMessage. Plus `state.adaptiveBreathPacing` boolean.
- **Adaptive PAUSE handler** — `speakScript` PAUSE marker code uses `waitForExhale` when: adaptiveBreathPacing on AND mic active AND consecutiveSilent < 3 AND seconds in [3,10] (between-breath range; <3s too short, >10s dramatic deepening). 1.5x ceiling. Falls through to `interruptibleSleep` otherwise.
- **Session lifecycle** — startSession kicks off startMic non-blocking when checkbox checked. finishSession + early-return finish path + emergency-end all call stopMic + reset opt-in checkbox.
- **Breath-driven viz pulse** — `getAudioReactivityForViz()` mixes `breathBoost = 1.0 + (state._mic.ema/0.3 clamped) * 0.3` into pulse channel. Gated on adaptiveBreathPacing.

#### Round 21b — Profile expansion (15 new tracked fields)
- **15 new profile fields** organized into 4 collapsible sections:
  - 📅 Practice Context — `pfTypicalTime`, `pfPracticeSpace`, `pfPosture`, `pfCoPresence`
  - 🌱 Current Life Context — `pfCurrentThemes`, `pfCurrentStressors`, `pfCurrentWins`, `pfHealthNotes`
  - 🎨 Style Preferences — `pfMetaphorDensity`, `pfAuthorityStyle`, `pfTouchComfort`, `pfSpiritualFrame`
  - ⚓ Personal Anchors — `pfSafePlace`, `pfPowerObject`, `pfResourceMemory`
- **Pervasive use wired through** 10 sites: `getProfile()`, `applyProfileToForm()`, `TRACKED_FIELDS` for diff bouncer, both `srcLabel` mappings (timeline + getProfileContext), input/change listeners (also fixed previously-missed Round 16 fields), `getProfileContext` with field-specific AI instructions, **walking-posture safety gate** in `setupVisual` (force-swap intense viz to 'none'), **practice-space soundscape nudge** in `_refreshSoundscapeHint` ("for bedroom sessions, try rain or ocean"), live refresh on practice-space change, one-time discovery toast for established users.
- **Privacy preserved** — all goal-adjacent personal fields ride under existing `shareGoals` toggle; style/safety fields (posture, authority style) always pass through since they're not personal.
#### Round 21c — Backlog closeout: pregen gating + security audit
- **#14 mobile/battery gating shipped** — `schedulePreGenForProgramDay` now skips when `navigator.connection?.effectiveType` is 'slow-2g'/'2g' OR `navigator.getBattery()?.level < 0.2` (and not charging). Both checks gracefully fall open when the API is missing (iOS/Safari for Connection API; Battery API being deprecated everywhere).
- **#8 security audit completed** — found that 4/5 sub-items were already satisfied: warning modal on key entry, escapeHtml everywhere persona/cloud-user data crosses into DOM, per-provider key cache, quick-clear button, per-session call counter with soft notification at 100 calls. The "scrub after 30 days" sub-item is N/A — keys are session-only by design (`_extAiKey` underscore prefix means never persisted via `_buildSettingsObj`). Only remaining item: server-side proxy template documentation.
- **ROADMAP §0 documented** — Rounds 17 / 18 / 19 / 20 / 21a / 21b / 21c now have full entries matching the existing format. Backlog items #7 (partially), #13, #14, #15 marked complete; #8 marked mostly-complete with audit findings.

#### Round 22 — Generation UI unified (Step 5 ↔ Guide Builder)
- Step 5 script generation now uses the same rich-row pattern as the Guide Builder. Each phase renders as `.gen-step-row` with status icon (○ pending / ◐ running / ● done / ⚠ failed) + uppercase label + per-row regen ↻ button + inline content area where streaming text lands.
- New shared CSS family `.gen-step-row` reuses Guide Builder's `gbPulse` and `streamBlink` keyframes. Old `.gen-phase` / `.gen-dot` kept as defensive fallback. Dead `.gen-stream-box` CSS removed.
- `renderGenProgress` rewritten to output rich rows with `data-phase-idx` + `data-phase-id` + per-row regen button bound via event delegation. `_GEN_STATUS_CLASS` and `_GEN_STATUS_ICON` maps replace ad-hoc state strings.
- `setStreamingPreview` refactored to write into the row's `[data-content-idx="N"]` element instead of a standalone box. Signature accepts numeric `phaseIdx`; legacy string-label form still works via backward-compat. `{hide:true}` is now a no-op since rows stay populated post-generation (UX improvement: users see what was generated per phase). 14 unit tests cover the render logic.
- `perchance_1.txt` plugin-import documentation added — comprehensive block teaching other Perchance users how to import HNE as a plugin in their own generators. Documents `freshSeeds` API, iframe embedding, contributing-new-exports guidance, and the inverse `communityPacks` flow.

#### Rounds 22a–22c — Research passes (no code; ROADMAP work)
- **22a — Profile expansion (15 new tracked fields)** — see Round 21b for the actual ship; 22a was the alignment pass.
- **22b — Tracking & profiling research pass.** Catalogued WHAT to add. 27 items in three single-turn-doable clusters (A: instrument actions we already see, B: derive new signals from existing data, C: extend Director with collected signals) plus a multi-turn cluster D for bigger structural items.
- **22c — Tracking refinement: drop / reformat / restructure.** Critical-eye companion to 22b. Catalogued WHAT to drop and HOW to restructure. 18 items in four clusters (R: pure reductions, F: reformats, S: schema restructuring, P: Perchance-native idioms) plus 3 retracted items from the 22b proposal that were over-collection on closer review.

#### Round 22d — Tracking refinement: Cluster R shipped (4 of 6 items)
- **F3 (replaced R1) · structured before/after diff snippets** — `_diffText` now stores `{before: 40-char, after: 40-char, changePct}` instead of single 80-char `preview`. Same storage cost, more information. Activity timeline renders "old text… → new text…" instead of just "new text…". Backward-compat fallback for legacy `preview`-only entries in `_renderTimelineEntry`.
- **R2 · default-omitting on every history write** — module-level `_HISTORY_RECORD_DEFAULTS` map (12 fields: affirmationMode='woven', voiceName='', speechRate=0.65, voicePitch=0.85, voiceVolume=1.0, droneVolume=0.35, duckingEnabled=true, profileName='Practitioner', startedUnsafe=false, preTension=null, preEnergy=null, intentionText='', userAffirmations=''). Plus `_VIZ_SETTINGS_DEFAULT` shallow-equal check. `_compactHistoryRecord(rec)` strips defaults; readers all use `record.field || DEFAULT` fallback so omission is read-side safe.
- **R5 · script-TTL** — `_writeHistoryCompact(history)` keeps `script` field only on newest 5 records; records 6+ get the field deleted. Idempotent. First-boot migration runs on legacy data automatically.
- **R6 · profile snapshot dedup** — `_stableJSON(profile)` (key-sorted, internal-prefix-stripped) → `_hashSeed` (FNV-1a 32-bit) yields stable `profileHash`. Always stored on every record. Full `profileSnapshot` only stored when hash differs from previous record's hash. Reader has graceful no-op fallback so missing snapshots = current profile state on replay.
- **All 7 history-write sites routed through `_writeHistoryCompact()`** — recordSession, boot-migration, importBackup, history-delete, post-checkin patch, journal-blur, sentiment-patch, journal-flush. Defaults can no longer creep back via normalize→resave round-trips.
- **R3 / R4 skipped** — see ROADMAP entries; resolver complexity > storage win for R3, R4 would lose per-session value history.
- **24 unit tests pass** covering omit-defaults round-trip, vizSettings deep-equal, R5 idempotence, R6 hash stability with key-order variation, R6 dedup logic, F3 structured diff. Net storage win: ~50-200KB per 50-session history window.

#### Round 22e — Tracking refinement: Cluster F shipped (5 of 5 items + bonus A5)
- **F1 · circular-buffer cfg** — `_CFG_CAP=32`, `_cfgPush`/`_cfgMean`/`_cfgSlope` helpers. Migration in `normalizeAnalyticsData` seeds buffers from legacy `{rateSum, ...}` mean × min(n, 32). `mergeAnalyticsData` concat-then-cap. `recordAnalytics` and `rebuildAnalytics` use `_cfgPush`. Cold-start defaults updated. Unblocks B6 (setting drift via `_cfgSlope`).
- **F2 · audit-log byte split** — `logAICall` accepts `parts` array, computes `contextBytes` (sharedContext + priorSummary) vs `taskBytes` (taskBlock + diversity). Activity timeline shows split when present. Optional — single-block callers omit.
- **F3 · structured before/after diff snippets** — already shipped Round 22d (was R1).
- **F4 · `feat` bitmask → map** — `feat` storage converted from bit-OR'd integer to `{[FEATURE_NAME]: firstUsedAt}`. FEAT bit constants kept for caller compatibility (12+ call sites unchanged) via `_FEAT_BIT_TO_NAME` reverse lookup. `_migrateFeatBitmask`, `featUsed`, updated `markFeatureUsed`, `mergeAnalyticsData`, `deriveInsights`, and `drawFeatureGrid`. Migration runs on read, defensively on write, on rebuild.
- **F5 · embed intention text alongside digest** — `embedAndStoreDigest` accepts paired text, batches into single `embedTexts` call. `intent_${sessionId}` namespace separation. New `findRelevantIntents` consumer. Embed cap doubled 100→200 for paired vectors. `findRelevantDigests` filters out intent-prefixed keys.
- **Bonus: A5 (first-use timestamps)** ships as a side-effect of F4. `drawFeatureGrid` now shows "✓ Programs · 14d ago" when ts > 0. Discovery time data unblocks future Director nudges ("you discovered X after N sessions, ready for the deeper ones?") without separate instrumentation.
- **26 unit tests for F4/A5** + **24 retained for Cluster R**. Parse clean at 32,275 lines.

---

### §1 Earlier shipped features

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

#### 7 · `textToImagePlugin` beyond backgrounds — ✅ partially shipped
The plugin is already imported for background generation. Two of three
candidate uses now shipped:

- ✅ **Personal symbols for crafted suggestions.** Shipped Round 20.
  `generateSuggestionSymbol` with FNV-1a hash-based caching by
  `trigger|coreResponse`. UI button in crafted-preview card.
- ✅ **Persona portraits.** Shipped earlier — `generatePersonaPortrait`
  with `PORTRAIT_KEY(id)` IDB cache + per-persona dedup map.
- ⏳ **Program-day achievement badges.** When a user completes day N
  of a multi-day program, generate a unique badge. Display in the
  heatmap sidebar. NOT YET SHIPPED — this is the cleanest remaining
  textToImagePlugin extension to pick up next.

Implementation notes for badges: reuse `bgCacheGetUrl` / `bgCacheStore`
infrastructure. Gate behind `state.remoteProcessingOptIn`. Add a
`programBadgeUrl` field on the program-day records.

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

#### 13 · Live adaptive pacing via mic — ✅ shipped Round 21a
Full mic module shipped per the spec — `AnalyserNode` over the mic
stream at 50Hz, falling-edge detector with 200ms debounce, exhale-end
events resolve `waitForExhale` Promises in the speakScript PAUSE handler.
Hard ceiling of 1.5× target prevents silent-user stalls. Falls back to
fixed timer after 3 silent breaths. Never transmits audio off-device.
8 unit tests pass.

Bonus: surfaced as part of a unified "Live sensing" panel in Step 5
that bundles mic + webcam under a single privacy disclosure. Breath
envelope also feeds the visualizer pulse channel for visible feedback.

### Medium impact

#### 14 · Pre-generate tomorrow's program session — ✅ shipped (Round 21c completed)
Most of the pre-gen system pre-existed (offer flow on session end →
`schedulePreGenForProgramDay` → `runPreGenForProgramDay` with state
snapshot/restore + 26h cache at `stillness_program_pregen_v1`).

Round 21c completed the missing sub-deliverable: **mobile/battery
gating**. `schedulePreGenForProgramDay` now skips with debug log when:
- `navigator.connection?.effectiveType` is 'slow-2g' or '2g'
- `navigator.getBattery()?.level < 0.2` AND not charging

Both checks fail open (allow pregen) when the API isn't available.
Connection API is missing on iOS/Safari; Battery API is being deprecated.

#### 15 · `prefers-reduced-motion` audit — ✅ shipped Round 20
All four deliverable items shipped:
- ⚠ motion badges added to 7 intense viz options in the dropdown
- Init-time dampening for first-run users with reduced-motion already on
  (halves intensity/speed, defaults to candle, drops audio-reactive
  dramatic→subtle, persists once)
- `setupVisual()` runtime gate re-evaluates OS preference on every call
- 60s toast debounce so users fighting the swap aren't spammed
- Round 21b extended this with a walking-posture safety gate that
  force-swaps intense viz to 'none' when profile.posture === 'walking'

### Security / technical debt

#### 8 · Browser API key exposure — ✅ mostly shipped (audit Round 21c)
Lines in `aiGenerate` put `x-api-key` (Anthropic) and `Authorization: Bearer`
(OpenAI) directly in a browser `fetch`, relying on
`anthropic-dangerous-direct-browser-access: true` to bypass CORS.

Mitigations status (audited Round 21c):

- ✅ **Explicit warning on key entry.** `_showApiKeyWarning()` modal fires
  on first non-empty key entry per session. Clears the field if the user
  cancels.
- ✅ **Sanitise shared content strictly.** Audited all sites where
  persona / cloud-user data crosses into DOM — every one uses
  `escapeHtml()` (renderPersonaGrid, renderPersonaDetail, persona chat,
  preset cards). Persona import (`saveImportedLore`) length-bounds all
  fields and routes through `normalizePersona()`. No innerHTML XSS surface.
- ✅ **N/A: Scrub keys after inactivity.** Keys are session-only by
  design — `_extAiKey` underscore prefix means never persisted via
  `_buildSettingsObj`. User has to re-paste on every reload, which is
  stricter than the 30-day scrub the original spec suggested.
- ✅ **Separate keys per provider.** `state._extAiKeysByProvider`
  (anthropic / openai) preserves both during provider switches.
- ✅ **Quick-clear button.** `extAiClearBtn` in the options modal wipes
  active key + per-provider cache, reverts provider to Perchance.
- ✅ **Per-session call counter** (`_refreshExtAiCallCount`) surfaces
  unexpected activity on the AI key UI; soft notification at 100 calls.
- ⏳ **Server-side proxy template documentation.** Still TODO — would
  benefit users who want to use external providers without exposing
  the key in the browser. Cloudflare Worker template + origin-check
  pattern is the cleanest delivery.

### Perchance skill-checklist items ✅

Running the §14 checklist from the `perchance-api` skill against the current code — all items now satisfied:

- [x] **`stopSequences` chat-format-bleed safety net** — `generateSinglePhase` uses `['\n\n\n\n', '\n\n[[', '\n[[']` (line ~18979). The `[[` patterns catch any model that drops into `[[Name]]:` chat completion format.
- [x] **iOS Safari `maximum-scale=1` viewport patch** is present (line ~2260)
- [x] **`tryPersistBrowserStorageData()`** — `navigator.storage.persist()` called inside `_initStorage` (line ~6594) AND a second time 5s after init (line ~30557) to catch the browser-grants-after-interaction case.
- [x] **Mobile AI preload delay** is correctly implemented:
      `if(window.innerWidth < 500) setTimeout(..., 5000)`
- [x] **Emergency export timer** — 10s failsafe at line ~30586 that surfaces a "download backup" button if the page hasn't progressed past step 1. Self-deletes on first click.
- [x] **`exportRawDb` / corrupt-item replacer** — `_corruptItemReplacer()` at line ~21577 used by `exportBackup` at ~21640 so a single bad record doesn't abort the whole export.

---

## Tracking & profiling — research pass (Round 22b backlog)

A systematic survey of the tracking, profiling, and auditing surfaces turned
up the gaps below. The system is mature in places (3-tier memory + embeddings
for semantic recall, weighted-reward learning over persona/method/soundscape/
length, feature bitmask, AI audit log with byte-size + provider + source) but
under-instrumented elsewhere — particularly around in-session behavior and
mid-flow user actions that contain the strongest engagement signal.

Items are clustered by effort. Each cluster is independently shippable in
one turn; items inside a cluster are independently doable so partial shipment
works. Within each cluster items are ordered least-to-most effort.

### Cluster A — instrument actions we already see (single turn, ~10 items)

We already fire these events; we just don't record them. Pure additive — no
schema changes for existing consumers, just new fields on `sessionRecord`
and incremental writes from existing handlers.

- **A1 · Pause telemetry on the session record** (~5 lines). `state._inactiveTotalMs` is already accumulated; just add `pauseCount` and `inactiveMs` to `sessionRecord` in `finishSession`. Pause count tells the Director "ran to completion but disrupted" vs "ran clean."
- **A2 · Mid-session setting adjustments** (~15 lines). Wire change handlers on `tempoSlider`, `speechRate`, `voiceVolume`, `droneVolume`, `vizSelect` to push to `state._inSessionAdjustments[]`. Persist in `sessionRecord`. Each adjustment is a tiny tuple `{at, field, from, to}`. High-signal: tempo adjusts mid-flow = pacing miscalibrated.
- **A3 · AI audit log enrichment** (~10 lines in `logAICall`). Currently captures `{ts, provider, source, bytes}`. Add `stopReason` (`stop`/`max_tokens`/`error`), `errorFlag`, `phaseId` (from opts), and `regenAttempt` (0 = first try, 1+ = regen). Closest thing we have to AI-quality data.
- **A4 · Phase regeneration tracking** (~5 lines). Round 22's per-row regen button has the phase id at click time; just push `{at, phaseId, attempt}` to `state._phaseRegens` and persist on the record.
- **A5 · Feature first-use timestamps** — ✅ shipped Round 22e as byproduct of F4. `feat` is now `{[FEATURE_NAME]: firstUsedAt}` map. `markFeatureUsed` only sets ts on first invocation (preserves discovery time across repeated touches). `drawFeatureGrid` shows "Nd ago" subtitle. Director nudges that depend on discovery time can now read directly from analytics.feat[NAME].
- **A6 · Backgrounded-tab tracking** (~8 lines). `visibilitychange` already fires for cloud presence; piggyback to push to `state._tabHiddenMs` during sessions and persist on record. Hidden during work-phase = strong disengagement signal.
- **A7 · Voice-test sampling counter** (~5 lines). Each voice preview click during config push to `_voiceSamplesThisSession`. High count = uncertainty; surface "you sampled 7 voices — try save-favorites?" as a Director nudge after the session.
- **A8 · Soundscape "settle time"** (~5 lines). Time between user picking a soundscape and clicking generate. Long delay = looking for something else; could surface alternatives proactively.
- **A9 · Generate→accept latency** (~5 lines). Time between full-script generation completing and user clicking begin-session. Long delay = reviewing/rejecting; correlate with mood-Δ to surface "scripts you took >2min to review had higher post-mood — slow down?" as Director insight.
- **A10 · "Why-now" tag chip dwell time** (~3 lines). When a chip is selected, record `{tag, ts}` in addition to the existing `_sessionTags` Set. Dwell + selection sequence reveals user's actual mental-state arc, not just final state.

### Cluster B — derive new signals from existing data (single turn, ~6 items)

No new instrumentation; pure computation over data we already have.

- **B1 · Profile snapshot diff-only storage** (~12 lines). Currently `sessionRecord.profileSnapshot` is full-copy, ~1-3KB per session. Hash the profile, store `profileHash` always but `profileSnapshot` only when hash differs from previous record. Saves ~50KB per 50-session window with no information loss (snapshots reconstruct from chain).
- **B2 · Cross-session continuity score** (~10 lines new fn `computeContinuityScore`). Read history; for each session, compute `hoursToNext`. `continuityScore` = sessions-within-24h / total. >0.5 = part of routine; <0.2 = sporadic. Surface in stats panel.
- **B3 · Affirmation freshness counter** (~8 lines). New analytics field `affirmAge: { lastEditedAt, sessionsSince }`. Decremented each session, reset on `pfUserAffirmations` edit. Surface in adaptive recommendations: "your affirmations are 30 sessions old — refresh?"
- **B4 · Reverse modality from affirmation language** (~25 lines, extends `inferModality`). Token-classify user-typed affirmations: "I see..." → visual+1, "I feel..." → kinesthetic+1, "I hear..." → auditory+1. Higher-confidence than the current config-only heuristic. Combine via weighted sum.
- **B5 · Adaptive prompt length from completion-by-length** (~15 lines). Read `analytics.ln` + `analytics.comp`; compute completion rate per length bucket. If `completionRate(long) < 0.3` and `completionRate(medium) > 0.7`, boost medium in the recommendation engine. Today we ignore this signal.
- **B6 · Setting drift detection** (~20 lines). Walk last 20 records' `speechRate` (or `tempoMultiplier`); compute linear regression slope. Significant positive slope (>0.01/session) = user fighting the default. Surface as Director nudge: "your speech rate has crept from X to Y — try `fast` preset?"

### Cluster C — extend the Director with signals already collected (single turn, ~5 items)

The Round 21b profile fields (typicalTime, posture, healthNotes, etc.) feed
the AI prompt but the LOCAL Director recommendation engine never reads them.
Each of these is a small extension to `getRecommendation` or a new pre-filter
applied before scoring.

- **C1 · Posture-aware method filter** (~10 lines). Walking posture → filter out `hand-heaviness`, `arm-catalepsy`, `progressive`, `floating-body`, `glove-anesthesia` from candidates. Lying-side posture → suppress symmetric-body methods. Today we warn at runtime via `setupVisual`; this is the recommendation-side equivalent.
- **C2 · Time-of-day persona biasing** (~15 lines). `analytics.hr` already has hour-of-day distribution. Cross with persona usage to compute "this persona is 80% used between 22:00–02:00." When current hour matches a persona's time profile, boost it in `getRecommendation`. Goes deeper than `pfTypicalTime` alone.
- **C3 · Health-notes safety gate** (~10 lines). Token-match `pfHealthNotes` against a small pattern list (`migraine|chronic pain|pregnant|insomnia`). On match, suppress methods with relevant contraindications (gamma flicker visuals for migraine, deep-deepener stacks for sleep-disorder users, etc.).
- **C4 · Spiritual-frame method filter** (~8 lines). `pfSpiritualFrame === 'secular'` filters out methods with `spiritual` tag (if any have it) and removes mystical language from method blurbs in the picker.
- **C5 · Co-presence soundscape biasing** (~5 lines). `pfCoPresence === 'shared' || 'public'` boosts low-attention soundscapes (drone, brown) in the recommendation engine — matches what the AI prompt already nudges but enforces it in local pick too.

### Multi-turn / bigger items (need design pass)

These are scope-larger because they require schema changes, new modules, or
explicit user-facing UX surfaces.

- **D1 · Embedding-based intention clustering.** Extend the existing `_embedCache` infrastructure to also embed `intentionInput` text per session. Run k-means (or HDBSCAN) over the last 50 vectors; surface clusters as named themes ("you've returned to grief work in 6 of the last 12 sessions"). Powerful but needs cluster-naming UX + UI surface.
- **D2 · Per-phase reward propagation.** Today reward is whole-session. Decompose: which phases correlated with the mood-Δ? Requires phase-level depth/mood probes (depends on Cluster A items A1+A2+A4 to have phase-level engagement signal) and a new weighted-reward map `adaptive.w.phaseTypes` keyed by phase base.
- **D3 · A/B test → adaptive weight integration.** When user picks a winner in A/B mode, that's GROUND TRUTH preference data. Currently stored in `_abResults` but never feeds adaptive weights. Add a `+5 reward` boost on the winner side of each pair.
- **D4 · Session abandonment funnel.** Track step-by-step where users drop out (Step 1 / Step 4 / Step 5 / pre-generation). Critical for UX improvement but needs per-step `_lastViewedAt` instrumentation + a new analytics surface to display drop-off rates. Not a single-turn item.
- **D5 · Crash/error telemetry.** Persistent capped error log capturing `{ts, source, message, stack}` for AI failures, audio errors, IDB write fails. Gated under remote-processing-opt-in for upload to a debug endpoint. Anonymized — no profile data, no script content.
- **D6 · Settings export/import telemetry round-trip.** When users restore from backup, do they immediately re-edit certain fields? That's a "backup-incomplete" signal. Track which fields get edited within 5 minutes of a restore.

## Tracking refinement — drop / reformat / restructure (Round 22c backlog)

The previous research pass (Round 22b above) catalogued what to ADD. This
pass is the critical-eye companion: what to DROP, what to REFORMAT, and what
to RESTRUCTURE. A concrete audit against the codebase turned up real waste —
redundant fields, lossy aggregations, duplicated free-text snippets, and a
schema that doesn't scale (38 profile fields means 5+ files to touch when
adding one more).

Verified gaps from the audit:
- `sessionRecord` has 35 fields; ~6 are redundant (name+id pairs, default
  values stored verbatim, full profile snapshot every session)
- `getProfileContext` has 28 if-blocks — no schema, every new field means
  hand-editing prompt assembly
- `analytics.cfg` keeps running sums only — outliers contaminate forever,
  no slope/distribution recoverable
- `_profileChangeLog` text entries store `changePct` AND a `preview` (~80
  chars); the preview duplicates the profile field's current value
- 13 sites gate on `shareGoals !== false` — a single toggle controls a third
  of the prompt context, with no granularity
- AI audit log has 4 fields; `bytes` conflates context-size with task-size
- Full session script (~5-15KB) stored on every record — ~550KB at the 50-
  record cap, mostly never consulted after a few days

Same single-turn clustering as 22b. Items ordered least-to-most effort.

### Cluster R — Drop redundancy and over-collection (single turn, ~6 items) — ✅ shipped Round 22d (4 of 6)

Pure reductions. No information loss in any item — what's dropped is either
duplicated elsewhere, derivable, or stale-by-design.

- **R1 → upgraded to F3** — On closer look, the change-log preview IS used in the timeline render (not pure duplication — it's a temporal snapshot of the field's value at edit time). Shipped as F3 instead: replaced single 80-char `preview` with structured 40-char `before` + 40-char `after`. Same storage cost, more information. Activity timeline now renders "old text… → new text…" instead of just "new text…". Backward-compat for legacy entries with only `preview`.
- **R2 · Drop default-valued fields from `sessionRecord`** — ✅ shipped via `_compactHistoryRecord(record)` helper. 12 default-valued fields stripped (affirmationMode='woven', voiceName='', speechRate=0.65, etc.) plus shallow vizSettings deep-equal check. All 7 history-write sites routed through `_writeHistoryCompact()` so defaults don't creep back via normalize-then-resave round-trips. Saves ~200 bytes/record average.
- **R3 · ~~Drop redundant name strings (`guide`, `method`)~~ — SKIPPED.** On audit: 70 chars × 50 records = 3.5KB saved, but breaks history rendering when a custom persona is deleted (id no longer resolves). Maintenance complexity > storage win.
- **R4 · ~~Drop `userAffirmations`/`intentionText` duplicates~~ — SKIPPED.** Diff log stores edits, not VALUES — removing from sessionRecord would lose per-session text history. Truncation-only variant rejected (small win, reduces fidelity).
- **R5 · TTL the full `script` field on session records** — ✅ shipped. Newest 5 records keep `script`; records 6+ get the field deleted on every history write. Idempotent. Digest in `stillness_memory_v1` is what `findRelevantDigests` actually consumes; the full script on old records was rarely re-read. Saves ~50-150KB per 50-record window. First-boot migration prunes existing legacy data automatically.
- **R6 · Drop full `profileSnapshot` per record** — ✅ shipped. `_stableJSON(profile)` (key-sorted, internal-prefix-stripped) → `_hashSeed` (FNV-1a 32-bit, already in codebase) yields a stable `profileHash`. Always stored; full `profileSnapshot` only stored when hash differs from previous record's hash. Reader at line ~22002 has graceful no-op fallback (`if (entry.profileSnapshot) applyProfileToForm(...)`) so missing snapshots just keep the user's current profile state on replay — arguably better UX. 24-test suite covers round-trip.

**Net storage win**: ~50-200KB per 50-session history window depending on profile churn rate. Plus eliminated default-field bloat at every write path (formerly creeping back via normalize→resave on every boot/import).

### Cluster F — Reformat for better signal (single turn, ~5 items) — ✅ shipped Round 22e (5 of 5 + bonus A5)

Same data, smarter shape. No information loss; in some cases information GAIN
(circular buffer recovers signal that running sums destroy).

- **F1 · `analytics.cfg` running sums → circular buffer** — ✅ shipped. `_CFG_CAP=32` constant + `_cfgPush(cfg, field, value)` (push + cap to 32) + `_cfgMean(arr)` + `_cfgSlope(arr)` (linear regression — unblocks B6 setting drift). Migration in `normalizeAnalyticsData` detects legacy `{rateSum,...}` shape and seeds buffer with `mean × min(n, 32)` entries (preserves the average; new samples gradually replace synthetic uniform). `mergeAnalyticsData` concat-then-cap with newest-wins semantics. `recordAnalytics` and `rebuildAnalytics` both use `_cfgPush`. Cold-start defaults updated everywhere.
- **F2 · AI audit `bytes` → `{contextBytes, taskBytes, totalBytes}`** — ✅ shipped via `logAICall(provider, source, instruction, parts)` accepting the same `parts` array used by `trimToTokenBudget`. When provided, splits into context (sharedContext + priorSummary) and task (taskBlock + diversity) byte counts. Optional — single-block callers (image-tag, craft-suggestion, journal-sentiment) leave it unset. Activity timeline shows "X ctx + Y task" subtitle when split is present.
- **F3 · `_profileChangeLog` text-diff format upgrade** — ✅ shipped Round 22d (replaced R1). `_diffText` returns `{changePct, before: 40-char, after: 40-char}` instead of single 80-char `preview`. Activity timeline renders "old text… → new text…" with backward-compat fallback for legacy entries.
- **F4 · `feat: bitmask` → `feat: {NAME: firstUsedAt}` map** — ✅ shipped Round 22e. FEAT bit constants kept for caller compatibility (12+ call sites unchanged); `_FEAT_BIT_TO_NAME` reverse lookup translates internally. `_migrateFeatBitmask(featRaw)` handles legacy data — sets ts=0 for "unknown discovery time" so subsequent UI can distinguish "discovered (time unknown)" from "discovered Nd ago". `featUsed(analytics, bit)` helper for legacy-style read sites. Migration runs in `normalizeAnalyticsData` and defensively in `markFeatureUsed`. `mergeAnalyticsData` merges with min-timestamp wins.
- **F5 · Embed `intentionInput` alongside session digest** — ✅ shipped. `embedAndStoreDigest(sessionId, digestText, intentionText)` accepts paired text, batches into a single `embedTexts` call. Stores digest at `[sessionId]` and intention at `intent_${sessionId}`. New `findRelevantIntents(queryText, k)` consumer surfaces "sessions whose user-typed intention was similar to today's" — semantically distinct from `findRelevantDigests` (AI-summarized script content). Embed cap doubled 100→200 for paired vectors. `findRelevantDigests` filters out `intent_`-prefixed keys via early continue.

**Bonus: A5 (first-use timestamps) shipped as a side-effect of F4** — every feature now carries its first-use timestamp in the same map slot, so the discovery-time data needed by future Director nudges ("you discovered programs after 12 sessions, ready for the deeper ones?") is available without separate instrumentation. `drawFeatureGrid` now renders "✓ Programs · 14d ago" subtitle when ts > 0; falls back to bare label when ts=0 (legacy migration with unknown first-use).

**Test coverage**: 26 unit tests for F4/A5 (migration both directions, featUsed map+legacy paths, mark first-use semantics, merge min-ts wins, null/empty edge cases). 24 unit tests for Cluster R still passing. Parse clean at 32,275 lines.

### Cluster S — Schema restructuring (single turn, ~4 items)

Bigger refactors but each pays off over time as the field count grows. Order
of priority within: do S1 + S2 together (they share a profile-storage migration);
S3 builds on S1; S4 is independent.

- **S1 · Group profile fields by category in storage** (~50 lines, mostly mechanical). Today profile is flat: `{name, age, pronouns, typicalTime, currentThemes, safePlace, ...}`. Restructure to `{identity:{name, pronouns, age}, practice:{typicalTime, practiceSpace, posture, coPresence}, lifeContext:{currentThemes, currentStressors, currentWins, healthNotes}, style:{vocalPace, energyPattern, metaphorDensity, authorityStyle, touchComfort, spiritualFrame}, anchors:{safePlace, powerObject, resourceMemory, preferredEnvironments, sensorySensitivities}, demographics:{experience, appearance, about, goals}}`. Backward-compat: `getProfile()` returns flattened shape for current consumers; new code can opt into `getProfile({grouped: true})`. Migration: one-time convert on first read.
- **S2 · Replace per-field share toggles with category-scope** (~20 lines, depends on S1). Currently 13 sites gate on `shareGoals !== false` because we shoved all goal-adjacent fields under one toggle. With S1's grouping, the toggle becomes per-category: `shareScope: { identity: bool, lifeContext: bool, style: bool, anchors: bool, demographics: bool }`. Practice-context (posture, time, space) stays always-shared since it's safety-relevant, not personal. UI becomes self-explanatory: "share my current life context with the AI" beats "share goals" when "goals" actually controls themes / stressors / wins / health-notes / safe-place / etc.
- **S3 · Schema-driven `getProfileContext`** (~80 lines but eliminates ~150 of if-blocks; depends on S1). Currently 28 if-blocks hand-format each field's prompt instruction. Replace with a metadata table:
  ```
  PROFILE_PROMPT_SCHEMA = [
    { field: 'practice.typicalTime', scope: 'practice', tpl: t => `Typical time: ${t}. ${TIME_HINTS[t] || ''}` },
    { field: 'practice.posture',     scope: 'practice', tpl: t => `Posture: ${t}. ${POSTURE_HINTS[t] || ''}`, mandatory: true },
    ...
  ]
  ```
  Iterate once over the schema, applying scope-gates, producing the prompt block. Adding a field = one schema entry, not 5 file edits.
- **S4 · Ring-buffer `_profileChangeLog` time-decayed cap** (~10 lines). Currently fixed at last 50 events. For sparse users that can stretch back 2 years; for active users it only covers a month. Switch to "last 50 OR last 90 days, whichever is smaller" — old entries drop off naturally without losing recent context for low-activity users.

### Perchance-native format wins (single turn, ~3 items)

Idiom-specific improvements that take advantage of Perchance's strengths.

- **P1 · Expose `PROFILE_SCHEMA` as a top-level Perchance function in `perchance_1.txt`** (~15 lines, depends on S3). Once the profile schema is a data table, exposing it as `root.profileSchema()` (returns `{ categories: [...], fields: [{id, label, type, scope, hint}, ...] }`) lets other Perchance generators build consistent hypnosis-profile UIs without copy-pasting our markup. Sister generators (sleep app, meditation tracker, etc.) can render compatible profiles, share data, and stay in sync as we add fields.
- **P2 · `gzipSync` on backup-file payload** (~8 lines in `exportBackup`). Perchance's environment exposes `gzipSync` for the wider ecosystem. Backup files are 200KB+ JSON; gzipping typically gets 70-80% reduction → 40-60KB. Faster downloads, easier sharing. Round-trip via `gunzipSync` in `importBackup`. Versioned: backup `version: 7` adds a magic byte to flag gzip-wrapped payloads, fall back to v6 plain JSON.
- **P3 · Lazy-load `_aiAuditLog` and `_metricsDays` at first consumer access** (~12 lines). Both currently load at boot via top-level `_loadAuditLog().catch(...)`. Boot delay matters in iframe contexts — Perchance generators reload on every navigation. Move to `getAuditLog()` / `getMetricsDays()` accessor pattern that loads on first call, primes consumers from cache thereafter. ~40-100ms saved on cold start for users with full history.

### Items from previous research that should be SKIPPED on closer review

The Round 22b research pass listed items that, on this critical-eye look, are
over-collection. Skip these even though they were proposed:

- **~~A7 · Voice-test sampling counter~~** — most users sample 1-2 voices and settle. The signal-to-noise of "sampled 7 voices" as a Director nudge trigger is too low to justify per-click tracking. SKIP.
- **~~A8 · Soundscape settle time~~** — same problem. Picking a soundscape can take 30s for legitimate reasons (reading the descriptions). Latency is not engagement signal. SKIP.
- **~~A10 · "Why-now" tag chip dwell time~~** — the chip selection IS the signal. Dwell time before selection adds no actionable information. The user picks a chip when they're ready; tracking how long they considered before picking is over-collection. SKIP.

That's 3 of 10 Cluster A items removed. Cluster A's value remains in A1
(pause telemetry), A2 (mid-session adjustments — but only `tempoMultiplier`,
not voice volume which is just context-aware), A3 (audit enrichment), A4
(phase regen tracking), A5 (feature first-use — pair with F4 above), A6
(backgrounded-tab), A9 (generate→accept latency).

### Sequencing notes

Several refinements unblock or improve previous-research items:
- **F1 (circular buffer)** is a prerequisite for **B6** (setting drift detection — needs samples, not running sums)
- **F4 (`feat` → Set)** is a prerequisite for **A5** (first-use timestamps — natural to merge into `featAt: { [name]: ts }`)
- **S1 + S2 (grouped profile + scope toggles)** unblock **S3** which unblocks **P1**
- **R6 (drop profileSnapshot redundancy)** supersedes the earlier B1 — same item, promoted to "drop" since it's pure reduction
- **R5 (TTL old scripts)** opens room to LIFT the 50-record cap to 100-200 records without storage growth

### Good as-is — DO NOT touch

The audit confirmed these surfaces are well-shaped and shouldn't be churned:
- **3-tier memory architecture** (digest → rolling → long-term) — proven pattern, AI-prompt consumers stable, embedding retrieval works.
- **Diff bouncer** — the debouncing logic correctly handles rapid edits; the bouncer-per-field architecture scales linearly.
- **Activity timeline merge** — composes 3 sources cleanly via `renderTimeline`; backward-compat shims in place.
- **Adaptive weights with decay (`updateWeights`)** — exponential decay (0.98/session) is well-tuned; reward function (mood-Δ + completion ± emergency) is grounded.
- **Embedding storage** (`_EMBED_KEY`) — capped at 100 vectors with oldest-first eviction; serialization to plain arrays for IDB is correct.
- **Pre-gen cache (Round 21c)** — 26h expiry + 3-entry cap is reasonable; mobile/battery gates close the spec.

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

Last updated: 2026-04-24
