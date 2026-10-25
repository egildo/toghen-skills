# Variables and styles

Part of the `toghen:figma` skill. Pinned to `@figma/plugin-typings` 1.142.0 and Plugin API Update 141; checked 2026-10-23. *Measured* marks what Toghen ran in a real file ([measured behaviour](https://codeberg.org/Toghen/figma/src/branch/main/.reference/figma/measured_behaviour.md)); it holds over the docs where they differ.

## Collections and modes

- `figma.variables.createVariableCollection(name)`, then `createVariable(name, collection, type)`; `addMode(name)`, `renameMode(id, name)`, `removeMode(id)`; `variable.setValueForMode(modeId, value)` ([working with variables](https://developers.figma.com/docs/plugins/working-with-variables/)).
- Resolved types: `BOOLEAN`, `COLOR`, `FLOAT`, `STRING`, and since Update 133 `EASING` and `TIMING` (typings 1.142.0).
- *Measured:* a new collection has exactly one mode, `Mode 1`. To make *n* modes, rename that one and add *n − 1*.
- *Measured:* the default mode is always `modes[0]`. `defaultModeId` and `modes` have no setter, and there is no `setModeOrder`. Removing `modes[0]` promotes the next. **The only safe moment to choose the default is when the collection is created.**
- *Measured:* the mode cap on a Professional file is 10 (`in addMode: Limited to 10 modes only`). Plans: Professional 10, Organization 20, Enterprise unlimited through extended collections ([plans table](https://help.figma.com/hc/articles/360040328273)). Parse the error softly: another plan or wording must read as "cap unknown", not as a wrong number.
- *Measured:* Figma does not clear a node's pinned mode when the mode is removed; the pin keeps the dead id.
- **Extended collections** (Enterprise): `collection.extend(name)` or `figma.variables.extendLibraryCollectionByKeyAsync(key, name)`. They inherit modes and variables and hold overrides only; no new variables or modes, no change of scope or description ([working with variables](https://developers.figma.com/docs/plugins/working-with-variables/), [extend a collection](https://help.figma.com/hc/en-us/articles/36346281624471-Extend-a-variable-collection)).

## Names and code syntax

- *Measured:* `/` in a name makes groups; `--` is accepted as written (`demo/tone--scheme`).
- `variable.setVariableCodeSyntax(platform, value)`, platform `WEB`, `ANDROID` or `iOS`; Dev Mode shows it. *Measured:* `codeSyntax` reads back what was written (`{ WEB: 'var(--demo-surface)' }`).

## Aliases and explicit modes

- `figma.variables.createVariableAlias(variable)` or `createVariableAliasByIdAsync(id)`, set as a mode's value ([working with variables](https://developers.figma.com/docs/plugins/working-with-variables/)).
- *Measured:* an alias into another collection can be made in the same run, to a variable made moments before.
- `node.setExplicitVariableModeForCollection(collection, modeId)` pins a mode on a frame or component.
- *Measured:* each collection takes its mode from the nearest ancestor that sets it, independently of the others, and an alias chain across collections resolves in the consuming node's full context. This is how CSS custom properties behave under attribute-scoped blocks. `variable.resolveForConsumer(node)` reads the result.

## Scopes

- Scopes decide which pickers offer a variable. They do **not** stop a binding through the API ([variables endpoints](https://developers.figma.com/docs/rest-api/variables-endpoints/) says so; *measured* too: a FLOAT scoped only `CORNER_RADIUS` binds to `paddingLeft`).
- The list in 1.142.0: `ALL_SCOPES`, `TEXT_CONTENT`, `CORNER_RADIUS`, `WIDTH_HEIGHT`, `GAP`, `ALL_FILLS`, `FRAME_FILL`, `SHAPE_FILL`, `TEXT_FILL`, `STROKE_COLOR`, `STROKE_FLOAT`, `EFFECT_FLOAT`, `EFFECT_COLOR`, `OPACITY`, `COLOR_OPACITY`, `FONT_FAMILY`, `FONT_STYLE`, `FONT_WEIGHT`, `FONT_SIZE`, `LINE_HEIGHT`, `LETTER_SPACING`, `PARAGRAPH_SPACING`, `PARAGRAPH_INDENT`.
- Figma refuses a scope that does not suit the type (`Invalid scope for this variable type`), `ALL_SCOPES` with another scope, and `ALL_FILLS` with another fill scope.
- `[]` hides a variable from every picker; it still works as an alias target and a binding.
- **`COLOR_OPACITY`** (Update 139) puts a FLOAT only in colour-opacity pickers ([update 139](https://developers.figma.com/docs/plugins/updates/2026/09/17/version-1-update-139/)). Its unit is not stated. Toghen refuses it in a declaration until the unit is measured.

## Opacity is a percentage

- *Measured by a consumer:* a FLOAT holding `1` bound to a layer's `opacity` reads `0.0099999`: Figma reads the variable as percent. A token's `0.55` must be written as `55`.
- Composed colours (below) use percent in Figma's own example (`opacity: 60`), though the docs do not say so.

## Composed colours (new in Update 139)

- A `COLOR` variable's value may be `{ color, opacity }`: `color` is an RGB(A) value or an alias, `opacity` a number or an alias to a FLOAT; at least one must be an alias (`VariableComposedColor`, typings 1.142.0; [working with variables](https://developers.figma.com/docs/plugins/working-with-variables/)).
- This is the first route for a paint's opacity to follow a variable. `setBoundVariableForPaint` still binds `color` only.
- Not measured by Toghen. The REST `POST` supports it too.

## Binding fields

- `node.setBoundVariable(field, variable)`; `null` unbinds. Paints, effects and layout grids go through `figma.variables.setBoundVariableForPaint`, `…ForEffect`, `…ForLayoutGrid`, which **return a new object** to assign back into `fills`, `strokes`, `effects` or `layoutGrids` ([working with variables](https://developers.figma.com/docs/plugins/working-with-variables/)).
- *Measured:*
  - A field outside the bindable list throws `Unknown field: '<name>'` (`x`, `rotation`, `strokeAlign`, `blendMode`, `layoutGrow`, `textCase` and others).
  - The type is checked, the scope is not.
  - A binding where it means nothing is accepted and waits: `minWidth` on a frame without auto-layout, `counterAxisSpacing` on a frame that does not wrap. It takes effect when the node gives it meaning. A LINE takes a `cornerRadius` binding it can never draw.
  - `cornerRadius` bound on a frame reads back as the four corners; `strokeWeight` as the four sides.
  - A paint's `opacity` does not bind: the helper accepts `color` only. A gradient refuses the helper; set `gradientStops[i].boundVariables = { color: alias }` and assign the gradient.
  - **A re-bound paint on an existing node draws the colour it carries, not the variable's.** Start the paint from `variable.resolveForConsumer(node).value`, then bind.
  - Shadows (drop and inner) bind `color`, `radius`, `spread`, `offsetX`, `offsetY`. Blurs bind `radius` only. Texture, glass and noise effects cannot be bound.
  - A layout grid's bindable fields depend on its alignment: columns at `STRETCH` take `count`, `gutterSize`, `offset`; at `CENTER`, not `offset`; at `MIN`, `sectionSize`.

## Typography

- Text and text styles bind `fontFamily`, `fontStyle`, `fontWeight`, `fontSize`, `lineHeight`, `letterSpacing`, `paragraphSpacing`, `paragraphIndent`; ranges with `setRangeBoundVariable` ([working with variables](https://developers.figma.com/docs/plugins/working-with-variables/)).
- *Measured:*
  - A bound `lineHeight` or `letterSpacing` is **pixels**, whatever unit the node had. Assigning `lineHeight` afterwards removes the binding.
  - Unbound, `lineHeight` and `letterSpacing` take `{ unit: 'PERCENT' | 'PIXELS', value }`; a bare number throws. A new text style's `letterSpacing` defaults to PERCENT 0.
  - `fontWeight` is read-only on a text node; bind it. On a family without a weight axis it snaps to a named style.
  - `paragraphSpacing` and `paragraphIndent` bind on a range, although the docs say they do not; only the first paragraph takes it.
  - `setRangeFontName` clears the range's text style. `setRangeTextStyleIdAsync` keeps a range on a style of its own.
  - Set a text's `characters` before its sizing: sized while empty, a text stays 0 wide.

## Colour

- *Measured:* channels are sRGB, gamma-encoded, 0 to 1: `figma.util.rgb('#808080')` gives `r: 0.50196…`. A DTCG `{ colorSpace: 'srgb', components: [0, 0.4, 0.8] }` is written unconverted.
- *Documented:* channels are in the document's colour profile (`documentColorProfile`). In a Display P3 document the same numbers mean P3; Toghen does not convert.
- Update 141 adds `sourceColor` on paints, gradient stops and effects (`SRGB`, `DISPLAY_P3`, `HSL`, `HSB`, `OKLAB`, `OKLCH`); `r`, `g`, `b` remain the projection for the document's profile. Variables are not mentioned ([update 141](https://developers.figma.com/docs/plugins/updates/2026/10/15/version-1-update-141/)). Not measured.
- *Measured:* storage is float32. `0.4` reads `0.4000000059604645`. **Compare with a tolerance, never with equality.**

## Styles

- `figma.createTextStyle()`, `createPaintStyle()`, `createEffectStyle()`; list with `getLocalTextStylesAsync()` and its siblings (the synchronous forms throw under dynamic-page).
- `StyleType` is `PAINT | TEXT | EFFECT | GRID | CUSTOM_ANIMATION` from typings 1.141.0; code that switches on it needs a default.
- A text style binds its typography fields with `setBoundVariable`; paint and effect styles take bound paints and effects as above.
- *Measured:* an imported style behaves like a local one (`setTextStyleIdAsync` works), but `getLocalTextStylesAsync()` does not list it. A style has no `getPublishStatusAsync` at runtime.

## Variables through REST

See [rest-and-mcp.md](rest-and-mcp.md). Enterprise only, reads included.
