# toghen-skills

A Claude Code plugin marketplace, its manifest at the repository's root. It holds the `toghen` plugin: skills for the pipeline from a repository to Figma and FigCraft, and for each app at a pinned version. No code.

Toghen is a design-tokens toolkit split into one repository per package. What the repositories share lives in house.

The rules are in the constitution, at `.specify/memory/constitution.md`: the shared principles and this package's own section. Read it before changing anything. This file holds only what is not a rule.

## The docs

| Doc | What it is |
|---|---|
| `guide.md` | How to use it: setup, workflows, examples |
| `specs.md` | Why it is built this way: architecture, contracts, decisions, open questions |
| `decisions.md` | The archive: dated rulings, what was considered and rejected, what the build found, superseded sections, open questions |

## Commands

```sh
claude plugin validate .                 # the marketplace
claude plugin validate plugins/toghen    # the plugin and its skills
pnpm check                               # coming in the tooling round
```
