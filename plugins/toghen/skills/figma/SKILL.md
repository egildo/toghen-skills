---
name: figma
description: Use when running, planning or debugging a Toghen pipeline into Figma, or when reasoning about what Figma itself allows. Covers which door reaches a file (the Plugin API in an open editor, the REST API, Figma's MCP server and its use_figma tool), who must be present and on which plan, variables and their mode limits, scopes and bindings, styles, components and their properties, slots, shared plugin data where Toghen keeps its stamps, dynamic page loading, libraries (what no API can do), undo, and what can run unattended. Prefers Toghen's measurements over Figma's docs where they disagree. For the pipeline's commands, use toghen:pipeline.
license: MIT
compatibility: "Documents Figma Plugin API 1.0.0 (Update 141), @figma/plugin-typings 1.142.0, REST API as of 2026-10-22; checked 23 October 2026"
metadata:
  documents: figma
  version: plugin-typings 1.142.0
  checked: "2026-10-23"
allowed-tools: Read, Grep, Glob, Bash, WebFetch
---

# Figma, for a Toghen pipeline

What an agent must know about Figma itself to run or reason about a pipeline that writes tokens and components into a Figma file. The commands, their order and their flags belong to **toghen:pipeline**; this skill does not restate them.

## Pinned

| Thing | Version | Date | Source |
|---|---|---|---|
| Plugin API (manifest `api`) | `1.0.0`, latest entry Update 141 | 2026-10-15 | [plugin updates](https://developers.figma.com/docs/plugins/updates/) |
| `@figma/plugin-typings` | 1.142.0 (Toghen builds against ^1.142.0) | 2026-10-16 | npm `@figma/plugin-typings` |
| REST API | `/v1` and `/v2` endpoints, changelog to 2026-10-22 | 2026-10-22 | [REST changelog](https://developers.figma.com/docs/rest-api/changelog/) |
| REST OpenAPI spec | `@figma/rest-api-spec` 0.46.0 | 2026-10-22 | npm `@figma/rest-api-spec` |
| Figma MCP server | unversioned; `use_figma` beta since 2026-03-24 | read 2026-10-23 | [tools and prompts](https://developers.figma.com/docs/figma-mcp-server/tools-and-prompts/) |
| Toghen's measurements | `measured_behaviour.md`, sections 1 to 15 | 2026-08-03 to 2026-10-22 | [measured behaviour](https://codeberg.org/Toghen/figma/src/branch/main/.reference/figma/measured_behaviour.md) |

Checked on 23 October 2026. If Figma has published an update since, check its entry in the plugin updates list before trusting a detail below.

**Which source wins.** Where Toghen measured something in a real file, the measurement holds over the docs, and this skill says so. Where Figma documented something Toghen has not measured, the docs hold, marked *documented*.

## Detail files

Read the one the task needs:

- [variables-and-styles.md](variables-and-styles.md): collections, modes and their limits, scopes, code syntax, aliases, binding fields, composed colours, colour spaces, styles, the measured traps.
- [components-and-slots.md](components-and-slots.md): component sets, the five property types, wiring, slots, instances, nested instances, the measured traps.
- [rest-and-mcp.md](rest-and-mcp.md): REST endpoints, plans, rate limits, tokens and scopes, webhooks; the MCP server's tools, limits and `use_figma`'s measured limits.

## Three doors into a file

| | Plugin API | REST API | Figma MCP server |
|---|---|---|---|
| Runs where | inside an open Figma editor, desktop or browser | anywhere, over HTTPS | remote server `mcp.figma.com/mcp`, or a desktop server at `127.0.0.1:3845` |
| Who must be present | a person who opened the file and ran the plugin | nobody; a token | a person's MCP client, signed in with OAuth |
| Writes the canvas | yes: nodes, components, variables, styles | **no**; only variables (`POST`), comments, dev resources, webhooks | remote `use_figma` runs Plugin API code in a file; the desktop server cannot write |
| Variables | read and write on every plan that has them | **Enterprise only**, reading included | through `use_figma` |
| Shared plugin data | read and write | read, with `plugin_data=shared` | through `use_figma` |
| Unattended (CI) | no | yes | no (a person's session, beta, rate limited) |

Sources: [how plugins run](https://developers.figma.com/docs/plugins/how-plugins-run/), [file endpoints](https://developers.figma.com/docs/rest-api/file-endpoints/), [variables endpoints](https://developers.figma.com/docs/rest-api/variables-endpoints/), [MCP server](https://developers.figma.com/docs/figma-mcp-server/), [get started with MCP](https://help.figma.com/hc/en-us/articles/39216419318551-Get-started-with-the-Figma-MCP-server).

This is why Toghen's command line has no Figma sign-in: no credential lets an outside process build a component set. Toghen reaches a file through its own plugin, open in the editor, with the command line talking to the plugin's UI over a localhost socket. The person who opened the plugin is the authentication.

## Plans and seats

- **Modes per collection** ([plans table](https://help.figma.com/hc/articles/360040328273)): Professional up to 10, Organization up to 20, Enterprise unlimited through extended collections; Starter shows none. The REST `POST` caps a collection at 40 modes.
  - *Measured:* a Professional file stops the 10th `addMode` (11 modes in all) with `in addMode: Limited to 10 modes only`.
  - Toghen treats 10 as the limit for every plan, by its own ruling. A modifier with more contexts must list at most 10 for Figma.
- **Variables REST API:** "full members of Enterprise orgs" for every endpoint, reads included ([variables endpoints](https://developers.figma.com/docs/rest-api/variables-endpoints/)).
- **Libraries** (publish components, styles, variables): Professional and up ([plans table](https://help.figma.com/hc/articles/360040328273)).
- **Extended collections:** Enterprise ([working with variables](https://developers.figma.com/docs/plugins/working-with-variables/)).
- **MCP canvas writing:** Professional, Education, Organization or Enterprise; a Full seat to write outside your drafts; edit access to the file ([write to canvas](https://developers.figma.com/docs/figma-mcp-server/write-to-canvas/)).

## The plugin's manifest

Toghen's plugin declares `editorType: ["figma"]`, `documentAccess: "dynamic-page"`, `permissions: ["teamlibrary"]`, `enablePrivatePluginApi: true`, and `networkAccess` with `allowedDomains: ["none"]` and `devAllowedDomains: ["ws://localhost:3055"]`. What each means ([manifest](https://developers.figma.com/docs/plugins/manifest/)):

- **`documentAccess: "dynamic-page"`** is required for every new plugin. See *Pages*, below.
- **`teamlibrary`** is required for `figma.teamLibrary`; its methods throw without it (*documented*, not measured in the desktop plugin).
- **`enablePrivatePluginApi`** lets a development plugin use private-plugin APIs. `figma.fileKey` reads only in a private plugin or such a development plugin; a published public plugin would lose it ([figma global](https://developers.figma.com/docs/plugins/api/figma/)).
- **`networkAccess`** is enforced by CSP. `["none"]` blocks every domain. A localhost entry in `allowedDomains` needs a `reasoning` string. `devAllowedDomains` holds only while the plugin is in development, which is how Toghen's bridge port is declared ([making network requests](https://developers.figma.com/docs/plugins/making-network-requests/)).
  - *Measured:* a plugin UI opens a WebSocket to localhost and a local process drives the Plugin API through it. Payloads of 10 million characters arrived whole. The socket's `Origin` is `null`. When the server goes away, the plugin stays open and reconnects when it returns.

## The UI and the main thread

- The main thread reaches the document and has no DOM. The UI is an iframe with browser APIs and no document access; by default its origin is `null`, so only APIs that answer any origin respond to it ([how plugins run](https://developers.figma.com/docs/plugins/how-plugins-run/)).
- UI to main: `parent.postMessage({ pluginMessage }, '*')`, received by `figma.ui.onmessage`. Main to UI: `figma.ui.postMessage(value)`, read from `event.data.pluginMessage`; queued until the iframe loads ([creating UI](https://developers.figma.com/docs/plugins/creating-ui/)).
- Messages travel as if serialized: plain objects, arrays, numbers, strings, booleans, `null`, `undefined`, `Date`, `Uint8Array`. No `Blob`, no `ArrayBuffer`, no class instances (same source).
- A plugin runs until `figma.closePlugin()`; no time limit is documented ([how plugins run](https://developers.figma.com/docs/plugins/how-plugins-run/)).

## Pages

Under `dynamic-page`, only the current page is loaded ([accessing the document](https://developers.figma.com/docs/plugins/accessing-document/)).

- `page.loadAsync()` loads a page without moving the person to it. `figma.setCurrentPageAsync(page)` loads and navigates.
- `figma.loadAllPagesAsync()` is needed before `findAll`, `findOne` or `findAllWithCriteria` on `figma.root`, and before the `documentchange` event. Figma advises against it on large files.
- Synchronous lookups throw: use `getNodeByIdAsync`, `getLocalVariablesAsync`, `getLocal*StylesAsync`.
- *Measured, and it holds over intuition:* after `loadAsync()` on every page, the plugin still may not call `findAllWithCriteria` on the document; the error names `loadAllPagesAsync`. Searching each loaded page is allowed. `use_figma` does allow the document-wide search, and does not support `loadAllPagesAsync`. **Search page by page; it works in both.**

## Plugin data: where Toghen's stamps live

- **Shared plugin data** (`setSharedPluginData(namespace, key, value)`) is readable by every plugin. Namespace of at least 3 alphanumeric characters. ([setSharedPluginData](https://developers.figma.com/docs/plugins/api/properties/nodes-setsharedplugindata/))
- **Private plugin data** (`setPluginData`) is readable only by the plugin with the same id, and lost if the id changes ([setPluginData](https://developers.figma.com/docs/plugins/api/properties/nodes-setplugindata/)).
- Both: **100 kB per entry**, no stated total. Both work on nodes, pages, the document, styles, variables and variable collections.
- Toghen writes **shared** data, namespace `toghen`: key `stamp` on each variable, style and node it wrote, key `lock` (sharded) on the document root. Shared, because `use_figma` and Toghen's plugin are different plugins and both must read it.
- **REST reads it without Figma open:** `GET /v1/files/:key?plugin_data=shared` (or `/nodes?ids=…&plugin_data=shared`) returns `sharedPluginData` ([file endpoints](https://developers.figma.com/docs/rest-api/file-endpoints/)). This needs only `file_content:read`, on any plan. It is a read: nothing can write a stamp from outside Figma.
- *Measured:* a node renamed by hand keeps its shared plugin data. A person's copy of a variable, style or node carries the original's data; the docs say nothing about copies.
- **`figma.clientStorage`** is different: 5 MB per plugin, on the person's machine, not synced, lost with the browser cache or a new plugin id ([clientStorage](https://developers.figma.com/docs/plugins/api/figma-clientStorage/)). Toghen keeps its recent documents there. Never keep anything another machine or person needs there.

## Libraries: what no API can do

- **Publishing a library, enabling it in a file, and accepting its updates are a person's steps.** No API does any of them ([teamLibrary](https://developers.figma.com/docs/plugins/api/figma-teamlibrary/) says libraries cannot be enabled from the Plugin API; Toghen measured the rest).
- `figma.teamLibrary` has two methods: `getAvailableLibraryVariableCollectionsAsync()` and `getVariablesInLibraryCollectionAsync(key)`. *Measured:* each entry is exactly `{ name, libraryName, key }` and `{ name, resolvedType, key }`; `libraryName` is the library file's name; an unknown collection key resolves to `[]`, it does not reject.
- Nothing lists a library's components or styles. They import by key only: `importComponentByKeyAsync`, `importComponentSetByKeyAsync`, `importStyleByKeyAsync`, `figma.variables.importVariableByKeyAsync`.
- *Measured:*
  - every local object has a 40-hex key, published or not;
  - an unpublished key does not import, even in its own file, and the error is the same as for a key that never existed;
  - a published variable imports without the library being enabled; enabling only makes the listing work;
  - a variable's or style's imported id is stable across calls, but the `Variable` object is a new one each time (`===` is false), so compare ids;
  - an imported component's node id changes between calls;
  - imported objects cannot be `remove()`d. With the library not enabled, unused imports dropped out within minutes.
- *Measured:* a style has no `getPublishStatusAsync` at runtime, although the typings give it one. An imported variable's `getPublishStatusAsync` throws.
- **In Toghen (figma 0.1.0):** a run takes names from a library only when `--use "<library name>"` declares it, by the name the target's lock holds; it imports by the lock's keys. Preferred values by `name=` are refused by name in Figma until the desktop app shows Figma takes a resolved component's key, local ones included; write `key=`. The commands are `toghen:pipeline`'s.
- **A library collection's mode is not pinned in Figma yet.** No API imports a collection by key: the route would be to import one of its variables by key, read its collection (*measured:* it comes with its modes, `remote: true`), and call `setExplicitVariableModeForCollection` on it. Importing is measured; pinning a remote collection is not. So Toghen refuses a `modes` line naming a `--use` library's collection, a dry run as a real one: `… a library collection's mode ("<collection>" in library "<library>") is not measured in Figma yet; pin it on FigCraft, or use a local collection`. FigCraft pins it.

## Undo: trust the measurement, not the docs

- *Documented:* plugin actions are not committed to undo history by default; `figma.commitUndo()` marks a boundary; `figma.triggerUndo()` reverts to the last commit ([commitUndo](https://developers.figma.com/docs/plugins/api/properties/figma-commitundo/), [figma global](https://developers.figma.com/docs/plugins/api/figma/)).
- *Measured, and it holds:* in five trials of six, `triggerUndo()` also reverted work from **before** the last `commitUndo()`, including a frame made before the commit. An undo after a failed run can delete a person's earlier work.
- So: **never use `triggerUndo()` to roll a run back.** Toghen does not. A run that fails partway leaves what it wrote, and the next run, which reconciles by stamp, completes it. Through `use_figma`, a failed script rolls back entirely by itself (*measured*).

## Automation limits

- **Nothing writes a Figma canvas unattended.** The Plugin API needs an open editor and a person who launched the plugin. `use_figma` needs a person's signed-in MCP client and is beta, to become usage-based paid ([write to canvas](https://developers.figma.com/docs/figma-mcp-server/write-to-canvas/)).
- **REST is the only unattended door**, and it reads only, except the Enterprise variables `POST`. A CI job can read a file, render images, read shared plugin data (stamps) and receive webhooks.
- **Webhooks:** `LIBRARY_PUBLISH` lists the components, styles and variables created, modified and deleted; `FILE_UPDATE` fires within 30 minutes of editing inactivity ([webhook events](https://developers.figma.com/docs/rest-api/webhooks-events/)). A CI job can react to a publish without polling.
- **Personal access tokens last at most 90 days**; plan access tokens (Organization, Enterprise) are not tied to a person ([REST changelog](https://developers.figma.com/docs/rest-api/changelog/)). A pipeline secret expires; plan for it.
- **Figma's own tokens pipeline** (`figma/variables-github-action-example`) uses the variables REST API and therefore needs Enterprise. It updates by name, never deletes, and assumes variable names unique across collections ([repository](https://github.com/figma/variables-github-action-example)).
- **Figma's native import** of modes as DTCG JSON exists, with limits: dimensions in px only, one font family name (no arrays), colours in sRGB or HSL ([modes for variables](https://help.figma.com/hc/en-us/articles/15343816063383-Modes-for-variables)). Toghen does not use it, and does not import from Figma.

## Where the docs and Toghen disagree

| Topic | Docs say | Toghen measured or holds | Which holds |
|---|---|---|---|
| `triggerUndo` | reverts to last commit | reverted past it, 5 of 6 trials | the measurement |
| Whole-document search | `loadAsync` per page is the dynamic-page route | plugin still needs `loadAllPagesAsync` for document-wide `findAllWithCriteria`; `use_figma` does not | the measurement: search per page |
| Style publish status | every style has `getPublishStatusAsync` (typings) | absent at runtime | the measurement |
| Slot wiring key | guide: `slotContentId`; typings 1.142.0 and the `SlotNode` page: not listed | `slotContentId` is what Figma uses | the guide and the measurement |
| `layoutWrap` on a vertical frame | allowed since Update 140 | accepted, measured before the docs said so | both agree now |
| Pattern fills | settable with `setFillsAsync` since Update 123 | an older note said they cannot be set | the docs; the note is stale |
| Mode limits | Professional 10, Organization 20 | 10 on Professional, measured | both; Toghen uses 10 everywhere |
| `clientStorage` quota | 5 MB per plugin, per machine | an older note said no quota is named | the docs |

## Before acting on a file

1. Know the plan. Variables REST and extended collections need Enterprise; more than 10 modes needs more than Professional.
2. Know who is present. Writing needs a person in the editor (plugin) or a person's MCP session (`use_figma`).
3. Never undo a run. Re-run instead.
4. Never assume a library step happened. Publishing, enabling and accepting updates are a person's.
5. Work on a duplicate or a draft first when trying something new; Figma advises the same for `use_figma`.
