# The REST API and Figma's MCP server

Part of the `toghen:figma` skill. REST as of its changelog of 2026-10-22; MCP server pages read 2026-10-23. *Measured* marks what Toghen ran ([measured behaviour](https://codeberg.org/Toghen/figma/src/branch/main/.reference/figma/measured_behaviour.md), section 8).

## REST: what it can and cannot do

- **No endpoint writes a file's node tree.** File endpoints are reads. Writes that exist: variables (`POST`, Enterprise), comments, dev resources, webhooks, and for Enterprise private OAuth apps, file duplication, folders and teams ([file endpoints](https://developers.figma.com/docs/rest-api/file-endpoints/), [changelog](https://developers.figma.com/docs/rest-api/changelog/)).
- Most endpoints live under `/v1/`; webhooks and the folder endpoints (2026-08-10) under `/v2/`. The v1 project endpoints are deprecated for the v2 folder ones.

### File endpoints

From [file endpoints](https://developers.figma.com/docs/rest-api/file-endpoints/):

| Endpoint | Use | Scope | Tier |
|---|---|---|---|
| `GET /v1/files/:key` | the document; `version`, `ids`, `depth`, `geometry`, `plugin_data`, `branch_data` | `file_content:read` | 1 |
| `GET /v1/files/:key/nodes?ids=…` | chosen nodes and their subtrees; `plugin_data` too | `file_content:read` | 1 |
| `GET /v1/images/:key?ids=…` | renders | `file_content:read` | 1 |
| `GET /v1/files/:key/images` | image fills | `file_content:read` | 2 |
| `GET /v1/files/:key/meta` | metadata | `file_metadata:read` | 3 |

- `plugin_data=shared` returns `sharedPluginData` on every node; `plugin_data=<plugin id>` returns that plugin's private data. This is how a CI job reads Toghen's stamps (namespace `toghen`, key `stamp`) and lock without Figma open.

### Variables endpoints (Enterprise only)

From [variables endpoints](https://developers.figma.com/docs/rest-api/variables-endpoints/):

- `GET /v1/files/:key/variables/local`, `GET /v1/files/:key/variables/published` (a main file key, not a branch), `POST /v1/files/:key/variables`.
- All three: available to "full members of Enterprise orgs". `POST` also needs edit access. Scopes `file_variables:read`, `file_variables:write`.
- `POST` limits: 4 MB body; 40 modes per collection; mode names up to 40 characters; 5,000 variables per collection.
- `POST` is one atomic operation: any validation error is a 400 and nothing persists. Applied in the order collections, modes, variables, mode values. Temporary ids are mapped back in `tempIdToRealId`.
- Supports `codeSyntax`, `scopes`, `hiddenFromPublishing`, extended collections (`parentVariableCollectionId`) and composed colours.
- Variables written by REST must be **published** before other files see them, and publishing is a person's step.

### Rate limits

From [rate limits](https://developers.figma.com/docs/rest-api/rate-limits/), in force since 2025-11-17; per minute unless noted:

| Tier | View or Collab seat, any plan | Dev or Full, Starter | Professional | Organization |
|---|---|---|---|---|
| 1 (files, images) | 20 per month | 10 | 15 | 20 |
| 2 (variables GET, webhooks, comments) | 5 | 25 | 50 | 100 |
| 3 (components, styles, metadata, variables POST) | 10 | 50 | 100 | 150 |

- The Enterprise column was not legible in the source. Limits follow the plan of the file requested, not only the caller's seat.
- A 429 carries `Retry-After` (seconds), `X-Figma-Plan-Tier`, `X-Figma-Rate-Limit-Type`, `X-Figma-Upgrade-Link`. Honour `Retry-After`.
- A View or Collab seat gets 20 file reads a **month**. A pipeline token must belong to a Dev or Full seat.

### Tokens and scopes

- Personal access tokens go in the `X-Figma-Token` header. Scopes are chosen at creation. **At most 90 days**; non-expiring tokens can no longer be made (since 2025-04-28) ([personal access tokens](https://developers.figma.com/docs/rest-api/personal-access-tokens/), [changelog](https://developers.figma.com/docs/rest-api/changelog/)).
- Plan access tokens (Organization, Enterprise; GA 2026-07-23) belong to the plan, not a person.
- Scopes a tokens pipeline meets: `file_content:read`, `file_metadata:read`, `file_variables:read` / `file_variables:write` (Enterprise), `library_content:read`, `team_library_content:read`, `webhooks:read` / `webhooks:write`. `files:read` is deprecated; use the granular ones ([scopes](https://developers.figma.com/docs/rest-api/scopes/)).

### Webhooks (v2)

From [webhooks](https://developers.figma.com/docs/rest-api/webhooks/) and [webhook events](https://developers.figma.com/docs/rest-api/webhooks-events/):

- Contexts: team (admins), folder (the `context` value is `project`; edit permission), file (edit permission). At most 20 per team, 5 per folder, 3 per file; file webhooks in all: 150 Professional, 300 Organization, 600 Enterprise.
- Events: `PING`, `FILE_UPDATE` (within 30 minutes of editing inactivity), `FILE_DELETE`, `FILE_VERSION_UPDATE` (a named version), `LIBRARY_PUBLISH`, `FILE_COMMENT`, `DEV_MODE_STATUS_UPDATE`.
- `LIBRARY_PUBLISH` lists created, modified and deleted components, styles and variables (`key`, `name`), and one publish may arrive as several events.
- Your endpoint must answer 200 quickly. Figma retries three times: after 5 minutes, 30 minutes, 3 hours.

## Figma's MCP server

### Two servers

From [get started](https://help.figma.com/hc/en-us/articles/39216419318551-Get-started-with-the-Figma-MCP-server) and [MCP server](https://developers.figma.com/docs/figma-mcp-server/):

- **Remote**, `https://mcp.figma.com/mcp`: recommended; no desktop app needed; sign in with a Figma account; the full tool set, writing included.
- **Desktop**, `http://127.0.0.1:3845/mcp`: enabled in the desktop app, in Dev Mode with nothing selected; a Dev or Full seat on a paid plan; read tools only (`get_screenshot`, `get_design_context`, `get_metadata`, `get_figjam`, `get_variable_defs`, `get_code_connect_map`, `add_code_connect_map`, prompt `create_design_system_rules`).

### Tools that matter for a tokens pipeline

From [tools and prompts](https://developers.figma.com/docs/figma-mcp-server/tools-and-prompts/):

| Tool | What | Server |
|---|---|---|
| `use_figma` | runs Plugin API code in a file: create, edit, inspect | remote |
| `get_variable_defs` | variables and styles used by a selection | both |
| `get_metadata` | a light XML outline of a selection or the pages | both |
| `search_design_system` | search libraries for components, variables, styles | remote |
| `get_libraries` | subscribed and available libraries | remote |
| `get_screenshot` | an image of one node | both |
| `create_new_file` | a blank file in your drafts | remote |
| `whoami` | your identity, plans, seats | remote |

The connector may also require a `create_session` call before file tools. That tool is not on Figma's tools page as of 2026-10-23; follow the connector's own instructions.

### `use_figma`

- Beta since 2026-03-24, free during the beta, to become usage-based paid. Professional, Education, Organization or Enterprise; edit access; a Full seat to write outside your drafts ([write to canvas](https://developers.figma.com/docs/figma-mcp-server/write-to-canvas/)).
- *Documented:* output up to 20 KB per call; cannot add new images (use `upload_assets`); fonts must be uploaded to the Figma account; test on a duplicate or draft. Figma's `figma-use` skill is advised before calling it.
- *Measured:*
  - The code is limited to 50,000 characters; a longer script is refused before it runs.
  - A return value is cut at about 20 kB: summarise large reports.
  - `figma.loadAllPagesAsync()` is not supported. Load each page with `loadAsync()` and search per page.
  - `fetch`, `DecompressionStream` and `TextDecoder` are not defined. `Object.hasOwn` and `BigInt` are.
  - `new Function(source)` works, so code kept in shared plugin data can be evaluated by a later call.
  - A failed script rolls back entirely (reported once, by another project's scripts). Never run two scripts at once against one file.
  - The page context resets on every call: `await figma.setCurrentPageAsync(page)` at most once per script.
  - `figma.fileKey` reads through the connector, which proves nothing for a plugin without `enablePrivatePluginApi`.
  - `figma.root.name` reads `Document`, not the file's name.
- *Measured:* what `use_figma` allows is not always what a plugin under `dynamic-page` allows (see SKILL.md, *Pages*). Code tried in `use_figma` must still be checked in the plugin.

### Rate limits

From [rate limits and access](https://developers.figma.com/docs/figma-mcp-server/rate-limits-access/), counting tool calls that read:

| Seat | Starter | Professional, Education | Organization | Enterprise |
|---|---|---|---|---|
| View, Collab | 20 per month | 20 per month | 20 per month | 20 per month |
| Dev, Full | not listed | 200 per day, 10 per minute | 200 per day, 15 per minute | 600 per day, 20 per minute |

Exempt: `add_code_connect_map`, `create_new_file`, `generate_figma_design`, `whoami`. `use_figma` is not named either way.
