# Figma, as documented on 23 October 2026: facts for skill writers

> Research notes, not Toghen documentation. Gathered on 2026-10-23 from Figma's official docs (developers.figma.com, help.figma.com, figma.com/pricing, github.com/figma) and npm. Each bullet names its source. Where Toghen's own measurement (`.reference/figma/measured_behaviour.md`) disagrees with the docs, the measurement is the stronger evidence; the disagreement is listed at the end.
>
> "Documented" below means a Figma page says it. "Measured" means Toghen ran it. Nothing here was measured today.

## Versions pinned

| Thing | Version | Date | Source |
|---|---|---|---|
| Plugin API, manifest `api` field | `"1.0.0"` (still the only value; not upgraded automatically) | n/a | https://developers.figma.com/docs/plugins/manifest/ |
| Plugin API, latest changelog entry | Version 1, Update 141 (`sourceColor`) | 2026-10-15 | https://developers.figma.com/docs/plugins/updates/ |
| `@figma/plugin-typings`, latest on npm | 1.142.0 | published 2026-10-16 | `npm view @figma/plugin-typings` |
| `@figma/plugin-typings`, what Toghen pins | `^1.139.0`, installed 1.139.0 | published 2026-09-23 | `toghen-figma/package.json`, `toghen-figma/node_modules/@figma/plugin-typings/package.json` |
| REST API | No single version. Most endpoints under `/v1/`; webhooks and the new folder endpoints under `/v2/`. Latest changelog entry 2026-10-22 | 2026-10-22 | https://developers.figma.com/docs/rest-api/changelog/ |
| REST API OpenAPI spec, `@figma/rest-api-spec` | 0.46.0 | modified 2026-10-22 | `npm view @figma/rest-api-spec` |
| Figma MCP server, remote | Unversioned; `https://mcp.figma.com/mcp`. Canvas writing (`use_figma`) launched 2026-03-24, still beta | tools page read 2026-10-23 | https://developers.figma.com/docs/figma-mcp-server/tools-and-prompts/ , https://developers.figma.com/docs/figma-mcp-server/write-to-canvas/ |
| Figma MCP server, desktop | Unversioned; `http://127.0.0.1:3845/mcp`, enabled from Dev Mode in the desktop app; read tools only | read 2026-10-23 | https://help.figma.com/hc/en-us/articles/39216419318551-Get-started-with-the-Figma-MCP-server |
| Code Connect CLI, `@figma/code-connect` | 2.0.2 | modified 2026-10-14 | `npm view @figma/code-connect` |
| DTCG spec Figma's native variable import links to | 2025.10 | n/a | https://help.figma.com/hc/en-us/articles/15343816063383-Modes-for-variables |

Mapping of typings to changelog, by publish date (inferred, not stated by Figma): 1.139.0 ≈ Update 139 (composed colours, `COLOR_OPACITY`); 1.141.0 ≈ Update 140 (custom animation styles, vertical wrap); 1.142.0 ≈ Update 141 (`sourceColor`).

### What changed in the typings from 1.139.0 to 1.142.0

Diffed locally (`npm pack` of both, `plugin-api.d.ts`):

- **Custom animation styles.** `figma.createCustomAnimationStyle()`, `getLocalCustomAnimationStylesAsync()`, two move methods; `StyleType` gains `'CUSTOM_ANIMATION'` and `BaseStyle` gains `CustomAnimationStyle`. Code that switches on `style.type` sees a new member (Toghen's two switches have a `default`).
- **Colour with its colour space.** New `ColorWithColorSpace` (`SRGB`, `DISPLAY_P3`, `HSL`, `HSB`, `OKLAB`, `OKLCH`), `RGBWithColorSpace` / `RGBAWithColorSpace` (an optional readonly `sourceColor`). Paint, gradient-stop and effect colours are typed with them. `figma.util.rgb`, `rgba`, `solidPaint` accept the new shapes; `rgba` takes an `alphaOverride`.
- **Layout wrap.** The `layoutWrap` comment now allows `VERTICAL` as well as `HORIZONTAL`.
- **Shader property metadata** (`propertyMetadata`).
- **Not changed:** `componentPropertyReferences` is still typed with `visible`, `characters`, `mainComponent` only; `slotContentId` is still not declared in 1.142.0. `VariableComposedColor`, `COLOR_OPACITY`, `EASING`/`TIMING` types, `SlotNode`, `createSlot()` and the `SLOT` property type were already in 1.139.0.

## Plugin API changelog, last twelve months

Source: https://developers.figma.com/docs/plugins/updates/ and `/page/2/`.

| Date | Entry | What |
|---|---|---|
| 2025-11-06 | Update 120 | Grid: `HUG` for grid containers and tracks, `FLEX` values other than 1. Breaking: setting a track size no longer turns `FLEX` into `FIXED`. Deprecated: `InstanceNode.resetOverrides`, use `removeOverrides`. |
| 2025-11-20 | Update 121 | Extended variable collections (Enterprise): create, override, remove overrides; `variableOverrides`, `mode.parentModeId`; `setValueForMode` on an extended mode sets an override. |
| 2026-01-14 | Update 122 | `rootVariableCollectionId` on extended collections; `slides` and `buzz` editor types documented. |
| 2026-01-26 | Update 123 | Figma Draw APIs; **pattern fills and strokes settable through `setFillsAsync` / `setStrokesAsync`**. |
| 2026-03-26 | Update 124 | `focusedNode` in Dev Mode. |
| 2026-04-22 | Update 125 | `SlotNode` typing gains `clone()`. |
| 2026-05-22 | Update 127 | Grid `gridAutoTracks`, `gridItemsPositioning`, row/column reordering; `noiseSizeVector`; `strokes`/`cornerRadius` on sections. |
| 2026-06-10 | Updates for 2026-06-10 | **Slots generally available**: `SlotNode`, `SlotSettings`, `createSlot()`, `resetSlot()`, `limitViolations`, the `SLOT` component property type. |
| 2026-06-23 | Update 130 | Motion (animation styles, keyframes, timelines); shaders. |
| 2026-07-16 | Update 131 | `exportAsync` to MP4, GIF, WebM. |
| 2026-07-29 | Update 132 | `figma.motion.playheadPosition`. |
| 2026-08-05 | Update 133 | `EASING` and `TIMING` variable types. |
| 2026-08-14 | Update 134 | `textWrapStyle` on text nodes and text styles. |
| 2026-08-24 | Update 135 | Typings maintenance only. |
| 2026-08-27 | Update 136 | `getCSSAsync` restored to the public typings (it always worked at runtime). |
| 2026-08-31 | Update 137 | `SPACE_EVENLY`, `SPACE_AROUND` for `primaryAxisAlignItems`. |
| 2026-09-03 | Update 138 | Variable fonts: `variationSettings`, optional `style` when loading fonts. |
| 2026-09-17 | Update 139 | **Composed colour variables** (`VariableComposedColor`) and the **`COLOR_OPACITY`** scope. |
| 2026-09-30 | Update 140 | Custom animation styles; **`layoutWrap = 'WRAP'` on vertical frames**. |
| 2026-10-15 | Update 141 | **`sourceColor`**: colours keep the space they were authored in. |

No entry in this period touches plugin data, `clientStorage`, `documentAccess`, `networkAccess`, undo or `figma.teamLibrary`.

## Plugin API: variables

- Create a collection with `figma.variables.createVariableCollection(name)`, a variable with `createVariable(name, collection, type)`; modes with `addMode`, `renameMode`; values per mode with `setValueForMode(modeId, value)`. The default mode is `defaultModeId`, which equals `modes[0].modeId`. https://developers.figma.com/docs/plugins/working-with-variables/
- Resolved types: `BOOLEAN`, `COLOR`, `FLOAT`, `STRING`, and since Update 133 `EASING`, `TIMING`. Typings 1.142.0.
- **Modes per collection by plan** (plans table): Professional "up to 10", Organization "up to 20", Enterprise "unlimited modes with extended collections"; Starter has no value in the row, and the modes article says modes are for Education, Professional, Organization and Enterprise. The REST `POST` caps a collection at **40 modes**. The plugin docs give no number. https://help.figma.com/hc/articles/360040328273 , https://help.figma.com/hc/en-us/articles/15343816063383-Modes-for-variables , https://developers.figma.com/docs/rest-api/variables-endpoints/ . The pricing page's text extraction did not map the values to columns unambiguously; Toghen's measurement (10 on a Professional file, error `in addMode: Limited to 10 modes only`) agrees with the mapping above.
- **Scopes** are a picker filter only: they decide which variables a property picker offers, and do not stop a binding made through the API (REST variables page says so explicitly; Toghen measured the same in the plugin). `VariableScope` in 1.142.0: `ALL_SCOPES TEXT_CONTENT CORNER_RADIUS WIDTH_HEIGHT GAP ALL_FILLS FRAME_FILL SHAPE_FILL TEXT_FILL STROKE_COLOR STROKE_FLOAT EFFECT_FLOAT EFFECT_COLOR OPACITY COLOR_OPACITY FONT_FAMILY FONT_STYLE FONT_WEIGHT FONT_SIZE LINE_HEIGHT LETTER_SPACING PARAGRAPH_SPACING PARAGRAPH_INDENT`. `COLOR_OPACITY` puts a `FLOAT` only in colour-opacity pickers. https://developers.figma.com/docs/plugins/updates/2026/09/17/version-1-update-139/
- **Code syntax:** `setVariableCodeSyntax(platform, value)` with platform `WEB`, `ANDROID` or `iOS`; Dev Mode shows it. Typings `CodeSyntaxPlatform`; REST `codeSyntax`.
- **Aliases:** `figma.variables.createVariableAlias(variable)` or `createVariableAliasByIdAsync(id)`, set as a mode's value. https://developers.figma.com/docs/plugins/working-with-variables/
- **Composed colour (new, Update 139):** a `COLOR` variable's value may be `{ color, opacity }` where at least one is an alias; `opacity` is a number or an alias to a `FLOAT`. The docs' example writes opacity `60`, which reads as a percentage; the unit is not stated. https://developers.figma.com/docs/plugins/working-with-variables/
- **Binding:** `node.setBoundVariable(field, variable)` for plain fields (`null` unbinds); `figma.variables.setBoundVariableForPaint`, `setBoundVariableForEffect`, `setBoundVariableForLayoutGrid` return a new object that must be assigned back to `fills`/`strokes`/`effects`/`layoutGrids`; `setRangeBoundVariable` for text ranges, where the docs say `paragraphSpacing` and `paragraphIndent` are not valid; text styles take `setBoundVariable`; `node.boundVariables` reads; `variable.resolveForConsumer(node)` resolves in a node's context. https://developers.figma.com/docs/plugins/working-with-variables/
- **Explicit modes:** `setExplicitVariableModeForCollection(collection, modeId)` on a node pins a mode (typings; the guide does not cover it).
- **Extended collections (Enterprise):** `collection.extend(name)` or `figma.variables.extendLibraryCollectionByKeyAsync(key, name)`; an extended collection inherits modes and variables and holds overrides only (`setValueForMode` with the extended mode id, `valuesByModeForCollectionAsync`, `variableOverrides`, `removeOverrideForMode`, `removeOverridesForVariable`). You cannot add variables or modes, or change a variable's description or scope, in an extended collection. https://developers.figma.com/docs/plugins/working-with-variables/ , https://help.figma.com/hc/en-us/articles/36346281624471-Extend-a-variable-collection
- **Library variables:** `figma.variables.importVariableByKeyAsync(key)` imports a published variable. `figma.teamLibrary` has two methods only: `getAvailableLibraryVariableCollectionsAsync()` (collections of libraries enabled in this file) and `getVariablesInLibraryCollectionAsync(key)`. They throw unless the manifest's `permissions` includes `"teamlibrary"`. Libraries cannot be enabled from the Plugin API. Nothing lists a library's components or styles. https://developers.figma.com/docs/plugins/api/figma-teamlibrary/
- **A variable's key** is present on local and published variables, but only published ones import (typings comment on `Variable.key`).

## Plugin API: styles

- Styles take bound variables: text styles through `setBoundVariable` (typography fields), paint styles through paints bound with `setBoundVariableForPaint`, effect styles through effects bound with `setBoundVariableForEffect`. https://developers.figma.com/docs/plugins/working-with-variables/
- `StyleType` is `PAINT | TEXT | EFFECT | GRID | CUSTOM_ANIMATION` from typings 1.141.0.
- `importStyleByKeyAsync(key)` imports a published style. https://developers.figma.com/docs/plugins/api/figma/
- Under `documentAccess: dynamic-page` the synchronous `getLocal*Styles()` throw; use the `…Async` forms (typings deprecation notes).

## Plugin API: components and properties

- Property types: `BOOLEAN` (visibility), `TEXT` (characters), `INSTANCE_SWAP` (which nested instance), `VARIANT` (component sets only), `SLOT` (GA 2026-06-10). https://developers.figma.com/docs/plugins/working-with-component-properties/
- `addComponentProperty(name, type, defaultValue, options?)` on a component or set returns the property id with a `#` suffix (`IconVisible#4:1`); a variant axis has no suffix. `editComponentProperty(id, {...})` changes name, default, `preferredValues`, `description`, `slotSettings`; **renaming changes the id**. `deleteComponentProperty(id)`. Same source.
- **Wiring** a sublayer: `componentPropertyReferences` with keys `visible`, `characters`, `mainComponent`, and `slotContentId` for a slot. The guide documents `slotContentId`; the typings (1.142.0) and the `SlotNode` reference page still list only the first three. Same source; https://developers.figma.com/docs/plugins/api/SlotNode/
- **Slots:** `componentNode.createSlot()` makes the slot node and its `SLOT` property together; or add a `SLOT` property and set `slotContentId` on an existing frame. `SlotSettings`: `minChildren`, `maxChildren`, `allowPreferredValuesOnly`; `limitViolations` reads `BELOW_MIN`, `ABOVE_MAX`, `HAS_NON_PREFERRED`. On an instance, `setProperties` does not set slot content: edit the slot's children with normal frame APIs, `resetSlot()` restores the default. `clone()` of a slot returns a plain `FrameNode`. Setting a slot reference on a non-frame or a grid frame throws. Same sources.
- Instances: `instance.componentProperties` reads, `instance.setProperties({ id: value })` writes; a variable alias is accepted as a property value or default. `resetOverrides` is deprecated for `removeOverrides` (Update 120). https://developers.figma.com/docs/plugins/working-with-component-properties/
- `importComponentByKeyAsync`, `importComponentSetByKeyAsync` reject when the asset is not published. https://developers.figma.com/docs/plugins/api/figma/

## Plugin API: plugin data and client storage

- `setPluginData(key, value)`: private to the plugin's id, inaccessible if the id changes, readable by anyone who decodes a `.fig`; private "for stability, not security". https://developers.figma.com/docs/plugins/api/properties/nodes-setplugindata/
- `setSharedPluginData(namespace, key, value)`: readable by any plugin; namespace at least 3 alphanumeric characters. https://developers.figma.com/docs/plugins/api/properties/nodes-setsharedplugindata/
- Both: **100 kB per entry** (namespace or plugin id, key and value together); no total limit stated. Available on nodes, `DocumentNode`, `PageNode`, styles, `Variable` and `VariableCollection`. Behaviour on copy is not documented. Same sources.
- The REST file endpoints return plugin data with `plugin_data=<plugin ids>` or `plugin_data=shared`, so shared plugin data can be read without a plugin. https://developers.figma.com/docs/rest-api/file-endpoints/
- `figma.clientStorage`: **5 MB per plugin**, keyed by plugin id, stored **on the user's local machine**, not synced; clearing the browser cache may erase it; values are JSON-like plus `Uint8Array`; `setAsync` rejects on failure. Methods `getAsync`, `setAsync`, `deleteAsync`, `keysAsync`. https://developers.figma.com/docs/plugins/api/figma-clientStorage/

## Plugin API: pages, network, UI, undo

- **`documentAccess: "dynamic-page"`** is required for all new plugins. Only the current page is loaded; `page.loadAsync()` loads another without navigating; `setCurrentPageAsync` loads and navigates; `figma.loadAllPagesAsync()` is needed before document-wide `findAll`, `findOne`, `findAllWithCriteria` and the `documentchange` event, and is discouraged on large files. `getNodeById` (sync) throws under dynamic-page; use `getNodeByIdAsync`. https://developers.figma.com/docs/plugins/manifest/ , https://developers.figma.com/docs/plugins/accessing-document/ , https://developers.figma.com/docs/plugins/api/figma/
- **`networkAccess`:** `allowedDomains` is required once the key is present; `["none"]` blocks everything; `*` and any localhost entry need `reasoning`; schemes `http`, `https`, `ws`, `wss`; wildcard subdomains; `devAllowedDomains` applies only in development. Enforced by CSP. https://developers.figma.com/docs/plugins/manifest/ , https://developers.figma.com/docs/plugins/making-network-requests/
- The main thread is a minimal sandbox with no DOM; the docs show a global `fetch` in plugin code. The default UI iframe has a `null` origin, so only `Access-Control-Allow-Origin: *` APIs answer it. https://developers.figma.com/docs/plugins/how-plugins-run/ , https://developers.figma.com/docs/plugins/making-network-requests/
- **UI messages:** UI to main with `parent.postMessage({ pluginMessage }, '*')`, received by `figma.ui.onmessage = (msg) => …`; main to UI with `figma.ui.postMessage(value)`, read from `event.data.pluginMessage`, queued until the iframe loads. Data travels as if serialized: objects, arrays, numbers, strings, booleans, `null`, `undefined`, `Date`, `Uint8Array`; not `Blob`, `ArrayBuffer` or other typed arrays. https://developers.figma.com/docs/plugins/creating-ui/
- **Undo:** plugin actions are not committed to undo history by default; `figma.commitUndo()` marks a boundary; `figma.triggerUndo()` is documented as reverting to the last commit. https://developers.figma.com/docs/plugins/api/properties/figma-commitundo/ , https://developers.figma.com/docs/plugins/api/figma/ (Toghen measured otherwise; see Contradictions.)
- **Manifest permissions:** `currentuser`, `activeusers`, `fileusers`, `payments`, `teamlibrary`. `enablePrivatePluginApi` unlocks private-plugin APIs, and lets a development plugin use them. `figma.fileKey` is for private plugins and Figma-owned resources only. `enableProposedApi` does not work in published plugins. https://developers.figma.com/docs/plugins/manifest/ , https://developers.figma.com/docs/plugins/api/figma/
- A plugin runs until `figma.closePlugin()`; a plugin that never closes runs indefinitely with a "Running" toast. No time limit is documented. https://developers.figma.com/docs/plugins/how-plugins-run/

## Colour

- Paint, gradient-stop and effect colours may carry `sourceColor` (SRGB, DISPLAY_P3, HSL, HSB, OKLAB, OKLCH); the `r`, `g`, `b` fields remain the projection for the document's colour profile. Variables are not mentioned. https://developers.figma.com/docs/plugins/updates/2026/10/15/version-1-update-141/

## REST API

- **Writes:** no endpoint writes a file's node tree. Writes that exist: variables `POST`, comments, dev resources, webhooks, and (Enterprise, private OAuth apps) file duplication, folders and teams. https://developers.figma.com/docs/rest-api/file-endpoints/ , https://developers.figma.com/docs/rest-api/changelog/
- **File endpoints:** `GET /v1/files/:key` (`version`, `ids`, `depth`, `geometry`, `plugin_data`, `branch_data`), `GET /v1/files/:key/nodes` (`ids` required), `GET /v1/images/:key` (renders), `GET /v1/files/:key/images` (image fills), `GET /v1/files/:key/meta`. Scope `file_content:read` (meta: `file_metadata:read`). Tier 1, except image fills Tier 2 and meta Tier 3. https://developers.figma.com/docs/rest-api/file-endpoints/
- **Variables endpoints:** `GET /v1/files/:key/variables/local`, `GET /v1/files/:key/variables/published` (main file key, not a branch), `POST /v1/files/:key/variables`. All three: "full members of Enterprise orgs"; `POST` also needs edit access. Scopes `file_variables:read` / `file_variables:write`. `POST`: 4 MB body, 40 modes per collection, mode names up to 40 characters, 5,000 variables per collection, one atomic operation (any failure is a 400 and nothing persists), temporary ids mapped in `tempIdToRealId`, applied in the order collections, modes, variables, mode values; supports `codeSyntax`, `scopes`, `hiddenFromPublishing`, extended collections (`parentVariableCollectionId`) and composed colours. Variables written by REST must be published before other files see them. https://developers.figma.com/docs/rest-api/variables-endpoints/ , https://developers.figma.com/docs/rest-api/variables/
- **Rate limits** (since 2025-11-17), per minute unless noted. Tier 1 (files, images): View/Collab 20 per month on all plans; Dev/Full 10 Starter, 15 Professional, 20 Organization. Tier 2 (variables GET, webhooks, comments…): 5; 25 / 50 / 100. Tier 3 (components, styles, metadata, variables POST): 10; 50 / 100 / 150. The Enterprise column was not legible. Limits follow the plan of the resource requested. 429 carries `Retry-After` (seconds), `X-Figma-Plan-Tier`, `X-Figma-Rate-Limit-Type`, `X-Figma-Upgrade-Link`. https://developers.figma.com/docs/rest-api/rate-limits/
- **Personal access tokens:** header `X-Figma-Token`; scopes chosen at creation; **at most 90 days**, non-expiring tokens no longer created (since 2025-04-28). **Plan access tokens** (Organization and Enterprise, GA 2026-07-23) are tied to the plan, not a person. https://developers.figma.com/docs/rest-api/personal-access-tokens/ , https://developers.figma.com/docs/rest-api/changelog/
- **Scopes:** `file_content:read`, `file_metadata:read`, `file_variables:read` and `file_variables:write` (Enterprise), `library_content:read`, `library_assets:read`, `team_library_content:read`, `webhooks:read`, `webhooks:write`, and others. Deprecated: `files:read` (use granular scopes), `projects:read`, `project_metadata:read` (replaced by folder scopes after the 2026-08-10 v2 folder endpoints). https://developers.figma.com/docs/rest-api/scopes/
- **Webhooks v2:** contexts team (admins), folder (`context: project`) and file (edit permission); at most 20 per team, 5 per folder, 3 per file; file-webhook totals 150 Professional, 300 Organization, 600 Enterprise. Events `PING`, `FILE_UPDATE` (within 30 minutes of editing inactivity), `FILE_DELETE`, `FILE_VERSION_UPDATE` (named version), `LIBRARY_PUBLISH` (created/modified/deleted components, styles and **variables**, split into several events per asset type), `FILE_COMMENT`, `DEV_MODE_STATUS_UPDATE`. Retries three times, at 5 minutes, 30 minutes, 3 hours. https://developers.figma.com/docs/rest-api/webhooks/ , https://developers.figma.com/docs/rest-api/webhooks-events/

## Figma MCP server

- Two servers. **Remote** `https://mcp.figma.com/mcp`, recommended, no desktop app, the full tool set. **Desktop** `http://127.0.0.1:3845/mcp`, enabled in Dev Mode with nothing selected; Dev or Full seat on a paid plan; tools `get_screenshot`, `get_design_context`, `get_metadata`, `get_figjam`, `get_variable_defs`, `get_code_connect_map`, `add_code_connect_map`, prompt `create_design_system_rules`; no canvas writing. https://help.figma.com/hc/en-us/articles/39216419318551-Get-started-with-the-Figma-MCP-server
- **Remote-only tools** relevant to tokens and components: `use_figma` (runs Plugin API code in a file), `search_design_system`, `get_libraries`, `create_new_file`, `upload_assets`, `whoami`, `generate_figma_design`, `download_assets`. Both servers: `get_variable_defs` (variables and styles used by a selection), `get_metadata`, `get_design_context`, `get_screenshot`, Code Connect tools. https://developers.figma.com/docs/figma-mcp-server/tools-and-prompts/
- `create_session`, which the connector's own instructions require before file tools, is not on the official tools page as of today.
- **`use_figma`:** beta, free during the beta, to become usage-based paid; Professional, Education, Organization or Enterprise; edit access; a Full seat to write outside drafts; output up to 20 KB per call; cannot add new images (use `upload_assets`); fonts must be uploaded to the Figma account, not just installed locally; Figma advises working on a duplicate or draft. Figma recommends its `figma-use` skill. No code-length limit is documented. https://developers.figma.com/docs/figma-mcp-server/write-to-canvas/
- **Rate limits** (tool calls that read): View/Collab 20 per month on any plan; Dev/Full 200 per day and 10 per minute on Professional and Education, 200/15 on Organization, 600/20 on Enterprise. Exempt: `add_code_connect_map`, `create_new_file`, `generate_figma_design`, `whoami`. `use_figma` is not named as exempt. https://developers.figma.com/docs/figma-mcp-server/rate-limits-access/

## Native variable import and export (help centre)

- Figma imports and exports variable **modes** as DTCG JSON (the article links the 2025.10 format): right-click a mode for *Export mode* / *Import mode*, a collection for *Export modes*; dropping files on a new collection makes one mode per file and a variable only for tokens present in every file with one type. https://help.figma.com/hc/en-us/articles/15343816063383-Modes-for-variables
- Supported on import: `color` (sRGB and HSL only), `dimension` (**px only**), `fontFamily` (**one name; arrays unsupported**), `duration` (s only), `number` (or boolean through `com.figma.type`), and a non-standard `string`. Aliases `{a.b}` within the file; cross-collection aliases through `$extensions["com.figma.aliasData"]`. Group paths become `/` names; on a duplicate the first wins. Same source.

## Automation: what runs without a person

- **Plugin API:** only inside an open Figma editor (desktop or browser), launched by a person; there is no headless runtime and no CI. A plugin acts as the person running it. https://developers.figma.com/docs/plugins/how-plugins-run/
- **`use_figma` (remote MCP):** runs Plugin API code in a file without the desktop app open, but needs an MCP client authenticated as a person (OAuth), counts against MCP limits, and is beta. Not a CI tool by design. https://developers.figma.com/docs/figma-mcp-server/
- **REST:** headless and CI-friendly, but reads only, except variables (`POST`, Enterprise) and a few non-canvas writes. Publishing a library and accepting library updates are not available in any API (Toghen measured; plugin docs say libraries cannot be enabled from the API).
- **Figma's own tokens-pipeline example:** `figma/variables-github-action-example`, two manual GitHub Actions using the Variables REST API with a PAT (read-only or read-write Variables scope); requires a full member of an Enterprise org (an editor seat to push). Tokens in a DTCG-draft-like format, one file per collection and mode. It never deletes, updates in place by name, assumes variable names unique across collections, leaves scopes and code syntax untouched when absent, and gets a 400 on string variables bound to text in a shared organisation font. https://github.com/figma/variables-github-action-example
- **Change notification:** a `LIBRARY_PUBLISH` webhook lists variables created, modified and deleted, so a CI job can react to a publish without polling. https://developers.figma.com/docs/rest-api/webhooks-events/

## Contradictions with Toghen's docs

1. **Mode limits per plan.** `toghen-figma/specs.md`, *Figma's mode limit*, says "4 on Starter, 10 on Professional, 20 on Organization, 40 on Enterprise (as Figma documents it)". The plans table now says Professional 10, Organization 20, Enterprise "unlimited modes with extended collections", and Starter has no value; only the REST `POST` still says 40 per collection. Toghen's ruling (10 for everyone) is unaffected; the sentence is stale. `measured_behaviour.md` §3 says the cap is "not documented anywhere": it is now in the plans table.
2. **`clientStorage` quota and scope.** `specs.md`, *Recently loaded documents*, says "Figma names no `clientStorage` quota" and that recents are kept "per user, not per file". The docs say 5 MB per plugin, on the user's local machine, not synced, lost if the plugin id changes or the cache is cleared. Five bundled documents can approach 5 MB; the shed-oldest-and-retry fallback already handles a refusal, but the stated reason is wrong and "per user" should read "per machine".
3. **Pattern fills.** `measured_behaviour.md` §7 says pattern fills "cannot be set from the Plugin API". Since Update 123 (2026-01-26) `setFillsAsync` / `setStrokesAsync` set them; Toghen's own vendored `api_node_properties.md` already says so.
4. **Vertical wrap.** `measured_behaviour.md` §14, *Settable where the typings say it throws*: `layoutWrap = 'WRAP'` on a `VERTICAL` frame is now documented (Update 140) and typed (1.141.0+). The measurement stands; the heading is stale.
5. **`slotContentId`.** §8 says it is a key "`@figma/plugin-typings` 1.139.0 does not declare". Still undeclared in 1.142.0, but the *Working with component properties* guide now documents it as the slot key. The `SlotNode` reference page still lists only three keys, so Figma's docs disagree with each other.
6. **`COLOR_OPACITY` and a paint's opacity.** `specs.md` (scopes section, around line 620) and `decisions.md` line 804 hold that `COLOR_OPACITY`'s unit "cannot be measured" from code, and §14 that a paint's opacity does not bind. Update 139 defines `COLOR_OPACITY` as a `FLOAT` scope for colour-opacity pickers, and introduces composed colour variables: a `COLOR` variable whose value is `{ color, opacity }`, with opacity a number or an alias to a `FLOAT`. The docs' example uses `60`, which points to 0–100, though the unit is not stated. So `setBoundVariableForPaint` still binds `color` only, but a paint's opacity can now follow a variable through a composed colour. The open question can now be answered from code: write a composed colour and read back what it resolves to.
7. **Display P3 caveat.** §1 and `figmaTokenAdapter.ts` note that colours are in the document's profile and that Toghen does not handle a P3 document. Update 141 adds `sourceColor` on paints, gradient stops and effects (OKLCH, OKLAB, P3 and others), and says `r`/`g`/`b` stay the profile's projection. The caveat holds; there is now an API that could write a DTCG `colorSpace` as authored, for paints and effects, not documented for variables.
8. **Undo.** The docs say `triggerUndo()` reverts to the last `commitUndo()`. §8 measured it reverting past the commit in five trials of six. Not stale: the measurement contradicts the docs, and Toghen's rule (never undo) should stay. A skill must not repeat the docs' claim.
9. **`teamlibrary` permission.** §15 left open whether the desktop plugin needs it. The docs say the `figma.teamLibrary` methods throw without it. Documented, not measured; the manifest already declares it.
10. **Vendored snapshots.** `.reference/figma/figma_dev_docs_TOC.md` gives captures of 2026-08-04 and 2026-03-10. They predate Updates 134–141: `textWrapStyle`, `getCSSAsync` in the typings, variable fonts, composed colours, `COLOR_OPACITY`, custom animation styles, `sourceColor`. The variables snapshots need a refresh.
11. **Typings pin.** `^1.139.0`, installed 1.139.0; latest 1.142.0. An upgrade adds `CUSTOM_ANIMATION` to `StyleType` and `CustomAnimationStyle` to `BaseStyle`. Toghen's two `switch (style.type)` (`main/utils/tokenStamps.ts`, `main/engines/operationExecutor.ts`) have defaults, but `pnpm typecheck` against 1.142.0 was not run.
12. **Headless reads of stamps.** `specs.md`, *Why there is no Figma sign-in*, is right that REST cannot write a file and that variables endpoints are Enterprise-only (confirmed: all three, reading included). It does not mention that `GET /v1/files/:key?plugin_data=shared` reads shared plugin data, Toghen's stamps and lock included, with a PAT on any plan. Not a contradiction; a capability the docs leave out.

## Open questions

- The exact plan-to-column mapping of the modes row: two text extractions of the plans page disagreed. One read Starter 10, Professional 20; the other Starter blank, Professional 10, Organization 20, which agrees with Toghen's measurement. Someone should read the live table by eye.
- Whether Enterprise has a per-collection cap in the Plugin API (REST says 40; the plans page says "unlimited with extended collections").
- The unit of a composed colour's `opacity` and of a `COLOR_OPACITY` variable (0–100 suggested by an example, not stated).
- Whether `sourceColor` applies to variable values, and whether writing it changes what `figma.util.rgb` returns.
- Why the `SlotNode` page and the typings omit `slotContentId` while the guide documents it; whether a typings release will add it.
- Whether `use_figma` counts against the MCP read limits (not listed as exempt, not listed as counted).
- `create_session` is used by the connector but not on Figma's tools page; whether it is a newer, still undocumented tool.
- Whether plugin data, shared or private, travels with a copied node or variable (Toghen measured that a copy carries the stamp; the docs say nothing).
- The status of a native, whole-collection DTCG export: the help centre covers modes; a community thread says the full feature was still rolling out in late 2025.
