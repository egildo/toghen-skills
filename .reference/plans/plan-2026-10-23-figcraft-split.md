# Plan: the FigCraft skills split by layer (23 October 2026)

Three skills describe FigCraft 0.10.0, and all three trigger on the word: `artcraft:figcraft`, `toghen:figcraft` and `personal:figcraft`. They overlap, and each is updated on its own, so they will drift apart. Egildo ruled one owner per layer:

| Layer | Owner | Where it lives |
|---|---|---|
| The app: driving FigCraft, its surfaces, formats, traps | `artcraft:figcraft` | GitHub `egildo/skills`, `plugins/artcraft/skills/figcraft/` (to be public; GitHub reports it private today) |
| Toghen into FigCraft: `--figcraft`, `--figcraft-file`, the stamps, what Toghen builds and refuses | `toghen:figcraft` | this repository, `toghen-skills/plugins/toghen/skills/figcraft/` |
| This Mac: the port, the casks, where files live, the work log | `personal:figcraft` | Codeberg `egildo/private-skills` (checkout `~/Privata/dev/skills`), `plugins/personal/skills/figcraft/` |

## 1. Findings

- **`artcraft:figcraft`** (97 lines and `scripts/fc.mjs`) is docs-first and already pinned to 0.10.0. It knows things `toghen:figcraft` does not: a failed pinned batch puts the old tab back in front; `file.save` overwrites a path another tab holds; `component.combineVariants` names the set "Component 1"; a font style the family lacks is accepted silently; `window.new` opens nothing while the window is hidden; the formats table.
- **`toghen:figcraft`** (584 lines in four files): its sections from *The references* to *Fonts*, `commands.md` and `gotchas.md` are about the app, about 400 lines. Only *Toghen into FigCraft* and `toghen-target.md` are Toghen's own. Its app material that artcraft lacks:
  - the field table (Figma's field, the `node.set` key, the `variable.bind` field, what `node.get` reads);
  - write-then-bind, with the per-corner and per-side cases (writing a plain value unhooks a bound field, `strokeWeight` included);
  - what `node.get` reports and what FigCraft only keeps (wiring, frame modes, paragraph spacing and indent, max lines, corner smoothing, line caps, normal against pass-through);
  - slots (`component.makeSlot` only inside a component, `instance.slotInsert`, an instance's slot layer as `parent`);
  - `node.reparent` against `node.reorder` indices; `node.get` reading one level whatever `depth`;
  - library copies' ids prefixed, their mode ids not;
  - headless `figcraft-cli mcp` as a long session.
- **`personal:figcraft`** (3.4.3) holds the app again, with its own copy of `fc.mjs`, plus this Mac's facts and the work log.
- **The checkouts.** `~/Privata/dev/skills` is the Codeberg repository (remote `origin`); its `github` remote points at `egildo/skills`, which is now a different repository holding only `artcraft`. No local checkout of that one exists.

## 2. Units

1. **artcraft absorbs the app material** (`egildo/skills`). Add the list above to `artcraft:figcraft`, in its own voice and template (`plugins/artcraft/TEMPLATE.md`), keeping SKILL.md short: the field table and the reported-or-kept list go in a linked file. Bump the artcraft plugin's version; a line in its history. First, since the other two units remove what this one takes in.
2. **`toghen:figcraft` keeps only Toghen into FigCraft** (this repository). Keep `toghen-target.md`; cut SKILL.md to what a Toghen run needs: the two flags, the pinned file, write-then-bind as Toghen meets it, the stamps and the `-toghen` divider, what is built and refused, a run's messages, about 30 lines of driving essentials so a reader without artcraft is not lost. Drop `commands.md` and `gotchas.md`. Narrow the description to Toghen together with FigCraft. Name `artcraft:figcraft` for the app. Bump the `toghen` plugin to 0.2.0; a decisions entry. `toghen:pipeline` already links to `toghen:figcraft` by name and needs no change.
3. **`personal:figcraft` shrinks to this Mac** (Codeberg). Keep: the port the app was started with and how to find it, the two casks (the app from Homebrew, the CLI from `egildo/artcraft`), where `.figcraft` files live, the asking-first habits peculiar to this machine, the work log. Drop the app material and its `fc.mjs`; point to `artcraft:figcraft` and its driver. Narrow the description so it fires on this machine's setup, not on every FigCraft task. Bump `personal`; a HISTORY entry.

Units 2 and 3 run in parallel after unit 1. Each repository is committed locally; nothing is pushed without Egildo's word.

## 3. Risks

- **A reader of Toghen's marketplace without artcraft** loses the depth on driving FigCraft. The essentials stay in `toghen:figcraft`; the depth is one install away once `egildo/skills` is public.
- **The two remaining skills still both mention FigCraft.** The descriptions are narrowed so each fires on its own layer: artcraft on FigCraft, toghen on Toghen with FigCraft, personal on this Mac's setup.

## 4. Questions

1. **A local checkout of `egildo/skills`** for unit 1.
   - (a) Clone it to `~/Privata/dev/public-skills`, beside the private one, as the name of the Codeberg repository suggests.
   - (b) Work in a throwaway clone in the scratchpad.

   Recommended: (a). It will be edited again at every FigCraft release.
2. **How `toghen` refers to artcraft.**
   - (a) By name, in the text: "for the app, `artcraft:figcraft`".
   - (b) As a declared dependency in `plugin.json`. Across marketplaces this needs `allowCrossMarketplaceDependenciesOn`, unmeasured here, and a public `egildo/skills`.

   Recommended: (a).
3. **Pushing.** Toghen goes through a pull request and merges on Egildo's word, as always. For `egildo/skills` and Codeberg:
   - (a) Committed locally, diffs shown, pushed on his word.
   - (b) Pushed as soon as each unit is checked.

   Recommended: (a).

## Answers (23 October 2026)

1. **A checkout at `/Volumes/dev/GitHub/skills`**, beside the other GitHub checkouts.
2. **(a)** `toghen` names artcraft in its text; no declared dependency.
3. **(a)** `egildo/skills` and Codeberg are committed locally, the diffs shown, and pushed on Egildo's word.
