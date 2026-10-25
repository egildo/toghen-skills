# toghen-skills — guide

Skills for an agent that keeps Figma and FigCraft files in step with a repository through Toghen. They are a [Claude Code](https://code.claude.com/docs/en/plugins) plugin, `toghen`, offered by a marketplace of the same name whose manifest sits at the root of this repository.

| Skill | What it teaches |
|---|---|
| `toghen:pipeline` | The pipeline from a repository's token files, resolver, component files and artboard files to CSS, Figma and FigCraft: the commands in order, what needs a person, how a run fails, and what to do then |
| `toghen:figma` | Figma at the version pinned: what the Plugin API, the REST API and Figma's own connector can and cannot do for such a pipeline, and the measured behaviour Toghen relies on |
| `toghen:figcraft` | Toghen into FigCraft at the version pinned: `--figcraft` and `--figcraft-file`, the pinned file, the stamps and the `-toghen` divider, what Toghen builds there and refuses. Driving FigCraft itself is `artcraft:figcraft`'s, in the `egildo/skills` marketplace |

## Install

```bash
claude plugin marketplace add egildo/Toghen
claude plugin install toghen@toghen
```

In a session: `/plugin marketplace add egildo/Toghen`. While working on the skills themselves, load them in place: `claude --plugin-dir toghen-skills/plugins/toghen`, then `/reload-plugins` after an edit.

## Update

The plugin's `version` is the version of the skills' text, not of the software they describe. A user receives a change only when it moves: `claude plugin marketplace update toghen`, then `claude plugin update toghen@toghen`.

Each skill says, in its frontmatter and in a table at its top, which software version it documents and when that was checked. When Figma or FigCraft moves, the skill is checked again against the new version (see [specs.md](specs.md), *Keeping a skill pinned*) and the plugin's version is bumped.
