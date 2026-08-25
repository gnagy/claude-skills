# Claude Code skills

Reusable [Claude Code](https://claude.com/claude-code) skills, kept in one place so several projects
can share them instead of each growing its own copy.

| Skill               | Use it when                                                           |
|---------------------|-----------------------------------------------------------------------|
| `bro`               | The last message needs saying again without the jargon                |
| `grill-with-wiki`   | Stress-testing a plan or design, capturing the outcome into the wiki  |
| `markdown-remark`   | Formatting, linting, or validating markdown; before any bulk reformat |
| `technical-writing` | Writing or reviewing a doc, RFC, wiki note, PR description or commit  |
| `unslop`            | Stripping the AI tells out of prose that is about to ship             |
| `wiki-docs`         | Reading or editing the project wiki; wiring a project to that skill   |

`bro` and `technical-writing` are **manual-only** (`disable-model-invocation: true`): they run on
`/bro` and `/technical-writing` and never load themselves. `bro` restates the previous message, which
only makes sense when a human asks; `technical-writing` is a deliberate review pass whose body is too
large to fire on every doc-touching turn. `unslop` stays model-invocable, because it is the catalog
the others defer to.

## Installing

A skill is a directory containing a `SKILL.md`. Claude Code loads them from `~/.claude/skills` (all
projects) or `.claude/skills` (one project). Install from a clone of this repo, run from its root.

Every skill, user-level:

```shell
npx skills add "$PWD" --skill '*' --agent claude-code --agent universal -g -y
```

A single skill:

```shell
npx skills add "$PWD" --skill markdown-remark --agent claude-code --agent universal -g -y
```

Into the current project rather than user-level — drop `-g`:

```shell
npx skills add "$PWD" --skill wiki-docs --agent claude-code --agent universal -y
```

Each of these writes one canonical copy to `~/.agents/skills/<name>/` and symlinks it into the
agent's directory (`~/.claude/skills/`). That layout is the point: one file, many agents, nothing
that can drift.

**Name `universal` whenever you scope agents.** `--agent claude-code` *alone* has no canonical store
to point at, so `skills add` writes a **real directory** into `~/.claude/skills/` instead of a
symlink — abandoning `~/.agents/skills/` and reintroducing exactly the duplication this layout
avoids. Pairing it with `--agent universal` is what keeps one copy and one link.

Verify with `ls -l ~/.claude/skills/`: every entry should be a symlink into `../../.agents/skills/`.
A real directory there is a copy that will drift.

**Installs are snapshots, and that is the point.** `npx skills update` does not refresh local-path
sources, so **re-run `skills add` after editing a skill** — that install step is the only thing that
exposes a change, which is what lets agent jobs keep running against the previous version while a
skill is being worked on.

**Do not symlink a skill into `~/.claude/skills/` to get live edits.** It works, and it destroys that
isolation: every half-finished edit becomes live for every session at once. If a skill needs
tighter feedback than reinstalling gives, reinstall more often.

A session that has already *invoked* a skill keeps its loaded copy until it is invoked again, so
verify a change in a fresh session.

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

**Quote the description if it contains a colon followed by a space.** `description: A layered
standard: Diátaxis for …` is invalid YAML, and `skills add` **skips the skill with a warning and
carries on**, so a bulk install reports "Installed 5 skills" and the sixth is simply absent. Wrap the
value in single quotes. Check the count the installer prints against the number of skill directories.

Two conventions hold across these skills:

- **Skills are project-agnostic.** Anything specific to one project belongs in that project's own
  files. `wiki-docs` is the worked example: the skill describes how a Foam wiki works, while the
  wiki's `meta/` notes define what goes in *this* one, and the `meta/` notes win on conflict.
- **Related skills cross-reference by name and say who owns what.** Foam owns the link graph; remark
  owns the documents. Without that, two skills quietly give contradictory advice about the same file.
- **A manual-only skill cannot be loaded by another skill.** Claude refuses to invoke anything marked
  `disable-model-invocation` and asks the user to run it instead, so **dependencies run one way**: a
  manual skill may load a model-invocable one, never the reverse. `technical-writing` loads `unslop`;
  `unslop` can only ask for `/technical-writing`. Keep the shared catalog model-invocable and the
  deliberate passes manual, or the two ends cannot reach each other.

  Marking the field `false` is the same as omitting it. Omit it.

Skills adapted from someone else's carry a **Lineage** section at the end: where they came from, the
licence, and what was changed. `bro`, `technical-writing`, `unslop` and `grill-with-wiki` all have
one.

Prefer writing down the failure modes over the happy path. The parts of these skills that earn their
place are the hazards — what silently corrupts a document, and how to verify a change before it
touches real files.
