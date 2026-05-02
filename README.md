# my-skills

A personal collection of [Claude Code](https://docs.claude.com/en/docs/claude-code) skills for the pre-implementation workflow — the part of a project before code gets written.

## Skills

| Skill | What it does |
|---|---|
| [`prior-art-research`](my-skills/skills/prior-art-research/SKILL.md) | Surveys existing solutions for a problem and produces an honest build-vs-reuse recommendation. |
| [`drafting-requirements`](my-skills/skills/drafting-requirements/SKILL.md) | Defines what a system must do (not how) — stakeholder-traced, with explicit non-goals and surfaced conflicts. |
| [`quint-literate-spec`](my-skills/skills/quint-literate-spec/SKILL.md) | Writes a [literate Quint](https://quint.sh/docs/literate) specification — prose and formal model in one markdown file, validated with `lmt` + `quint typecheck`. |

Each skill is standalone — use any one without the others.

## Install

In Claude Code:

```
/plugin marketplace add zdavison/claude-skills
/plugin install my-skills@zdavison
```

Replace `zdavison/claude-skills` with whatever GitHub path this repo lives at.

That's it. Skills become available immediately and are invoked the same as any other skill (e.g. `/prior-art-research`, or by Claude auto-activating on a matching trigger phrase).

## Updates

The plugin manifest pins no version, so Claude Code treats every push to `main` as an update and pulls it on startup.

To force an update without restarting:

```
/plugin marketplace update zdavison
```

## Uninstall

```
/plugin uninstall my-skills@zdavison
/plugin marketplace remove zdavison
```

## Layout

```
.claude-plugin/
  marketplace.json         # marketplace catalog
my-skills/                 # the plugin
  .claude-plugin/
    plugin.json
  skills/
    prior-art-research/SKILL.md
    drafting-requirements/SKILL.md
    quint-literate-spec/SKILL.md
```

Skills under `my-skills/skills/` are auto-discovered by Claude Code — adding a new one is just a new directory with a `SKILL.md`.
