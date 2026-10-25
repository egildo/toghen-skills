# toghen-skills — specs

## What this package is

A Claude Code plugin marketplace kept inside Toghen. It holds one plugin, `toghen`, whose content is skills: instructions an agent loads when its task matches. The skills teach the use of Toghen's published surface (its command line, its file formats, its plugin) together with the two design apps it targets. They contain no code Toghen runs.

The purity rule holds here as everywhere in Toghen: no skill names a design system, a consumer, a brand or a consumer's paths. Examples use synthetic names.

## Layout

```
.claude-plugin/marketplace.json            the marketplace, at the repository's root
toghen-skills/plugins/toghen/
  .claude-plugin/plugin.json                the plugin: its name and the version of its text
  skills/<name>/SKILL.md                    a skill, under 500 lines
  skills/<name>/*.md                        detail a skill links to, read when needed
```

**Why the manifest is at the repository's root.** Claude Code adds a marketplace from a repository, not from a folder inside one, so `claude plugin marketplace add egildo/Toghen` needs `.claude-plugin/marketplace.json` at the root. Its `source` points into this package, and the skills stay here. *Not chosen:* the manifest inside `skills/`, which works for a local path but needs a setting (`extraKnownMarketplaces` with a `path`) that the documentation does not confirm for a subfolder.

**One plugin, three skills.** One version number for all three. *Not chosen:* a plugin per skill, three versions to keep in step for skills that are used together.

## Keeping a skill pinned

A skill about software that moves weekly has to say which version it describes, or its advice outlives its truth. Every skill pins:

- in its frontmatter, `compatibility` (free text, kept by every way of installing a skill) and `metadata` (`documents`, `version`, `checked`);
- in a table at its top, the same facts with their sources.

The plugin's own `version` counts changes to the text. A skill is checked again when the software it pins moves, or when Toghen changes a command it teaches:

1. Read the new release notes and the documentation at the new version's tag, never at the latest.
2. Measure what changed against the running software, in files of one's own.
3. Update the skill, its pins and its date; bump the plugin's version.
4. Record what changed in [decisions.md](decisions.md).

The research a check rests on is kept in `.reference/skills-research/`, dated.

## Validation

`claude plugin validate .` at the repository's root checks the marketplace; `claude plugin validate toghen-skills/plugins/toghen` checks the plugin and its skills.
