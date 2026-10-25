# A worked example: `mini`

*A synthetic design system. `toghen-figma check`, `toghen-figma members`, `toghen-query check` and the CSS build were run on these files on 24 October 2026, at figma 7.0.0, lib 6.0.0 and postcss 0.21.0. The Figma and FigCraft runs were not; what they should end with is read from the code.*

## The files

```
mini/
├── package.json                  the scripts in commands.md
├── tokens/
│   ├── mini.resolver.json        set base; modifiers theme (light, dark) and density (regular, compact)
│   ├── vocabulary.tokens.json    parts: state { hover, pressed }
│   ├── primitives.tokens.json    palette.*, hidden from Figma's pickers
│   ├── roles.tokens.json         surface.*, text.*, space.*, radius.full, type.label
│   ├── theme.dark.tokens.json
│   ├── density.compact.tokens.json
│   └── chip.tokens.json          the component: keyed by CSS property, with hover and pressed parts
├── components/chip.component.kdl
├── docs/chip.artboard.kdl
├── docs/chip.data.json
├── styles/index.css
└── scripts/build-css.mjs         the script in commands.md, *The CSS build*
```

`tokens/mini.resolver.json`:

```json
{
  "version": "2025-11-01",
  "name": "mini",
  "description": "A tiny synthetic design system.",
  "sets": {
    "base": {
      "$extensions": { "com.toghen": { "version": "0.1.0" } },
      "sources": [
        { "$ref": "vocabulary.tokens.json" },
        { "$ref": "primitives.tokens.json" },
        { "$ref": "roles.tokens.json" },
        { "$ref": "chip.tokens.json" }
      ]
    }
  },
  "modifiers": {
    "theme": {
      "default": "light",
      "contexts": { "light": [], "dark": [{ "$ref": "theme.dark.tokens.json" }] },
      "$extensions": { "com.toghen": {
        "css": { "selector": "[data-theme='{context}']" },
        "figma": { "include": ["light", "dark"] }
      } }
    },
    "density": {
      "default": "regular",
      "contexts": { "regular": [], "compact": [{ "$ref": "density.compact.tokens.json" }] },
      "$extensions": { "com.toghen": { "css": { "selector": "[data-density='{context}']" } } }
    }
  },
  "resolutionOrder": [
    { "$ref": "#/sets/base" },
    { "$ref": "#/modifiers/theme" },
    { "$ref": "#/modifiers/density" }
  ]
}
```

`tokens/vocabulary.tokens.json`:

```json
{ "$extensions": { "com.toghen": { "css": { "parts": {
  "state": { "hover": ":hover", "pressed": ":active" }
} } } } }
```

`tokens/primitives.tokens.json`. An empty `figma.scopes` hides the variables from every picker; they still serve as alias targets:

```json
{
  "palette": {
    "$type": "color",
    "$extensions": { "com.toghen": { "figma": { "scopes": [] } } },
    "white": { "$value": { "colorSpace": "srgb", "components": [1, 1, 1], "hex": "#ffffff" } },
    "ink":   { "$value": { "colorSpace": "srgb", "components": [0.1, 0.1, 0.12], "hex": "#1a1a1f" } },
    "blue":  { "$value": { "colorSpace": "srgb", "components": [0.1, 0.4, 0.9], "hex": "#1a66e6" } },
    "navy":  { "$value": { "colorSpace": "srgb", "components": [0.06, 0.24, 0.56], "hex": "#0f3d8f" } }
  }
}
```

`tokens/roles.tokens.json`:

```json
{
  "surface": { "$type": "color",
    "page": { "$value": "{palette.white}" },
    "accent": { "$value": "{palette.blue}" },
    "accent-strong": { "$value": "{palette.navy}" } },
  "text": { "$type": "color",
    "default": { "$value": "{palette.ink}" },
    "on-accent": { "$value": "{palette.white}" } },
  "space": { "$type": "dimension",
    "xs": { "$value": { "value": 4, "unit": "px" } },
    "sm": { "$value": { "value": 8, "unit": "px" } } },
  "radius": { "$type": "dimension", "full": { "$value": { "value": 9999, "unit": "px" } } },
  "type": { "label": { "$type": "typography", "$value": {
    "fontFamily": ["Inter", "sans-serif"], "fontSize": { "value": 14, "unit": "px" },
    "fontWeight": 500, "lineHeight": 1.25 } } }
}
```

`tokens/theme.dark.tokens.json` and `tokens/density.compact.tokens.json`:

```json
{ "surface": { "$type": "color", "page": { "$value": "{palette.ink}" } },
  "text": { "$type": "color", "default": { "$value": "{palette.white}" } } }
```

```json
{ "space": { "$type": "dimension", "sm": { "$value": { "value": 4, "unit": "px" } } } }
```

`tokens/chip.tokens.json`:

```json
{ "chip": {
  "backgroundColor": { "$type": "color", "$value": "{surface.accent}" },
  "color":           { "$type": "color", "$value": "{text.on-accent}" },
  "paddingInline":   { "$type": "dimension", "$value": "{space.sm}" },
  "paddingBlock":    { "$type": "dimension", "$value": "{space.xs}" },
  "borderRadius":    { "$type": "dimension", "$value": "{radius.full}" },
  "font":            { "$type": "typography", "$value": "{type.label}" },
  "hover":   { "backgroundColor": { "$type": "color", "$value": "{surface.accent-strong}" } },
  "pressed": { "backgroundColor": { "$type": "color", "$value": "{surface.accent-strong}" } }
} }
```

`components/chip.component.kdl`. The root binds the `chip` group by its keys, and the parts give it a `state` axis:

```kdl
component Chip tokens="{chip}"
description "A compact label on the accent colour."

properties {
    TEXT Label default="Chip"
}

FRAME Chip layoutMode=HORIZONTAL layoutSizingHorizontal=HUG layoutSizingVertical=HUG counterAxisAlignItems=CENTER {
    TEXT Caption tokens="{chip}" characters=(property)Label textAutoResize=WIDTH_AND_HEIGHT
}
```

`docs/chip.artboard.kdl`:

```kdl
artboard "Chip reference"

FRAME Page layoutMode=VERTICAL layoutSizingHorizontal=HUG layoutSizingVertical=HUG itemSpacing="{space.sm}" paddingTop="{space.sm}" paddingBottom="{space.sm}" paddingLeft="{space.sm}" paddingRight="{space.sm}" {
    fills "{surface.page}"
    TEXT Title characters=(input)title textStyle="{type.label}" {
        fills "{text.default}"
    }
    each c in=(input)examples key=label {
        INSTANCE Example component=Chip {
            set Label=(c)label
        }
    }
}
```

`docs/chip.data.json`:

```json
{
  "version": "2025-11-01",
  "name": "chip-docs",
  "sets": { "base": { "sources": [ {
    "title": { "$value": "Chip" },
    "examples": { "$value": [ { "label": "Default" }, { "label": "Selected" } ] }
  } ] } },
  "resolutionOrder": [ { "$ref": "#/sets/base" } ]
}
```

`styles/index.css`:

```css
@layer tokens {
    @toghen contexts;
}

@toghen mixins;

.chip {
    display: inline-flex;
    @apply --chip;
}
```

## The run

### A. Checks (no target)

```sh
pnpm tokens:check     # incomplete: (none), holes: (none), non-factorable: (none), collisions: (none)
pnpm figma:lint       # 2 files read: no errors
pnpm css              # dist/mini.css; no [object Object], no t( or toghen( left
pnpm exec toghen-figma members tokens/mini.resolver.json components/chip.component.kdl
                      # Label TEXT; state VARIANT [rest, hover, pressed]; members state=rest, state=hover, state=pressed
```

A broken copy shows the form of a problem:

```text
bad/chip.component.kdl:8:115 "width" takes a token reference or a (literal) number: a bare 12 would be a number nobody chose, write (literal)12 or a token reference
bad/dot.component.kdl:2:11 token {radius.xl} is not in the design system "mini"
bad/dot.component.kdl:3:5 token {chip.hover.backgroundColor} is the "hover" part of "chip.backgroundColor", which is one variable with a mode per "state": name {chip.backgroundColor}; the member pins the mode (a tokens= axis, or modes state=hover)
2 files read: 3 errors
```

The third line is a folded part: `chip.hover.backgroundColor` is the `hover` mode of the variable `chip/backgroundColor`, not a variable of its own. A text written `characters="{{space.sm}}"` in the same file shows `{space.sm}`, and is no error.

### B. Figma, first build

Needs a file with a page named `Components`, open in the desktop app, with the plugin running.

```sh
pnpm figma:all:check  # in the background: hand over "Pairing code: …", wait for "The plugin is paired."
                      # the artboard's instance of Chip is listed as "pending"
pnpm figma:all        # reconnects by itself
```

The dry run counts the set the chip's file would create as present, so the artboard's `INSTANCE Example component=Chip` is checked by name; its `set Label=…` is checked when the real run builds the chip. Every later build is the same two commands.

What the file should hold afterwards:

- **Collections.**
  - `mini`, with one mode `default`: the primitives, and the tokens that change only through a reference, as aliases.
  - `theme`: `light` and `dark`.
  - `density`: `regular` and `compact`.
  - `state`: `rest`, `hover` and `pressed`. It holds `chip/backgroundColor`, folded from the parts.
- **Variables.** Each has its CSS name as web code syntax, such as `var(--chip-background-color)`. `palette/*` is in no picker.
- **Text styles.** `type/label` and `chip/font`, from the default mode. The plan warns that styles have no modes.
- **The set `Chip`.** Properties `Label` and `state`; three members, each pinning its `state` mode. Fills, paddings, radius, caption fill and text style are bound to the variables and the style.
- **The artboard `Chip reference`.** A title and two `Chip` instances, labelled `Default` and `Selected`.

### C. FigCraft, live

```sh
open -n -a FigCraft --args --control 7979
for i in $(seq 60); do nc -z 127.0.0.1 7979 && break; sleep 1; done   # a minute at most
# show the target file in FigCraft's front window, with a page "Components" if --page names one
pnpm figcraft:check
pnpm figcraft:sync
# the artboard with them: the same commands with docs/chip.artboard.kdl --input docs/chip.data.json added
```

### D. FigCraft, headless (CI)

```sh
mkdir -p design
pnpm figcraft:file:check   # no --page: a new file has only "Page 1"
pnpm figcraft:file         # ends "Saved design/mini.figcraft."
git add design/mini.figcraft
```

A second run with nothing changed plans nothing and writes nothing: `Variable plan: 0 creates · 0 updates · <n> skips (<n> unchanged)`, `Chip (dry run): … 0 would update`.

## The CSS it makes (excerpt, verified)

```css
@layer tokens {
    @layer defaults, theme, density;
    @layer defaults { :root { --surface-accent: #1a66e6; --chip-background-color: #1a66e6; --chip-hover-background-color: #0f3d8f; … } }
    @layer theme { [data-theme='light'] { … } [data-theme='dark'] { --surface-page: #1a1a1f; --text-default: #ffffff } }
    @layer density { [data-density='regular'] { --space-sm: 8px; --chip-padding-inline: 8px } [data-density='compact'] { --space-sm: 4px; --chip-padding-inline: 4px } }
}
@mixin --chip {
    --_chip-background-color: initial;
    background-color: var(--_chip-background-color, var(--chip-background-color, #1a66e6));
    …
    &:where(:hover) { --_chip-background-color: var(--chip-hover-background-color, #0f3d8f) }
    &:where(:active) { --_chip-background-color: var(--chip-pressed-background-color, #0f3d8f) }
}
```
