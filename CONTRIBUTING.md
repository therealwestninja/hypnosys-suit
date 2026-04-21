# Contributing

Thank you for your interest in improving the Hypnosis Script Generator. This document covers how to contribute content, code, and bug reports.

## Architecture

The entire project is a **single HTML file** designed to run in Perchance.org's HTML panel. There is no build step, no bundler, no framework. The file contains:

1. `<style>` — all CSS (~700 lines)
2. HTML body — wizard UI, session screen, post-session screen, modals, live panel (~400 lines)
3. `<script>` — all JavaScript (~2800 lines)

### Key JavaScript sections (in order)

| Section | Purpose |
|---|---|
| IDB Storage | IndexedDB key-value wrapper (`idbGet`, `idbSet`) |
| Toast System | Non-blocking notification toasts |
| iOS Safari Fix | Viewport meta for touch devices |
| Data: Methods | 37 induction method definitions |
| Data: Pacing Rules | Per-method pacing instructions for the AI |
| Data: Intention Types | 20 intention type definitions with AI guidance |
| Data: Personas | 25 guide persona definitions |
| State | Global state object and phase durations |
| Step Management | Wizard navigation, validation, UI updates |
| Rendering | Grid renderers, detail panels, config summary |
| Configure Panel | Dynamic Step 4 context rendering |
| Voices | TTS voice loading and quality scoring |
| Soundscape Engine | `SoundscapeEngine` class (10 ambient types) |
| Visual Renderers | 16 Canvas 2D render functions + `VIZ_RENDERERS` map |
| Visual Setup | `setupVisual()` — mounts renderers with ResizeObserver |
| Persona Editor | Modal CRUD for custom personas |
| Crafter | `craftSuggestion()` — AI suggestion generation |
| Script Generator | `generateFullScript()` — AI full script generation |
| Library | Save/load/delete scripts, session history, backup/restore |
| Export & Share | Script download, URL sharing, backup export/import |
| Transcript | Populate and toggle script transcript |
| Session Player | `startSession()`, `speakScript()`, `speak()`, `_doSpeak()` |
| Post-Session | `finishSession()` — summary, mood, grounding, cooldown |
| Emergency | Persona re-alert scripts, `emergencyEnd()` |
| Handlers | All event listeners (~65) |
| Init | Async startup: IDB migration, rendering, AI preload |

### Adding a new persona

Add an entry to `BUILTIN_PERSONAS`:
```js
{ id: 'unique-id', name: 'The Name', tagline: 'Short description.', 
  category: 'contemplative|showman|somatic|authoritarian', 
  systemPrompt: `Voice instructions for the AI. Describe tone, vocabulary, pacing, signature phrases.` },
```

Then add a matching re-alert script to `PERSONA_REALERTS`:
```js
'unique-id': "Re-alert script with [PAUSE N] markers. Match the persona's voice.",
```

### Adding a new method

Add an entry to `METHODS`:
```js
{ id: 'unique-id', name: 'Method Name', tagline: 'One-line description.', 
  tags: ['tag1', 'tag2'], category: 'therapeutic|somatic|demonstration|rapid|advanced', 
  detail: 'Full description of the method and how it works.', 
  visual: 'breathing|tunnel|vortex|candle|moire|starfield|...',  // any VIZ_RENDERERS key
  drone: true|false,  // whether soundscape is recommended
  pacing: 'monotony|structured|narrative|focused|...',  // PACING_RULES key
  recommended: true  // optional — shows a badge
},
```

If the method needs a new pacing style, add it to `PACING_RULES`:
```js
'pacing-key': `Pacing instructions for the AI. Describe phrase length, pause timing, tone.`,
```

### Adding a new visual renderer

Write a function with the signature `(ctx, W, H, t, color)`:
```js
function renderMyEffect(ctx, W, H, t, color) {
  const rgb = hexToRgb(color);
  // ... Canvas 2D drawing using t (time in seconds) as the animation driver
}
```

Register it in `VIZ_RENDERERS`:
```js
const VIZ_RENDERERS = {
  // ...existing...
  'my-effect': renderMyEffect,
};
```

Add an `<option>` for it in both the Configure tab `vizSelect` and the live panel `liveVizSelect`.

### Adding a new soundscape

Add a new `else if` branch in `SoundscapeEngine.start()`:
```js
} else if (type === 'my-sound') {
  // Create oscillators, noise buffers, filters, etc.
  // Connect everything to `m` (master gain)
  // For periodic events, use setTimeout with `if (!this.running) return;` guard
}
```

Add `<option>` entries in both `soundscapeSelect` (Configure tab) and `liveSoundscapeSelect` (live panel).

## Code style

- No build tools, no transpilation, no modules — everything is vanilla ES2020+ in a single `<script>`.
- Use `const` and `let`, never `var`.
- Use `?.` optional chaining and `??` nullish coalescing freely.
- Template literals for HTML generation — always escape user content via `escapeHtml()`.
- Prefer `notify.warn/error/success/info` over `alert()`.
- Always `linearRampToValueAtTime` for Web Audio gain changes — never set `.value` directly.
- Guard all `Number` inputs with `Number.isFinite()` before passing to Web Audio.
- All canvas renderers must respect `state.abortFlag` (check `if (state.abortFlag) return;` in rAF loops).
- Cap visual flicker at ≤2 Hz per WCAG 2.3.1.

## Bug reports

Include:
1. Browser and version
2. Console errors (right-click → Inspect → Console)
3. Steps to reproduce
4. Which step of the wizard you were on

## Testing

Since this runs on Perchance, testing is manual:
1. Extract the `<script>` content and run `node --check` to verify syntax.
2. Open the generator on Perchance and verify no console errors.
3. Walk through all 5 wizard steps.
4. Start a session and verify TTS, visuals, and soundscape all work.
5. Test emergency end → grounding exercise.
6. Test backup export → import on a fresh browser profile.
