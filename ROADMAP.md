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

#### 7 · `textToImagePlugin` beyond backgrounds
The plugin is already imported for background generation. It's currently
under-leveraged. Candidate uses:

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

Last updated: 2026-04-24
