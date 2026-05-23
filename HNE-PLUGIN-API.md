# HNE Plugin API — Integration Brief for Another Claude

You're working on a Perchance generator that wants to use the Advanced Hypnosis Narration Engine (HNE) as a function library. This brief tells you what's available, how to use it, when to use which export, and what to watch out for.

**TL;DR:** HNE exposes three pure functions for content rotation. `freshSeeds` and `freshMotifs` return curated content from HNE's bank (meditation-coded register). `pickFromBank` is a generic rotation utility that runs against *your* data — use this when HNE's register doesn't match your project's voice.

---

## 1. Importing

In your generator's top editor (`perchance_1.txt`-equivalent):

```
hne = {import:advanced-hypnosis-narration-engine}
```

That makes the three exports available as `root.hne.freshSeeds(...)`, `root.hne.freshMotifs(...)`, `root.hne.pickFromBank(...)` from anywhere in your generator — your top editor, your `<script>` block, your phase functions, anywhere.

**The alias `hne` is arbitrary.** Use whatever name fits your code. Just match it in your call sites.

---

## 2. The Three Functions

### 2.1 `root.hne.freshSeeds(opts) → string[]`

Returns evocative sensory anchor **phrases** from HNE's curated 60+ phrase corpus. Tagged across 12 categories: `sleep`, `anxiety`, `focus`, `performance`, `creativity`, `pain`, `trance`, `confidence`, `behavior`, `grounding`, `release`, `emotional`.

**Signature:**
```js
root.hne.freshSeeds({
  profileTags: string[],   // any-match tag filter; empty = all tags
  n:           number,     // how many to return (default 5)
  excludeSet:  string[],   // phrases to skip (rotation memory)
})
// returns: string[] — up to n phrases, randomized, shallow copies
```

**Use it for:** scattering varied sensory anchors through an AI prompt to break mode-collapse. Each call returns *different* phrases, never settling on a fixed image.

**Example:**
```js
let anchors = root.hne.freshSeeds({
  profileTags: ["sleep", "release"],
  n: 3,
  excludeSet: lastUsedAnchors,
});
// → ["a held shoulder finally dropping",
//    "ice giving way to slow steady water",
//    "breath leaving longer than it came in"]
```

**Register:** meditation/hypnosis-coded. Grounding, somatic, kinesthetic. Examples: *"the slow pull of evening tide"*, *"a held shoulder finally dropping"*, *"the breath remembering its own rhythm"*. If your project's voice is different — occult, playful, clinical, noir — these will dilute your tone. Use `pickFromBank` instead.

---

### 2.2 `root.hne.freshMotifs(opts) → motif[]`

Returns full motif **objects** from HNE's curated 24-motif corpus. Each motif is a structured "through-line" image with metadata for threading across narration, visuals, and audio.

**Signature:**
```js
root.hne.freshMotifs({
  profileTags: string[],   // any-match tag filter
  n:           number,     // how many to return (default 1)
  excludeSet:  string[],   // motif IDs to skip
})
// returns: motif[] — each {id, image, essence, sense, tags, palette, viz, cue}
```

**Motif schema:**
```js
{
  id:      "brass-key",
  sense:   "tactile",                          // visual|auditory|kinesthetic|tactile
  tags:    "trance|behavior|confidence",       // pipe-separated, any-match
  image:   "a small brass key, warm from being held",
  essence: "something that opens, already in your hand",
  palette: { a: [0.42, 0.32, 0.14],            // RGB triples, 0..1 floats
             b: [0.78, 0.6, 0.26] },           //   .a is darker, .b is brighter
  viz:     ["candle", "mandala", "colorwash"], // preferred visualizers
  cue:     "bell"                              // bell|bowl|chime|drop|none
}
```

**Use it for:** picking ONE recurring image a session is meant to repeat deliberately. Where `freshSeeds` rotates VARIED anchors (anti-collapse), `freshMotifs` picks ONE image to be the spine of the session. The metadata fields let you thread it consistently: feed `image` + `essence` to your AI prompt, use `palette` to tint your visualizer, play a soft `cue` at the deepest moment.

**Example:**
```js
const [motif] = root.hne.freshMotifs({
  profileTags: ["focus", "trance"],
  n: 1,
  excludeSet: recentMotifIds,
});
if (motif) {
  // Inject into AI prompt:
  prompt += `\nMOTIF (weave 2-4 times across the session): "${motif.image}". ${motif.essence}.`;
  // Tint visualizer:
  vizColor = blendHex(accentColor, paletteToHex(motif.palette.b), 0.3);
  // Play cue at deepest phase:
  if (motif.cue !== "none") playSoftTone(motif.cue);
}
```

**Register:** same caveat as `freshSeeds`. Examples: *"a lighthouse beam, passing and returning"*, *"a single ember holding its heat beneath grey ash"*, *"the long, easy line of a spine that has stopped bracing"*. Meditation-coded.

---

### 2.3 `root.hne.pickFromBank(opts) → item[]` ★ The one you usually want

Generic rotation utility. **No bundled content** — operates on YOUR bank. Use this when HNE's meditation register would dilute your project's voice.

**Signature:**
```js
root.hne.pickFromBank({
  bank:         any[],     // YOUR bank — strings or objects (required)
  profileTags:  string[],  // any-match tag filter (objects only)
  n:            number,    // how many to return (default 1)
  excludeSet:   any[],     // items to skip, matched by excludeField
  tagField:     string,    // object field with tags (default "tags")
  excludeField: string,    // object field for dedupe (default "id")
})
// returns: item[] — shallow copies of matched items
```

**Behavior:**
- **Auto-detects bank shape** from the first non-null item:
  - Strings → matched against `excludeSet` by value equality. No tag filtering possible.
  - Objects → filtered by `tagField` (pipe-separated tags, any-match), deduped by `excludeField`.
- **Cold-start fallback:** if tag-filtered set has fewer than `n` items, broadens to ANY non-excluded item.
- **Returns shallow copies** of objects — you can't mutate the source bank by reference.
- **Untagged items are universal** — an object missing the `tagField` value matches every profileTags query.

**Example — fortune-teller picking a tarot card by question relevance:**
```js
const fortuneMotifs = [
  { card: "Queen of Cups",   tags: "love|intuition",
    image: "tea leaves settling into the shape of a bird" },
  { card: "Ace of Cups",     tags: "love|beginning",
    image: "a chalice brimming, water trembling at the rim" },
  { card: "Two of Swords",   tags: "decision|conflict",
    image: "a blindfolded figure holding two crossed blades" },
  { card: "The Hermit",      tags: "solitude|wisdom",
    image: "a lantern held aloft at the edge of a cliff" },
  // …
];

const [draw] = root.hne.pickFromBank({
  bank:         fortuneMotifs,
  profileTags:  ["love"],          // user asked about love
  n:            1,
  excludeSet:   recentlyDrawn,     // ["Queen of Cups", "The Tower"]
  excludeField: "card",            // dedupe by card name, not id
});
// → { card: "Ace of Cups", tags: "love|beginning", image: "..." }
```

**Example — string bank, simple rotation:**
```js
const myPhrases = ["a", "b", "c", "d", "e"];
const picked = root.hne.pickFromBank({
  bank:       myPhrases,
  n:          2,
  excludeSet: ["a", "b"],   // value-equality match
});
// → e.g. ["d", "c"]
```

**Use it for:** any project that needs variety-with-rotation-memory but where HNE's curated content would clash. Tarot readers, dream journals, noir adventure games, oracle generators, custom sleep apps with their own voice, etc.

---

## 3. Decision Tree: Which Function?

```
Do you need rotation-aware content selection?
├─ No → don't use HNE; you don't need it
└─ Yes
   ├─ Do you have your own corpus already?
   │  ├─ Yes → use `pickFromBank` against your data
   │  └─ No
   │     ├─ Want phrases (just strings)? → `freshSeeds`
   │     └─ Want structured motifs (image + palette + viz + cue)? → `freshMotifs`
   └─ Does HNE's meditation register fit your project's voice?
      ├─ Yes (you're a meditation/sleep/hypnosis app) → use `freshSeeds` / `freshMotifs`
      └─ No (occult, playful, clinical, noir, etc.) → use `pickFromBank`
```

**You can mix.** It's legitimate to use `pickFromBank` against your own thematic motifs for the spine of each session, AND sprinkle `freshSeeds` phrases into your AI prompt as supplementary sensory anchors. The two registers can coexist if the role separation is clear: your bank carries the *voice*, HNE's seeds add variety scaffolding the AI can pick from.

---

## 4. Common Patterns

### 4.1 Per-session rotation with cross-session memory

```js
// Persist to IndexedDB / localStorage so motifs don't repeat across reloads
const RECENT_KEY = "myapp_recent_motif_ids";
const ROTATION_CAP = 12;

let recentIds = JSON.parse(localStorage.getItem(RECENT_KEY) || "[]");

const [motif] = root.hne.pickFromBank({
  bank: myMotifs,
  profileTags: userTags,
  n: 1,
  excludeSet: recentIds,
  excludeField: "id",
});

if (motif) {
  recentIds.push(motif.id);
  if (recentIds.length > ROTATION_CAP) {
    recentIds = recentIds.slice(-ROTATION_CAP);
  }
  localStorage.setItem(RECENT_KEY, JSON.stringify(recentIds));
}
```

### 4.2 Derive profile tags from user input

If the user types a question / intention, derive tags by keyword matching:

```js
function deriveTags(userInput) {
  const text = (userInput || "").toLowerCase();
  const tags = new Set();
  if (/\b(love|romance|relationship|partner)\b/.test(text))  tags.add("love");
  if (/\b(work|career|job|business|money)\b/.test(text))     tags.add("career");
  if (/\b(decide|decision|choose|fork|crossroads)\b/.test(text)) tags.add("decision");
  if (/\b(future|ahead|forward|soon|next)\b/.test(text))     tags.add("future");
  if (/\b(past|memory|remember|childhood|before)\b/.test(text)) tags.add("past");
  // …
  return [...tags];
}

const [draw] = root.hne.pickFromBank({
  bank: fortuneMotifs,
  profileTags: deriveTags(question),
  n: 1,
  excludeSet: recentlyDrawn,
  excludeField: "card",
});
```

Empty `profileTags` is fine — the cold-start fallback gives you a random non-excluded item.

### 4.3 Layering: project motif + HNE seeds for variety

Some projects want their own motif AND a sprinkle of HNE's sensory phrases (to give the AI options without forcing them):

```js
const [motif] = root.hne.pickFromBank({
  bank: myCorpus, profileTags: userTags, n: 1,
  excludeSet: recentMotifIds, excludeField: "id",
});

// HNE seeds as supplementary anchors — the AI can use or ignore them
const sensoryAnchors = root.hne.freshSeeds({
  profileTags: userTags,
  n: 3,
  excludeSet: recentAnchors,
});

const prompt = `
MOTIF (recurring spine): "${motif.image}". ${motif.essence}.
OPTIONAL SENSORY ANCHORS (use if they fit, ignore otherwise):
${sensoryAnchors.map(a => "  • " + a).join("\n")}

Now write the reading...
`;
```

The motif carries your project's *voice*; the anchors are register-neutral enough to coexist as "use if it fits" candidates.

---

## 5. Gotchas

### 5.1 Don't use HNE's register where it doesn't belong

If your generator's voice is *"the cards whisper of what's to come, dear seeker..."*, dropping in *"the breath remembering its own rhythm"* will sound jarring. Use `pickFromBank` against your own thematic bank. The reviewer who triggered the creation of `pickFromBank` was right about this.

### 5.2 Mixed string/object banks behave undefined-ly

`pickFromBank` probes the FIRST non-null item to decide whether the bank is strings or objects. If you mix them (some strings, some objects), behavior is undefined. Pick one shape and stick to it.

### 5.3 `excludeField` defaults to `"id"`

If your objects don't have an `id` field, dedupe doesn't work — every item appears unique. Either add `id` to your data or pass `excludeField` explicitly (e.g. `"card"`, `"name"`, `"key"`).

### 5.4 Untagged items are universal

An object missing the `tagField` value (or with `tags: ""`) matches every `profileTags` query. This is intentional — it lets you mark items as "always eligible". But if you have a typo in your tag field, you'll get this behavior accidentally.

### 5.5 `excludeSet` is value-equality, not deep equality

For string banks, exclusion is straight `Set.has(item)`. For object banks, exclusion is `Set.has(item[excludeField])`. Don't pass whole objects in `excludeSet` — pass the dedupe keys (IDs, names, whatever your `excludeField` is).

### 5.6 Returned objects are shallow copies — nested fields aren't cloned

You can mutate `result[0].image = "..."` safely. But mutating `result[0].palette.a[0]` would still modify the source bank's nested array, because the shallow copy only clones top-level properties. If you need full isolation, deep-clone the returned items.

### 5.7 Cold-start fallback can ignore your tags

If your tag filter produces zero matches, the function falls through to ANY non-excluded item. That's deliberate — callers always get something useful, no need to handle empty results. But if you DO want to know "tag filter produced no matches", you'd need to do that check yourself before calling.

---

## 6. End-to-End: Building a Tarot Reader

This is the canonical use case `pickFromBank` was designed for.

**Top editor (`perchance_1.txt`-equivalent):**
```
$meta
  title = Madame Aurelia's Carnival Reading

hne = {import:advanced-hypnosis-narration-engine}
aiTextPlugin = {import:ai-text-plugin}

fortuneMotifs
  // Each line is one card. Tags are pipe-separated for any-match filtering.
  {card:"Queen of Cups",  tags:"love|intuition|emotion",  image:"tea leaves settling into the shape of a bird, just where the cup narrows"}
  {card:"Ace of Cups",    tags:"love|beginning|renewal",   image:"a chalice brimming, water trembling at the very rim"}
  {card:"Two of Cups",    tags:"love|union|partnership",   image:"two figures pouring their cups into a single rising stream"}
  {card:"Three of Swords", tags:"heartbreak|grief|wound",  image:"three blades crossing a single beating heart, but the blood is wine"}
  {card:"The Hermit",     tags:"solitude|wisdom|search",   image:"a lantern held aloft at the edge of a cliff, with all the night patient behind"}
  {card:"The Tower",      tags:"upheaval|sudden|truth",    image:"a crown of lightning, falling stones that turn to white birds in flight"}
  {card:"The Star",       tags:"hope|future|guidance",     image:"a girl pouring water into a still pool, and the pool full of stars"}
  // ... add as many cards as you want; the function scales
```

**HTML pane `<script>`:**
```js
const RECENT_KEY = "aurelia_recent_cards";
const ROTATION_CAP = 7;

function getRecentDraws() {
  try { return JSON.parse(localStorage.getItem(RECENT_KEY) || "[]"); }
  catch { return []; }
}

function deriveTags(question) {
  const t = (question || "").toLowerCase();
  const tags = new Set();
  if (/love|heart|romance|partner|relationship|crush/.test(t)) tags.add("love");
  if (/decide|choose|fork|two paths|stuck/.test(t)) tags.add("decision");
  if (/future|ahead|soon|will|tomorrow/.test(t)) tags.add("future");
  if (/lonely|alone|isolation|withdraw/.test(t)) tags.add("solitude");
  if (/end|breakup|loss|over|finished/.test(t)) tags.add("heartbreak");
  if (/hope|wish|dream|aspire/.test(t)) tags.add("hope");
  return [...tags];
}

async function readForUser(question) {
  // Pull bank from the top editor.
  const bank = root.fortuneMotifs.map(item => item);

  // HNE rotation against OUR bank — not HNE's meditation corpus.
  const recentCards = getRecentDraws();
  const [draw] = root.hne.pickFromBank({
    bank,
    profileTags:  deriveTags(question),
    n:            1,
    excludeSet:   recentCards,
    excludeField: "card",
  });

  if (!draw) return null;

  // Update rotation memory.
  recentCards.push(draw.card);
  while (recentCards.length > ROTATION_CAP) recentCards.shift();
  localStorage.setItem(RECENT_KEY, JSON.stringify(recentCards));

  // Build prompt for Madame Aurelia's voice — your project's register, not HNE's.
  const prompt = `
You are Madame Aurelia, a carnival fortune teller with a thick accent and theatrical bearing. The seeker has asked:

  "${question}"

The card you have drawn for them is: ${draw.card}.
The image you see in it is: ${draw.image}.

Speak to them as Madame Aurelia would. Reference the image. Give the reading 2-3 short paragraphs. End with a single piece of advice.
  `.trim();

  const result = await root.aiTextPlugin({
    instruction: prompt,
    startWith: "Ahh, my dear...",
  });

  return { card: draw.card, image: draw.image, reading: result.generatedText };
}
```

**That's it.** HNE's rotation machinery handles the variety-with-memory, your bank carries Madame Aurelia's occult voice, your AI prompt is yours. Three lines of HNE integration, no register clash.

---

## 7. Don't Forget

- HNE is a Perchance generator at `perchance.org/advanced-hypnosis-narration-engine`. Importing it loads its top editor into your scope. You don't need to load the whole HTML app — only the function exports are imported.
- All three functions are **pure** — no DOM, no state, no side effects. They work in headless contexts (Perchance's `$meta.dynamic` for share-link previews, server-side rendering, etc.).
- All three are **synchronous**. No `await` needed.
- The functions are **defensive** — passing garbage (non-array `bank`, non-numeric `n`, missing `opts`) returns `[]` rather than throwing.
- If your generator already has its own rotation logic and you just want HNE for its curated content, use `freshSeeds` / `freshMotifs` and skip `pickFromBank`. Don't import what you won't use.

---

## 8. If You Need More

The full HNE source is at `perchance.org/advanced-hypnosis-narration-engine?edit`. The top editor (`perchance_1.txt`) holds the three exported functions and the data-pack import wiring. The HTML pane (`perchance_2.txt`, the main app) consumes its own exports the same way you would — `root.freshSeeds(...)`, `root.freshMotifs(...)` — so you can study how HNE itself wires them in.

The motif corpus (24 entries) is inlined in the `freshMotifs` body. The seed corpus (60+ entries) is inlined in `freshSeeds`. Both functions document the schema in their headers — read the source if you want exhaustive detail.
