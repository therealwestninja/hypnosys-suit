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
