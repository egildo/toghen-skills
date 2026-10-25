# Commands, flags and exit codes

*Pinned to `@toghen/figma` 0.1.0 and `@toghen/query` 0.1.0 (7.0.1 and 0.2.0 before the split), checked 24 October 2026 against the usage text the command prints. The authority is the usage text in `figma/cli/main.ts` and `figma/reference.md` (*The command line: toghen-figma*, *Into FigCraft*, *Headless*, *The lock*).*

## How to call it

- In a design system's repository, from an npm script, or with `pnpm exec toghen-figma …`.
- Inside the Toghen checkout: `pnpm --filter @toghen/figma toghen-figma <args>`.
- Paths are read from the directory the command was typed in. pnpm passes it as `INIT_CWD`, which `cli/bin.mjs` uses.

## `toghen-figma`

```
toghen-figma variables  <entry.json>…          [--name <n>] [--overrides <map.json>] [--collection <n>] [--dry-run] [--allow-breaking] [--overwrite-drift] [--verbose] [--figcraft <port> | --figcraft-file <path.figcraft>] [--library <library>]
toghen-figma components <file.kdl>…            [--resolver <entry.json>] [--name <n>] [--overrides <map.json>] [--input <doc>] [--page <n>] [--dry-run] [--allow-breaking] [--overwrite-drift] [--verbose] [--figcraft <port> | --figcraft-file <path.figcraft>] [--library <library> | --use <library>…]
toghen-figma sync       <entry.json>… <file.kdl>… [--name <n>] [--overrides <map.json>] [--collection <n>] [--input <doc>] [--page <n>] [--dry-run] [--allow-breaking] [--overwrite-drift] [--verbose] [--figcraft <port> | --figcraft-file <path.figcraft>] [--library <library> | --use <library>…]
toghen-figma check      <entry.json>… <file.kdl>… [--name <n>] [--overrides <map.json>] [--input <doc>]
toghen-figma members    <entry.json>… <file.kdl>… [--name <n>] [--overrides <map.json>] [--input <doc>]
toghen-figma lock       <library name>
toghen-figma lock       <library.figcraft> (--figcraft <port> | --figcraft-file <path.figcraft>) [--dry-run]
```

A `<library>` is the library's name as Figma names it, or, with `--figcraft` or `--figcraft-file`, the library's `.figcraft` file.

| Command | What it does | Target |
|---|---|---|
| `variables` | Loads the entry; syncs variables, then styles | Figma, FigCraft |
| `components` | Checks and runs component files, then artboard files, against variables and styles already in the file. Takes the design system only through `--resolver` | Figma, FigCraft |
| `sync` | All of the above, in order. Needs at least one `.kdl` file | Figma, FigCraft |
| `check` | Reads each file against the design system, with no target. Problems on standard error, a summary on standard output | none |
| `members` | Reads each file the same way; prints `[{file, component, artboard?, members, problems}]` as JSON. The types are in `figma/reference.md` (*Component files*) | none |
| `lock` | Figma: pairs a plugin in the library file and one in the target file; copies the library's keys into the target's lock. FigCraft: in the target only, enables the library, accepts its updates and writes the lock | Figma, FigCraft |

### Arguments and flags

| Argument or flag | Commands | Meaning |
|---|---|---|
| `*.json` positionals | variables, sync, check, members | The entry: one resolver document, one token file, or several token files merged in order, the last winning. `components` refuses one |
| `*.kdl` positionals | components, sync, check, members | `*.component.kdl` and `*.artboard.kdl`. Component files run first, in dependency order (each after the files defining the components it names; ties keep the order given; a cycle is refused), then artboards |
| `--resolver <entry.json>` | components | Loads the design system for files that take axes or bindings from the tokens (`tokens=`, `(modifier)`). Syncs nothing |
| `--name <n>` | all but lock | The collection's name for token files. Default: their common folder. Required when they share none |
| `--overrides <map.json>` | all but lock | One JSON object: a `$ref` as written → the file to read instead (`lib/reference.md`, *The overrides map*) |
| `--collection <n>` | variables, sync | The base collection's name. Default: the document's `name`. Keep it the same on every run |
| `--page <n>` | components, sync | The page sets and artboards live on. It must exist. Default: the current page |
| `--input <doc>` | components, sync, check, members | The data an artboard reads with `(input)`: a resolver-like document. Refused when no artboard file is given. One per run |
| `--dry-run` | variables, components, sync; lock in FigCraft | Plans everything, writes nothing. Saves nothing headless |
| `--verbose` | variables, components, sync | Lists every skip of a plan, `unchanged: <path>` included, with no list cut at ten, and every patch of a component set as `patch <member> <address>: <fields>`. Without it, a plan lists the rows of every cause but `unchanged`, ten per cause |
| `--allow-breaking` | variables, components, sync | Takes the steps that break instances or delete a mode. The person's call |
| `--overwrite-drift` | variables, components, sync | Writes the source over every hand edit. All or nothing. The person's call |
| `--figcraft <port>` | variables, components, sync, lock | Runs into the file FigCraft shows, through `127.0.0.1:<port>` |
| `--figcraft-file <path.figcraft>` | variables, components, sync, lock | Runs into that file with no app, through `figcraft-cli mcp`, and saves it if the run completes. The path must end in `.figcraft` |
| `--library <library>` | variables, components, sync | Builds in the library file, then fills the target's lock. Figma: the library's name; a person publishes and accepts. FigCraft: its `.figcraft` file; the run publishes and saves it, enables it in the target, accepts its updates and writes the lock |
| `--use <library>` | components, sync | Repeatable. Declares a library the files may take components, tokens' variables and text and effect styles from, through the target's lock. Figma: its name as the lock holds it. FigCraft: its `.figcraft` file, enabled and its lock entry refreshed first (a dry run neither). Not with `--library` |

### Exit codes

| Command | 0 | 1 |
|---|---|---|
| `variables`, `components`, `sync` | Every step succeeded (a dry run with breaking steps listed included) | Anything failed: arguments, a file, the check, component files naming each other in a cycle, a stopped plan, a refused breaking step, a refused write, the plugin gone, a component not built, a headless save that failed |
| `check`, `members` | No file has an error | A file has an error or cannot be read |
| `lock` | Figma: keys copied. FigCraft: enabled, accepted and the lock written, or, in a dry run, what waits said | The plugins or FigCraft did not answer, or the lock was not written |

### Environment

| Variable | Effect |
|---|---|
| `FIGCRAFT_CLI` | The `figcraft-cli` binary a headless run starts. Default: `figcraft-cli` on `PATH` |
| `FIGCRAFT_NO_SYSTEM_FONTS=1` | Reaches the headless `figcraft-cli`: FigCraft's own fonts only, the same on every machine. It also narrows `text.fonts`, which the style plan reads, so a text style in a system font falls back or is skipped |
| `INIT_CWD` | Set by pnpm; where relative paths are read from |

Files the command keeps: the pairing code in `~/.toghen/figma-pairing`, and the library's in `~/.toghen/figma-pairing.library`.

## `toghen-query`

```
toghen-query [--doc <resolver.json> | --tokens <file>…] [--overrides <map.json>] [--build sum|product|modifier=context,…] [--json] [<query> | --session <file>]
```

Useful queries in a pipeline (`query/guide.md`):

| Query | Answers |
|---|---|
| `check` | Holes, tokens incomplete in some mode, non-factorable tokens, the collisions `load` found. Exit 1 for any but collisions |
| `system` | Modifiers, contexts, defaults, modes built; with a map, its hash and unused keys |
| `value <path> across` | One row per mode |
| `parts <group> axes` | The axes a component's parts give it, the ones a component file takes |
| `applies <component> state=hover` | Which token supplies each key under a selection |
| `css <lookup path>` | A token's custom property name |

Exit, for one query: 0 for an answer, `help` included; 1 for an `error: …`, a usage error, an unknown command, or a `check` with `incomplete`, `holes` or `non-factorable` entries (collisions do not fail it). A replayed `--session` exits 1 when it differs; the prompt exits 0; 2 when the document does not load.

## The CSS build

`@toghen/postcss` ships no command. The design system builds its own:

```js
import { readFile, writeFile, mkdir } from 'node:fs/promises';
import postcss from 'postcss';
import postcssMixins from '@csstools/postcss-mixins';
import { load, Resolver } from '@toghen/lib';
import toghen from '@toghen/postcss';

const { document } = await load('tokens/mini.resolver.json', { read: (p) => readFile(p, 'utf8') });
const system = new Resolver(document).resolveContextsToDesignSystem();
const { css } = await postcss([toghen({ system, onUnresolved: 'throw' }), postcssMixins({ preserve: true })])
  .process(await readFile('styles/index.css', 'utf8'), { from: 'styles/index.css' });
await mkdir('dist', { recursive: true });
await writeFile('dist/mini.css', css);
```

- List Toghen before the mixins polyfill, or every `@apply` stays in the output.
- `onUnresolved: 'throw'` makes a missing token fail the build. Use it in CI.
- The stylesheet holds `@layer tokens { @toghen contexts; }` and `@toghen mixins;` at its top level. See `postcss/guide.md`.

## npm scripts, a full set

```json
{
  "scripts": {
    "tokens:check":    "toghen-query --doc tokens/mini.resolver.json check",
    "figma:lint":      "toghen-figma check tokens/mini.resolver.json components/*.component.kdl docs/chip.artboard.kdl --input docs/chip.data.json",
    "css":             "node scripts/build-css.mjs",

    "figma:check":     "toghen-figma sync tokens/mini.resolver.json components/*.component.kdl --page Components --dry-run",
    "figma:sync":      "toghen-figma sync tokens/mini.resolver.json components/*.component.kdl --page Components",
    "figma:all:check": "toghen-figma sync tokens/mini.resolver.json components/*.component.kdl docs/chip.artboard.kdl --input docs/chip.data.json --page Components --dry-run",
    "figma:all":       "toghen-figma sync tokens/mini.resolver.json components/*.component.kdl docs/chip.artboard.kdl --input docs/chip.data.json --page Components",

    "figcraft:check":  "toghen-figma sync tokens/mini.resolver.json components/*.component.kdl --figcraft 7979 --dry-run",
    "figcraft:sync":   "toghen-figma sync tokens/mini.resolver.json components/*.component.kdl --figcraft 7979",

    "figcraft:file:check": "toghen-figma sync tokens/mini.resolver.json components/*.component.kdl docs/chip.artboard.kdl --input docs/chip.data.json --figcraft-file design/mini.figcraft --dry-run",
    "figcraft:file":       "toghen-figma sync tokens/mini.resolver.json components/*.component.kdl docs/chip.artboard.kdl --input docs/chip.data.json --figcraft-file design/mini.figcraft",

    "figma:library":   "toghen-figma sync tokens/mini.resolver.json components/*.component.kdl --page Components --library \"Mini Library\"",
    "figma:lock":      "toghen-figma lock \"Mini Library\"",
    "figma:app":       "toghen-figma components docs/chip.artboard.kdl --resolver tokens/mini.resolver.json --input docs/chip.data.json --use \"Mini Library\""
  }
}
```

A FigCraft library and a file that uses it, headless: `design/kit.figcraft` holds the variables, styles and components; `design/app.figcraft` places them.

```json
{
  "scripts": {
    "kit:check": "toghen-figma sync tokens/mini.resolver.json components/*.component.kdl --figcraft-file design/app.figcraft --library design/kit.figcraft --dry-run",
    "kit":       "toghen-figma sync tokens/mini.resolver.json components/*.component.kdl --figcraft-file design/app.figcraft --library design/kit.figcraft",
    "kit:lock":  "toghen-figma lock design/kit.figcraft --figcraft-file design/app.figcraft",
    "app:check": "toghen-figma components docs/chip.artboard.kdl --resolver tokens/mini.resolver.json --input docs/chip.data.json --figcraft-file design/app.figcraft --use design/kit.figcraft --dry-run",
    "app":       "toghen-figma components docs/chip.artboard.kdl --resolver tokens/mini.resolver.json --input docs/chip.data.json --figcraft-file design/app.figcraft --use design/kit.figcraft"
  }
}
```

`kit` builds into `kit.figcraft`, publishes and saves it, then enables it in `app.figcraft`, accepts its updates, writes the lock entry `kit` and saves `app.figcraft`. `app` takes the chip and its tokens from the library through that lock. Run `kit` before `app`; `kit:lock` is for a library published by hand. Live, the same with `--figcraft <port>` in place of `--figcraft-file design/app.figcraft`: the target is the file FigCraft shows, and the person saves it.

No script carries `--allow-breaking` or `--overwrite-drift`. A person adds them to one run, after reading its dry run:

```sh
pnpm figma:sync -- --allow-breaking
```

## A CI job

Figma cannot run in CI. FigCraft files can.

```yaml
- run: pnpm tokens:check
- run: pnpm figma:lint
- run: pnpm css
- run: pnpm figcraft:file:check
- run: pnpm figcraft:file
```

The first line fails when `toghen-query check` lists incomplete tokens, holes or non-factorable tokens. It passes with collisions, which `load` reports as advice. To fail on them too, read the output: a clean section reads `collisions: (none)`, one with entries is a bare `collisions:` with them beneath (`pnpm --silent tokens:check > query.txt && ! grep -qE '^collisions:\s*$' query.txt`). A first-ever `figcraft:file:check` may hold the artboard: the chip its component file would create counts as present (see the skill, *Dry run first*).
