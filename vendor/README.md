# Vendored Upstream

Files in this directory are **unmodified copies** of upstream source. Do not edit them directly.

## `perchance-ai-character-chat/`

The current baseline of Perchance's official AI Character Chat generator.

- `perchance_1.txt` — top DSL (plugin imports, list declarations, async functions)
- `perchance_2.txt` — HTML panel (chat UI, event handlers, storage layer, AI pipeline, sandbox bridge)

Source: https://perchance.org/ai-character-chat (viewable via the generator editor)

Last synced: 2026-04-17 (initial scaffold commit)

### How to sync upstream

When Perchance updates the generator:

1. Open the generator's editor
2. Copy the full contents of the top zone — paste over `perchance_1.txt`
3. Copy the full contents of the HTML panel — paste over `perchance_2.txt`
4. Commit as: `vendor: sync upstream YYYY-MM-DD`
5. Re-run the build. Test. Fix any extension-point breakage in `src/profile/mount.js`.

### Why vendor it

- Deterministic baseline for diffs and builds
- Ability to see exactly what upstream changed between syncs
- No dependency on fetching live upstream at build time
- Offline development

## License

The upstream `ai-character-chat` source in this directory is explicitly MIT-licensed by its author. The header comment of `perchance_2.txt` grants the right to copy, host, modify, and use the code (including commercially). This project reuses it under those terms.
