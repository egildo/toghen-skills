---
name: pipeline
description: Use when a repository's design tokens (DTCG token files, a resolver document) and component or artboard files (.component.kdl, .artboard.kdl) must become CSS, Figma variables, styles and component sets, or the same in a FigCraft file, through Toghen's toghen-figma command line. Covers the order of a run and why, what a token's value may hold, dry runs first (skips by cause, --verbose), pairing with the Figma plugin without blocking, FigCraft live and headless (--figcraft, --figcraft-file), libraries in both (--library, lock, --use), checks with no target, drift, --overwrite-drift and --allow-breaking, exit codes, what needs a person, and the assets Toghen does not make.
license: MIT
compatibility: "Documents Toghen @toghen/figma 0.1.10 (lib 0.1.0, postcss 0.1.0, query 0.1.0; before the split lib 6.0.0, postcss 0.21.0, query 0.2.0), checked 27 October 2026"
metadata:
  documents: "toghen"
  version: "0.1.10"
  checked: "2026-10-27"
allowed-tools: Bash Read Grep Glob
---

# Toghen pipeline: from a repository to CSS, Figma and FigCraft

## Pinned

| What | Version | Source |
|---|---|---|
| `@toghen/figma` (the `toghen-figma` command and the Figma plugin) | 0.1.10 (0.1.0 was 7.0.1 before the split) | `figma/package.json` |
| `@toghen/lib` | 0.1.0 (6.0.0 before the split) | `lib/package.json` |
| `@toghen/postcss` | 0.1.0 (0.21.0 before the split) | `postcss/package.json` |
| `@toghen/query` (`toghen-query`) | 0.1.0 (0.2.0 before the split) | `query/package.json` |
| FigCraft app and `figcraft-cli` | measured against 0.9.0 and 0.10.0; a headless run needs `figcraft-cli` 0.8.0 or later | `figma/decisions.md` (*What the build found: finishing the FigCraft target*, *headless FigCraft runs*) |
| Figma | the desktop app only; the browser app is not measured | `figma/decisions.md` (*Open questions › The command line*) |
| Checked | 27 October 2026, against figma 0.1.10: the usage text, `--skip-unchanged`, `--prune` and the messages added since 0.1.0 (`figma/decisions.md`, the entries of 25 to 27 October). Before that, 24 October 2026, against figma 7.0.0, now 0.1.0 (computed values, skips by cause, the new `check` refusals, components in dependency order) and query 0.2.0, now 0.1.0: the usage text, `check`, `members`, `toghen-query check` and the CSS build run on the example | `.reference/skills-research/pipeline-2026-10-23.md`; `.reference/plans/toghen-figma/plan-2026-10-23-figcraft-libraries.md`; `.reference/plans/toghen-lib/plan-2026-10-24-computed-values.md`; `.reference/plans/toghen-figma/plan-2026-10-24-issues.md` (each plan's *What the build found*) |

If the installed Toghen is newer, read `figma/cli/main.ts` (the usage text) and the guide before trusting a flag here.

Detail, read when needed:

- [commands.md](commands.md): every command and flag, exit codes, environment variables, npm scripts.
- [messages.md](messages.md): the messages a run prints, what each means, what to do.
- [example.md](example.md): a small synthetic system, `mini`, from files to CSS, Figma and FigCraft.

The other skills of this plugin: `toghen:figma` for Figma itself, `toghen:figcraft` for what a run does in FigCraft. FigCraft itself, the app, is `artcraft:figcraft`'s, in another marketplace (`egildo/skills`).

---

## What the pipeline is

One token source. Many targets. Every target reads the same `load()` from `@toghen/lib`, so none redefines a token.

```
tokens/*.tokens.json ─┐
tokens/*.resolver.json ┼─ @toghen/lib load() ─┬─ CSS             @toghen/postcss, in the design system's own build
--overrides map.json ─┘                       ├─ Figma file      toghen-figma ── ws://localhost:3055 ── Toghen plugin
components/*.component.kdl ─┐                 ├─ FigCraft, live  toghen-figma --figcraft <port>
docs/*.artboard.kdl ────────┤                 ├─ FigCraft file   toghen-figma --figcraft-file <path.figcraft> (no app)
--input data.json ──────────┘                 └─ checks          toghen-figma check / members, toghen-query (no target)
```

The formats are documented once, in Toghen. Read them there:

- Tokens, resolver, parts, vocabulary: `lib/model.md`, `lib/guide.md`, `lib/reference.md` (*load*).
- Component and artboard files: `figma/component-files.md`.
- The command line, the plugin, the stamps: `figma/guide.md` (*From the repository to Figma*, *Syncing from a repository*, *Syncing into FigCraft*), `figma/reference.md` (*The command line: toghen-figma*, *Into FigCraft*, *Headless*).
- CSS: `postcss/guide.md` (*Set up the pipeline*).

### What a token's value may hold

Lib stores every value as CSS computes it without an element (`lib/reference.md`, *Token values*, *Reference grammar*):

- **Colours:** all of CSS Color 4 (hex, named colours, `transparent`, `rgb()`, `hsl()`, `oklch()`, `color()` and the rest) and `color-mix()`, nested or over references. So a shadow whose colour is `color-mix(in srgb, {ink} 5%, transparent)` becomes an effect style in Figma and FigCraft.
- **Lengths:** `calc()`, `min()`, `max()`, `clamp()` and `round()` over `px`, `rem`, `pt` and `mm`; a mix of `rem` and absolute units is stored in `rem`. Composite members (shadow, typography, gradient, border) are stored computed too.
- **Refused with a sentence**, naming the token, the mode and the reason: what needs the element, such as `currentColor` inside a mix, `var()`, relative colour syntax, `light-dark()`, `em` outside a letter spacing, `%` and viewport units. A bare `currentColor` is kept, and Figma cannot hold it. In a plan such a token is one skip under `cannot be written` or `cannot be built`, carrying lib's sentence; in CSS, `onUnresolved` decides.
- **A literal brace is written doubled:** `{{size.sm}}` is the text `{size.sm}`, never the token. The same in a token's value, a data file's JSON and a component file's KDL. Still open: a data value that arrives as exactly one reference and lands on `characters` is read as a token reference.
- **Not computed yet:** `number`, `duration`, `cubicBezier` and `transition` tokens, stored as written; `color-mix()` without `in <space>` or with more than two colours.

---

## Set up once

1. **A Toghen checkout, built.** `pnpm install && pnpm build` at its root. The root build makes `@toghen/ui` before `@toghen/figma`, which needs it.
2. **A current Node first on `PATH`.** The repository is developed on Node 25. An old Node fails without saying why.
3. **In the design system's repository:** `@toghen/figma` is private, so depend on it as a linked checkout (`"link:../toghen/toghen-figma"`). Its command needs `tsx` and `esbuild` from that checkout. `@toghen/lib` and `@toghen/postcss` install by git URL with pnpm (`lib/guide.md`, *From the repository*). Pin them to the commit the checkout is at.
4. **For Figma:** in the desktop app, Plugins → Development → Import plugin from manifest… → `figma/dist/manifest.json`. **Build the plugin from the same checkout as the command.** Nothing checks that they match.
5. **For FigCraft headless:** `figcraft-cli` 0.8.0 or later, the same version as the app that opens the file. Put it on `PATH`, or name it with `FIGCRAFT_CLI=/path/to/figcraft-cli`.

---

## The order, and why

`toghen-figma sync` runs these steps, always in this order (`figma/cli/run.ts`):

1. **Read from disk.** Load the entry, read every `.kdl`, read `--input`.
2. **Reach the target.** Pair with the plugin, or connect to FigCraft.
3. **Load and check, writing nothing.** Every file is read and checked. With `sync`, every token and style a file names is held against the design system.
4. **Variables.** Plan, print, then execute.
5. **Styles.** Plan, print, then execute.
6. **Component files**, one at a time, in dependency order: each after every file that defines a component it names. Ties keep the order given; a name a `--use` library holds is no dependency. Files that name each other in a cycle stop the run before the variable plan.
7. **Artboard files**, one at a time.
8. **End of run.** Strays are removed (only with `--allow-breaking`), and with `--prune` the sets and artboards no file declares (only with `--allow-breaking` too), and Toghen's sets are stacked again on each page it touched.

Why:

- **Variables before styles.** A text style binds its size or font to variables. Without them, the style takes plain values.
- **Styles before components.** A component finds its text and effect styles by name, and its tokens by variable name (`space.sm` is the variable `space/sm`). A name found nowhere, or twice, stops the run before it writes.
- **The check before the variables.** The check tells the variable sync which components use `layout=parts`. A run that skips the check folds every component. A `layout=parts` file then fails with `names no variable`. **Use `sync`. Do not split it into `variables` then `components` when any file uses `layout=parts`.**
- **Components before artboards.** An artboard places instances of components. An artboard's `INSTANCE` takes its size (`width`, `height`, the four limits, `layoutSizingHorizontal`, `layoutSizingVertical`), written after its properties and again after a variant switch, which resizes an instance. Measured in FigCraft; written the same way in Figma, not measured there.
- **Library before target.** A target file can only use a library's keys once the library is published and its update accepted: in Figma by a person, in FigCraft by the `--library` run.

---

## The ways in

| Way in | Command | Needs open or running | Runs in CI? |
|---|---|---|---|
| **Checks, no target** | `toghen-figma check …`, `toghen-figma members …`, `toghen-query --doc … check`, the CSS build | Nothing | Yes |
| **Figma, paired** | `toghen-figma variables \| components \| sync …` | Figma desktop; the target file open; the Toghen plugin running in it; paired; port 3055 free; every `--page` existing | No |
| **FigCraft, live** | the same, with `--figcraft <port>` | FigCraft started with `--control <port>`; the target file shown in its front window; every `--page` existing; nobody else driving that instance | No |
| **FigCraft, headless** | the same, with `--figcraft-file <path.figcraft>` | `figcraft-cli` ≥ 0.8.0; the folder of the path existing | Yes |

### Checks, no target

Run these at every step of a change. They need no app.

```sh
toghen-query --doc tokens/mini.resolver.json check
toghen-figma check tokens/mini.resolver.json components/*.component.kdl docs/*.artboard.kdl --input docs/chip.data.json
toghen-figma members tokens/mini.resolver.json components/*.component.kdl > members.json
node scripts/build-css.mjs     # @toghen/postcss with onUnresolved: 'throw'
```

- `check` prints `<file>:<line>:<column> <message>` per problem, then `N files read: no errors`. It cannot see a Figma file: whether a variable exists, or exists twice, is the dry run's to say.
- It also refuses, each with the fix in its sentence:
  - a folded part's name, such as `{chip.hover.backgroundColor}`, and says which to use: `name {chip.backgroundColor}; the member pins the mode`;
  - `#null` on a `width` or `height` (a size cannot be cleared; write `HUG` or `FILL`). The four limits keep `#null`;
  - a `width` or `height` bound to a token, written or from `tokens=`, beside `HUG` or `FILL` on its axis, or beside a text's `textAutoResize` that sizes that axis. A bound size is a fixed size, and the target would settle on one of the two;
  - a frame that hugs an axis every child in its flow fills, which has nothing to size it by.
- `members` prints each file's component, members and problems as JSON, the members a build would make. Use it to draw members somewhere Toghen does not.
- `toghen-query … check` prints `incomplete:`, `holes:`, `non-factorable:` and `collisions:`. Each says `(none)` when clean. It exits 1 when `incomplete`, `holes` or `non-factorable` has entries. Collisions do not fail it: `load` reports them as advice, the last source winning.
- After the CSS build, search the output for `[object Object]` and an unresolved `t(` or `toghen(`. There should be none.

### Figma, paired

```sh
toghen-figma sync tokens/mini.resolver.json components/*.component.kdl --page Components --dry-run
toghen-figma sync tokens/mini.resolver.json components/*.component.kdl --page Components
```

The first run needs a person to pair. See *Pairing without blocking*.

### FigCraft, live

```sh
open -n -a FigCraft --args --control 7979     # a new instance; elsewhere than macOS: figcraft --control 7979
for i in $(seq 60); do nc -z 127.0.0.1 7979 && break; sleep 1; done   # wait up to a minute
nc -z 127.0.0.1 7979 || echo "FigCraft did not open port 7979"         # stop and tell the person
toghen-figma sync tokens/mini.resolver.json components/*.component.kdl --figcraft 7979 --dry-run
toghen-figma sync tokens/mini.resolver.json components/*.component.kdl --figcraft 7979
```

- The run writes into the file shown when it starts, and keeps writing there if someone looks at another tab.
- Each step is one undo step. The last is *Toghen stamps*.
- Closing a tab before the target during a run moves the target, and the run stops. The step it was on may have gone to the other file: undo it there.
- Always start a new instance with `-n`. Plain `open -a FigCraft --args …` drops the arguments when FigCraft already runs, and the port never opens. Never restart the person's instance.
- One program per FigCraft instance. Toghen pins every batch to its file, but another program's unpinned commands would land in whatever tab the run just showed, and the person's view jumps between tabs. Give the other program, or the person, a second instance on another port.

### FigCraft, headless

```sh
toghen-figma sync tokens/mini.resolver.json components/*.component.kdl --figcraft-file design/mini.figcraft --dry-run
toghen-figma sync tokens/mini.resolver.json components/*.component.kdl --figcraft-file design/mini.figcraft
```

- No app, no window, no person. The command starts `figcraft-cli mcp` itself.
- **Saved whole or not at all.** A dry run saves nothing. A run that stops partway saves nothing and exits 1: `design/mini.figcraft was not saved: the run did not complete.` A run that completes prints `Saved design/mini.figcraft.`
- A path with no file starts a new file. Its folder must exist, or the save fails and the run exits 1.
- A new file has one page, `Page 1`. Omit `--page`, or create the page in FigCraft once, save, and commit the file.
- Fonts are the machine's. `FIGCRAFT_NO_SYSTEM_FONTS=1` gives every machine FigCraft's own fonts only. It also narrows what `text.fonts` lists, which the style plan reads: a text style in a system font then falls back to a later family in its stack, or is skipped.
- Commit the saved file, or keep it as the job's artifact (`figma/guide.md`, *A FigCraft file with no app: continuous integration*).

### What FigCraft still refuses

FigCraft refuses these by name before it writes anything, the dry run included. The run stops with `will not build "<component>": nothing was written` and one line each:

- a component named by a Figma key (`(key)"…"`); FigCraft's keys are `<library id>:<asset id>`;
- preferred values by `key=` on an `INSTANCE_SWAP` or `SLOT` property: name them with `name=`;
- `textTruncation`, because FigCraft's truncation is a sizing mode;
- the line cap `TRIANGLE_FILLED`;
- a `TEXT` or `BOOLEAN` property whose default is a token;
- any field outside the built table: `"<component>" <address>: the field <name> is not built in FigCraft yet`.

Libraries are built in FigCraft: see *Libraries, in short*. Team libraries on a `figcraft-server` are not; a share link to `--library`, `--use` or `lock` is refused before anything runs.

FigCraft also has these known differences:

- It binds a text style's family, style, size, line height and letter spacing, but not its weight. A typography whose weight refers to a token is written again on every run. That is not a regression.
- It keeps some fields without reporting them, so Toghen keeps its own copy beside the stamp and reads that wherever FigCraft reports nothing: a layer's wiring to a property, a frame's pinned modes, corner smoothing, a line's cap, maximum lines, paragraph spacing and indent, an exposed instance, and a slot's description and settings. Blend mode and constraints are reported, except normal against pass-through and the constraints FigCraft leaves out. A hand change to what is not reported is not seen.

The list is `refuses` and `BUILT` in `figma/cli/figcraftApply.ts`, and `figma/specs.md` (*FigCraft*).

---

## Pairing without blocking

A Figma run waits for the plugin **with no time limit**. Never run it in the foreground and wait.

1. **Run it in the background.** Read its output as it comes.
2. **Wait for one of these lines:**
   - `Pairing code: ABCD-2345`: a person must type it (step 3).
   - `Looking for the plugin paired earlier on ws://localhost:3055…`: a plugin paired before may come back by itself, within 3 seconds.
   - `The plugin is paired.`: go on reading the plan.
3. **Hand the code over in one message**, in these words or close:
   > In Figma, open the file, run **Plugins → Development → Toghen**. On the **Load** pane, in the **Source** card at its foot, type `ABCD-2345` and press **Connect**.
4. **Wait for `The plugin is paired.`** Then read the rest of the output.

What to know:

- The pairing lasts until the plugin closes. Later runs find the plugin by themselves, with no code. Closing, rebuilding or reopening the plugin ends it.
- The command's hint names the same place: the Source card at the foot of the Load pane.
- With `--library`, the command prints two codes, one for the library file and one for the target file. Each goes into the plugin in its own file.
- **Only one run at a time.** A second run while one waits fails: `port 3055 is taken: …`.
- Ask the person not to press buttons in the plugin during a run. The command takes the first answer of the type it expects, and a click can produce one.
- A run never pairs by itself. Typing the code is the person's consent. Do not type it into their Figma unless they asked you to.

---

## Dry run first

Every write run has a dry run before it, with the same arguments plus `--dry-run`. There is no rollback. Toghen refuses everything it can foresee before its first write. If Figma refuses a write in the middle, what was written stays, and the next run completes it.

Read the dry run's output in this order:

1. **Problems** (on standard error, `<file>:<line>:<column> …`). Fix them first.
2. **`breaking:` lines.** Steps that would break instances or delete a mode. They are the person's call (next section).
3. **`drifted:` lines** for variables and styles, and **`left:` lines** for components (`"<field>" was changed in Figma since Toghen wrote it`). These are hand edits, kept unless the person decides otherwise.
4. **`warning:` and `skipped:` lines.** Font fallbacks, values Figma cannot hold, the mode limit. Skips are grouped by cause: every cause but `unchanged` lists its rows as `skipped: <reason>`, ten per cause, then `…and N more; --verbose lists them`. `--verbose` lists every skip, `unchanged: <path>` included, and every patch of a set as `patch <member> <address>: <fields>`.
5. **`pending:` lines.** Names a dry run counts as present because an earlier step would create them: variables, styles, modes, and the sets a component file earlier in the run builds. They are expected in a dry run.
6. **The counts.** `Variable plan: 3 creates · 0 updates · 239 skips (180 unchanged · 40 cannot be written · 19 unsupported type)`, most frequent cause first, then `modes`, `scopes`, `warnings` and `breaking` when not zero; the same for `Style plan:`. Then `<Component> (dry run): N steps · c created · n would update · d deleted`: the members a real run would write to. An artboard's dry run counts layers, `N steps · c would create · u would update · d would delete`, and lists them. A second run with nothing changed plans nothing: `0 would update`, in FigCraft too.
7. **With `--prune`, `breaking: the set … has no source`.** A set or artboard no file of the run declares. The dry run exits 1 for it. If its file was only left out of this run, add it back; if it was deleted, removing it is the person's call (`--allow-breaking`).

Two dry runs serve two purposes. Without flags it reads and compares everything, and finds hand edits. With `--skip-unchanged`, after one real run has stamped the fingerprints, it leaves alone each set and artboard whose inputs have not changed, and answers in seconds on a large file; it does not look for hand edits there.

What a dry run proves, and what it does not:

| Proves | Does not prove |
|---|---|
| Every file reads and checks, and every name resolves or is pending, a set a file earlier in the run builds included | That a published key imports (a dry run imports nothing) |
| The variable and style plans, drift, breaking steps, the mode limit | The properties and slots set on instances of a set the same run would create: they are checked on the run that builds it |
| Which font each text style would take on this machine | That Figma accepts every write |
| FigCraft's refusals | The look. Compare by eye or by screenshot |

**A first build is one run.** Component files run in dependency order, and a first-ever dry run counts each set a file earlier in the run would create as present. An instance of it is checked by name, and the dry run says once per component: `pending: … the properties and slots of instances of "<name>" are checked when it is built`. A name nothing would create still stops the dry run.

---

## Drift, `--overwrite-drift`, `--allow-breaking`

Toghen stamps everything it writes, and a second run writes only the difference.

- **Drift** is a field Toghen owns that someone changed by hand since Toghen wrote it. A run keeps the edit and reports it on every run. A field the file stops declaring keeps its value and is reported once.
- **A bound paint is compared by its variable only.** Its opacity is the variable's, in whatever mode the layer is in, so a hand change to a bound paint's opacity is not drift. An explicit paint opacity in the file is still compared.
- **`--overwrite-drift`** writes the source's value over **every** drifted value, scope, style and component field. All or nothing. It throws away designer edits.
- **Breaking** steps lose something in the file:
  - a member removed (a variant value removed or renamed);
  - a layer replaced by another kind;
  - a property removed or retyped;
  - a stray member removed;
  - an artboard layer removed (once, with what it holds);
  - with `--prune`, a set or artboard no file declares, removed at the end of the run; a page it leaves empty stays;
  - a context's mode deleted.

  Without `--allow-breaking`, a real run stops before writing. Instances of a removed member keep pointing at it in every file that uses the library. Someone must swap them.

**Both flags are the person's decision, never the agent's.** Run the dry run, then show the person the `breaking:` and `drifted:`/`left:` lines and ask:

> The dry run lists 2 breaking steps (the member "state=pressed" is gone…) and 1 hand edit ("paddingTop" in "Chip/state=rest"). Take the breaking steps with `--allow-breaking`? Keep the hand edit, or write over it with `--overwrite-drift`?

Never add either flag to make a run pass. Never put either in a default npm script.

---

## Exit codes, and how to read them

| Command | 0 | 1 | Other |
|---|---|---|---|
| `toghen-figma variables \| components \| sync` | Every step succeeded. A dry run with breaking steps listed still exits 0 | A check failed; component files name each other in a cycle; a plan stopped; a breaking step was refused; Figma or FigCraft refused a write; the plugin went away; a component was not built; a headless save failed | — |
| `toghen-figma check`, `members` | No file has an error (warnings do not count) | A file has an error, or cannot be read | — |
| `toghen-query` one query | An answer, `help` included; a `check` that lists only collisions | An `error: …`, a usage error, an unknown command, a `check` with `incomplete`, `holes` or `non-factorable` entries; a replayed `--session` that differs | 2: the document did not load. The prompt always exits 0 |

- **A failed component does not stop the others.** Look for `<file>: was not built:` or `was not planned:` lines, even when later components succeed.
- Problems go to standard error. Plans and reports go to standard output. Read both.

The full list of messages: [messages.md](messages.md).

---

## What needs a person

Hand each over in one short message that says what to do, where, and why the run waits.

- **Figma:** opening the desktop app, the file and the plugin; importing the plugin; typing the pairing code once per plugin session; not touching the plugin during a run.
- **Pages and files:** `--page` must name a page that exists (`no page named "Docs" in this file`). Toghen creates no page or file in Figma. FigCraft can (`page.add`, `file.new`), outside Toghen.
- **Libraries, in Figma:** publishing the library; enabling it in the target file; accepting its update. No plugin can do these. In FigCraft the `--library` run does all three.
- **Decisions:** `--allow-breaking` and `--overwrite-drift`; swapping the instances of a removed member.
- **Unblocking a refusal:**
  - a component set of the file's name that Toghen did not make: rename or delete it;
  - a set Figma reports broken: delete or rename the members it names;
  - two sets stamped as one.
- **Fonts:** installing every face the typography tokens name where Figma runs.
- **Export to DTCG:** the plugin's Sync pane, Export; a person downloads it. It writes the loaded document at one combination of contexts, with the file's stamps if `$extensions` is ticked. It reads nothing back from Figma, so the repository already holds everything in it but the stamps. There is no command (`figma/specs.md`, *Not in this design*).

Runs no Figma step can do headless. FigCraft files can: `--figcraft-file`.

---

## Libraries, in short

A library file holds the variables, styles and components; a target file uses them through its lock. Two runs: `--library` builds the library and fills the target's lock, then `--use` runs the target's own files against it.

**Figma:**

```sh
toghen-figma sync tokens/mini.resolver.json components/*.component.kdl --library "Mini Library" --dry-run
toghen-figma sync tokens/mini.resolver.json components/*.component.kdl --library "Mini Library"
# a person publishes the library, then accepts its update in the target file
toghen-figma lock "Mini Library"
toghen-figma components docs/chip.artboard.kdl --resolver tokens/mini.resolver.json --input docs/chip.data.json --use "Mini Library"
```

- Both files must be open, with the plugin running in each. Give the library's name exactly as Figma shows it.
- The run builds in the library file. The target file gets only the lock: the key of each set, style and variable.
- `lock` builds nothing. It copies the keys again and lists anything not yet published as it is.

**FigCraft**, live or headless; a `.figcraft` file names the library:

```sh
toghen-figma sync tokens/mini.resolver.json components/*.component.kdl --figcraft-file design/app.figcraft --library design/kit.figcraft --dry-run
toghen-figma sync tokens/mini.resolver.json components/*.component.kdl --figcraft-file design/app.figcraft --library design/kit.figcraft
toghen-figma components docs/chip.artboard.kdl --resolver tokens/mini.resolver.json --input docs/chip.data.json --figcraft-file design/app.figcraft --use design/kit.figcraft
```

- The run builds in `kit.figcraft` (made if it does not exist), publishes and saves it, then in the target enables it, accepts its updates and writes the lock entry `kit`. Headless, both files are open in one session and the target is saved last. Live, the target is the file FigCraft shows, and the person saves it.
- A dry run builds, publishes and accepts nothing, and says what updates wait. A failed build publishes nothing.
- `toghen-figma lock design/kit.figcraft --figcraft-file design/app.figcraft` does the target's half alone, after a library published by hand.
- Keys are `<library id>:<asset id>`. A Figma key in a component file is refused by name. Name components by name and declare the library.
- `toghen:figcraft` has the details: copies, the relink, what is not built.

**`--use`, in both:**
- Repeatable, on `components` and `sync`. In Figma the library's name as the lock holds it; in FigCraft its `.figcraft` file, which a real run enables and whose lock entry it refreshes first.
- Names (components, tokens' variables, text and effect styles) resolve through the overrides map, then a declared library through the lock, then the file. A name in both a library and the file stops the build. Without `--use`, nothing changes.
- When a library makes a component again, the next `--use` run swaps Toghen's instances to it (`swap <layer>` in an artboard's plan). A person's instances are left.
- `--use` and `--library` together are refused.

**A library collection's mode.** A component's or artboard's `modes` may name a collection of a `--use` library: one whose variables' ids in the lock begin with its name. Found in the file and in a library, or in two libraries, it stops the build naming each.
- FigCraft pins it with `library.setVariableMode`, which copies the collection in and checks the mode's name, and a second run compares it. A dry run resolves through the lock as it is, so lock the library before one. Not yet run against a real FigCraft: the command's parameters come from FigCraft's command list, and its refusal's wording is not measured.
- **Figma does not do it yet.** It refuses the pin, in a dry run as in a real one, until the desktop app measures it: `modes <collection>=<mode> (line <n>): a library collection's mode ("<collection>" in library "<library>") is not measured in Figma yet; pin it on FigCraft, or use a local collection`.

**Preferred values by name.** `preferred COMPONENT name="…"` and `preferred COMPONENT_SET name="…"`, beside `key=`, resolve like an instance swap's default (the file, or a declared library). FigCraft writes them as ids. Figma keeps `key=` and refuses `name=` by name until it is measured in the desktop app.

---

## Assets: what Toghen does not make

| Asset | Toghen does | The pipeline must |
|---|---|---|
| Fonts in CSS | Writes `font-family` stacks | Write `@font-face`, or embed fonts, in the design system's own build |
| Fonts in Figma and FigCraft | A text style takes the first family in the stack that the target lists with the style asked for. The plan says what it passed over. A style none of whose families loads is skipped. In Figma, any component with text also needs Inter Regular (unverified for FigCraft) | Install the faces where Figma runs. In FigCraft, check `text.fonts {family}`. Headless, use the machine's fonts, or `FIGCRAFT_NO_SYSTEM_FONTS=1`, which leaves FigCraft's own fonts only, so system-font styles fall back or are skipped |
| Icon fonts | Nothing in CSS. In Figma, an icon can be a `TEXT` layer holding the icon's character, with a text style in the icon face. That is not exercised yet (`ui/decisions.md`, *The icon font's cost*) | Write the glyph rules and embed the font in its own build. Install the face where Figma opens the file |
| Vector icons, SVG | No vector layer kind exists (`FRAME`, `SLOT`, `TEXT`, `INSTANCE`, `LINE` only) | Make each icon a component in the file first, then reference it: `INSTANCE Mark component=Icon`. Figma: another tool or a person. FigCraft: `file.import {path}` turns an SVG into a frame, then `component.create` |
| Raster images | Not supported; an image `background` is skipped into the notes | Place them outside Toghen. FigCraft: `image.place {path, parent, mode}` |

A component **set** that Toghen did not make, with the name of a component file, blocks that file. Name outside icons differently from your component files.

---

## Known limits, and what to do today

| Limit | Workaround |
|---|---|
| `layout=parts` is learned only from the check inside a run | Use `sync`, not `variables` then `components` |
| A first dry run checks instances of a set the same run creates by name only | Read the real run's report; a wrong property stops that build with its own message |
| Figma: a `--use` library collection's mode in `modes` is refused until measured | Pin it in FigCraft, or use a local collection |
| Figma: an instance's size after a variant switch, and a bound size beside `HUG`, are not measured (`check` refuses the second) | Look at the result in the desktop app |
| A data value that arrives as exactly one reference and lands on `characters` is read as a token reference, `{{…}}` included | Keep text around the braces, or put the text in the artboard file |

---

## The ten mistakes an agent makes most

1. **Blocking on pairing.** Run Figma commands in the background and hand the code over. After the first pairing, runs reconnect by themselves.
2. **Splitting the run.** `variables` then `components` with a `layout=parts` file, or `components` on `tokens=` files without `--resolver`, ends in `names no variable`. Use `sync`.
3. **Trusting a first dry run past its names.** It checks instances of a set the same run would create by name only, and it never imports a key. Read the real run's report too.
4. **Adding `--allow-breaking` or `--overwrite-drift` to get a green run.** They break instances or throw away edits. Ask the person.
5. **Writing to Figma another way.** No `use_figma` scripts, no REST, no set made by hand under a component file's name, no edited stamps, no rebuilt set. Toghen adopts nothing by name. A rebuilt set gets a new key and breaks every instance.
6. **Changing `--collection` between runs, or dropping it once.** A second base collection duplicates every base token. Later builds stop on `is ambiguous`.
7. **Expecting Toghen to make pages, files, fonts, icons or images.** `--page` must exist. Fonts must be installed or listed. Icons must already be components.
8. **Writing component files loosely.** A bare number or colour is refused: write a token, or `(literal)8` with a comment saying why. These are refused too:
   - a field beside `textStyle` that the style also sets;
   - `tokens=` on an `INSTANCE`;
   - `(property)` inside `set`;
   - the collection name in a token reference;
   - a folded part's name (`{chip.hover.backgroundColor}`): name the token it folds into, and let the member pin the mode;
   - `#null` on a `width` or `height`;
   - a bound `width` or `height` beside `HUG`, `FILL` or a text's `textAutoResize` on that axis;
   - a frame that hugs an axis every child in its flow fills: give the frame a fixed size, or a child a size of its own.

   A brace meant as text is doubled: `"{{space.sm}}"`.

   Run `toghen-figma check` after every edit (`figma/component-files.md`, *What is checked*).
9. **Misreading exit codes.**
   - `toghen-query check` passes with collisions listed.
   - `check` cannot see the Figma file.
   - A failed component does not stop the others.
   - A dry run exits 0 with breaking steps listed.
10. **Expecting FigCraft to match Figma.** Figma keys, preferred values by `key=` and `textTruncation` are refused, a `--library` run publishes and accepts where Figma needs a person, a library collection's mode is pinned in FigCraft and refused in Figma, and some fields are trusted, not read. Run one program per instance. Headless, nothing is saved unless the run completes.

Close behind:

- running two runs at once (port 3055);
- a plugin built from another checkout than the command;
- more than 10 contexts in a modifier, or a `figma.include` without the default;
- removing a context and expecting no breaking step.
