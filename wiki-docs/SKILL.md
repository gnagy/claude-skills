---
name: wiki-docs
description: Use this skill to read, search, or edit the project wiki — a Foam markdown knowledge base of `[[wikilink]]`-connected notes served by the foam-wiki MCP server, usually an Open Knowledge Format (OKF) bundle. Look something up here before changing it, and capture what you learn as linked notes. Also covers bootstrapping a wiki, and other projects' wikis mounted read-only alongside.
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
>
> **A `meta/` note is authoritative about its project, not about the machine.** It settles what this
> wiki does; it cannot settle what tools exist. A note saying a project has no formatter pipeline
> wired up is not a note saying no formatter is installed — see [Markdown tooling](#markdown-tooling-what-a-wiki-adds).

---

## Orient first

The wiki root is whatever `.mcp.json` passes as `--workspace` to the `foam-wiki` server; by convention
`docs/wiki/`. All URIs below are relative to that root.

**There may be more than one foam server.** A project can mount other projects' wikis read-only
alongside its own — see [Other wikis](#other-wikis-read-only-mounts). Exactly one is writable: yours.
`get_workspace_info` reports `root_dir` and `read_only`, so confirm which server you are talking to
before writing, rather than inferring it from the server's name.

Three notes define how this particular wiki works, and a fourth appears only in projects that exchange
knowledge with other wikis. Read them before writing anything:

| URI                         | What it defines                                                         |
|-----------------------------|-------------------------------------------------------------------------|
| `index.md` (or `README.md`) | Home — what this wiki covers and an index of the notes that exist       |
| `meta/conventions.md`       | **Front matter vocabularies, folder layout, filenames, linking rules**  |
| `meta/scope.md`             | What belongs in the wiki vs. which files in the repo own a fact instead |
| `meta/interop.md`           | *Only if present* — see [Other wikis](#other-wikis-read-only-mounts)    |

```
read_resource(uri: "index.md")
read_resource(uri: "meta/conventions.md")
```

If those notes don't exist, read `setup.md` beside this file.

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
The server ships a description and schema for every tool, so this section carries only what those
can't: which tool to reach for, in what order, and the rules that involve tools foam doesn't know
exist. Reference: <https://docs.foam.md/tools/cli/mcp/>

| Tool                                       | Reach for it when                                                       |
|--------------------------------------------|-------------------------------------------------------------------------|
| `get_workspace_info`                       | First call — is the server live, and is `root_dir` the wiki you mean    |
| `search_resources`                         | Default entry point for a topic                                         |
| `list_tags` → `search_by_tag`              | In that order — see the vocabulary before coining a near-duplicate      |
| `search_by_property`                       | By front-matter axis, e.g. everything still `draft`                     |
| `list_resources`                           | Enumerating by `tag` — **not** by front-matter `type`, see below        |
| `read_resource`                            | The note itself, by URI relative to the wiki root                       |
| `get_outline`                              | Scanning a long note before committing to a full `read_resource`        |
| `get_resource`                             | Front matter alone, without the body                                    |
| `get_connections`                          | Widening from one hit to its neighbours — the highest-yield second call |
| `traverse_graph`                           | Mapping a whole topic, multi-hop                                        |
| `get_placeholders`                         | The backlog: links pointing at notes worth writing                      |
| `get_orphans` / `get_deadends`             | Under-connected notes — a navigation gap, or a missing index entry      |
| `get_graph_summary`                        | Graph-level stats                                                       |
| `list_queries` / `get_query` / `run_query` | Saved Foam queries, if the workspace defines any                        |

> **`list_resources`'s `type` is not your front matter.** It filters Foam's own *resource kind* —
> `note` versus `attachment` — so every markdown note is reported as `type: note` whatever its front
> matter says, and `list_resources(type: "adr")` comes back `[]` with no error. An empty result reads
> as *"no such notes exist"* rather than *"wrong tool"*, which is what makes this worth a warning.
> **Use `search_by_property(property: "type", value: …)` for the front-matter axis.** Filtering by
> `tag` behaves as you'd expect, because tags are Foam-native.

### Writes

Available only when the server runs with `--allow-writes`.

- **`move_resource`, never `mv`.** It rewrites every inbound wikilink; a plain move silently breaks
  the graph. Likewise **`rename_tag`** rather than `sed` — it updates every note that carries the tag.
- **`update_resource` overwrites.** Passing `content` replaces the **whole file**, not a section, and
  `properties` merges front matter only while `merge_properties` is true — `false` replaces it
  wholesale. For a prose edit inside a note, use the normal Edit tool instead.
- **`add_tags` / `remove_tags`** for tags on one note; **`create_resource` / `delete_resource`** for
  whole notes.

---

## Lookup workflow

1. **`get_workspace_info`** — confirm the server is up, and that `root_dir` is the wiki you mean.
2. **If `meta/interop.md` exists**, check the inbox it declares — messages from other wikis, oldest
   first. Resolve or consciously defer them before starting new work.
3. **Start at `index.md`** (or `README.md`) — it indexes what exists and frames the subject matter.
4. **Search** — `search_resources` for the topic, or `list_tags` then `search_by_tag`.
5. **Widen** — `get_connections` on each hit to pick up neighbours you'd otherwise miss.
6. **Follow the note to its source.** A good note points at the authoritative file. Read that file
   when the detail matters — **the source wins over the note**, and a note found to be stale should be
   fixed as part of the work.

---

## Editing

Read `meta/conventions.md` first. Beyond whatever it says:

- **One concept per note.** Prefer a new linked note over growing an existing one past its subject.
- **Link notes with `[[wikilinks]]`** — always, no confirmation needed (see
  [Wikilinks vs. OKF links](#wikilinks-vs-okf-links)). Markdown links are for external URLs and files
  outside the wiki.
- **Link liberally**, including to notes that don't exist yet — an unresolved `[[link]]` is a valid
  to-do, discoverable via `get_placeholders`.
- **Closing a placeholder is two operations.** Writing the note resolves the `[[link]]`, but the prose
  around it still says the note doesn't exist — *"`[[backups]]` is still unwritten"*, *"…and the
  reason `[[backups]]` needs writing"*. Keep the backlinkers `get_placeholders` reported **before** you
  created it — they are gone from that list afterwards — then grep them for
  `unwritten|not yet written|needs writing|can now be written` and fix the sentences. A link resolving
  is not the same as a sentence becoming true, and nothing connects the two.
- **Never duplicate an authoritative source** into the wiki — no config file contents, no data rows.
  Link to the path and explain the shape and the reasoning instead. Duplicated facts go stale silently.
- **Record open questions** rather than guessing. A note that says "these two sources disagree, unverified"
  is more useful than one that quietly picks a side.
- Keep `title` in sync with the H1. Move `status` forward as a note matures.
- Add new notes to the home index (`index.md`) or the relevant MoC — otherwise they're orphans.
  `foam list orphans` is the check; run it rather than trusting that you remembered.
- Use `move_resource` / `rename_tag` for renames so links and tags stay intact.
- **Always format markdown tables.** Never leave a ragged `|---|---|` table behind.
- No secrets, ever — the wiki is committed.

Tables are written as aligned rectangles, not left ragged — see the `markdown-remark` skill for the
mechanics and `meta/conventions.md` for which delimiter style this project uses.

### No display-text overrides

**Use plain stems: `[[note-stem]]`, never `[[stem|text]]`.** A rename — `move_resource`, or `foam
rename` on the CLI — rewrites the *target* of a labelled link but preserves the *label*, so
`[[target|the old description]]` becomes
`[[power-supply-unit|the old description]]` — still rendering the old name, pointing at a note that is
no longer called that. It renders fine, so nothing looks broken.

A label is a copy of the note's identity that no tool will ever update.

**When a bare link won't fit the sentence, put the phrase in prose and the link beside it.** Never
reach for a label to fix the grammar:

- `Dashboards are [[two-tier-state|tier 2]]` → `Dashboards are tier 2 ([[two-tier-state]])`
- `everything [[deploy-to-instance|rsync]] touches` → `everything rsync touches (see [[deploy-to-instance]])`

Both forms resolve identically in Foam's graph, so this is about drift, not connectivity.

---

## Other wikis (read-only mounts)

A project may mount other projects' wikis as additional read-only foam servers, conventionally named
`foam-<project>`. Only your own wiki is writable.

**`meta/interop.md` is the switch.** If the wiki has one, this project exchanges knowledge with
others, and that note declares which wikis are mounted, what each is authoritative for, and where
this project's own inbox is. If the wiki has no such note, none of this applies.

Three rules hold whether or not you read further, because breaking any of them is silent:

- **Never edit another project's notes.** The read-only mount is the guarantee; don't route around it
  with Write/Edit. If something there is wrong, send it a message or go work in that repo.
- **Never write a cross-wiki reference as a `[[wikilink]]`.** Foam registers any unresolved `[[…]]`
  as a placeholder, so it lands in `get_placeholders` permanently and poisons the one signal that
  means "note worth writing". Name the wiki and the note in plain prose instead.
- **Never mount a wiki that depends on yours.** Mounts run one way, down the dependency: the project
  that already depends on the other mounts it, never the reverse — a depended-on project has to build
  and be worked on with no sibling checkout present. Two wikis mounting each other is a cycle, and it
  usually arrives as "mirroring their setup". See *Which way a mount points* in `interop.md`.

**Read `interop.md` beside this file** before adding a mount, consulting a mounted wiki, copying
anything out of one, or sending it a message. It covers provenance, foreign front-matter
vocabularies, the inbox protocol and the message format.

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

OKF expresses the graph as bundle-relative markdown links; Foam and Obsidian need `[[wikilinks]]`. You
can't have both. **Always use wikilinks — don't ask, don't propose converting.** They buy rename-safe
links (`move_resource` rewrites them), Obsidian compatibility, and placeholders as a backlog; the cost
is that a generic OKF consumer sees the notes but none of the edges.

That's a **declared deviation**, not a conformance break. Record it once in `meta/conventions.md`, and
if it's missing there, add it — a documentation fix, not a decision to raise:

```markdown
## Link syntax (OKF deviation)

Links are `[[wikilinks]]`, not OKF bundle-relative markdown links — deliberate, for rename-time link
rewriting, backlinks, placeholders and Obsidian compatibility. Generic OKF consumers see the notes but
not the edges.
```

## Markdown tooling: what a wiki adds

**Foam owns the link graph; remark owns the documents.** They don't overlap — Foam has no formatter
(`foam lint` only checks links) and remark has no concept of backlinks. Load the **`markdown-remark`**
skill for the toolchain itself: config, plugins, hazards, and how to verify a formatter before it
writes. Check `meta/conventions.md` for whether this project has it wired up and under what commands.

> **`mdfmt` is a standalone CLI, not a project pipeline.** It carries its own config and plugins, so it
> formats any directory without that project installing anything — it works in a repo with no
> `node_modules`, no `.remarkrc` and no npm script. **"No remark pipeline is wired up here" and "no
> formatter is available" are different facts**, and a `meta/` note stating the first is routinely read
> as the second. Check `mdfmt --version` before hand-aligning a table or writing a script to do it; if
> it isn't on `PATH`, say so rather than reinventing it.

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

**`--workspace` resolves against the current directory, not the repo root.** Run these from the repo
root; from inside the wiki itself, `--workspace docs/wiki` looks for `docs/wiki/docs/wiki` and the
`ENOENT` names a doubled path that appears in nothing you typed.

Run the project's markdown check in the same pass — the two cover different link types, so neither
alone is sufficient. `meta/conventions.md` names the command; conventionally it is an npm script:

```shell
npm run docs:check                            # remark: lint, links, front-matter schema
```

`0 errors` means every wikilink resolves. `stale-definitions` warnings concern the autogenerated
GitHub-compatibility link blocks (`--fix` adds them); whether to keep them is a per-project choice —
follow whatever the existing notes do.

---

## Setup and bootstrapping

`.mcp.json` server config, mounting other projects' wikis, and the skeleton for a wiki that doesn't
exist yet are in **`setup.md` beside this file**. Read it when `get_workspace_info` finds no server,
when the `meta/` notes are missing, or when asked to add a wiki to a project.

---

## Rendering the wiki as a site

A wiki is written for agents and for whoever edits it, and **neither is a reader**. A project can
render its own wiki into a browsable static site — backlinks, graph, search, hover previews — and,
where several wikis reference each other, into links that actually cross between them.

**The project that owns the wiki builds its own site**, from a `site/` directory in that repo. There
is no central builder: a wiki always lives inside the repo that owns its subject matter, so anything
else means submoduling a subdirectory (git cannot) or handing a build host a credential to a repo
whose markdown is the only part it needs.

Two things worth knowing before reading further, because both come up as questions rather than as
tasks:

- **Do not gitignore `site/` wholesale.** `quartz.config.yaml` and `site/.gitignore` are tracked; the
  renderer clone and the build output are not.
- **A cross-wiki reference is a prefixed markdown link** — `[conventions](otherwiki:meta/conventions.md)`
  — never a `[[wikilink]]`, for the placeholder reason in
  [Other wikis](#other-wikis-read-only-mounts). It stays inert in Foam and the editor, and becomes a
  real link only in the rendered site.

**Read `site.md` beside this file** before setting a site up, changing its configuration, running or
debugging a build, or making a cross-wiki reference resolve. It covers the pinned renderer, what git
tracks, build versus publish mode, the link-graph check that verifies the renderer against Foam, the
cross-wiki resolver and its registry, and the traps — including why a wiki whose home is `README.md`
serves a 404 at its root.
