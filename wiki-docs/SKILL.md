---
name: wiki-docs
description: Use this skill to read, search, or edit the project wiki — a Foam markdown knowledge base served by the foam-wiki MCP server, usually also an Open Knowledge Format (OKF) bundle. Use it to look up context before making a change, and to capture new knowledge as linked notes. Covers the foam-wiki MCP tools, the lookup and editing workflow, OKF conformance and its front-matter vocabularies, and how to bootstrap a wiki in a project that doesn't have one yet.
---

# Wiki Docs

Use this skill when you need to **look up how or why something works before changing it**, or when
asked to read or update the project wiki.

The wiki is a [Foam](https://docs.foam.md/) workspace: plain markdown notes with YAML front matter,
connected by `[[wikilinks]]`, served to agents through the `foam-wiki` MCP server — and usually also
an **[OKF](#open-knowledge-format-okf) bundle**. It holds the **narrative** knowledge — how the
pieces fit together, the decisions behind them, and the open questions — as opposed to the code,
config, and data files that are the executable truth.

> **This skill is project-agnostic on purpose.** Everything specific to *this* project — what belongs
> in the wiki, the folder layout, the front-matter vocabularies — is defined in the wiki's own `meta/`
> notes. Read those, don't assume; and when this skill and a `meta/` note disagree, the `meta/` note
> wins.

---

## Orient first

The wiki root is whatever `.mcp.json` passes as `--workspace` to the `foam-wiki` server; by convention
`docs/wiki/`. All URIs below are relative to that root.

Three notes define how this particular wiki works. Read them before writing anything:

| URI                         | What it defines                                                         |
|-----------------------------|-------------------------------------------------------------------------|
| `index.md` (or `README.md`) | Home — what this wiki covers and an index of the notes that exist       |
| `meta/conventions.md`       | **Front matter vocabularies, folder layout, filenames, linking rules**  |
| `meta/scope.md`             | What belongs in the wiki vs. which files in the repo own a fact instead |

```
read_resource(uri: "index.md")
read_resource(uri: "meta/conventions.md")
```

If those notes don't exist, see [Bootstrapping](#bootstrapping-a-new-wiki) at the end.

**Never invent front-matter fields, values, or folders.** The controlled vocabularies live in
`meta/conventions.md`; extending them is a decision to raise with the user, not to make silently.
Front matter is at minimum:

```yaml
---
title: <human-readable title>   # matches the H1
type: <type>                    # see meta/conventions.md
description: <one sentence>
status: <status>                # see meta/conventions.md
tags: [<tag>, <tag>]
---
```

---

## foam-wiki MCP tools

Prefer these over `grep`/`find` for wiki lookups — they understand the link graph and front matter.
Reference: <https://docs.foam.md/tools/cli/mcp/>

### Discovery and navigation

**`get_workspace_info`** — note/tag/orphan/placeholder counts. Call it to confirm the server is live
and to gauge the wiki's size before exploring.

**`list_resources`** — list notes, optionally filtered.

```
list_resources()
list_resources(tag: "<tag>")
list_resources(type: "moc")
```

**`list_tags`** — the full tag vocabulary. Check it *before* searching by tag, and before adding a tag
to a new note, so you reuse an existing one instead of coining a near-duplicate.

**`search_resources`** — full-text search across title, alias, tag, and front-matter properties.

**`search_by_tag`** — exact tag match.

**`search_by_property`** — search any front-matter property, e.g. `status`, `type`.

### Reading content

**`read_resource`** — raw markdown of a note, by URI relative to the wiki root.

```
read_resource(uri: "meta/conventions.md")
```

**`get_outline`** — heading structure only. Use it to scan a long note before committing to a full read.

**`get_resource`** — front matter and URI without the body.

### Graph navigation

**`get_connections`** — outgoing links and/or backlinks for a note. The fastest way to widen a search
from one relevant note to its neighbourhood.

```
get_connections(uri: "<uri>", direction: "both")
```

**`traverse_graph`** — multi-hop traversal from a starting note, for mapping a whole topic.

**`get_placeholders`** — link targets that don't exist yet. This is the backlog of notes worth writing.

**`get_orphans`** — notes nothing links to. Either a navigation gap or a note that should be linked
from an index.

**`get_deadends`** — notes with no outgoing links; usually under-connected.

**`get_graph_summary`** — high-level graph stats.

### Saved queries

**`list_queries`** / **`get_query`** / **`run_query`** — saved Foam queries, if the workspace defines any.

### Writes

Available when the server runs with `--allow-writes`:

- **`move_resource`** — moves or renames a note **and rewrites every wikilink pointing at it**. Always
  use this rather than `mv`; a plain move silently breaks the graph.
- **`update_resource`** — front-matter patches.
- **`add_tags`** / **`remove_tags`** / **`rename_tag`** — tag mutations. `rename_tag` updates every note.
- **`create_resource`** / **`delete_resource`** — whole notes.

For prose edits inside a note, the normal Edit/Write tools are fine.

---

## Lookup workflow

1. **`get_workspace_info`** — confirm the server is up.
2. **Start at `index.md`** (or `README.md`) — it indexes what exists and frames the subject matter.
3. **Search** — `search_resources` for the topic, or `list_tags` then `search_by_tag`.
4. **Widen** — `get_connections` on each hit to pick up neighbours you'd otherwise miss.
5. **Follow the note to its source.** A good note points at the authoritative file. Read that file
   when the detail matters — **the source wins over the note**, and a note found to be stale should be
   fixed as part of the work.

---

## Editing

Read `meta/conventions.md` first. Beyond whatever it says:

- **One concept per note.** Prefer a new linked note over growing an existing one past its subject.
- **Link liberally**, including to notes that don't exist yet — an unresolved `[[link]]` is a valid
  to-do, discoverable via `get_placeholders`.
- **Never duplicate an authoritative source** into the wiki — no config file contents, no data rows.
  Link to the path and explain the shape and the reasoning instead. Duplicated facts go stale silently.
- **Record open questions** rather than guessing. A note that says "these two sources disagree, unverified"
  is more useful than one that quietly picks a side.
- Keep `title` in sync with the H1. Move `status` forward as a note matures.
- Add new notes to the home index (`index.md`) or the relevant MoC — otherwise they're orphans.
- Use `move_resource` / `rename_tag` for renames so links and tags stay intact.
- **Always format markdown tables.** Never leave a ragged `|---|---|` table behind.
- No secrets, ever — the wiki is committed.

Tables are written as aligned rectangles, not left ragged — see the `markdown-remark` skill for the
mechanics and `meta/conventions.md` for which delimiter style this project uses.

---

## Open Knowledge Format (OKF)

A markdown-plus-frontmatter wiki is almost exactly
[OKF](https://github.com/GoogleCloudPlatform/knowledge-catalog/tree/main/okf), Google Cloud's
vendor-neutral spec for knowledge bundles agents can consume. If a project asks for a schema, propose
OKF instead of inventing one — check `meta/conventions.md` for whether it's already adopted, and
which version.

The parts that constrain how you write notes:

| Rule                        | Detail                                                                                      |
|-----------------------------|---------------------------------------------------------------------------------------------|
| `type` is required          | A non-empty **string**, never a list. The one field every non-reserved `.md` file must have |
| Values are producer-defined | OKF registers no `type` vocabulary centrally; the project's list is the project's           |
| `index.md` is reserved      | A directory listing. **No front matter**, except `okf_version` at the bundle root           |
| `log.md` is reserved        | A chronological history, newest first, `YYYY-MM-DD` headings                                |
| `status`                    | `draft` · `stable` · `deprecated`. Absent means `stable`                                    |
| `sources`                   | A list of **mappings**, each with `resource` — not a list of strings                        |
| Extensions are legal        | Extra keys are fine; consumers must preserve them. Local axes don't break conformance       |

**Trust tiers** come from `verified` (a list of `{by, at}`): absent → *unverified*, non-`human:`
actor → *machine-confirmed*, `human:<id>` → *human-reviewed*. `generated` records who authored a
note. Actors are `<producer>/<version>`, `human:<id>`, or `process:<id>`. Don't add `verified` to a
note you haven't actually verified — the tier is the whole point, and inflating it is worse than
leaving it absent.

**Consumers must not reject** a bundle for missing optional fields, unknown `type` values, unknown
keys, broken links, or missing `index.md`. That tolerance is what makes the next point workable.

### Wikilinks vs. OKF links

OKF expresses the graph as bundle-relative markdown links (`[Ingress](/hosts/ingress.md)`). Foam and
Obsidian need `[[wikilinks]]`. **You cannot have both properties at once**, and the trade is real:

- Keep wikilinks → Foam/Obsidian backlinks, placeholders and rewrite-on-move all work; a generic OKF
  consumer sees every note but **none of the edges**.
- Convert to markdown links → the graph is portable; you lose the Foam authoring affordances.

Neither breaks conformance, because unknown body content is tolerated. If a project keeps wikilinks,
that is a **declared deviation** and `meta/conventions.md` should say so explicitly — a reader
otherwise can't tell a deliberate trade from an oversight.

## Markdown tooling: what a wiki adds

**Foam owns the link graph; remark owns the documents.** They don't overlap — Foam has no formatter
(`foam lint` only checks links) and remark has no concept of backlinks. Load the **`markdown-remark`**
skill for the toolchain itself: config, plugins, hazards, and how to verify a formatter before it
writes. Check `meta/conventions.md` for whether this project has it wired up and under what commands.

Everything below is what a *wiki* adds on top of that.

> ### Wikilinks are the hazard
>
> **Never run remark over a Foam wiki without the `remark-wiki-link` plugin.** Plain remark escapes
> every `[[wikilink]]` to `\[\[wikilink]]`. It still *renders* as a wikilink, so nothing looks
> broken — while every backlink, placeholder and `get_connections` result silently disappears. The
> graph is gone and the diff looks cosmetic.

This is the general "custom inline syntax gets escaped" hazard, and it is worth being paranoid about
here specifically, because a wiki's entire value is the edges. Practical consequences:

- **Use the project's own scripts.** An ad-hoc `npx remark …` bypasses the config — and remark
  resolves config by walking up from each *file*, so it also happens when you point remark at a copy
  of the wiki outside the project.
- **After any bulk markdown operation, verify the graph, not just the text.** `grep -r '\[\[' `
  counts and `foam lint` reporting `0 errors` are the check that matters. Text-level diffs won't
  show you a destroyed graph.
- **`![[embeds]]` need one more check.** The plugin tokenises `[[…]]` only, so the `!`-prefixed
  transclusion form is *not* covered by it — plain remark escapes it to `!\[\[note]]`, which stops
  being an embed in both Foam and Obsidian. `mdfmt` repairs that on the way out; any other pipeline
  needs its own post-pass. Grep for `'!\[\[' ` separately after a bulk run, since the wikilink count
  above is unchanged by it.
- **An embed is invisible to remark either way.** Even repaired it stays a text node, not a
  `wikiLink`, so `remark-validate-links` and toolkit helpers like `wikiLinks()` never see the target.
  Foam does see it, so `foam lint` remains the authority on whether an embed points anywhere.

Two other wiki-flavoured uses of the same toolchain:

- **Front-matter schemas** are the natural way to enforce the closed vocabularies in
  `meta/conventions.md` — `type`, `status`, and whatever else the project fixed. Prose conventions
  decay; schema violations fail a check. Map different schemas to different folders when the rules
  differ between them.
- **Link checking is split.** `foam lint` covers `[[wikilinks]]`; `remark-validate-links` covers
  ordinary `[text](path.md#anchor)` links and heading anchors. A wiki usually has both, so run both.

---

## Health checks

The `foam` CLI backs the same workspace and is useful for verification after a batch of edits:

```shell
foam lint --workspace <wiki-root>            # broken links, stale link definitions
foam list placeholders --workspace <wiki-root>
foam list orphans --workspace <wiki-root>
```

Run the project's markdown check in the same pass — the two cover different link types, so neither
alone is sufficient. In this repo:

```shell
npm run docs:check                            # remark: lint, links, front-matter schema
```

`0 errors` means every wikilink resolves. `stale-definitions` warnings concern the autogenerated
GitHub-compatibility link blocks (`--fix` adds them); whether to keep them is a per-project choice —
follow whatever the existing notes do.

---

## Setup

The server is configured per project in `.mcp.json`:

```json
{
  "mcpServers": {
    "foam-wiki": {
      "command": "npx",
      "args": ["-y", "foam-cli", "mcp", "--allow-writes", "--workspace", "docs/wiki"]
    }
  }
}
```

`--workspace` is relative to the project root. Drop `--allow-writes` for a read-only wiki. `npx -y`
fetches `foam-cli` on demand; install it globally to avoid that on every start:

```shell
npm install -g foam-cli
```

Adding or changing the server requires restarting the agent session and approving it.

---

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

**Propose OKF rather than inventing a vocabulary** (see below) — a spec someone else maintains beats
one you have to defend. If the user prefers their own:

- **`type`**: a string, one of `note` · `moc` · `adr` · whatever the domain needs.
- **`status`**: `draft` · `stable` · `deprecated`.
- **Filenames**: kebab-case, globally unique basenames — that keeps `[[stem]]` links unambiguous
  regardless of folder.
- **Folders**: start flat. Split only when navigation actually hurts, using `move_resource`.
- **Classification**: tags first. Add front-matter axes (`area`, `domain`, `owner`, …) only when a real
  navigation need appears — each one is a vocabulary someone has to maintain.

Seed the wiki from what the repo already documents (READMEs, docs directories, agent instruction files),
and write down contradictions between sources as open questions rather than resolving them by guesswork.
