# Toghen's FigCraft target, at `@toghen/figma` 0.1.20 (0.1.0 was 7.0.1 before the split)

What `toghen-figma` does when it writes into FigCraft, as an agent needs it to read a run and to know what will be refused. The commands, their order and the CSS and Figma sides are `toghen:pipeline`'s. The authority is Toghen's own `figma/specs.md` (*FigCraft*) and `reference.md` (*Into FigCraft*, *Headless*, *The lock*).

## Two ways in

| Flag | Into | Saved by | Needs |
|---|---|---|---|
| `--figcraft <port>` | the file the app shows when the run starts | the person | the app started with `--control <port>` |
| `--figcraft-file <path.figcraft>` | that file, headless (a new one when the path has none) | the run, once it completed | `figcraft-cli` 0.8.0 or later (0.15.1 or later only from figma 0.1.16), from `FIGCRAFT_CLI` or the PATH; the file's folder |

Both go on `variables`, `components` and `sync`, and on `lock`. They exclude each other. With either, `--library` and `--use` take a `.figcraft` file (*Libraries*, below).

- **Over the socket**, the run prints `Writing into "<file>" in FigCraft on port <port>.`
  - It pins that file for every batch: its tab's index, and the file by the `id` `doc.list` gives it, or by its name and path where it gives none.
  - If the file leaves its tab, the operation stops with `FigCraft no longer has "<file>" at the tab it was in …`, and every later request is refused before it is sent.
  - Nothing answering on the port is exit 1: `Nothing answered on 127.0.0.1:<port>: … Start FigCraft with --control <port>.`
- **Headless**, the run prints `Writing into <path> headless, with figcraft-cli <version>.`
  - On success it ends with `Saved <path>.`
  - A run that stopped partway saves nothing and says `<path> was not saved: the run did not complete.` The file on disk is as it was.
  - A dry run never saves.
  - A CLI older than 0.8.0 stops before opening anything. A missing binary says `<cli> was not found: put figcraft-cli on the PATH, or name it with FIGCRAFT_CLI.`
  - A folder that does not exist, for the file or for a `--library` the run builds, stops the run before it reaches FigCraft: `<flag> <path>: the folder <folder> does not exist, so the run could not save the file there; nothing was written`.
  - `initialize` is waited for 60 s, every later request 10 minutes. Past that the session and the run stop, exit 1: `<cli> mcp did not answer <command> within <s> s, so the session was stopped`. A line on the child's standard output that is not MCP stops them too: `<cli> mcp wrote what is not MCP on its standard output: "<line>"`.
  - The fonts are the machine's; `FIGCRAFT_NO_SYSTEM_FONTS=1` in the environment reaches the child.

## How a run writes

- **One operation, one `command.batch`**: one undo step in FigCraft, named after the operation, rolled back whole if a step fails.
- **Variables and styles.**
  - Read from `variable.list`, `style.list` and `text.fonts`; library copies (prefixed with the library's id) are left out, styles as variables.
  - Written with the `variable.*` and `style.*` commands.
- **Components and artboards.**
  - Read a level at a time with `page.list` and `node.get`.
  - Written by a component writer, one batch per step.
  - A new set is one batch: members built whole, combined, named, given properties and wired.
  - A member added to an existing set gets the set's properties and its wiring before it joins: FigCraft 0.10.0 gives a joining member none.
  - A library's component copies belong to no page, and are never read as the file's own.
  - A new artboard whose instances have slots takes two batches: the frame and its layers, then what fills the slots.
  - An instance's sizing is written after its properties (`instance.setProperties`), and again whenever a step writes them, since a variant switch resizes an instance.
  - Component files run in dependency order, each after the files defining the components it names; artboards last.
- **Write whole, then bind.** Lists with parts (corners, sides, paints, effects, grids) are written whole, then each part is bound.
  - A layer with a corner of its own binds each corner, never `cornerRadius`.
  - A layer with a side of its own binds each side, never `strokeWeight`.

## Libraries

| Command | Library file | Target |
|---|---|---|
| `variables \| components \| sync … --library <library.figcraft>` | built, published (`library.publish`, local), saved | enabled (`library.add`), that library's updates accepted, lock written, then saved last headless |
| `lock <library.figcraft> (--figcraft <port> \| --figcraft-file <target.figcraft>) [--dry-run]` | untouched | enabled, updates accepted, lock written |
| `components \| sync … --use <library.figcraft>…` | untouched | each library enabled and its lock entry refreshed, no update accepted; then the files resolve through the lock, a `modes` collection of the library included |

- **Both files at once.** The library is pinned by its path: `file.open` (headless, `open_file`) opens it or shows its tab, and the `index` it answers is that tab. The file is then known by its `id`. Live, a tab the run opens is left open. Headless, the library is a second document in the same `figcraft-cli mcp` session. Before figma 0.1.17 the tab was found by `doc.list`'s `path`, which `figcraft-cli` 0.18.1 gives no MCP call, so every headless `--library` run failed. A library file that does not exist is made, with no path until the run saves it; one the run made and did not save is closed. The save is checked by what `file.save` answers: a tab gone, or holding another file, by then stops the run with `FigCraft no longer has "<name>" at the tab it was in: it was not saved at <path>.`
- **Order.** The library is published and saved only after its build completed; the target is touched only after that. A failed build publishes nothing and writes no lock.
- **A dry run** of `--library` or `lock` builds, publishes, enables and accepts nothing; it reports what the target's `library.updates` lists from that library. It looks the library up by the id the target's lock holds, and by its path where the lock holds none, so a moved library is found where it is enabled: `Dry run: "<target>" has the library at <the path FigCraft lists>; a run would point it at <path>.` A dry run with `--use` neither enables nor refreshes, and resolves through the lock as it is.
- **The lock entry** is keyed by the library file's name without `.figcraft`, and holds `id` beside `components`, `styles` and `variables`, filled from the target's `library.list {assets}`, and no path: every run is given it again. An entry with a `path`, from figma 0.1.17 or earlier, is still read, and the path ignored and dropped on the next write. It lives in the root's data, kept in the record on the `-toghen` divider. The id is what makes a moved file the same library: `library.add` with the new path re-points it. FigCraft's own link to the library stays absolute, so a committed target still changes when a run on another machine points it at that machine's folder.
- **Keys** are `<library id>:<asset id>`. A component file's `(key)` is accepted only in that form. Imports go by `library.import` and `library.importVariable`; a library style is applied with `library.applyStyle`. A dry run imports nothing.
- **The relink.** A component the library made again is a new asset, not an update, so the target keeps the old copy. The next `--use` run swaps every instance Toghen made (`swap <layer>` in an artboard's plan, a dry run included) by `instance.swap`. A person's instances, which carry no stamp, are left. FigCraft's `library.swap`, which would move the person's too, is not used.
- **The library's own stamps travel** onto its copies and onto every instance in the target. Toghen never takes them for the target's.
- **A library collection's mode.** A `modes` line naming a collection no local one is named after is looked for among the `--use` libraries, as the one whose variables' ids in the lock begin with `<collection>/`. Found in the file and in a library, or in two libraries, it stops the build naming each. It is written by `library.setVariableMode {library, collection, mode, ids}`, which copies the collection in, linked and read-only, and checks the mode's name; FigCraft's refusal comes back as its own message, then `; this batch pinned a library collection's mode at <what>`. It is kept in `toghen.unreported` as `library:<library>/<collection>`, so a second run compares it and writes nothing. A dry run needs the library locked first. **Not yet run against a real FigCraft:** the parameters come from the 0.9.0 command list, the same in 0.18.1's, and the refusal's wording is not measured.
- **Not built:** team libraries (`library.publishToServer`, share links), a library's extended collections.

## The stamps

A stamp is what Toghen wrote, kept on what it wrote, so the next run can tell a hand edit from a change in the source.

- **Variables and styles hold no data in FigCraft.** Their stamps live together in the page data of a divider, `-toghen`, under the key `toghen`, keyed by object id.
  - The divider is found by its data, not its name. A dry run never makes it.
  - The record is written once, at the end of a run or when a run fails partway, as one undo step labelled *Toghen stamps*, and only when the run stamped something.
- **Layers keep their own stamp** in their layer data, under `toghen.stamp`.
- **What FigCraft keeps without reporting** is kept beside the stamp, under `toghen.unreported`, as the file declares it. A hand change to these is not seen:
  - property wiring and frame modes, a library collection's mode included;
  - normal against pass-through, corner smoothing, a line's cap, maximum lines, paragraph spacing and indent;
  - constraints where FigCraft leaves them out;
  - an exposed instance;
  - on a set, a slot property's description and settings, which FigCraft's slots do not hold.
- **Reused ids.** Since variable and style ids are reused, each record entry notes what it was written on. An entry that no longer matches its object is dropped.
- **A lost record** (the divider deleted, the *Toghen stamps* step undone, a killed run) makes the next run a first run. That run adopts existing objects by name and stamps them.

A paint bound to a variable is compared by its variable only: FigCraft paints it with the variable's value in the layer's mode, alpha included, so its opacity is not compared and a hand change to it is not drift. The opacity is still written at the defaults' alpha; whether FigCraft draws a member pinned to another mode with that opacity on top of the colour's alpha is not checked.

With stamps, a value changed by hand is drift: kept, and reported as `member "<m>", layer "<address>": "<field>" was changed in Figma since Toghen wrote it, and was left as it is; pass --overwrite-drift to write the file's value`. The wording names Figma in both targets. `--overwrite-drift` writes over it.

## What it builds

- **Variables:** collections, modes added and removed (never the default), the four types, values per mode, aliases, scopes, Web code syntax, renames, deletes.
- **Styles:**
  - text styles with family, style, size, line height and letter spacing bound;
  - effect styles with drop and inner shadows;
  - paint styles with gradients.
- **Component sets:**
  - members, and variant, boolean, text, instance-swap and slot properties;
  - wiring of `visible`, `characters` and `mainComponent`;
  - instances with their properties: a token drives a text or boolean property, and a swap goes by component id;
  - preferred values by name, from the file or a declared library, written as ids;
  - instances of a declared library's components, its variables bound and its text and effect styles applied;
  - an axis renamed, members added and deleted, layers added, replaced, moved, patched and deleted.
- **Artboards:** frames of layers and instances, pinned to modes (a `--use` library's collection included), with what fills their instances' slots, and each instance's size (`width`, `height`, the four limits, both sizings).
- **Fields:**
  - auto layout and grid layout with tracks and placement, wrap and content alignment;
  - sizing with min and max, absolute position, padding and gaps;
  - corners and per-side strokes, stroke align, join, cap and dashes;
  - paints with solid colours and gradients, effects, layout grids;
  - constraints, blend modes, opacity, visibility, rotation and position;
  - text fields and text styles, effect styles, explicit modes, slots, exposed instances.

## What it refuses by name

These are refused before anything is written, a dry run included. The run stops with `will not build "<component>": nothing was written` and one line each:

| Refused | The line says | Why |
|---|---|---|
| a component named by a Figma key | `a component named by a Figma key is not built in FigCraft, whose keys are <library id>:<asset id>; name the component, and declare its library with --use` | a Figma key names nothing in FigCraft |
| preferred values by `key=` | `the preferred values of "<property>" name Figma's published keys, which FigCraft does not hold; name the components with name=` | `name=` is resolved through the file or a declared library and written as ids |
| `textTruncation` | `textTruncation is not built in FigCraft, whose truncation is a text's sizing (textAutoResize "truncate"), not Figma's ending` | the two mean different things |
| the line cap `TRIANGLE_FILLED` | `the line cap TRIANGLE_FILLED has no match in FigCraft, whose one triangle is ARROW_EQUILATERAL` | no such cap |
| a text or boolean property whose default is a token | `the default of "<property>" is a token, which FigCraft does not bind to a property yet` | |
| any field outside the built table | `"<component>" <address>: the field <name> is not built in FigCraft yet` | not built yet |

The FigCraft target also refuses the operations that bind layers by convention or by the plugin's Apply (`BIND_VARIABLE`, `UNBIND_VARIABLE`, `APPLY_TOKEN`): `Binding layers is not built for FigCraft yet; the FigCraft target syncs variables and styles.` No command-line run plans them; layer bindings come from component and artboard files.

Not written, without a refusal:
- **A property's description:** FigCraft's properties have none.
- **A text style's weight bound to a token:** FigCraft binds no weight. The weight is written into the font style, and the style is planned as an update on every run.

## Reading a run's result in FigCraft

- `doc.list` and `page.list` show the `-toghen` divider beside the pages.
- `node.get` on a set, a member or a layer Toghen made shows `pluginData["toghen.stamp"]`. A layer may also show `toghen.unreported`.
- A second run with no change in the source plans nothing and writes nothing: `0 created · 0 updated · 0 deleted`, `0 would update` in a set's dry run, `Variable plan: 0 creates · 0 updates · <n> skips (<n> unchanged)`. Two causes of rewrites were found and closed in figma 7.0.0, both reproduced against `figcraft-cli` 0.10.0: a bound paint's alpha, which depends on the layer's mode (now compared by its variable only), and a bound size beside `HUG`, which FigCraft settles as fixed (now refused by `check`). Anything else after an unchanged source is drift or a FigCraft change worth reporting.

## Since 0.1.0

Up to 0.1.16, each was reported by a consumer's FigCraft run and reproduced with synthetic files: against `figcraft-cli` 0.10.0 up to 0.1.15, against 0.15.1 for 0.1.16. 0.1.17's came from the review of 23 November 2026; its library fix rests on what `figcraft-cli` 0.18.1 answers headless. 0.1.18's came from the skills' measurement of 25 November 2026 (`figma/decisions.md`, 25 October to 26 November 2026).

| Version | What changed | What an agent sees |
|---|---|---|
| 0.1.1 | A frame that hugs an axis every child in its flow fills is refused by the check: FigCraft sets the child back to `FIXED`, so it never settled. An instance set to `HUG` or `FILL` with no size of its own lets its main's bound size go first, since FigCraft keeps a bound size fixed | `FRAME "Face" fills the width of FRAME "Chip", which hugs it: …`; an instance that fills, filling |
| 0.1.2 | A slot's default content, stamped as the component's, is the component's, not the artboard's | No more removals planned for it on every run |
| 0.1.3 | The file is read once a run, and again only where a write reached | A large run in seconds or minutes, not an hour |
| 0.1.4 | An artboard's dry run counts its layers; a new instance whose slot the artboard fills can be added to an artboard that exists | `would create · would update · would delete` |
| 0.1.5 | A dry run takes a library's variable or style already copied into the file, as the real run would | No longer `115 would update` straight after a real run |
| 0.1.6 | `--skip-unchanged`: a fingerprint of what the run would write is stamped on each set and artboard; a run with the flag leaves an unchanged one unread | `unchanged, its fingerprint the same; not read`. A new version reads everything once |
| 0.1.7, 0.1.8 | The design system's sum and its snapshots are built once a run | A skipped set costs milliseconds, not a second, on a large document |
| 0.1.9 | A removed layer takes what it holds in one step; the writer passes over a layer already gone | A run that removes a frame with layers in it no longer stops on `no such node` |
| 0.1.10 | `--prune`; and the removals at the end of a run reach the file | Before 0.1.10, a stray the end of a run reported `removed` stayed in the file: the next run with `--allow-breaking` removes it |
| 0.1.11 | A guard written with no body by a part key's name is supplied; a token named outright that some members' parts replace is warned of | `warning: {…} names the rest's variable, which a member's parts replace: …` |
| 0.1.12 | A `SLOT` built into FigCraft no longer keeps the white fill a new frame starts with | A slot built before keeps its white until it is cleared by hand or the set is rebuilt |
| 0.1.13 | `textAlignHorizontal`, `textAlignVertical`, `textCase` and `textDecoration` reach FigCraft | Before, none was written, and every run planned the same patch again |
| 0.1.14 | A member binds the variable that carries what the precedence picks, folded parts included | A member whose folded part wins a tie binds that part's variable, and draws what the stylesheet draws |
| 0.1.15 | The stamp records the variables each field bound; a field whose variable the sync removed is written as moved, not kept as drift. A set's summary counts its drift | `· 3 drifted` on the summary line. A stamp from before needs one run with `--overwrite-drift` |
| 0.1.16 | The stamps are read with no `depth` | Before 0.1.16, every headless run against `figcraft-cli` 0.15.1 or later stops at reading the stamps: `unknown parameter "depth"` in `node.get` |
| 0.1.17 | A library's tab is the index its opening answers, and a file is known by its `id`. A missing folder stops the run before it starts. A headless session gives up after 60 s on `initialize`, 10 minutes on any other request. A set's dry run counts what it would create and delete. A set or artboard whose update fails loses its fingerprint | Headless `--library` runs against `figcraft-cli` 0.18.1, where before every one failed. `N would create · N would update · N would delete`. The next run reads a failed set or artboard whole, `--skip-unchanged` or not |
| 0.1.18 | The lock keeps a library's id and no path. A path in a message is as typed. A dry run finds a library by the lock's id, and names only the sets the restack would move | `Saved design/app.figcraft.`, not an absolute path. A moved library's dry run says `a run would point it at <path>`. After a real run, the next dry run has no `a run would restack` line |
| 0.1.19 | A first headless run names its new file as later runs do. `lock`'s dry run says what `lock` would do. A set's step count leaves out the re-layout. An entry lib could not read is named once | `Enabled design/kit.figcraft in "app"` on the first run, not `in "Untitled"`. `Dry run: "app" has 0 updates from design/kit.figcraft waiting; a run would accept them, then write the lock.` An unchanged set prints `Chip: 0 steps · 0 created · 0 updated · 0 deleted`, a first build still `1 step`. `the entry document "/…/tokens/nope.resolver.json" could not be read. ENOENT: no such file or directory` |
| 0.1.20 | A file of a list of token files that does not load is named once, on lib 0.2.2 | `the entry's token file "/…/tokens/nope.tokens.json" could not be read. ENOENT: no such file or directory`, where 0.1.19 named it three times |
