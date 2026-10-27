# Components, properties and slots

Part of the `toghen:figma` skill. Pinned to `@figma/plugin-typings` 1.142.0 and Plugin API Update 141; checked 2026-10-23. *Measured* marks what Toghen ran in a real file ([measured behaviour](https://codeberg.org/Toghen/figma/src/branch/main/.reference/figma/measured_behaviour.md), sections 7, 8, 12 and 13). Section 7 came from another project's hand-written scripts and was measured once; treat it as weaker than the rest.

## Component sets

- `figma.combineAsVariants(components, parent)` makes a set. Name each member `Axis=value, Axis=value` **when it is created**. *Measured:* renaming members one by one breaks `variantProperties` partway through.
- *Measured:*
  - Same-named properties on the members merge into one property of the set.
  - The default variant is the member of all first values, placed top-left.
  - A component appended to an existing set joins it, and its properties merge.
  - Deleting a member removes its combination from the axis.
  - A set may hold sparse combinations; Figma warns when a missing one is picked.
  - A deleted member is retained by Figma: `getNodeByIdAsync` on its old id still returns a `COMPONENT` with no parent, on no page.
  - A variant cloned into another set loses its property references; rebind them after moving it.

## The five property types

From [working with component properties](https://developers.figma.com/docs/plugins/working-with-component-properties/):

| Type | Controls | Wired by |
|---|---|---|
| `BOOLEAN` | a layer's visibility | `componentPropertyReferences.visible` |
| `TEXT` | a text layer's characters | `componentPropertyReferences.characters` |
| `INSTANCE_SWAP` | which component a nested instance shows | `componentPropertyReferences.mainComponent` |
| `VARIANT` | the member shown; sets only | the members' names, not references |
| `SLOT` | a free area the instance fills | `componentPropertyReferences.slotContentId` |

- `addComponentProperty(name, type, defaultValue, options?)` returns the property's id with a `#` suffix (`Label#12:3`); a variant axis has no suffix. `options` carries `preferredValues`, `description`, `slotSettings`.
- `editComponentProperty(id, { name, defaultValue, preferredValues, description, slotSettings })`. **Renaming changes the id**; later calls must use the new one.
- `deleteComponentProperty(id)`.
- A variable alias may be a property's default or an instance's value.

*Measured traps:*

- **One name per property across types.** A `BOOLEAN` or `VARIANT` with a slot's name is renamed silently (`Nav` becomes `Nav2`). A variant and a text property of the same name make the set unpublishable, silently: the rest of the library publishes and that set is missing. Check `getPublishStatusAsync()` on every set after a publish.
- **A `BOOLEAN` binds `visible` only.** Anything that changes structure or a value must be a `VARIANT`, with values `"true"` and `"false"`.
- **A `TEXT` property has one default for the whole set.** A text that follows the variant while staying wired cannot exist. An instance's override survives a variant switch.
- **Unwiring** with `componentPropertyReferences = {}` keeps the characters and detaches the property; an instance's override then reaches nothing.
- An `INSTANCE_SWAP` default is the component's node id.

## Slots (generally available since 2026-06-10)

From [component properties](https://developers.figma.com/docs/plugins/working-with-component-properties/), [SlotNode](https://developers.figma.com/docs/plugins/api/SlotNode/) and the [2026-06-10 update](https://developers.figma.com/docs/plugins/updates/page/2/):

- `component.createSlot()` makes a `SlotNode` and its `SLOT` property together. Or add a `SLOT` property and set `slotContentId` on an existing frame.
- `SlotSettings`: `minChildren`, `maxChildren`, `allowPreferredValuesOnly`. `slot.limitViolations` reads `BELOW_MIN`, `ABOVE_MAX`, `HAS_NON_PREFERRED`.
- On an instance, `setProperties` does not fill a slot: edit the slot's children with frame APIs; `resetSlot()` restores the default content.
- `slot.clone()` returns a plain `FrameNode`.
- A slot reference on a non-frame or on a grid frame throws.
- **Docs disagree with each other:** the guide documents `slotContentId`; the `SlotNode` page and typings 1.142.0 list only `visible`, `characters`, `mainComponent`. *Measured:* `slotContentId` is the key Figma uses. Expect a TypeScript cast until the typings catch up.

*Measured:*

- A slot's property is named after the slot node; its key has the `#id` suffix.
- Slots with the same name across members become one property of the set.
- A slot with no preferred values reports `preferredValues: []`.
- Slots clip by default; set `clipsContent = false` for content drawn outside.
- A library slot of fixed height cannot be resized in an instance; it can be set to HUG.
- A slot may not be a grid, but it may sit in one.

## Instances and overrides

- `instance.componentProperties` reads; `instance.setProperties({ id: value })` writes. `removeOverrides()` replaces the deprecated `resetOverrides()` (Update 120) ([component properties](https://developers.figma.com/docs/plugins/working-with-component-properties/), [updates](https://developers.figma.com/docs/plugins/updates/page/2/)).
- `instance.getMainComponentAsync()` reads the main component; under dynamic-page, `instance.mainComponent` is write-only (typings 1.142.0).
- *Not measured in Figma:* what a variant switch does to a size written on an instance, and what binding a `width` or `height` does to a `HUG` axis. Toghen (figma 7.0.0) writes an instance's sizing after its properties and again after a switch, and its `check` refuses a bound size beside `HUG`, `FILL` or a text's `textAutoResize` on that axis. Both were measured in FigCraft 0.10.0, where a switch puts a fixed axis back to the variant's size and a bound size fixes the axis. Since figma 0.1.1, `check` also refuses a frame that hugs an axis every child in its flow fills (FigCraft sets the child back to `FIXED`); and in FigCraft only, an instance set to `HUG` or `FILL` with no size of its own lets its main's bound size go first. Neither is measured in Figma.

## Nested instances: no pass-through

*Measured, and Figma's UI agrees:*

- A host component's property cannot be wired to a nested instance's property. Setting `componentPropertyReferences` with a nested property's key throws (`Unrecognized key(s)`); on a layer inside the nested instance it throws `Cannot set component property references on instance sublayer`.
- **Exposure is the only route:** `nested.isExposedInstance = true` lists the nested instance in the host instance's `exposedInstances`, with its own properties settable there. It does not make a property of the host.

## Layout traps that hit component builds

*Measured:*

- An empty auto-layout frame is 100 px tall until resized.
- `resize()` resets hug sizing: call it first, then set the axis back to HUG.
- `layoutSizingHorizontal` or `layoutSizingVertical` set to `FILL` or `HUG` throws on a node with no auto-layout parent: append first.
- A new frame clips its content; a component read as a set member does not.
- HUG on a component's root is accepted although its parent, the set, is not auto-layout.
- Per-side stroke weights need `strokeAlign = 'INSIDE'`, and `strokeWeight` before the sides.
- Grid tracks: set the count, then each track's type and value. A FLEX track is invalid on an axis where the grid hugs. Bound grid gaps draw only once children are placed in cells (`setGridChildPosition`).
- `layoutWrap = 'WRAP'` works on a `VERTICAL` frame and wraps into columns (documented since Update 140).
- Pattern fills and strokes are set with `setFillsAsync` / `setStrokesAsync` (since Update 123); a plain `fills =` assignment cannot set them.

## Identity across runs

Toghen finds what it built by the stamp in shared plugin data (namespace `toghen`, key `stamp`), not by name or id: a person may rename or move a node. *Measured:* a node renamed by hand keeps its shared plugin data; a copy carries the original's. See SKILL.md, *Plugin data*.
