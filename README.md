# Claude Code skills

Reusable [Claude Code](https://claude.com/claude-code) skills, kept in one place so several projects
can share them instead of each growing its own copy.

| Skill             | Use it when                                                           |
|-------------------|-----------------------------------------------------------------------|
| `markdown-remark` | Formatting, linting, or validating markdown; before any bulk reformat |
| `wiki-docs`       | Reading or editing a Foam wiki through the `foam-wiki` MCP server     |

## Installing

A skill is a directory containing a `SKILL.md`. Claude Code loads them from `~/.claude/skills` (all
projects) or `.claude/skills` (one project), so either symlink or copy:

```shell
ln -s "$PWD/markdown-remark" ~/.claude/skills/markdown-remark
```

Symlinking keeps every project on the same version, which is usually what you want — the point of
this repo is that fixing a skill once fixes it everywhere.

## Writing one

The front-matter `description` is the whole triggering mechanism: Claude reads it to decide whether
to load the body. Write it as "Use this skill when …" with concrete trigger phrases, and make it
self-contained — it is matched without the body in context.

```markdown
---
name: my-skill
description: Use this skill when … Covers … Also read it before …
---
```

Two conventions hold across these skills:

- **Skills are project-agnostic.** Anything specific to one project belongs in that project's own
  files. `wiki-docs` is the worked example: the skill describes how a Foam wiki works, while the
  wiki's `meta/` notes define what goes in *this* one, and the `meta/` notes win on conflict.
- **Related skills cross-reference by name and say who owns what.** Foam owns the link graph; remark
  owns the documents. Without that, two skills quietly give contradictory advice about the same file.

Prefer writing down the failure modes over the happy path. The parts of these skills that earn their
place are the hazards — what silently corrupts a document, and how to verify a change before it
touches real files.
