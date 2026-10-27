# Messages a run prints, and what to do

*Pinned to `@toghen/figma` 0.1.10 (lib 0.1.0, postcss 0.1.0) and `@toghen/query` 0.1.0 (lib 6.0.0, postcss 0.21.0 and query 0.2.0 before the split), checked 27 October 2026; lib, postcss and query as of 24 October 2026. Text in `<angle brackets>` varies. Sources: `figma/cli/*.ts`, `figma/main/components/resolve.ts`, `figma/reference.md`, `postcss/reference.md` (*Messages*).*

Problems go to standard error, plans and reports to standard output. A message that ends a run is marked **stops**.

## Arguments and files

| Message | Meaning | Do |
|---|---|---|
| `no entry given: a resolver document or token files (.json)` **stops** | No `.json` argument | Add the entry |
| `components takes the design system with --resolver; <x> is not a component file` **stops** | A `.json` given to `components` | `--resolver <x>`, or use `sync` |
| `variables takes the entry and nothing else; sync runs component files too` **stops** | `.kdl` given to `variables` | Use `sync` |
| `no component file given; variables syncs the tokens alone` **stops** | `sync` with no `.kdl` | Use `variables`, or add files |
| `--input is read only when an artboard file (<name>.artboard.kdl) is among the files; none is` **stops** | | Drop `--input`, or add the artboard |
| `--library takes the library's .figcraft file with --figcraft or --figcraft-file; got <x>` / the same with `--use` or `lock` **stops** | In FigCraft a library is its `.figcraft` file | Name the file |
| `--library <x> names a team library on a figcraft-server, which reaches outside this machine and is not built: name a .figcraft file` / the same with `--use` or `lock` **stops** | A share link; team libraries are not built | Name a `.figcraft` file |
| `--use and --library do not go together: --library runs the files into the library, --use runs them into a file that uses libraries` **stops** | | Two runs: the library first, then the file that uses it |
| `lock takes the library's .figcraft file, and nothing else` / `lock takes the library's name, as Figma names it, and nothing else` **stops** | | One library per `lock` |
| `lock --dry-run is FigCraft's: in Figma, lock writes nothing a dry run could hold back` **stops** | | Drop `--dry-run` |
| `--figcraft and --figcraft-file do not go together: …` **stops** | | Choose the app or the file |
| `--figcraft-file takes a .figcraft file, which the run saves; got <path>` **stops** | | Use a `.figcraft` path |
| `unknown option <flag> for <command>` / `<flag> needs a value` **stops** | | Fix the arguments |
| `--overrides <file> cannot be read as JSON: …` / `… must hold one JSON object: …` **stops** | | Fix the map |
| `<file> cannot be read: …` **stops** | | Fix the path; paths are relative to where the command was typed |
| `the component files name each other in a cycle, so none of them can be built first: a.component.kdl → b.component.kdl → a.component.kdl; nothing was written` **stops** | Files run in dependency order, and these cannot | Take one of the instances out |
| `<entry>: $ref "<ref>" in "<file>" …` **stops** | `load` could not follow a reference: a missing or non-JSON file, a pointer to nothing, a cycle, token files with no common folder and no `--name` | Read the message; it names the reference and its file |

## Pairing (Figma)

| Message | Meaning | Do |
|---|---|---|
| `Pairing code: <XXXX-XXXX>` then `Open the Toghen plugin in the target file, type the code into the Source card at the foot of its Load pane, and press Connect.` then `Waiting for the plugin on ws://localhost:3055…` | The run waits for the plugin, with no limit | Hand the code over: Load pane, Source card at its foot, field *Code from toghen-figma*, **Connect** |
| `In the library file "<n>", enter <code>; in the target file, enter <code>.` | A `--library` run: two codes | Each code into the plugin in its own file |
| `Looking for the plugin paired earlier on ws://localhost:3055…` | A kept code; 3 seconds' grace | Wait |
| `The plugin is paired.` / `Both plugins are paired.` | | Read on |
| `port 3055 is taken: another toghen-figma, or another program, is listening on it. …` **stops** | One run at a time | Stop the other run |
| `5 connections gave a wrong pairing code, so the run stopped. …` **stops** | | Check what else uses port 3055; run again |
| `the plugin disconnected: it was closed, or Figma lost it` / `the plugin is not connected` **stops** | Earlier writes stay | Reopen the plugin, pair, run again; the next run completes the writes |
| Plugin screen: `Nothing answered on localhost:3055 (tried at <time>). Run toghen-figma; this keeps trying.` | No run is waiting | Start the run |
| Plugin screen: `The command line did not accept that code (<time>). Type the one it printed.` | A code from before the plugin closed | Type the code the current run printed |

## The check (`check`, and step 3 of every run)

Each problem is `<file>:<line>:<column> <message>`; a warning is `<file>:<line>:<column> warning: <message>`. Summary: `N files read: no errors` or `N files read: K errors[, M warnings]`. Any error **stops** a run before it writes.

| Message | Do |
|---|---|
| `token {space.xl} is not in the design system "mini"` | Fix the reference or add the token |
| `"width" takes a token reference or a (literal) number: a bare 12 would be a number nobody chose, write (literal)12 or a token reference` | Write a token, or `(literal)12` with a comment. On a limit (`minWidth` and the rest) the sentence adds `, or #null` |
| `"width=#null": a width cannot be cleared; to let the layer size itself, write layoutSizingHorizontal=HUG (or FILL)` | Write the sizing; `#null` is for the four limits |
| `token {chip.hover.backgroundColor} is the "hover" part of "chip.backgroundColor", which is one variable with a mode per "state": name {chip.backgroundColor}; the member pins the mode (a tokens= axis, or modes state=hover)` | Name the token it folds into |
| `FRAME "Chip": "height" is bound to {chip.height} while layoutSizingVertical=HUG: a bound height is a fixed size; write layoutSizingVertical=HUG and drop the height token, or keep the token and drop HUG` | Choose one. Beside a text's `textAutoResize` the sentence ends `or keep the token and write textAutoResize=NONE` (`HEIGHT` for a width) |
| `component tokens="{nothing}" names no token and no group of the design system` | Fix the group |
| `FRAME "Face" fills the width of FRAME "Chip", which hugs it: a frame cannot hug children that only fill it; give "Chip" a fixed width, or a child a width of its own` (or `height`; several children: `FRAME "A", FRAME "B" fill …`) | A hugging frame whose every child in its flow fills it has nothing to size it by, and never settles in FigCraft. Do as it says |
| `VARIANT "State" takes its values from the modifier "state", which the design system does not have` | Remove the `(modifier)` axis; states are parts (`figma/guide.md`, *Move a component file off state modifiers*) |
| `… is not one Figma gets` | A context left out of `figma.include` is named in an axis, a guard, `exclude` or `modes` |
| `breaking: "<name>" (<id>) is a member outside any set, left by an earlier run; it is removed with --allow-breaking.` | A stray. Stops a real run without `--allow-breaking`; the person decides |
| `breaking: the set "<n>" (<id>) on page "<p>" has no source: no file in the run declares component "<n>"; it is removed with --allow-breaking.` (or `the artboard …`, `declares artboard …`) | `--prune` only: its file is gone. Stops a real run without `--allow-breaking`; a dry run lists it and exits 1. If the file was only left out of this run, add it back |
| `one set or artboard has no source; a run with --prune and --allow-breaking removes it.` / `N sets and artboards have no source; …` (exit 1) | The end of a `--prune` dry run that listed any. The person decides |
| A broken set: its members at fault, with ids, `ours` or `made by hand`, ending `delete or rename the members named, then run again` | A person fixes the set in Figma |
| `warning: … wired to nothing …` | A property that does nothing in some member. Never stops anything; read it |

The whole list of checks: `figma/component-files.md` (*What is checked*).

## Variables and styles

The plan prints `Variable plan: <c> creates · <u> updates · <s> skips (<n> <cause> · …)[ · modes · scopes · warnings · breaking]`, the causes most frequent first, then one line per row worth reading; `Style plan:` the same. The causes, in the line's words: `unchanged`, `drifted`, `copies`, `unsupported type`, `refused`, `in a closed collection`, `type conflict`, `cannot be written`, `cannot be built`, `no font`, `font not loaded`. Every cause but `unchanged` lists its rows as `skipped: <reason>`, ten per cause, then `…and <n> more; --verbose lists them`. With `--verbose` every row is listed, `unchanged: <path>` included.

| Message | Meaning | Do |
|---|---|---|
| `Variable sync stopped, so nothing was written and no style or component was run.` **stops** | A plan warning that stops, such as `Collection "<n>" was built by an earlier sync: it holds mode(s) …`, or a modifier with no default | Rename or remove that collection; give the modifier a `default` |
| `"<modifier>" has 14 contexts for Figma and Figma allows 10 modes per collection; keep at most 10 of them in figma.include.` **stops** | | List at most 10 in `$extensions["com.toghen"].figma.include`, the default among them |
| `The part axis "state" folds into a collection of that name, and the base collection or a modifier already has it. …` **stops** | | Rename the axis in the vocabulary, or `layout=parts` |
| `Variable sync will not run: N step(s) would lose something in Figma, and nothing was written; pass --allow-breaking to take it` **stops** | With `breaking: the mode "<ctx>" of collection "<m>" is gone` | The person decides |
| `skipped: "<path>" cannot be written in <context>: <reason>` / `skipped: "<path>" cannot be built in <mode>: <reason>` | A value Figma cannot hold, or one lib refuses with its sentence, such as `… cannot be computed: currentColor has no value until an element uses it.` | Read the reason; change the token, or give it `figma.value` where the guide allows |
| `drifted: <path> — <modes that moved>` | A hand edit in Figma or FigCraft, kept | Report it; the person decides on `--overwrite-drift` |
| `skipped: "<path>" is currentColor, which Figma cannot hold; skipped.` | | Add `figma.value` (`figma/guide.md`, *A value only CSS can hold*) |
| `skipped: "<path>" is a letter spacing in em, …; skipped.` | | Same |
| `skipped: "<path>" is defined only in theme=dark and has no value where theme=light, the default; …` | | Give it a default value |
| `note: N variables are offered to every field: …; declare figma.scopes to narrow them.` | | Optional: add `figma.scopes` |
| `"<style>" uses "<family>": "<a>" and "<b>" are not available in Figma.` | A font fallback on this machine | Install the face, or accept it |
| `"<component>": "<axis>" stays in the path, one variable per part: …` | How parts became variables | Informational |
| `Variable sync: <a> applied · <s> skipped · <f> failed` then `failed: <reason>` and `Variable sync did not complete, so no component was run.` **stops** | Figma refused a write. `skipped` counts skips, failed prerequisites and cancellations, not warnings or notes | Read the reason; run again after fixing |

## Components and artboards

The report prints `<Name>: N steps · c created · u updated · d deleted[ · b breaking]`. A set's dry run says `<Name> (dry run): N steps · c created · n would update · d deleted`, the members a real run would write to, and with `--verbose` one line per patch, `  patch <member> <address>: <fields>`. An artboard's dry run counts layers: `<Name> (dry run): N steps · c would create · u would update · d would delete`; a removed layer counts once, with what it holds. With `--skip-unchanged`, a set or artboard left alone says `<Name> (dry run): unchanged, its fingerprint the same; not read` (without `(dry run)` in a real run). Then lines starting `pending:`, `breaking:`, `warning:`, `left:`. An artboard's dry run lists its layers: `create`, `patch`, `replace`, `move`, `remove`, and `swap` for an instance Toghen made whose component the lock now answers with another (the relink).

A component that cannot run prints `<file>: was not planned: <headline>` or `<file>: was not built: <headline>`, then `  - <problem>` lines. **The run goes on to the next file and exits 1 at the end.**

| Problem | Do |
|---|---|
| `token {space.xl} names no variable in this file: looked for "space/xl"; run the variable sync first` | Run `sync`. Also the sign of a `layout=parts` file after a split run, or `components` without `--resolver` |
| `token {x} is ambiguous, 2 variables in this file are named "x/y"` | Two base collections (a changed `--collection`) or a hand-made duplicate. A person removes one |
| `component "Icon" (line N) is not in this file` | No file of the run defines it, and the file does not hold it: create or import it first, or add its component file to the run |
| `modes <collection>=<mode> (line N): a library collection's mode ("<collection>" in library "<library>") is not measured in Figma yet; pin it on FigCraft, or use a local collection` | Figma does not pin a `--use` library's collection yet. Pin it in FigCraft, or use a local collection |
| `modes <collection>=<mode> (line N): no local variable collection named "<collection>" (this file has …), and no library is declared` | Fix the name, or `--use` the library that holds it |
| `modes …: the collection "<collection>" is in this file and in library "<library>": a name found in both stops the build; rename one` | A person renames one |
| `a component set named "Chip" already exists (<ids>) and Toghen did not make it; rename or delete it, or build on a page where it is not. Toghen never adopts a set by name` | A person renames or deletes it |
| `no page named "Docs" in this file` | Create the page first |
| `the member "state=pressed" is gone: instances of it lose their variant` (under `breaking`) | The person decides on `--allow-breaking`, then swaps the instances |
| `token <path> in member "…" is bound to opacity but is not an opacity; declare figma.scopes ["OPACITY"] on it` | Fix the token or the field |
| `<what>: <from> did not import: not published, or not this key: publish the library, then accept its update in this file` | A person publishes and accepts |

| Line | Meaning |
|---|---|
| `pending: token {space.xl} (line 12): the variable sync would create "space/xl"` | Expected in a dry run |
| `pending: component "Chip" (line N): a file earlier in this run would create it; the properties and slots of instances of "Chip" are checked when it is built` | Expected in a first dry run; the real run checks the rest |
| `left: member "<m>", layer "<address>": "paddingTop" was changed in Figma since Toghen wrote it, and was left as it is; pass --overwrite-drift to write the file's value` | Drift on a component or artboard field. The wording says Figma in FigCraft runs too. With `--overwrite-drift` it is a `warning:` line ending `; the file's value is written over it` |
| `left: … is a copy of Toghen's …, left alone` | A person's duplicate; never touched |
| `removed "<x>" (<id>)`, `moved "Chip" to <x>, <y>`, `note: …` | The end of the run: strays removed, sets restacked |
| `removed the set "<n>" (<id>) from page "<p>"` (or `the artboard`), `note: page "<p>" holds nothing now; it is left as it is` | The end of a `--prune --allow-breaking` run. The page is the person's to delete |

## FigCraft, live and headless

| Message | Meaning | Do |
|---|---|---|
| `Writing into "<file>" in FigCraft on port <port>.` | Live: the pinned target | Check it is the right file |
| `Nothing answered on 127.0.0.1:<port>: …. Start FigCraft with --control <port>.` **stops** | | Start FigCraft with the port; wait for it |
| `FigCraft shows no file: open the file to sync into, then run again.` **stops** | | Show the target file |
| `FigCraft no longer has "<file>" at the tab it was in (a tab before it closed?): nothing more is written, and the step before this may have gone to "<other>".` **stops** | | Undo that step in the other file |
| `Writing into <path> headless, with figcraft-cli <version>.` | Headless start | |
| `<cli> was not found: put figcraft-cli on the PATH, or name it with FIGCRAFT_CLI.` **stops** | | Install it, or set `FIGCRAFT_CLI` |
| `<cli> did not start: <reason>` **stops** | | Check the binary and its permissions |
| `<cli> is <version>: a headless run needs figcraft-cli 0.8.0 or later, …` **stops** | | Upgrade to the app's version |
| `Saved <path>.` | Headless, completed | Commit the file, or keep it as an artifact |
| `<path> was not saved: the run did not complete.` (exit 1) | Headless, stopped partway | The file is as it was; fix and rerun |
| A save error such as `No such file or directory` (exit 1) | The folder does not exist | Create the folder |
| `will not build "<component>": nothing was written` **stops**, then lines such as: | FigCraft refuses something by name | Leave that out of FigCraft, or build it in Figma only |
| ` "<c>" <address>: a component named by a Figma key is not built in FigCraft, whose keys are <library id>:<asset id>; name the component, and declare its library with --use` | | Name it, and `--use` its library |
| ` "<c>": the preferred values of "<p>" name Figma's published keys, which FigCraft does not hold; name the components with name=` | | `preferred COMPONENT name="…"` |
| ` "<c>" <address>: textTruncation is not built in FigCraft, whose truncation is a text's sizing (textAutoResize "truncate"), not Figma's ending` | | |
| ` "<c>" <address>: the line cap TRIANGLE_FILLED has no match in FigCraft, whose one triangle is ARROW_EQUILATERAL` | | |
| ` "<c>": the default of "<p>" is a token, which FigCraft does not bind to a property yet` | | |
| ` "<c>" <address>: the field <name> is not built in FigCraft yet` | | |
| `FigCraft does not list <family> <style>` | A font FigCraft does not have | Install it, or change the typography |
| `FigCraft target: an effect style of <type> is not translated.` / `… a paint style of <type> is not translated.` | Only shadows and gradients | |
| `The default mode "<m>" of collection "<c>" is never deleted.` | | |
| `The stamps were not written into FigCraft: …. The next run adopts what this one wrote by name.` | | Rerun |
| FigCraft's own message, then `; this batch pinned a library collection's mode at <what>` | FigCraft refused a `--use` library collection's mode, such as a mode the collection lacks | Fix the `modes` line it names |

## Libraries

| Message | Meaning |
|---|---|
| `Library "<n>": C components · S styles · V variables · …` | What the library file holds |
| `"<set>" is not published as it is now (UNPUBLISHED\|CHANGED): publish the library, then accept its update in the target file.` | Figma: a person's step |
| `Dry run: the lock in the target file was not written.` | |
| `The lock in the target file was not written: the build in the library file did not complete.` (exit 1) | Fix the library build first |
| `The lock in the target file holds "<n>", written by the build.` | Done |
| `library "<n>" is declared, and the lock holds nothing for it: discover its variables, or pick its components, in the Build pane` | A `--use` library the lock does not hold. In Figma, run `--library` or `lock` first. In FigCraft a real run fills it; a dry run reads the lock as it is |
| `<what> is in this file (<where>) and in library "<n>": a name found in both stops the build; rename one, or name the one to use in the overrides map` | A name in the file and in a declared library |
| `<what> is in libraries "<a>", "<b>": name the one to use in the overrides map, as (key)"…"` | A name in two declared libraries |
| `"<c>": the preferred values of "<p>" by name are not written in Figma until measured in the desktop app; name them by key (line <n>)`, under `will not build "<c>": nothing was written` **stops** | Figma takes preferred values by `key=` only, for now |

### Libraries in FigCraft

The run publishes, enables and accepts; nothing is a person's step but saving the target, live.

| Message | Meaning |
|---|---|
| `Building the library <path>, in "<tab>".` / `Building the library <path>, a new file.` | A `--library` run: the build goes into the library |
| `Published the library <path> as version <v>, and saved it.` | |
| `Enabled <path> in "<tab>", and accepted <n> updates from it.` / `…, with no update to accept.` | The target's half |
| `The lock in "<tab>" holds "<name>" (<given>), written by the build: <c> components · <s> styles · <v> variables.` | Done; `written by the command line` after `lock` |
| `Using <path>: the lock in "<tab>" holds "<name>", <c> components · <s> styles · <v> variables.` | A `--use` run refreshed the entry |
| `<path> has <n> updates for "<tab>" not accepted: a run with --library accepts them, or lock does.` | `--use` enables, never accepts |
| `Dry run: "<tab>" has <n> updates from <path> waiting; a run would publish the library first, then accept them.` / `Dry run: <path> is not enabled in "<tab>"; a run would enable it.` / `Dry run: "<tab>" cannot read <path>: <reason>` | What a dry run sees in the target |
| `Dry run: the library was not published or saved, and the lock in the target was not written.` | |
| `The library was not published and the lock in the target was not written: the build in the library did not complete.` (exit 1) | Fix the library build; nothing left the library |
| `The target is the library <path> itself: show, or name, the file that uses it.` **stops** | |
| `<what> with key "<key>" not found: a FigCraft key is <library id>:<asset id>, and a Figma key names nothing in FigCraft` | A Figma key reached FigCraft, through the overrides map: use FigCraft's form, or the name and `--use` |

## CSS (`@toghen/postcss`)

| Message | Do |
|---|---|
| `@toghen/postcss needs its options: it has no design system to read. …` | Call the plugin with `{ system }`; never list it bare |
| `Cannot resolve token "ds.x": Token 'x' not found in collection 'ds' for mode 'light/regular'.` | A typo or a mode not built; with `onUnresolved: 'throw'` it stops the build |
| `Cannot write context '<c>' of modifier '<m>': the resolver document gives it no CSS selector. …` | Add `$extensions["com.toghen"].css` to the modifier |
| `Cannot write the mixins of "ds" inside a layer: …` | Put `@toghen mixins;` at the top level |
| `"a.b" and "a-b" are both written as the mixin --a-b. …` | Rename one path |
| Every `@apply` left in the output | List Toghen before the mixins polyfill |

## `toghen-query`

Each answer ends with `— <export>`. A one-shot query exits 1 on an `error: …` line, a usage error or an unknown command, and 0 for an answer or `help`. `check` prints `incomplete:`, `holes:`, `non-factorable:` and `collisions:`. Each reads `(none)` on its own line when clean, and is a bare header with indented entries when not. `check` exits 1 when any but `collisions:` has entries.
