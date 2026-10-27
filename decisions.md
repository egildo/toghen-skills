# toghen-skills — decisions

## Ruled: a skills marketplace in Toghen (23 October 2026)

Egildo asked for Toghen to keep a Figma skill and a FigCraft skill, updated often and pinned to the version they document, and a skill for the pipeline from a repository to both; Toghen gains a package that is a skills marketplace. Researched first, by four agents: Figma's documentation online, FigCraft's latest release measured live, Toghen's own pipeline, and the marketplace format (`.reference/skills-research/`, 23 October 2026).

- **Where the marketplace lives.** Ruled: the manifest at the repository's root, the skills in this package. Rejected: everything inside the package, which a remote user could not add without an unconfirmed setting.
- **How many plugins.** Ruled: one, `toghen`, with three skills. Rejected: three plugins.
- **How a skill pins a version.** Ruled: `compatibility` and `metadata` in the frontmatter, and a table at the top; the plugin's version counts the text. Rejected: a table alone.
- **Which FigCraft.** Ruled: 0.10.0, the app and its command line both, upgraded on this machine.
- **Order.** Ruled: the FigCraft target finished first (figma 6.3.0), the skills after, so they describe it finished.

## What the writing found (23 October 2026)

The three skills were drafted by the agents that did the research, then checked against each other and against the code: the pipeline's author checked every Toghen command, flag, message and refusal in the other two; the FigCraft researcher checked the pipeline's statements about FigCraft against 0.10.0. Eleven errors were fixed before the first commit, among them a FigCraft start that would have waited forever (`open -a` hands an argument only to a new instance), `--library` listed as a per-component refusal when it is an argument error, and "FigCraft has no libraries", when FigCraft 0.10.0 has local libraries and lacks only keys.

**Defects in Toghen the research found, taught by the skills as they are today and not fixed here:**

- The command line's pairing hint names a *System pane*; the code field is in the Load pane's Source card (`cli/run.ts`).
- `toghen-query check` exits 0 when it lists problems, so it cannot gate a CI step on its own.
- Export to DTCG exists only on the plugin's screen; no command.
- `--library` beside a FigCraft flag says FigCraft has no libraries; it has local ones, without keys (`cli/main.ts`).
- `toghen-figma`'s docs are stale against Figma's documentation as of 23 October 2026 in twelve places (`.reference/skills-research/figma-docs-2026-10-23.md`, *Contradictions*): among them the per-plan mode limits, `clientStorage`'s 5 MB quota per plugin, pattern fills, vertical wrap, and the typings, at 1.139 where 1.142 is current.

## Fixed: the four defects the skills taught (23 October 2026)

The defects listed above under *What the writing found* were fixed in Toghen's round of the small defects (`.reference/plans/toghen-figma/plan-2026-10-23-small-defects.md`), and the skills stop teaching the workarounds. Plugin 0.3.0; the skills pin figma 6.3.1 and query 0.2.0.

- **The pairing hint** names the Source card at the foot of the Load pane (figma 6.3.1).
- **`toghen-query check`** exits 1 when it lists incomplete tokens, holes or non-factorable tokens, and a one-shot query exits 1 on an error or a usage error (query 0.2.0). Collisions do not fail it; the CI recipe keeps a grep for them as an optional stricter step.
- **Export** was recorded wrongly: it writes the loaded document at one combination, with the file's stamps, and reads nothing back from Figma. The skill says so; no command was built (`toghen-figma/specs.md`, *Not in this design*).
- **`--library` with FigCraft** now says FigCraft's libraries are local files with no published keys (figma 6.3.1).
- **Typings:** `toghen:figma` says Toghen builds against `^1.142.0`. The stale Figma facts were corrected in `toghen-figma`'s docs; the skill already stated them as Figma documents them.
- **Kept:** the *Known defects* table is now *Known limits*, holding the two that remain (a first dry run cannot see what the run creates; `layout=parts` learned only inside a run).

## Ruled: the FigCraft skills split by layer (23 October 2026)

Three skills described FigCraft 0.10.0 and all three fired on the word: `artcraft:figcraft` (the `egildo/skills` marketplace), `toghen:figcraft` and Egildo's personal one. Each was updated on its own, so they would drift apart. Planned in [plan-2026-10-23-figcraft-split.md](.reference/plans/plan-2026-10-23-figcraft-split.md).

- **One owner per layer.** Ruled by Egildo:
  - `artcraft:figcraft` owns the app: driving it, its surfaces, fields and traps.
  - `toghen:figcraft` owns Toghen into FigCraft: `--figcraft`, `--figcraft-file`, the pinned file, the stamps and the `-toghen` divider, what is built and refused, a run's messages.
  - The personal skill owns one machine.

  Rejected: three skills each holding the app, which is what drifts.
- **What moved.** The app material of `toghen:figcraft` went to `artcraft:figcraft` 1.1.0: the field table, write-then-bind with the corner and side cases, what is reported and what is only kept, slots, `node.reparent` against `node.reorder`, library copies' prefixes, the headless session. `commands.md` and `gotchas.md` went with it. `toghen:figcraft` keeps about thirty lines of driving essentials, so a reader without artcraft can follow a run.
- **How `toghen` refers to artcraft.** By name, in the text. Rejected: a declared dependency in `plugin.json`, which across marketplaces needs `allowCrossMarketplaceDependenciesOn`, unmeasured here.
- **Pushing.** `egildo/skills` and the personal repository are committed locally and pushed on Egildo's word; this repository goes through a pull request as always.

**What the move found:**
- **`node.get` reads one level whatever `depth` it is given, but only as a command.** That covers a batch, `run_command` and `figcraft-cli run`. The MCP `get_node` tool honours `depth` (measured against `figcraft-cli` 0.10.0). The first `toghen:figcraft` said it of `node.get` in general. Toghen reads through the command in pinned batches, so the target is unaffected.
- **`toghen:pipeline` named `toghen:figcraft` "for FigCraft itself"**; it now names it for what a run does in FigCraft, and `artcraft:figcraft` for the app. The guide's row changed the same way.

## Updated: FigCraft libraries (23 October 2026)

The skills now teach FigCraft libraries as built (`.reference/plans/toghen-figma/plan-2026-10-23-figcraft-libraries.md`): `--library <file.figcraft>` with FigCraft, `lock` in FigCraft, `--use` in both targets, and preferred values by name; the old refusal of `--library` beside a FigCraft flag is gone. Pinned to figma 6.4.0, plugin 0.4.0.

## Re-checked: the round of 24 October 2026 (24 October 2026)

Toghen moved to lib 6.0.0, postcss 0.21.0 and figma 7.0.0 (`.reference/plans/toghen-lib/plan-2026-10-24-computed-values.md`, `.reference/plans/toghen-figma/plan-2026-10-24-issues.md`). The skills were checked again by the routine in [specs.md](specs.md): against `toghen-figma/reference.md`, `guide.md`, `component-files.md`, `toghen-lib/reference.md`, both plans' *What the build found*, and the command line itself. Its usage text was read, and `check`, `members`, `toghen-query check` and the CSS build were run on the `mini` example and on broken copies of it, in a scratch folder. No Figma or FigCraft run was made. Plugin 0.5.0; the skills pin figma 7.0.0, lib 6.0.0, postcss 0.21.0 and query 0.2.0.

**What was untrue, and is fixed:**

- **The first build is no longer two phases** (`pipeline`: *Dry run first*, *Known limits*, the ten mistakes; `example.md`, B and D; `commands.md`, *A CI job*). Component files run in dependency order, and a first dry run counts the sets a file earlier in the run would create as present, checking instances of them by name only.
- **The plan lines** (`messages.md`, `SKILL.md`): `Variable plan:` and `Style plan:` count skips by cause, list every cause but `unchanged` ten rows at a time, and say `…and N more; --verbose lists them`. A set's dry run says `n would update`, not `0 updated`. After execute, `skipped` counts skips only.
- **The bare-number message** (`messages.md`, `example.md`): on a `width` or `height` it no longer offers `#null`. The example's broken copy now prints what the command prints, the folded-part error included.
- **A library collection's mode** (`figcraft/toghen-target.md`, which listed it as not built): built for FigCraft, not yet run against a real FigCraft; refused in Figma with its sentence until measured.

**What was added:** what a token's value may hold (CSS Color 4, `color-mix()`, `calc()` and its kin, composites computed, what needs the element refused with a sentence), the `{{` escape and its one open case, the three new `check` refusals, `--verbose` and the cycle message, `modes` through `--use`, an artboard instance's size, a bound paint compared by its variable only, and an unchanged FigCraft run that writes nothing.

**Kept as it was:** the Figma skill's Figma facts, still pinned to 23 October 2026; only its Toghen lines moved to 7.0.0, with the library-mode refusal and the two sizing behaviours not measured in Figma.

**The plan moved.** The skills' plan of 23 October now lives at [`.reference/plans/toghen-skills/plan-2026-10-23-figcraft-split.md`](.reference/plans/plan-2026-10-23-figcraft-split.md), beside the other packages' plans, and every pointer to a plan names its new path.

**What the docs contradict, left for Toghen's own docs:** `toghen-figma/specs.md:900` (a colour's alpha on a paint) still says a hand-edited opacity on a bound paint is drift, where the ruling of 24 October and `reference.md` (*The receipt*, line 1230) say a bound paint is compared by its variable only. The command's usage text (`cli/main.ts:44`) says component files run first, then artboards, and does not say they run in dependency order.

## Moved: a repository of its own, numbered from 0.1.0 (25 October 2026)

**Status:** done.

**The change.** The package left the Toghen monorepo for its own repository, [`Toghen/skills`](https://codeberg.org/Toghen/skills) on Codeberg, in one first commit. Its history before the split is not carried over. Its version starts again at 0.1.0; the last version before the split was 0.5.0 (the plugin). Links that crossed into other packages now point at their repositories.

**Why.** Ruled by Egildo on 25 October 2026: one repository per package, and a [house](https://codeberg.org/Toghen/house) repository for what they share (the constitution, the speckit preset, the Biome config, the purity check). The plan is in house, `.reference/plans/workspace/plan-2026-10-25-tooling-codeberg.md`.

**With it.** Biome lints and formats the code, `pnpm check` runs every gate, and speckit with the house preset runs the rounds. The plugin is numbered 0.1.0, and every pin names the new numbers, with the old ones beside them. Paths in the skills name the repository (`figma/guide.md`) instead of the monorepo folder.

## Checked again: `@toghen/figma` 0.1.10 (27 October 2026)

**Status:** done. The plugin is 0.1.1.

**Why.** The skills pinned figma 0.1.0, and ten releases followed in three days, each from a consumer's headless FigCraft run (`figma/decisions.md`, 25 to 27 October). Two of them add flags a pipeline uses: `--skip-unchanged` and `--prune`.

**How it was checked.** Each change was reproduced headless against `figcraft-cli` 0.10.0 with synthetic files when it was built; the usage text of 0.1.10 was printed and compared with `commands.md`; the new messages were read from the source and the tests.

**What changed in the skills.**

- `pipeline/commands.md`: the two flags in the usage, the flags table and the exit codes (a `--prune` dry run that lists anything exits 1); a quick check script with `--skip-unchanged`, and `--prune` on the full one.
- `pipeline/messages.md`: the hug-and-fill refusal, the `has no source` lines and the dry run's closing line, the `unchanged, its fingerprint the same` line, an artboard's `would create · would update · would delete`, and what the end of a pruning run says.
- `pipeline/SKILL.md`: the pins; the end of a run; the hug-and-fill refusal among the check's; the two dry runs and what each proves; a set or artboard with no source among the breaking steps.
- `figcraft/SKILL.md` and `toghen-target.md`: the pins, large files, and a table of what each release changed, with the warning that before 0.1.10 a stray reported removed in FigCraft stayed in the file.
- `figma/`: the Toghen lines name 0.1.10; the hug-and-fill refusal and FigCraft's instance sizing, neither measured in Figma.

**Kept as it was:** the Figma and FigCraft facts, pinned to 23 October 2026 and FigCraft 0.10.0; lib, postcss and query, unchanged since 24 October.
