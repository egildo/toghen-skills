---
name: figcraft
description: Use when Toghen writes into FigCraft, the open-source Rust design app with Figma's model (github storytold/figcraft) — `toghen-figma variables | components | sync | lock` with `--figcraft <port>` or `--figcraft-file <path.figcraft>` — or when reading what such a run left behind: its messages, the `-toghen` divider, `toghen.stamp` layer data, drift, the lock, and what Toghen builds in FigCraft and refuses by name. Covers the pinned file, write-then-bind as Toghen meets it, headless runs in CI, FigCraft libraries (`--library <file.figcraft>`, `lock`, `--use`, keys, preferred values by name, a library collection's mode in `modes`), instance sizes on an artboard, and the few driving essentials a reader needs. For driving FigCraft itself, use `artcraft:figcraft`. Not the Figma plugin of the same name.
license: MIT
compatibility: Documents FigCraft 0.10.0 (app and figcraft-cli), docs tag v0.10.0, through Toghen @toghen/figma 0.1.0 (7.0.1 before the split); FigCraft checked 23 October 2026, Toghen 24 October 2026
metadata:
  documents: figcraft
  version: "0.10.0"
  checked: "2026-10-24"
allowed-tools: Bash Read
---

# Toghen into FigCraft

| Pinned | Value | Source |
|---|---|---|
| App | FigCraft 0.10.0 (22 October 2026) | `gh release list -R storytold/figcraft` |
| Command line | `figcraft-cli` 0.10.0; a headless run needs 0.8.0 or later | the same release |
| Docs | tag `v0.10.0` | `docs/` in the repository at the tag, never `main` |
| Toghen | `@toghen/figma` 0.1.0 (7.0.1 before the split): the FigCraft target with libraries, an unchanged run that writes nothing, a library collection's mode, instance sizes | `figma/specs.md`, *FigCraft*; `reference.md`, *Into FigCraft*, *Headless*, *The lock*; `.reference/plans/toghen-figma/plan-2026-10-23-figcraft-libraries.md`; `.reference/plans/toghen-figma/plan-2026-10-24-issues.md` (*What the build found*) |
| Checked | 23 October 2026, against a running 0.10.0 app and `figcraft-cli` 0.10.0, in files of the check's own. 24 October 2026, the Toghen side against figma 7.0.0's docs (now 0.1.0) and usage text, with no FigCraft run of the check's own | |

FigCraft runs no Figma plugin, so Toghen reaches it from its command line. It uses FigCraft's own control socket, or a headless `figcraft-cli mcp` with no app. The plans, the dry run, the stamps and the report are the Figma run's; only the reads and the writes are FigCraft's.

**Who owns what.**
- This skill covers Toghen's side: the two flags, what a run does, libraries, what it builds and refuses, and how to read what it leaves.
- The app itself belongs to `artcraft:figcraft` (marketplace `egildo/skills`): driving FigCraft, its commands and fields, and its traps.
- The commands of a whole pipeline and their order belong to `toghen:pipeline`.

## Safety

- **`--figcraft` writes into the file the app shows when the run starts.** Before running, make sure that file is the target. Never let it be a file the person has open for other work unless they said so.
- **One program per FigCraft instance.** Toghen pins its file in every batch, so its own writes land where they should. But another agent's unpinned commands then go to whatever tab Toghen brought forward. Run Toghen against an instance nobody else is driving, or a second instance on its own port.
- **Never quit or restart the person's instance.** It may hold unsaved work; start a second one with `--control <port>`. On macOS, `open -a` hands arguments only to a new instance, so use `open -n -a FigCraft --args --control <port>`.
- **Do not edit Toghen's data by hand:** the `-toghen` divider page, a layer's `toghen.stamp`, a layer's `toghen.unreported`. A lost record is not an error, but the next run becomes a first run.
- **Dry-run first.** `--dry-run` writes nothing and saves nothing, headless included.
- **`--library` saves the library file, live too.** The run publishes the library and saves it at its path, since FigCraft shows a library to other files only once it is saved. Name a library file that is the run's to save.

## Driving essentials

Enough to follow a run and check its result. The full material, with the field table, is in `artcraft:figcraft`.

**The ways in.**
- **The control socket.** The app started with `--control <port>` listens on 127.0.0.1 only. It takes one JSON object per line: `{"id", "method", "params"}`, answered by `{"id", "ok", "result" | "error"}`. `engine.execute {command, params}` runs any command; `engine.commands` lists them all, with a doc of their parameters.
- **MCP.** `figcraft-cli mcp --attach <port>` wraps the same socket. `figcraft-cli mcp` alone is a long headless session with no window: `run_command {command, params}` runs the same commands, `open_file` and `save_file` handle the file.
- **One-shot scripts.** `figcraft-cli run <script> [--in f] [--out f]` is all-or-nothing, and a failure loses the reads before it. Toghen does not use it.

**Versions.** Keep the app and `figcraft-cli` on one version. Before 0.8.0, `node.get` reports no layer data, so every stamp would read as absent.

**The pinned batch.** A command acts on the tab the window shows, never on a file it names, and tabs move. Every Toghen write is one request:

```json
{"command": "command.batch", "params": {"label": "<operation>", "commands": [
  {"command": "doc.switch", "params": {"index": <the target's tab>}},
  {"command": "command.batch", "params": {"commands": [<the steps>]}},
  {"command": "doc.list", "params": {}}]}}
```

Over the socket it also names the `window`. One batch is one undo step, rolled back whole when a step fails. `{"$step": n}` in a step's params is the `id` step n returned, counted within its own `command.batch`.

**Reading back.** The `node.get` command reads one level of children, whatever `depth` says, so Toghen walks a tree a level at a time. Layer data reads back as `pluginData: {"toghen.stamp": "…"}`. To look at a run's result yourself, run the commands in a batch pinned to the file, as above.

## What a run does

**`--figcraft <port>`, live.**
- It prints `Writing into "<file>" in FigCraft on port <port>.` and pins that file: the tab's index, name and path.
- Someone looking at another tab during the run sees the target come back, and the write still lands in it.
- If a tab before the target closes, the target moves and the run stops: `FigCraft no longer has "<file>" at the tab it was in …`. The step it was on may have gone into the file now at that index; undo it there.
- Nothing listening on the port stops the run with exit 1: `Nothing answered on 127.0.0.1:<port>: … Start FigCraft with --control <port>.`
- The person saves the file.

**`--figcraft-file <path.figcraft>`, headless.**
- It starts `figcraft-cli mcp`: the binary named by `FIGCRAFT_CLI`, else the one on the PATH. A version below 0.8.0 stops the run before anything is opened.
- It prints `Writing into <path> headless, with figcraft-cli <version>.`
- It opens the file, or starts a new one when the path has none; the folder must exist.
- **It saves whole or not at all.** `Saved <path>.` comes only after every step and the stamps. A run that stopped partway says `<path> was not saved: the run did not complete.`, exits 1, and leaves the file on disk as it was. A dry run never saves.
- **Fonts are the machine's.** `FIGCRAFT_NO_SYSTEM_FONTS=1` in the environment narrows them to FigCraft's own on every machine. It also changes what `text.fonts` lists, so a text style naming a system family falls back or is skipped.
- This is the CI route: Figma cannot run in CI, but a `.figcraft` file can be synced and committed.

**A dry run of a set** says `<Name> (dry run): N steps · c created · n would update · d deleted`, the members a real run would write to; `--verbose` adds `patch <member> <address>: <fields>` per patch. So a freshness check in CI can read `0 would update`.

**One operation, one batch.** A variable with its scopes, a style with its bindings, or a whole new component set lands as one undo step, or not at all. The last step of a run is *Toghen stamps*.

## Libraries

A FigCraft library is a `.figcraft` file, published, and enabled in another file by its path. Everything is local: the run calls no server. Toghen keeps the same model as in Figma (a library file, a target that uses it, a lock in the target), and the run does the steps a person does in Figma.

**The library run.** `--library` names the library's file; the target is the file FigCraft shows, or the `--figcraft-file`:

```sh
toghen-figma sync tokens/mini.resolver.json components/*.component.kdl --figcraft 7979 --library design/kit.figcraft
toghen-figma sync tokens/mini.resolver.json components/*.component.kdl --figcraft-file design/app.figcraft --library design/kit.figcraft
```

1. **Both files are pinned.** Live, the library is used in its tab if open, else opened in a tab and left open. Headless, both files are open in one `figcraft-cli mcp` session. A library file that does not exist yet is made. A target that is the library itself stops the run.
2. **The build goes into the library**: variables, styles, component files, as any run.
3. **The library is published** (`library.publish`, local) **and saved** at its path.
4. **In the target:** the library is enabled (`library.add`, which also re-points a moved file: the library's id lives in its file), that library's updates are accepted and no other's, and the target's lock is written from the target's own listing of it (`library.list {assets}`).
5. **The target is saved last**, headless, only when everything before completed. Live, the person saves it.

- **A dry run** builds, publishes, enables and accepts nothing. It says whether the target has the library enabled and what updates wait.
- **A failed build publishes nothing** and leaves the target untouched.
- **`lock`** does step 4 alone: `toghen-figma lock design/kit.figcraft --figcraft-file design/app.figcraft`. Use it after publishing by hand. It takes `--dry-run`.

**The lock in FigCraft** has the shape of Figma's (`reference.md`, *The lock*), kept in the record on the `-toghen` divider. Its entry is keyed by the library file's name without `.figcraft` (`kit`), and holds `path` and `id` beside the components, styles and variables.

**Keys** are `<library id>:<asset id>`: sixteen hex digits, then the library's own id for the asset (a node id, `VariableID:n`, `S:n`). A `(key)` in a component file is accepted in FigCraft only in that form; a Figma key is refused by name. A file meant for both targets names components by name.

**A run into a file that uses libraries.** `--use <library.figcraft>`, repeatable, on `components` and `sync`, declares the libraries the component and artboard files may take names from: components, tokens' variables, text and effect styles, through the target's lock.

```sh
toghen-figma components docs/*.artboard.kdl --resolver tokens/mini.resolver.json --input docs/chip.data.json --figcraft-file design/app.figcraft --use design/kit.figcraft
```

- Before resolving, the run enables each library and refreshes its lock entry, without accepting updates. A dry run does neither, and resolves through the lock as it is.
- A name in the file and in a declared library stops the build. Without `--use`, every name is found in the file, as before.
- `--use` and `--library` together are refused.

**Copies.** What a file takes from a library is a read-only copy: variables, collections and styles under the library's id as a prefix, components on no page. Toghen never reads a copy as the file's own.

**The relink.** When a library makes a component again (a new asset under the same name), FigCraft keeps the old copy and lists no update. The next `--use` run swaps each instance Toghen made to the new one: a `swap <layer>` line in an artboard's plan, a dry run included. A person's instances are left on the old copy.

**A library collection's mode.** `modes <collection>=<mode>` in a component or artboard file may name a collection of a `--use` library, found by the lock's variable ids. The run pins it with `library.setVariableMode`, which copies the collection in, linked and read-only, and checks the mode's name. FigCraft reports no pinned mode, so the pin is kept in `toghen.unreported` as `library:<library>/<collection>`, and a second run compares it. A dry run resolves through the lock as it is: lock the library before one. **Not yet run against a real FigCraft:** the command's parameters come from FigCraft's command list, and its refusal's wording is not measured. Figma refuses the same pin until it is measured there.

**Not built:** team libraries on a `figcraft-server` (`library.publishToServer`, share links). A share link to `--library`, `--use` or `lock` is refused before anything runs.

## Write, then bind

FigCraft unhooks a variable when a plain value is written to the bound field, so Toghen writes every value first and binds after.

- **Lists with parts are written whole, then each part bound.** That covers the four corners, the four stroke sides, paints, `effects` and `grids`, for example `fills/0/stops/1/color`, `effects/0/radius`, `layoutGrids/0/gutter`.
- **A corner of its own.** In FigCraft, a corner bound beside a bound `cornerRadius` does nothing. So a layer with a corner of its own has the four corners written as a list and each corner bound, never `cornerRadius`.
- **A side of its own.** Writing a side unhooks every side binding and a bound `strokeWeight`. So a layer with a side of its own binds each side, never `strokeWeight`.
- **A patch to one corner or side** writes again every corner, or every side, that the file declares.
- **Paints.** A paint goes out with a stand-in colour, then its colour is bound. A token below full alpha becomes the paint's opacity.

## Stamps, drift, and the `-toghen` divider

- **Variables and styles hold no data in FigCraft.** Their stamps live together in the page data of a divider page named `-toghen`, under the key `toghen`, keyed by object id.
  - FigCraft shows the divider as a line in the pages list and never opens it.
  - It is found by its data, so renaming or moving it changes nothing.
  - The first run that writes makes it; a dry run makes nothing.
- **A layer keeps its own stamp** in its layer data, `toghen.stamp`. A set, a member, an artboard and each layer Toghen made carry one.
- **What FigCraft keeps but never reports** is kept beside the stamp in `toghen.unreported`, as the file declares it. A hand change to these is not seen:
  - a layer's wiring to a property, a frame's modes;
  - normal against pass-through, corner smoothing, a line's cap, maximum lines, paragraph spacing and indent;
  - constraints where FigCraft leaves them out;
  - an exposed instance;
  - a slot's description and settings.
- **Ids can come back.** FigCraft reuses variable and style ids (the highest in use plus one), so each record entry notes what it was written on. An entry that no longer matches is dropped. Node ids never come back.
- **A bound paint is compared by its variable only.** FigCraft paints a bound colour with the variable's value in the layer's mode, alpha included, so its opacity is not compared, and a hand change to it is not drift.
- **Drift.** A value changed by hand is drift: kept, and reported on every run as `"<field>" was changed in Figma since Toghen wrote it, and was left as it is`. The wording names Figma in both targets. `--overwrite-drift` writes over it.
- **A lost record** (the divider deleted, the *Toghen stamps* step undone, a killed run) makes the next run a first run, which adopts existing objects by name.
- **An unchanged source plans nothing and writes nothing:** `0 created · 0 updated · 0 deleted` (`0 would update` in a dry run), `Variable plan: 0 creates · 0 updates · <n> skips (<n> unchanged)`. Reproduced against `figcraft-cli` 0.10.0, headless, on 24 October 2026; the first run after figma 7.0.0 may restamp bound paints once, writing nothing. Anything else after an unchanged source is drift, or a FigCraft change worth reporting.

## Built and refused

**Built:**
- **Variables:** collections, modes, the four types, aliases, scopes and Web code syntax.
- **Styles:**
  - text styles with family, style, size, line height and letter spacing bound;
  - effect styles of drop and inner shadows;
  - paint styles of gradients.
- **Component sets:** members, and variant, boolean, text, instance-swap and slot properties; wiring; instances and swaps. A member added to an existing set gets the set's properties and its wiring before it joins.
- **Libraries:** a library built, published and saved; enabled and its updates accepted in the target; the lock; components, variables and styles taken from a declared library; the relink; a declared library collection's mode pinned on a frame (not yet run against a real FigCraft).
- **Preferred values by name** (`preferred COMPONENT name="…"`, `preferred COMPONENT_SET name="…"`), from the file or a declared library, written as ids.
- **Artboards,** with what fills their instances' slots, and an instance's size: `width`, `height`, the four limits and both sizings, written after its properties and again after a variant switch. Measured in FigCraft 0.10.0: a switch puts a fixed axis back to the variant's size and a hug axis to its hugged size, a fill axis keeps its size, and writing `width` on a hugging instance fixes that axis.
- **Fields:** auto layout and grid layout with tracks, sizing, padding and gaps, corners and per-side strokes, paints with gradients, effects, layout grids, constraints, dashes, blend modes, lines and their caps, text fields and styles, visibility, modes, slots.

**Refused by name before anything is written,** a dry run included. The run stops with `will not build "<component>": nothing was written` and one line each:
- a component named by a Figma key; FigCraft's keys are `<library id>:<asset id>`;
- preferred values by `key=`, which names a published key: use `name=`;
- `textTruncation`, since FigCraft's truncation is a text's sizing;
- the line cap `TRIANGLE_FILLED`;
- a text or boolean property whose default is a token;
- any field outside the built table.

The variables run also refuses to bind layers by convention (`Binding layers is not built for FigCraft yet`). `--library`, `--use` or `lock` given anything but a `.figcraft` file is an argument error before anything runs. Not written, without a refusal: a property's description, and a text style's font weight bound to a token (FigCraft binds no weight, so that style is planned again on every run).

[toghen-target.md](toghen-target.md) has every message, the table of what is built, and the reasons.

## When FigCraft or Toghen moves

- **A FigCraft release:**
  1. Check the target against it in a file of your own, under the safety rules.
  2. Diff `engine.commands` against the last list.
  3. Run a sync twice, and a library run and a `--use` run twice: each second run must plan nothing.
  4. Update the pins and the date here.
- **A Toghen release** that changes the target updates *Built and refused* and `toghen-target.md`.
- **The app's own facts** are re-pinned in `artcraft:figcraft`.
