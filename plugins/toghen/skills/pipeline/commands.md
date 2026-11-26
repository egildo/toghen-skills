# Commands, flags and exit codes

*Pinned to `@toghen/figma` 0.1.18, `@toghen/query` 0.1.2 (0.1.0 was 0.2.0 before the split) and `@toghen/postcss` 0.2.0, checked 26 November 2026 against the usage text in `figma/cli/main.ts` and `query/src/main.ts` and against `query/guide.md`, each at its tag, then measured the same day on the worked example; checked before on 25 November 2026, at figma 0.1.17, query 0.1.1 and postcss 0.2.0, and on 23 November 2026, at figma 0.1.16 and query 0.1.0. The authority is the usage text in `figma/cli/main.ts` and `figma/reference.md` (*The command line: toghen-figma*, *Into FigCraft*, *Headless*, *The lock*).*

## How to call it

- In a design system's repository, from an npm script, or with `pnpm exec toghen-figma …`.
- Inside a checkout of figma: `pnpm toghen-figma <args>`.
- Paths are read from the directory the command runs in: where it was typed, with `pnpm exec`; in an npm script, the package's folder, or the one the script changes to first (`cd design && toghen-figma …`). `INIT_CWD` is not read.
- `toghen-figma --help`, or `-h`, prints the usage on standard output and exits 0.

## `toghen-figma`

```
toghen-figma variables  <entry.json>…          [--name <n>] [--overrides <map.json>] [--collection <n>] [--dry-run] [--allow-breaking] [--overwrite-drift] [--verbose] [--figcraft <port> | --figcraft-file <path.figcraft>] [--library <library>]
toghen-figma components <file.kdl>…            [--resolver <entry.json>] [--name <n>] [--overrides <map.json>] [--input <doc>] [--page <n>] [--dry-run] [--allow-breaking] [--overwrite-drift] [--skip-unchanged] [--prune] [--verbose] [--figcraft <port> | --figcraft-file <path.figcraft>] [--library <library> | --use <library>…]
toghen-figma sync       <entry.json>… <file.kdl>… [--name <n>] [--overrides <map.json>] [--collection <n>] [--input <doc>] [--page <n>] [--dry-run] [--allow-breaking] [--overwrite-drift] [--skip-unchanged] [--prune] [--verbose] [--figcraft <port> | --figcraft-file <path.figcraft>] [--library <library> | --use <library>…]
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
| `--skip-unchanged` | components, sync | Leaves unread each set and artboard whose stamp holds the fingerprint of what this run would write for it, and says `unchanged, its fingerprint the same; not read`. Every real run stamps the fingerprint. It trusts it: a hand edit in a skipped set or artboard is not looked for, so keep a run without it for that. A new Toghen version reads everything once: run one real run after upgrading. In FigCraft a dry run with it reads a frame's insides only where it compares it |
| `--prune` | components, sync | The files given are the whole system for the file the run writes. Each set and artboard Toghen made there that none declares has no source: listed as `breaking:`; a dry run that lists any exits 1; a real run stops unless `--allow-breaking` is given too, and with it removes them at its end. A page left empty stays. Give it every file, or it takes the ones left out for gone |
| `--figcraft <port>` | variables, components, sync, lock | Runs into the file FigCraft shows, through `127.0.0.1:<port>` |
| `--figcraft-file <path.figcraft>` | variables, components, sync, lock | Runs into that file with no app, through `figcraft-cli mcp`, and saves it if the run completes. The path must end in `.figcraft`, and its folder must exist before the run starts |
| `--library <library>` | variables, components, sync | Builds in the library file, then fills the target's lock. Figma: the library's name; a person publishes and accepts. FigCraft: its `.figcraft` file; the run publishes and saves it, enables it in the target, accepts its updates and writes the lock |
| `--use <library>` | components, sync | Repeatable. Declares a library the files may take components, tokens' variables and text and effect styles from, through the target's lock. Figma: its name as the lock holds it. FigCraft: its `.figcraft` file, enabled and its lock entry refreshed first (a dry run neither). Not with `--library` |

### Exit codes

| Command | 0 | 1 |
|---|---|---|
| `variables`, `components`, `sync` | Every step succeeded (a dry run with breaking steps listed included, but for `--prune`) | Anything failed: arguments, a file, the check, component files naming each other in a cycle, a stopped plan, a refused breaking step, a refused write, the plugin gone, a component not built, a `--figcraft-file` or built `--library` whose folder does not exist, a headless request not answered in time, a headless save that failed; and a dry run with `--prune` that lists a set or artboard with no source |
| `check`, `members` | No file has an error | A file has an error or cannot be read |
| `lock` | Figma: keys copied. FigCraft: enabled, accepted and the lock written, or, in a dry run, what waits said | The plugins or FigCraft did not answer, or the lock was not written |

### Environment

| Variable | Effect |
|---|---|
| `FIGCRAFT_CLI` | The `figcraft-cli` binary a headless run starts. Default: `figcraft-cli` on `PATH` |
| `FIGCRAFT_NO_SYSTEM_FONTS=1` | Reaches the headless `figcraft-cli`: FigCraft's own fonts only, the same on every machine. It also narrows `text.fonts`, which the style plan reads, so a text style in a system font falls back or is skipped |

Files the command keeps: the pairing code in `~/.toghen/figma-pairing`, and the library's in `~/.toghen/figma-pairing.library`.

## `toghen-query`

```
toghen-query [--doc <resolver.json> | --tokens <file> [--tokens <file> …]] [--overrides <map.json>] [--build sum|product|modifier=context,…] [--json] [<query> | --session <file>]
```

Give `--tokens` once for each file: `--tokens a.json --tokens b.json`. A second file after one `--tokens` is read as the query (`unknown command 'b.json'`).

Useful queries in a pipeline (`query/guide.md`):

| Query | Answers |
|---|---|
| `check` | Holes, tokens incomplete in some mode, non-factorable tokens, tokens whose paths give one custom property, references that do not resolve in some mode (a cycle included), the collisions `load` found. Exit 1 for any but collisions |
| `system` | Modifiers, contexts, defaults, modes built; with a map, its hash and unused keys |
| `value <path> across` | One row per mode, with the core's error on a row it throws for |
| `parts <group> axes` | The axes a component's parts give it, the ones a component file takes |
| `applies <component> state=hover` | Which token supplies each key under a selection |
| `css <path>` | A token's custom property name, with or without the collection: `css surface.page` and `css mini.surface.page` both answer `--surface-page`. A path that names no token, or a group, is an `error: …`, exit 1 |

The flags may stand before or after the query. From the query's first word on, a `--word` is the query's unless it is one of the flags, so `css --surface-raised theme` asks for a link's custom property. After `--`, every word is the query's.

Exit, for one query: 0 for an answer, `help` included; 1 for an `error: …`, a usage error, an unknown command, an `across` with a row the core threw for, or a `check` with `incomplete`, `holes`, `non-factorable`, `same-css-name` or `unresolvable` entries (collisions do not fail it). A replayed `--session` exits 1 when it differs; the prompt exits 0; 2 when the document does not load.

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

- List Toghen before the mixins polyfill, or no `@apply` is expanded. With `preserve: true` each `@apply` stays in the output beside what it expands to.
- List Toghen before anything that runs Lightning CSS, a PostCSS plugin such as Tailwind's included. Run first, Lightning CSS reads a number after a dot as a number, so it prints `t(palette.orange.100)` as `t(palette.orange.1)`; Toghen then reports a token that is not found, or reads another token that has the new path. Keep Vite's `css.transformer` at its default, `'postcss'`.
- A Vite 8 production build minifies its CSS with Lightning CSS by default, and at Vite's default targets Lightning CSS lowers each `light-dark()` pair. Lowered, a pair is decided at the root, so a subtree's own `color-scheme` is ignored; with no `color-scheme` in the stylesheet, the colour is lost. Exclude the lowering: `css: { lightningcss: { exclude: Features.LightDark } }` in `vite.config.js`, with `Features` from `lightningcss` in the dev dependencies. See `postcss/guide.md` (*When Lightning CSS runs after the plugin*).
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

    "figcraft:file:check": "toghen-figma sync tokens/mini.resolver.json components/*.component.kdl docs/chip.artboard.kdl --input docs/chip.data.json --figcraft-file design/mini.figcraft --dry-run --prune",
    "figcraft:file:quick": "toghen-figma sync tokens/mini.resolver.json components/*.component.kdl docs/chip.artboard.kdl --input docs/chip.data.json --figcraft-file design/mini.figcraft --dry-run --skip-unchanged",
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

`--prune` in a check is safe: it lists, and fails the check, but removes nothing without `--allow-breaking`. `figcraft:file:quick` answers in seconds once a real run has stamped the fingerprints, and does not look for hand edits; `figcraft:file:check` reads everything, finds hand edits, and fails on a set or artboard whose source is gone.

## A CI job

Figma cannot run in CI. FigCraft files can.

```yaml
- run: pnpm tokens:check
- run: pnpm figma:lint
- run: pnpm css
- run: pnpm figcraft:file:check
- run: pnpm figcraft:file
```

The first line fails when `toghen-query check` lists incomplete tokens, holes, non-factorable tokens, tokens of one custom property, or references that do not resolve. It passes with collisions, which `load` reports as advice. To fail on them too, read the output: a clean section reads `collisions: (none)`, one with entries is a bare `collisions:` with them beneath (`pnpm --silent tokens:check > query.txt && ! grep -qE '^collisions:\s*$' query.txt`). A first-ever `figcraft:file:check` may hold the artboard: the chip its component file would create counts as present (see the skill, *Dry run first*).
