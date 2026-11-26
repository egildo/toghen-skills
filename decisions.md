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

## Corrected: no Biome and no `pnpm check` here (23 November 2026)

The entry of 25 October, *With it*, says Biome lints and formats the code and `pnpm check` runs every gate. Neither holds for this repository: it has no code and no `package.json`. Its gate is the purity check, `node scripts/check-purity.mjs`, which CI runs; `claude plugin validate` checks the marketplace and the plugin by hand.

## Checked again: figma 0.1.17, lib 0.2.0, postcss 0.2.0, query 0.1.1 (25 November 2026)

**Status:** done. The plugin is 0.1.2.

**Why.** The fixes of the review of 23 November 2026 (house, `.reference/plans/workspace/review-2026-11-23.md`) were released on 24 and 25 November: figma 0.1.17, lib 0.2.0, postcss 0.2.0 and query 0.1.1. FigCraft stays 0.18.1. The check of 23 November, at figma 0.1.16, left no entry here and no bump; its commit says what it changed.

**How it was checked.** By the routine in [specs.md](specs.md), everything read at its tag with `git show <tag>:<path>`: the decisions entries of 23 November 2026 in all four repositories, the usage text in `figma/cli/main.ts` and `query/src/main.ts`, the lines `figma/cli/*.ts` and `figma/main/engines/styleSyncEngine.ts` print, `figma/reference.md`, `postcss/reference.md` (*Messages*), `postcss/guide.md` and `query/guide.md`. No command was run, so step 2 of the routine is not done.

**What changed in the skills.**

- **The pins.** Every skill names figma 0.1.17; the pipeline also lib 0.2.0, postcss 0.2.0 and query 0.1.1, with the numbers before the split kept. Each *Checked* row adds this check and keeps the earlier ones.
- **Headless library mode is no longer a limit.** figma 0.1.17 finds a FigCraft tab by the index its opening answers, and knows a file by its `id`, so `--figcraft-file` with `--library` runs against `figcraft-cli` 0.18.1 (`figma/decisions.md`, *Library mode headless*; `cli/figcraft.ts`, `pinPath` and `savePinned`). The row in the pipeline's *Known limits* is gone. The sentences in `pipeline` and `figcraft` that said such a run fails now say it needs 0.1.17.
- **Two FigCraft lines mean something else.** `FigCraft did not open <path>.` still exists, but prints only when `doc.list` has no tab at the index FigCraft answered. `… it was not saved at <path>.` prints when the library's tab is gone, or holds another file, at its save.
- **Paths** are read from `process.cwd()`, not `INIT_CWD` (`cli/bin.mjs`). `commands.md` says where that is, and its `INIT_CWD` row is gone.
- **A missing folder** for `--figcraft-file`, or for a `--library` the run builds, stops the run before it starts, with its own sentence (`cli/run.ts`). Before, the save failed at the end.
- **A set's dry run** says `N would create · N would update · N would delete` (`cli/run.ts`), in `pipeline`, `messages.md` and `figcraft`.
- **New lines in `messages.md`:**
  - the headless waits, 60 s and 10 minutes, and a standard output that is not MCP (`cli/figcraft.ts`);
  - a plugin window another one replaced (`cli/bridge.ts`);
  - a literal `fontWeight` a text style cannot hold (`main/engines/styleSyncEngine.ts`), which `example.md` now expects too;
  - postcss's three new reports, which a build under `'throw'` can now fail on (`postcss/reference.md`, *Messages*).
- **`toghen-figma --help`** exits 0 (`cli/main.ts`).
- **`toghen-query`.** An `across` with a row the core threw for shows the core's error on that row and exits 1. Flags may follow the query (`query/guide.md`, *Three ways in*).
- **The Lightning CSS order rule**, unit D of house's `.reference/plans/workspace/plan-2026-11-23-lightningcss.md`: two bullets in `commands.md`, *The CSS build*, taken from `postcss/guide.md` (*When Lightning CSS runs after the plugin*). Toghen runs before Lightning CSS, and a Vite 8 build excludes `Features.LightDark`.
- `figcraft/toghen-target.md` gains a 0.1.17 row in *Since 0.1.0*.

**What the check found.**

- The skills said a usage error of `toghen-query` exits 1. At 0.1.0 it exited 2, since `main` rejected; 0.1.1 makes it 1, so the sentence is true now.
- `toghen-query check` still prints four sections. Lib 0.2.0's `sameCssName` is not among them.

**Kept as it was:** the Figma and FigCraft facts, and lib's value rules in the pipeline (*What a token's value may hold*), which the fixes of 23 November do not change.

## Measured: the skills against the released tools (25 November 2026)

**Status:** done. The plugin is 0.1.3.

**Why.** The check that made 0.1.2 read the sources at their tags and ran nothing, so step 2 of the routine in [specs.md](specs.md) was open. This entry closes it, for everything but Figma.

**How it was measured.** The worked example was rebuilt from `example.md` in a scratch folder. figma 0.1.17, query 0.1.1, postcss 0.2.0 and lib 0.2.0 were installed from npm. FigCraft ran headless only, against `figcraft-cli` 0.18.1, with `FIGCRAFT_NO_SYSTEM_FONTS=1`. Figma and the FigCraft app were not run.

**What held.**

- `toghen-figma --help` exits 0, and paths are read from the directory the command runs in.
- `check`, `members` and `toghen-query check` give what `example.md` expects, including the broken copy's three errors.
- `toghen-query`'s `across`, flags after the query, and every exit code in `commands.md`; `--session` and the prompt were not run.
- The CSS build matches the example.
- The dry run's counts, the styles warning, and the `fontWeight` warning for both `type/label` and `chip/font`.
- A second run plans nothing and saves the same bytes.
- A missing folder stops the run before it starts, for `--figcraft-file` and for `--library`.
- **Headless library mode works at figma 0.1.17**: `kit.figcraft` is published and saved, `app.figcraft` uses it and is saved, and a rerun changes neither.
- Lightning CSS run first turns `t(palette.orange.500)` into `t(palette.orange.5)`. Toghen then reports it not found, or reads `palette.orange.1` with no error where that token exists.

**What the skills said wrongly, now corrected.**

- `Saved <path>.` prints the path absolute.
- A `left:` line is not only a hand edit: every report note prints as `left:`, such as a key of a `tokens` group a layer has no field for.
- With `preserve: true`, each `@apply` stays in the output even in the right order. The sign of the wrong order is an `@apply` with nothing expanded before it.
- `toghen-query --tokens` takes one file. Each file needs its own `--tokens`.
- `toghen-query css` wants the collection first; without it the answer is wrong with no error.
- `example.md`: `chip/font` is built but bound by nothing, since the Caption binds `type/label`; the artboard's frame is `Page`; the header now says what ran on 25 November.
- `messages.md` gains the `pending: modes …` line.

**What the measurement found in the tools.** These are behaviours of figma and query, not of the skills. They are reported to those repositories rather than taught here:

- A FigCraft library's lock entry, inside the target file, holds the library's path absolute. Whether a run on another machine, where that folder does not exist, still finds the library by its id was not measured.
- A missing entry document's message repeats the path: `the entry document "<path>": "<path>" could not be read`.
- A dry run after a real run says `a run would restack "Chip", "Page"`, and the real run moves nothing.
- The library lines mix absolute paths (Published, Enabled, Saved) with relative ones (Building, Using).
- `toghen-query`'s usage text, `--tokens <file>…`, reads as several files after one flag.
- `toghen-query css surface.page` answers `--page` with no error.
- `toghen-query check` passes a reference to a token that does not exist, `{nope}`, and exits 0.

## Measured again: figma 0.1.18, lib 0.2.1, query 0.1.2 (26 November 2026)

**Status:** done. The plugin is 0.1.4.

**Why.** The fixes the measurement of 25 November 2026 asked for were released on 26 November: figma 0.1.18, lib 0.2.1 and query 0.1.2. The plan is house's `.reference/plans/workspace/plan-2026-11-25-measurement-fixes.md`, ruled "1b 2a 3a 4a 5a". postcss stays 0.2.0, FigCraft 0.18.1.

**How it was checked.** By the routine in [specs.md](specs.md). Read at each tag: the entries *Fixed and built: what the measurement of 25 November 2026 found* in figma, lib and query; `figma/reference.md` (*The command line*, *Headless*, *Libraries in FigCraft*, *The lock*); `lib/reference.md` (`check()`, `cssPropertyNameOf`, *load*); `query/guide.md` and `query/specs.md`; and the diffs of `figma/cli/` and `query/src/` since the last tags. Then the worked example was copied to a fresh scratch folder, and the four packages installed from npm; `npm ls` showed one lib, 0.2.1. FigCraft ran headless only, against `figcraft-cli` 0.18.1, with `FIGCRAFT_NO_SYSTEM_FONTS=1`. Figma and the FigCraft app were not run.

**What held.**

- `check`, `members`, the broken copy's three errors, and the CSS build, with no `[object Object]` and no `t(` left.
- The dry and real runs, twice. The second plans nothing and saves the same bytes.
- Library mode: `kit` twice, `app` twice, and `lock`. Each second run changes neither file.
- A missing folder stops the run before it starts, dry runs included, for `--figcraft-file` and for `--library`.
- `--tokens` twice reads both files. A second file after one `--tokens` is still `unknown command`, so that note stays.

**What changed in the skills, and the line that showed it.**

- **`Saved <path>.` is as typed again:** `Saved design/mini.figcraft.` The correction of 25 November is undone, in `pipeline`, `messages.md` and `example.md`.
- **Paths in messages.** `messages.md` states the rule once: a typed path prints as typed, and a path nobody typed prints as it is.
- **The lock holds the id and no path:** the entry read back `{"id":"…","by":"build"}`. A target whose lock figma 0.1.17 wrote kept its `path` until the first `--use` run, which dropped it. In `figcraft` and `toghen-target.md`.
- **A moved library**, with its old file gone: `Dry run: "app" has the library at /…/kits/kit.figcraft; a run would point it at lib2/kit.figcraft.` `lock --dry-run` says the same. The real run pointed it there and changed nothing else. In `pipeline`, `messages.md`, `figcraft` and `toghen-target.md`.
- **The restack.** A dry run after a real run has no `a run would restack` line, and a file 0.1.17 built has none either. The line that names the sets a run would move is read from figma's reference, not measured. In `messages.md`, `example.md` and `toghen-target.md`.
- **`toghen-query check`** prints `same-css-name:` and `unresolvable:`, and exits 1 on either. Measured: `--space-a-b: space.a.b, space.a-b`, `color.broken in default: {nope} (not-found)`, `shadow.card in default: {color.missing} (not-found)` and `loop.a in default: {loop.a} (circular)`. In `pipeline`, `commands.md`, `messages.md` and `example.md`.
- **`toghen-query css`** takes the path with or without the collection: `css surface.page` and `css mini.surface.page` both answer `--surface-page`. `css nope.nothing` and `css surface` exit 1 with lib's error. The correction of 25 November is undone, in `commands.md` and `messages.md`.
- **The usage** reads `--tokens <file> [--tokens <file> …]` (`commands.md`).
- **Read errors.** `components/nope.component.kdl cannot be read: ENOENT: no such file or directory`. `messages.md` gains rows for a missing or broken entry document, in figma and in query.
- The pins, dates and *Checked* rows in every skill. The figma skill's Toghen line names 0.1.18. `toghen-target.md` gains a 0.1.18 row.

**What the measurement found in the tools.** These are behaviours of figma and query, reported to those repositories rather than taught here as faults:

- figma's line for a missing entry still holds its path twice, as typed and then as lib's absolute location: `tokens/nope.resolver.json: the entry document "/…/tokens/nope.resolver.json" could not be read. ENOENT: no such file or directory`. Each copy follows the rule of question 2a; query's line holds it once.
- A dry run with nothing waiting says `Dry run: "app" has 0 updates from design/kit.figcraft waiting; a run would publish the library first, then accept them.` `lock --dry-run` prints the same line, though `lock` publishes nothing.
- A `--doc` that does not load prints the usage after the error, though the arguments were right. It exits 2, as at 0.1.1.
- Still as the plan noted: an unchanged run prints `Chip: 1 step · 0 created · 0 updated · 0 deleted`, and a first headless library run names its target `"Untitled"`.

## Measured again: figma 0.1.19, query 0.1.3 (26 November 2026)

**Status:** done. The plugin is 0.1.5.

**Why.** The five leftovers that *Measured again: figma 0.1.18, lib 0.2.1, query 0.1.2* found in the tools were fixed and released the same day: figma 0.1.19 and query 0.1.3. The plan is house's `.reference/plans/workspace/plan-2026-11-26-leftovers.md`, ruled "1a 2a". lib stays 0.2.1, postcss 0.2.0, FigCraft 0.18.1.

**How it was checked.** By the routine in [specs.md](specs.md). Read at each tag: figma's *Fixed and built: the leftovers of the measurement of 26 November 2026* and query's *Fixed: the usage after a document that does not load*, with the diffs of `figma/cli/` and `query/src/main.ts` since the last tags. Then the worked example was copied to a fresh scratch folder, with no FigCraft file in it, and the packages installed from npm; `npm ls` showed one lib, 0.2.1. FigCraft ran headless only, against `figcraft-cli` 0.18.1, with `FIGCRAFT_NO_SYSTEM_FONTS=1`. Figma and the FigCraft app were not run.

**What changed in the skills, and the line that showed it.**

- **A missing entry is named once**, in lib's line alone: `the entry document "/…/tokens/nope.resolver.json" could not be read. ENOENT: no such file or directory`, from `check` and from a `sync` dry run, exit 1. Any other load error still starts with the entry as typed. In `messages.md`.
- **query prints no usage after a document that does not load:** `error: the entry document "/…/tokens/nope.resolver.json" could not be read. ENOENT: …`, exit 2, and nothing after it. In `messages.md`.
- **A set's step count leaves out the re-layout:** a second run prints `Chip: 0 steps · 0 created · 0 updated · 0 deleted`, as the artboard's `Chip reference: 0 steps · …`. A first build still prints `Chip: 1 step · 3 created …`. In `messages.md`, `figcraft` and `toghen-target.md`.
- **`lock --dry-run`** says `Dry run: "app" has 0 updates from design/kit.figcraft waiting; a run would accept them, then write the lock.`, then `Dry run: the lock in the target was not written.` A library run's dry run keeps both of its lines. In `messages.md`.
- **A first headless run names its new file as later runs do:** its dry run said `Dry run: design/kit.figcraft is not enabled in "app"` and left `design/` empty, and its real run `Enabled design/kit.figcraft in "app", with no update to accept.` and `The lock in "app" holds "kit" …`. No line said `Untitled`. No skill had taught the old name. In `toghen-target.md`.
- The pins, dates and *Checked* rows in every skill. `toghen-target.md` gains a 0.1.19 row.

**What the measurement found in the tools.** Nothing new. Two notes stay in figma's own `decisions.md`, under *Noticed, not fixed*: the bridge test that flakes, and a list of token files with one missing, whose line still holds paths twice.

## Measured again: figma 0.1.20, lib 0.2.2 (26 November 2026)

**Status:** done. The plugin is 0.1.6.

**Why.** The last note of *Measured again: figma 0.1.19, query 0.1.3*, a list of token files with one missing, was fixed and released the same day on Egildo's "Fix the token-file list too": lib 0.2.2 and figma 0.1.20. query stays 0.1.3 and takes lib 0.2.2 by its range. postcss stays 0.2.0, FigCraft 0.18.1.

**How it was checked.** By the routine in [specs.md](specs.md). Read at each tag: the entries *Fixed: a file of a list of token files named once* in lib's and figma's `decisions.md`, and the diffs of `lib/src/resolver/load.ts` and `figma/cli/run.ts`. Then the packages were installed from npm into a fresh scratch folder; `npm ls` showed one lib, 0.2.2. FigCraft ran headless only, a `sync --dry-run --figcraft-file`, which wrote nothing. Figma and the FigCraft app were not run.

**What changed in the skills, and the line that showed it.**

- **A file of a list of token files that does not load is named once**, in lib's line alone: `the entry's token file "/…/tokens/nope.tokens.json" could not be read. ENOENT: no such file or directory`, from figma's `check` and a `sync` dry run, exit 1, and `… is not JSON. …` for one that is not JSON. query's `--tokens` prints the same after `error: `, exit 2. 0.1.19 named the file three times and cited `tokens.resolver.json`, lib's wrapper, which is on no disk. In `messages.md` and `toghen-target.md`.
- The pins, dates and *Checked* rows in every skill. `toghen-target.md` gains a 0.1.20 row.

**What the measurement found in the tools.** Nothing new. lib's own `decisions.md` notes three messages left as they are: a listed resolver document and a listed file that holds no group still cite the wrapper, and a mapped reference names its location twice.
