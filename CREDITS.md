# Credits

This project builds on work done by others. Attribution matters.

## Upstream

**Perchance AI Character Chat** — the generator and plugin ecosystem this project forks.
- Site: https://perchance.org/ai-character-chat
- Plugins used: `ai-text-plugin`, `text-to-image-plugin`, `upload-plugin`, `super-fetch-plugin`, `comments-plugin`, `tabbed-comments-plugin-v1`, `fullscreen-button-plugin`, `combine-emojis-plugin`, `bug-report-plugin`, `dynamic-import-plugin`, `ai-character-chat-dependencies-v1`

The vendored copy of the upstream source lives in [`vendor/perchance-ai-character-chat/`](vendor/perchance-ai-character-chat/) and is unmodified. Any improvements upstream ships can be pulled in as clean diffs against that baseline.

## Inspiration — Code and Patterns

The following projects are MIT-licensed and informed specific parts of this design. No code is copied verbatim; patterns and architectural ideas were studied and adapted.

**[Perchance Memory Trimmer Tool (PMT)](https://github.com/therealwestninja/Perchance-Memory-Trimmer-Tool)** — by therealwestninja. MIT.
- Inspired the approach to local-first analysis of Perchance chat data
- Entry-ID hashing pattern (djb2 on trimmed entry text) — we adopt compatible IDs so PMT and this fork can interoperate on shared memory data in the future
- Snapshot / restore pattern for reversible operations
- Module boundary discipline (`core/` / `host/` / `ui/`)

**[Style Pilot](https://greasyfork.org/en/scripts/...)** — MIT.
- Drift-tracking concept (variety scoring across recent replies) may be revisited if we add writing-quality signals
- Macro-expansion pattern for user-defined reusable snippets

## License Compatibility

This project is MIT-licensed. All inspiration sources cited above are MIT-compatible. If you contribute code derived from a non-MIT-compatible source, disclose it in your PR so we can evaluate.
