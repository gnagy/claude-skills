# Setting up and bootstrapping a wiki

Reference file for the `wiki-docs` skill. Read it when there is no wiki server configured yet, when
the wiki's `meta/` notes are missing, or when asked to add a wiki to a project. `SKILL.md` beside this
file covers using a wiki that already exists.

## Server setup

The server is configured per project in `.mcp.json`:

```json
{
  "mcpServers": {
    "agent-wiki-toolbox": {
      "command": "awt",
      "args": ["mcp", "--allow-writes", "--workspace", "docs/wiki"]
    }
  }
}
```

`--workspace` is relative to the project root. Drop `--allow-writes` for a read-only wiki. `awt` has
to be on `PATH` — `awt --version` says whether it is, and which install answered.

Adding or changing the server requires restarting the agent session and approving it.

### Mounting other projects' wikis

Mount them as additional servers, with an absolute path and **no** `--allow-writes`:

```json
"awt-homeit": {
  "command": "awt",
  "args": ["mcp", "--workspace", "${HOME}/Ops/homeit/docs/wiki"]
}
```

### The older server, and why a project may still have it

`foam-cli mcp` served this role before the toolbox existed, and a project part-way through the switch
runs **both**: the toolbox as the graph, Foam as an independent check on it. That is deliberate and is
not something to tidy up — the second opinion is the only thing that can catch the first being wrong.

```json
"foam-wiki": {
  "command": "npx",
  "args": ["-y", "foam-cli", "mcp", "--allow-writes", "--workspace", "docs/wiki"]
}
```

Retiring it is the owning project's call, and belongs after the two have been seen to agree across
real edits — not on the day the toolbox is installed.

Which project mounts which is not free choice: mount only down the dependency — see *Which way a
mount points* in `interop.md` beside this file.

`.mcp.json` is committed, so a mount added there is one every clone gets. For a mount that should
exist only on this machine, use `claude mcp add --scope local` (local is the default scope) — it is
stored outside the repo.

A mount is only half of it: what the mounted wikis are for, and the rules for reading and messaging
them, are in `interop.md` beside this file. Declare the mounts in the wiki's own `meta/interop.md`.

## Bootstrapping a new wiki

If the wiki root or its `meta/` notes don't exist yet, create the skeleton — then **stop and agree the
structure with the user** before writing a large number of notes. Layout and taxonomy are much cheaper
to settle before there are fifty notes to migrate.

```
<wiki-root>/
  index.md             # home + index of notes (OKF reserves this name; no front matter)
  meta/conventions.md  # front matter vocabularies, folders, filenames, linking
  meta/scope.md        # what belongs here vs. which repo files own a fact instead
```

`meta/conventions.md` gets the wikilink rule and its OKF deviation note (see *Wikilinks vs. OKF links*
in `SKILL.md`) as part of the skeleton — not something to agree first.

**Propose OKF rather than inventing a vocabulary** — a spec someone else maintains beats one you have
to defend. See *Open Knowledge Format* in `SKILL.md` for what it constrains. If the user prefers their
own:

- **`type`**: a string, one of `note` · `moc` · `adr` · whatever the domain needs.
- **`status`**: `draft` · `stable` · `deprecated`.
- **Filenames**: kebab-case, globally unique basenames — that keeps `[[stem]]` links unambiguous
  regardless of folder.
- **Folders**: start flat. Split only when navigation actually hurts, using `move`.
- **Classification**: tags first. Add front-matter axes (`area`, `domain`, `owner`, …) only when a real
  navigation need appears — each one is a vocabulary someone has to maintain.

Seed the wiki from what the repo already documents (READMEs, docs directories, agent instruction files),
and write down contradictions between sources as open questions rather than resolving them by guesswork.

Add `meta/interop.md` only if the project actually exchanges knowledge with another wiki — see
`interop.md` beside this file for what it must declare.

**Wire the project to the skill in the same pass.** A wiki whose project never loads this skill gets
restructured by whatever the next session improvises. `adoption.md` beside this file carries the block
for the project's `CLAUDE.md` and the guard hook — both are part of bootstrapping, not a later tidy-up.

## The front-matter schema

**A vocabulary written only in `meta/conventions.md` is a vocabulary that drifts.** Put it in a JSON
Schema at the same time you agree it, so a violation fails a check instead of surviving until someone
reads all the notes. What follows is the wiki-shaped starting point, saved as `.remark/note.json` and
referenced from `awt.config.mjs`.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "additionalProperties": false,
  "required": ["title", "type", "description", "status"],
  "properties": {
    "title": { "type": "string", "minLength": 1 },
    "type": { "type": "string", "enum": ["note", "moc", "adr", "reference"] },
    "description": { "type": "string", "minLength": 1 },
    "status": { "type": "string", "enum": ["draft", "stable", "deprecated"] },
    "tags": { "type": "array", "items": { "type": "string" } },
    "sources": {
      "type": "array",
      "items": {
        "type": "object",
        "required": ["resource"],
        "properties": {
          "resource": { "type": "string", "pattern": "^([a-z][a-z0-9-]*:|https?://)" },
          "at": { "type": "string", "format": "date" }
        }
      }
    }
  }
}
```

Four things in there are worth keeping whatever else the project changes:

- **`additionalProperties: false`** — the only thing that stops an invented field from looking
  official. A vocabulary that accepts anything is not a vocabulary.
- **The `sources[].resource` pattern.** OKF requires `sources` to be mappings but says nothing about
  what a `resource` looks like, so `minLength: 1` accepts *"UniFi controller via unifi-network MCP,
  queried 2026-08-11"* — prose in a machine-readable field, which defeats the repo→note direction the
  field exists for. **The scope descriptors are per project; the shape is not.** Fix the shape here —
  a lowercase scheme prefix (`repo:`, `skill:`, `<wiki>:`) or a URL — and let
  `meta/conventions.md` declare which prefixes this project actually uses.
- **`type` as a closed `enum`**, matching whatever `meta/conventions.md` lists. Extend both together
  or neither.
- **`status` as OKF defines it** — `draft` · `stable` · `deprecated`, nothing else.

**Map several schemas to disjoint globs** when folders have genuinely different rules — it is the only
way to require a field in one folder and forbid it in another. Two rules that a schema cannot reach,
because JSON Schema never sees the body: `title` matching the H1, and anything asserting a
front-matter field against something in the prose. Those stay conventions until a check exists.
