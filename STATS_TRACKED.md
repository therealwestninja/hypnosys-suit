# Stats Tracked — Advanced Hypnosis Narration Engine

All data is stored locally on the user's device. Nothing is transmitted to any server.

---

## Storage Locations

| Store | Key | Contents | Max Size |
|---|---|---|---|
| IndexedDB | `stillness_analytics_v1` | Aggregated analytics (this document) | ~500 bytes |
| IndexedDB | `stillness_history_v1` | Session history (last 50 sessions) | ~200 KB |
| IndexedDB | `stillness_program_v1` | Active structured program state | ~200 bytes |
| localStorage | `stillness_settings_v1` | User settings (audio, viz, HUD) | ~800 bytes |
| localStorage | `stillness_profile_v1` | User profile (bio, appearance) | ~1 KB |
| localStorage | `stillness_safety_v1` | Safety screen attestation timestamp | 13 bytes |
| localStorage | `stillness_tour_v2` | Tour completion flag | 1 byte |
| Perchance KV | `stillness_settings_v1` | Mirror of localStorage settings | ~800 bytes |
| Perchance KV | `stillness_profile_v1` | Mirror of localStorage profile | ~1 KB |

---

## Analytics Store (`stillness_analytics_v1`)

O(1) storage — fixed size regardless of session count. Updated incrementally after each session.

### Schema Version

| Field | Type | Description |
|---|---|---|
| `v` | `number` | Schema version. Currently `1`. Used for future migrations. |

### Frequency Maps

Each is an object where keys are IDs and values are usage counts. Only IDs the user has actually used appear.

| Field | Key Format | Description | Source |
|---|---|---|---|
| `p` | Persona ID (e.g. `"driver"`, `"therapist"`) | How many times each guide persona has been used | Step 1 selection → `finishSession()` |
| `m` | Method ID (e.g. `"highway"`, `"spiral"`) | How many times each induction method has been used | Step 2 selection → `finishSession()` |
| `i` | Intention type (e.g. `"emotional"`, `"sleep"`) | How many times each intention type has been chosen | Step 3 selection → `finishSession()` |
| `s` | Soundscape type (e.g. `"rain"`, `"drone"`, `"none"`) | How many times each soundscape has been used | Step 4 selection → `finishSession()` |
| `z` | Visual effect (e.g. `"tunnel"`, `"breathing"`, `"auto"`) | How many times each visual effect has been used | Step 4 selection → `finishSession()` |
| `ln` | Session length (e.g. `"short"`, `"medium"`, `"long"`) | How many times each session length has been chosen | Step 4 selection → `finishSession()` |

### Time Patterns

| Field | Type | Description | Source |
|---|---|---|---|
| `hr` | `number[24]` | Session count by hour of day. Index 0 = midnight, 23 = 11pm. | `new Date(record.date).getHours()` in `recordAnalytics()` |
| `dw` | `number[7]` | Session count by day of week. Index 0 = Sunday, 6 = Saturday. | `new Date(record.date).getDay()` in `recordAnalytics()` |

### Mood & Wellbeing

| Field | Type | Description | Source |
|---|---|---|---|
| `mood.preSum` | `number` | Running sum of all pre-session mood scores (0–10 scale) | Pre-session state slider → `finishSession()` |
| `mood.postSum` | `number` | Running sum of all post-session mood scores (0–10 scale) | Post-session state slider → `recordPostMood()` |
| `mood.deltaSum` | `number` | Running sum of (post − pre) mood deltas. Positive = improvement. | Computed in `recordPostMood()` |
| `mood.n` | `number` | Count of sessions with mood data recorded | Incremented when `preMood` is non-null |

**Derived values** (computed by `deriveInsights()`):
- `avgPreMood` = `preSum / n`
- `avgPostMood` = `postSum / n`
- `avgImprovement` = `deltaSum / n`

### Completion Statistics

| Field | Type | Description | Source |
|---|---|---|---|
| `comp.total` | `number` | Total sessions started | Incremented in every `recordAnalytics()` call |
| `comp.finished` | `number` | Sessions completed normally (reached emergence phase) | `record.completed && !wasEmergency` |
| `comp.emergency` | `number` | Sessions ended via emergency end (Escape key or End button) | `wasEmergency === true` |

**Derived value**: `completionRate` = `finished / total × 100`

### Duration

| Field | Type | Description | Source |
|---|---|---|---|
| `dur.sum` | `number` | Total seconds across all sessions | `record.durationSec` from elapsed timer |
| `dur.n` | `number` | Count of sessions with duration data | Incremented when `durationSec > 0` |

**Derived value**: `avgDurationSec` = `sum / n`

### Settings Averages

Running sums to compute what audio levels the user typically prefers.

| Field | Type | Description | Range | Source |
|---|---|---|---|---|
| `cfg.rateSum` | `number` | Sum of speech rate values used | Each value: 0.5–1.0 | `record.speechRate` |
| `cfg.pitchSum` | `number` | Sum of voice pitch values used | Each value: 0.5–1.2 | `record.voicePitch` |
| `cfg.volSum` | `number` | Sum of narration volume values used | Each value: 0.0–1.0 | `record.voiceVolume` |
| `cfg.droneSum` | `number` | Sum of soundscape volume values used | Each value: 0.0–1.0 | `record.droneVolume` |
| `cfg.n` | `number` | Count of sessions with settings data | Incremented per session |

**Derived value**: `avgSpeechRate` = `rateSum / n`

### Feature Discovery

| Field | Type | Description |
|---|---|---|
| `feat` | `number` (8-bit bitmask) | Which features the user has tried at least once |

| Bit | Value | Feature | Trigger |
|---|---|---|---|
| 0 | `1` | Affirmations | User types in the affirmations textarea |
| 1 | `2` | Structured Programs | User starts a multi-day program |
| 2 | `4` | Custom Persona | User saves a custom persona |
| 3 | `8` | Skip-to-Script | User clicks "I already have a script" |
| 4 | `16` | Timing Fixer | User clicks "Fix Timings" button |
| 5 | `32` | Share Config | User clicks the share button |
| 6 | `64` | Backup | User exports a backup file |
| 7 | `128` | Double Induction | User runs the Double Induction method |

### Timestamp

| Field | Type | Description |
|---|---|---|
| `ts` | `number` | Unix timestamp (ms) of last analytics update |

---

## Session History Records (`stillness_history_v1`)

Array of up to 50 session records, newest first. Each record contains:

| Field | Type | Description | Used For |
|---|---|---|---|
| `id` | `string` | Unique ID (`"sess_" + timestamp`) | Deduplication on backup import |
| `date` | `number` | Unix timestamp when session started | History display, heatmap, time patterns |
| `guide` | `string` | Display name of the persona used | History card display |
| `method` | `string` | Display name of the method used | History card display |
| `intention` | `string` | Intention type ID | Analytics frequency map |
| `length` | `string` | Session length (`"short"`, `"medium"`, `"long"`) | Analytics, replay |
| `soundscape` | `string` | Soundscape type ID | Analytics, replay |
| `durationSec` | `number` | Actual elapsed seconds | Stats display, analytics |
| `completed` | `boolean` | Whether session completed normally | Completion rate |
| `preMood` | `number\|null` | Pre-session state score (0–10) | Sparkline chart, analytics |
| `postMood` | `number\|null` | Post-session state score (0–10) | Sparkline chart, analytics |
| `journal` | `string\|undefined` | Post-session reflection text | Pattern tracking |
| `script` | `string` | Full generated script text | Replay, edit, clinical review |
| `personaId` | `string` | Persona ID for replay | Replay button |
| `methodId` | `string` | Method ID for replay | Replay button |
| `intentionText` | `string` | User's typed intention description | Replay, history display |
| `voiceName` | `string` | Selected TTS voice name | Replay |
| `speechRate` | `number` | Voice speed (0.5–1.0) | Replay, analytics |
| `voicePitch` | `number` | Voice pitch (0.5–1.2) | Replay |
| `voiceVolume` | `number` | Narration volume (0.0–1.0) | Replay |
| `droneVolume` | `number` | Soundscape volume (0.0–1.0) | Replay, analytics |
| `vizOverride` | `string` | Visual effect selection | Replay |
| `vizSettings` | `object` | `{speed, intensity, complexity, trail}` | Replay |
| `duckingEnabled` | `boolean` | Whether audio ducking was on | Replay |

---

## User Profile (`stillness_profile_v1`)

| Field | Type | Description | Used For |
|---|---|---|---|
| `name` | `string` | Display name (max 30 chars) | Sidebar header, AI prompt injection |
| `pronouns` | `string` | Pronouns (`"he/him"`, `"she/her"`, `"they/them"`, etc.) | AI prompt injection |
| `age` | `string` | Age range (`"18-24"`, `"25-34"`, etc.) | AI prompt injection |
| `experience` | `string` | Experience level (`"beginner"`, `"some"`, `"experienced"`, `"professional"`) | AI prompt injection |
| `about` | `string` | Free-text bio (max 300 chars) | AI prompt injection |
| `appearance` | `string` | Physical description (max 300 chars) | AI personalization (body scan, visualization) |
| `goals` | `string` | Therapeutic goals (max 300 chars) | AI prompt injection |

---

## User Settings (`stillness_settings_v1`)

| Field | Type | Description |
|---|---|---|
| `selectedPersonaId` | `string` | Last selected guide persona |
| `selectedMethodId` | `string` | Last selected induction method |
| `intentionType` | `string` | Last selected intention type |
| `length` | `string` | Last selected session length |
| `soundscape` | `string` | Last selected soundscape |
| `droneEnabled` | `boolean` | Whether soundscape is enabled |
| `duckingEnabled` | `boolean` | Whether audio ducking is on |
| `speechRate` | `number` | Voice speed (0.5–1.0) |
| `voicePitch` | `number` | Voice pitch (0.5–1.2) |
| `voiceVolume` | `number` | Narration volume (0.0–1.0) |
| `droneVolume` | `number` | Soundscape volume (0.0–1.0) |
| `vizOverride` | `string` | Selected visual effect |
| `vizSettings` | `object` | `{speed, intensity, complexity, trail}` |
| `hud` | `object` | `{fixationDot, textDisplay, phaseDots, phaseLabel, progressBar, sessionTime, labels}` |
| `personaFilter` | `string` | Last active persona filter tab |
| `methodFilter` | `string` | Last active method filter tab |
| `intentionInput` | `string` | Last typed intention text |
| `userAffirmations` | `string` | Last typed affirmations |
| `voiceName` | `string` | Last selected TTS voice name |

---

## Derived Insights (computed at display time, not stored)

| Insight | Computation | Displayed In |
|---|---|---|
| Favourite guide | Most-used key in `p` map | Sidebar stats |
| Favourite method | Most-used key in `m` map | Available via `deriveInsights()` |
| Favourite soundscape | Most-used key in `s` map | Sidebar stats |
| Favourite length | Most-used key in `ln` map | Available via `deriveInsights()` |
| Top 3 personas | Top 3 keys in `p` map | Available via `deriveInsights()` |
| Top 3 methods | Top 3 keys in `m` map | Available via `deriveInsights()` |
| Average mood improvement | `mood.deltaSum / mood.n` | Sidebar stats (green) |
| Peak practice hour | Index of max value in `hr` array | Sidebar stats |
| Peak practice day | Index of max value in `dw` array | Available via `deriveInsights()` |
| Completion rate | `comp.finished / comp.total × 100` | Sidebar stats |
| Average session duration | `dur.sum / dur.n` | Available via `deriveInsights()` |
| Average speech rate | `cfg.rateSum / cfg.n` | Available via `deriveInsights()` |
| Features discovered | Bitmask decode of `feat` | Available via `deriveInsights()` |

---

## Heatmap Data (computed from history, not stored separately)

The GitHub-style activity heatmap is rendered from `stillness_history_v1` session dates. For each of the last 112 days (16 weeks), it counts how many sessions occurred on that day and maps to a 5-level green intensity scale.

| Level | Sessions | Color |
|---|---|---|
| 0 | None | Dark wood background |
| 1 | 1 session | `rgba(57,255,142,0.15)` |
| 2 | 2 sessions | `rgba(57,255,142,0.35)` |
| 3 | 3 sessions | `rgba(57,255,142,0.55)` |
| 4+ | 4+ sessions | `rgba(57,255,142,0.8)` |

---

## Sparkline Chart (computed from history, not stored separately)

The rising-score sparkline plots the last 20 sessions that have both `preMood` and `postMood` values. Two lines are drawn:

- **Cyan line**: Post-session mood scores over time
- **Faded cyan area**: Pre-session mood scores (fill below)

The chart shows therapeutic progress — a rising post-mood line indicates sessions are increasingly effective.
