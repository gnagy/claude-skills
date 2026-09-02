---
name: wiki-docs
description: Use this skill to read, search, or edit the project wiki — a markdown knowledge base of `[[wikilink]]`-connected notes served by the agent-wiki-toolbox MCP server, usually an Open Knowledge Format (OKF) bundle. Look something up here before changing it, and capture what you learn as linked notes. Read it before restructuring, splitting, renaming or bulk-editing notes with any tool, and before running any script over the wiki directory. Also covers bootstrapping a wiki, other projects' wikis mounted read-only alongside, and wiring a project's CLAUDE.md and meta/ notes to this skill or bringing them up to date with a new release of it.
---

# Wiki Docs

Use this skill when you need to **look up how or why something works before changing it**, or when
asked to read or update the project wiki.

The wiki is plain markdown notes with YAML front matter, connected by `[[wikilinks]]`, served to
agents through the **`agent-wiki-toolbox` MCP server** — and usually also an
**[OKF](#open-knowledge-format-okf) bundle**. It holds the **narrative** knowledge — how the pieces
fit together, the decisions behind them, and the open questions — as opposed to the code, config, and
data files that are the executable truth.

> **List the tools you actually have before reaching for one from this file.** What a project mounts
> is the authority on what you can call: a server this file does not describe, or a second one
> alongside, will differ in its tool names. This file is the authority on how to use what is there.

---

## Who owns what, and who wins

This skill is project-agnostic on purpose. A project's own files — its `CLAUDE.md` and the wiki's
`meta/` notes — carry what is specific to it. Read those, don't assume. **Precedence runs by subject,
not by file:**

| Subject                                                                                                                                      | Authority                          |
|----------------------------------------------------------------------------------------------------------------------------------------------|------------------------------------|
| **Choices** — wiki root, folder layout, front-matter vocabularies, the navigation axis, what belongs in the wiki, the project's own commands | The `meta/` notes and `CLAUDE.md`  |
| **Mechanics** — tool names and flags, the wikilink hazard, OKF, `awt` usage, cross-wiki links, the message format, setup                     | This skill and the files beside it |

**A local file restating a mechanic is not a second authority — it is a stale copy.** It was written
against some earlier version of this file, nothing updates it, and it reads as local law to the next
agent. Delete it and leave a pointer: a documentation fix, not a decision to raise. Changing a
*choice* is the opposite — that is the project's, and yours to ask about rather than edit.

**Extracting a wiki's own notes into a skill flips their authority the day it lands.** They were the
authority right up until the skill existed; afterwards they are the stale copy above, and the worst
kind of it — local, detailed, and outranking the skill for everyone already working in that repo.
**Reduce them to choices, or mark them superseded, as part of the extraction**, not as a later
cleanup: between the two there is a window where both exist and the wrong one wins. `adoption.md` §3
carries the extraction as a sweep trigger.

**Refer, never quote.** Point at this skill by name and section heading — *"see the `wiki-docs` skill,
'Editing'"* — so the reference survives an edit inside that section. A copied sentence does not, and a
copied command, flag or field name is the form that fails silently, because it still looks
authoritative long after it stops being true.

**A `meta/` note is authoritative about its project, not about the machine.** It settles what this
wiki does; it cannot settle what tools exist. A note saying a project has no formatter pipeline wired
up is not a note saying no formatter is installed — see [Markdown tooling](#markdown-tooling-awt).

**Nothing except the tool is evidence about the tool.** Not a `meta/` note, not your own system
prompt, not a habit carried from another project. Before concluding that `awt` cannot do something —
reformat a table, rename across the graph, tokenise an embed — run `awt --help`. Working around a
limit you inferred rather than observed is how a wiki ends up restructured by hand-rolled scripts
while the tool that does it safely sits on `PATH`.

---

## Orient first

**A wiki has one home, and the project names it once.** `rootDir` in `awt.config.mjs` at the
project root names the directory — `wiki/` when unsaid — and everything the wiki owns is a fixed name
inside it: `notes/` (the wiki root, every path below is relative to it), `site/`, `schemas/` and
`inbox/`. The server and every `awt` command find that config by walking up, so `.mcp.json` names no
path and neither does anything else. `workspace_info` reports `notesDir` and, when the project has one,
`rootDir` — two directories, two names, never inferred from each other. A project laid out the old way,
`docs/wiki/` beside `site/`, still works and is told so once; `adoption.md` beside this file says how
to move it.

**There may be more than one wiki server.** A project can mount other projects' wikis read-only
alongside its own — see [Other wikis](#other-wikis-read-only-mounts). Exactly one is writable: yours.
`workspace_info` reports `notesDir` and `allowWrites`, so confirm which server you are talking to before
writing, rather than inferring it from the server's name.

Three notes define how this particular wiki works, and a fourth appears only in projects that exchange
knowledge with other wikis. Read them before writing anything:

| URI                         | What it defines                                                         |
|-----------------------------|-------------------------------------------------------------------------|
| `index.md` (or `README.md`) | Home — what this wiki covers and an index of the notes that exist       |
| `meta/conventions.md`       | **Front matter vocabularies, folder layout, filenames, linking rules**  |
| `meta/scope.md`             | What belongs in the wiki vs. which files in the repo own a fact instead |
| `meta/interop.md`           | *Only if present* — see [Other wikis](#other-wikis-read-only-mounts)    |

**Read them with your own `Read` tool.** There is no tool here for opening a file, deliberately: a
tool earns a place on this surface only if it answers something a file cannot.

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

## The MCP tools

Prefer these over `grep`/`find` for wiki lookups — they understand the link graph and front matter.
The server ships a description and schema for every tool, so this section carries only what those
cannot: which tool to reach for, in what order, and the rules that involve things the server does not
know about.

**The surface is small on purpose.** A tool is here only if it answers something that cannot be
answered by opening a file — so there is no tool for reading a note, listing its headings or writing
prose into it. Your own `Read`, `Write` and `Edit` do those, and better.

| Tool             | Reach for it when                                                                |
|------------------|----------------------------------------------------------------------------------|
| `workspace_info` | First call — is the server live, is `notesDir` the wiki you mean, may you write  |
| `search`         | Default entry point. Note **bodies**, titles, front matter and tags, in one call |
| `connections`    | Widening from one hit to its neighbours — the highest-yield second call          |
| `resolve`        | What a link points at; **and what else matched**, when it is ambiguous           |
| `check`          | Is the graph healthy — every problem, plus the placeholder backlog, in one call  |

`search` takes a plain string (case-insensitive) or `/regex/flags`, and filters by `tag`, `type`,
`area` and `topic`. `workspace_info` returns the **whole tag vocabulary with counts**, which is what
to look at before coining a tag that already nearly exists.

`check` is the one that replaces four separate questions. It returns:

| In `check`     | What it means                                                                                                                                                                                |
|----------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `problems`     | Real defects: **ambiguous links**, **two files served as one page**, **a link that misses a note which exists**, broken section anchors, broken relative links, and a `[[` that never closed |
| `placeholders` | **The backlog, not a defect** — a `[[stem]]` naming a note that does not exist yet                                                                                                           |
| `orphans`      | Notes with no links either way                                                                                                                                                               |
| `deadends`     | Notes with **no way out** — you can reach them and not leave                                                                                                                                 |
| `unreferenced` | Notes nothing links to; reachable by search and nothing else                                                                                                                                 |

> **Two files on one page is an error too, and it is the one you cannot see.** A wiki can hold two
> files that render to the same address — `x/x.md` beside `x/index.md`, `a b.md` beside `a-b.md`, two
> spellings that differ only in case. Nothing links to them wrongly, so no link-level rule fires;
> `check` reports `colliding-view` on **every file in the group**, and the fix is to rename all but
> one. Which of them the index keeps is directory order, so leaving it is not a choice, it is a
> coin flip.

> **A link can miss a note that exists, and that is an error.** Resolution follows Quartz's
> shortest-path rule, and it has gaps — `[[folder/index]]` does not reach a listing one level down,
> though `[[a/b/index]]` reaches one two levels down. `check` reports `unreachable-note` wherever a
> link resolves to nothing while a file sits at exactly that address, so it can never be mistaken for
> backlog. **Reach a folder listing with a relative markdown link.**

> **An ambiguous link is an error, not a guess.** Two notes sharing a basename means a `[[stem]]`
> naming either one resolves to neither, and the rendered site 404s while nothing warns. `resolve`
> names the candidates; the fix is a folder segment in the link or a rename, and it is worth doing at
> once.

### Writes

Available only when the server runs with `--allow-writes`. Each is something that corrupts the wiki
when done by hand:

| Tool                               | Instead of                                                           |
|------------------------------------|----------------------------------------------------------------------|
| `rename` · `move`                  | `mv`, which silently breaks every inbound link                       |
| `delete`                           | `rm`, which leaves nothing to tell you what pointed at it            |
| `split_by_heading` · `merge_files` | Re-emitting both documents as tokens                                 |
| `rename_tag`                       | `sed`, which cannot tell front matter from prose                     |
| `build_listing`                    | Hand-maintaining the notes table in `index.md`                       |
| `fmt`                              | Aligning a table by hand, or any formatter that is not the toolbox's |

**Ordinary prose edits are yours**, with `Write` and `Edit` — and they are exactly what `fmt` is for.
The verbs above already serialise everything they write, so the unformatted markdown in a wiki is the
prose you wrote yourself. Its `paths` are workspace-relative like every other tool's, and no path
means the whole wiki; `dryRun` is what the CLI calls `--check`. The index is recomputed on the next
call, so nothing has to be told an edit happened.

Three things about how they behave, all of which change how you use them:

- **Every one is re-runnable, and re-running is how a partial run gets finished.** Nothing locks —
  several agents edit one wiki at once — so a file that changed under a verb is *skipped and
  reported*, never overwritten. Read `skipped` in the return and run it again.
- **Read `unresolved` too.** It carries the links the tool would not guess at: an ambiguous stem, a
  bare `[[parent]]` after a split that could mean any of the children. Those are yours to settle.
- **`split_by_heading` invents no names.** You supply the heading-to-path plan and the source note's
  fate (`delete`, `stub` or `keep`), because a section title becoming a filename is judgment. It
  refuses outright if a basename is already in the wiki.

### On the command line

The same operations, same names in kebab-case, when a shell is easier than a tool call:

```shell
awt check                          # the whole graph, one call — the project's notes, from anywhere in it
awt search 'listing budget'        # bodies as well as titles
awt resolve 'stem#a-heading'       # …and what else matched
awt move a.md design/a.md          # paths relative to the notes, like every tool's
awt --help                         # every operation
```

`-w` still names a notes directory outright — a scratch wiki, another project's — and otherwise a
command means the project it is run inside.

## Editing

Read `meta/conventions.md` first. Beyond whatever it says:

- **One concept per note.** Prefer a new linked note over growing an existing one past its subject.
  **A `[[note#section]]` reference is the signal.** A section worth linking to is usually a note worth
  splitting out, and splitting it turns a reference that breaks when someone rewords the heading into a
  link that survives a rename.
- **Link notes with `[[wikilinks]]`** — always, no confirmation needed (see
  [Wikilinks vs. OKF links](#wikilinks-vs-okf-links)). Markdown links are for external URLs and files
  outside the wiki.
- **Link liberally**, including to notes that don't exist yet — an unresolved `[[link]]` is a valid
  to-do, and `check` reports it under `placeholders`.
- **Closing a placeholder is two operations.** Writing the note resolves the `[[link]]`, but the prose
  around it still says the note doesn't exist — *"`[[backups]]` is still unwritten"*, *"…and the
  reason `[[backups]]` needs writing"*. Keep the backlinkers `check` reported **before** you
  created it — they are gone from that list afterwards — then grep them for
  `unwritten|not yet written|needs writing|can now be written` and fix the sentences. A link resolving
  is not the same as a sentence becoming true, and nothing connects the two.
- **Never duplicate an authoritative source** into the wiki — no config file contents, no data rows.
  Link to the path and explain the shape and the reasoning instead. Duplicated facts go stale silently.
- **A decision register is an authoritative source too — cite the row and stop.** Write *"whether it
  moves is `[[some-decisions]]` 28"*, never that plus a retelling of what 28 says. A row number reads
  like a citation, so the copied clause after it does not feel like duplication, and it ages
  separately from the row with nothing to detect the drift. **Counts drift the same way**: "thirty-five
  rows, one still open" in an index or a MoC is a copy of the register's own table, wrong the moment a
  row changes status. **An identifier range is a count wearing a hat** — *"decisions 6–13 are the
  current world"*, *"questions 3–5 and 21–23 are open"* — and it reads like a citation while ageing
  like a copy. One such list in a Map of Content named six questions as open that had been resolved
  days earlier. Cite the register and count nothing.
- **Record open questions** rather than guessing. A note that says "these two sources disagree, unverified"
  is more useful than one that quietly picks a side.
- **A measured figure has one home.** *"0 of 39 rows match"*, *"181 of 181 notes are conformant"* — the
  note that did the measuring owns it, and every other note links there. The same figure written down
  twice is two claims about a wiki that has since changed, with nothing connecting them.
- **No changelog section in a note.** Git owns the history and `log.md` owns the wiki's chronology. A
  *What changed* heading inside a note is a third record that only grows, and it is the first part to
  go stale when the note around it is rewritten.
- **A folder listing one level down cannot be linked.** `[[index]]` reaches the wiki home and
  `[[a/b/index]]` resolves, but `[[folder/index]]`, `[[folder/index.md]]` and `[[folder/]]` reach
  nothing — Quartz's own shortest-path rule, which the toolbox mirrors, so it is not a gap that will
  quietly close. Folder segments are not the problem; `[[folder/note-b]]` resolves. Link a nested
  listing with a **relative markdown link** — `[the folder listing](folder/index.md)` — which `check`
  validates. `check` errors on the wikilink form rather than filing it as backlog, so this costs you a
  build, not a graph.
- Keep `title` in sync with the H1. Move `status` forward as a note matures.
- Add new notes to the home index (`index.md`) or the relevant MoC — otherwise they're orphans.
  `check` reports `orphans` and `unreferenced`; run it rather than trusting that you remembered.
- Use `move` / `rename` / `rename_tag` for renames, so links and tags stay intact.
- **Always format markdown tables.** Never leave a ragged `|---|---|` table behind.
- No secrets, ever — the wiki is committed.

Tables are written as aligned rectangles, not left ragged — the `fmt` tool does it, `awt fmt` is the
same thing from a shell, and `meta/conventions.md` says which delimiter style this project uses.

### No display-text overrides

**Use plain stems: `[[note-stem]]`, never `[[stem|text]]`.** A rename rewrites the *target* of a
labelled link and leaves the *label* alone, so
`[[target|the old description]]` becomes
`[[power-supply-unit|the old description]]` — still rendering the old name, pointing at a note that is
no longer called that. It renders fine, so nothing looks broken.

A label is a copy of the note's identity that no tool will ever update.

**When a bare link won't fit the sentence, put the phrase in prose and the link beside it.** Never
reach for a label to fix the grammar:

- `Dashboards are [[two-tier-state|tier 2]]` → `Dashboards are tier 2 ([[two-tier-state]])`
- `everything [[deploy-to-instance|rsync]] touches` → `everything rsync touches (see [[deploy-to-instance]])`

Both forms resolve identically, so this is about drift, not connectivity.

---

## Other wikis (read-only mounts)

A project may mount other projects' wikis as additional read-only servers, conventionally named
after the project. Only your own wiki is writable, and `workspace_info` says so.

**`meta/interop.md` is the switch.** If the wiki has one, this project exchanges knowledge with
others, and that note declares which wikis are mounted, what each is authoritative for, and where
this project's own inbox is. If the wiki has no such note, none of this applies.

Three rules hold whether or not you read further, because breaking any of them is silent:

- **Never edit another project's notes.** The read-only mount is the guarantee; don't route around it
  with Write/Edit. If something there is wrong, send it a message or go work in that repo.
- **Never write a cross-wiki reference as a `[[wikilink]]`.** The graph registers any unresolved `[[…]]`
  as a placeholder, so it lands in the backlog permanently and poisons the one signal that means
  "note worth writing". Write a **prefixed markdown link** instead —
  `[conventions](otherwiki:meta/conventions.md)` — which the graph reads as external, `resolve`
  reports as `crossWiki`, and a rendered site turns into a real URL. See *Cross-wiki references* in
  `interop.md`.
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

OKF expresses the graph as bundle-relative markdown links; this estate and Obsidian need `[[wikilinks]]`. You
can't have both. **Always use wikilinks — don't ask, don't propose converting.** They buy rename-safe
links (`move` rewrites them), Obsidian compatibility, and placeholders as a backlog; the cost
is that a generic OKF consumer sees the notes but none of the edges.

That's a **declared deviation**, not a conformance break. Record it once in `meta/conventions.md`, and
if it's missing there, add it — a documentation fix, not a decision to raise:

```markdown
## Link syntax (OKF deviation)

Links are `[[wikilinks]]`, not OKF bundle-relative markdown links — deliberate, for rename-time link
rewriting, backlinks, placeholders and Obsidian compatibility. Generic OKF consumers see the notes but
not the edges.
```

## Markdown tooling: `awt`

**Format a wiki with the `fmt` tool, or with `awt fmt` from a shell.** They are one formatter, the
toolbox's own, and it is wikilink-aware — which is the whole reason not to reach for anything else
here (see the hazard below). On the command line it takes `-w` like every other subcommand, so a note
path is a note path there too: `awt fmt -w <notes-dir> meta/conventions.md` — or, from inside the
project, `awt fmt -w wiki/notes meta/conventions.md`.

> **`awt` is a standalone CLI, not a project pipeline.** It carries its own config and plugins, so it
> formats any directory without that project installing anything — it works in a repo with no
> `node_modules`, no `.remarkrc` and no npm script. **"No remark pipeline is wired up here" and "no
> formatter is available" are different facts**, and a `meta/` note stating the first is routinely read
> as the second. Check `awt --version` before hand-aligning a table or writing a script to do it; if
> it isn't on `PATH`, say so rather than reinventing it.

**It is the only formatter for a wiki, and "the other one works too" is not an exception.** Another
formatter producing identical output on this wiki today is not a reason to name it in a `meta/` note:
what is being bought is a guarantee, and two commands that happen to agree is not one that cannot
disagree. A note offering both — *"either one does"* — is the drift itself, and it is written by
someone who checked and was right about the output, which is exactly why it does not read as an error.

**Reach for `markdown-remark` only when the target is not a wiki** — a README, a `CLAUDE.md`, a docs
tree with no link graph in it. That skill owns the remark toolchain itself: config, plugins, and how to
verify a formatter before it writes. For wiki work, `awt` is the tool and this file is the authority.

Everything below is what a *wiki* adds on top of that.

> ### Wikilinks are the hazard
>
> **Never run remark over a wiki without a plugin that knows what a wikilink is.** Plain remark
> escapes every `[[wikilink]]` to `\[\[wikilink]]`. It still *renders* as a wikilink, so nothing looks
> broken — while every backlink, placeholder and `connections` result silently disappears. The graph is
> gone and the diff looks cosmetic.

This is the general "custom inline syntax gets escaped" hazard, and it is worth being paranoid about
here specifically, because a wiki's entire value is the edges. Practical consequences:

- **Use the project's own scripts.** An ad-hoc `npx remark …` bypasses the config — and remark
  resolves config by walking up from each *file*, so it also happens when you point remark at a copy
  of the wiki outside the project.
- **After any bulk markdown operation, verify the graph, not just the text.** `grep -rc '\[\['`
  counts and `check` reporting no problems are what matter. Text-level diffs will not show you a
  destroyed graph.
- **`![[embeds]]` are ordinary links here, and are not everywhere else.** `awt` tokenises the
  `!`-prefixed form, so an embed is a node, a graph edge, and something `rename` rewrites like any
  other link. **Every other remark pipeline escapes it** to `!\[\[note]]`, which stops being an embed
  in Obsidian and in Foam — so grep for `'!\[\['` separately after a bulk run by anything that is not
  `awt fmt`, since the wikilink count above is unchanged by it.

Two other wiki-flavoured uses of the same toolchain:

- **Front-matter schemas** are the natural way to enforce the closed vocabularies in
  `meta/conventions.md` — `type`, `status`, and whatever else the project fixed. Prose conventions
  decay; schema violations fail a check. Map different schemas to different folders when the rules
  differ between them.
- **Link checking is not split any more.** `awt check` covers `[[wikilinks]]`, ordinary
  `[text](path.md#anchor)` links, and heading anchors on both — one call, one answer. A project may
  still add front-matter schema validation on top, which `awt fmt --check` runs from its config.

---

## Health checks

One call, after a batch of edits:

```shell
awt check                         # every problem, plus placeholders, orphans and dead ends
awt check --json                  # the same answer, for a script
```

Both mean the project's own notes from anywhere inside it; `-w` names another directory.

It exits non-zero when there are **problems** and zero when there are only placeholders, which is the
distinction that matters: a placeholder is a note worth writing, not a defect, and a check that failed
on them would fail on every wiki with a backlog.

Run the project's own markdown check in the same pass if it has one — `awt fmt --check` covers
formatting, and a project may add front-matter schemas and relative-link validation on top.
`meta/conventions.md` names the command.

---

## Setup and bootstrapping

`.mcp.json` server config, mounting other projects' wikis, and the skeleton for a wiki that doesn't
exist yet are in **`setup.md` beside this file**. Read it when there is no wiki server at all,
when the `meta/` notes are missing, or when asked to add a wiki to a project.

---

## Wiring a project to this skill

**A description is matched when the model happens to consider it; a project's `CLAUDE.md` is in
context every turn.** So the thing that actually makes a project use this skill is a short mandate in
its `CLAUDE.md`, and the thing that catches the sessions ignoring it is a `PreToolUse` hook on the
wiki path. Both are shipped here as fixed text, so that a project holds a pointer with no content in
it to go stale — the failure this whole split exists to prevent.

**Read `adoption.md` beside this file** when installing this skill into a project or when a new
release of it lands — and on four triggers that have no event behind them: **having just written or
rewritten a `meta/` note or that block**, **having extracted content out of a wiki into a skill**, and
**any request to reconcile a project's local files with the skills it uses**, whether that is asked as
*is `meta/` still in step*, *why do these two disagree*, or *clean up the duplication in here*. The
last kind is this sweep under another name, and a pass that never opens a skill cannot see the copy or
the conflict that matters, nor settle one once it has. `adoption.md` carries the
`CLAUDE.md` block to paste verbatim, the guard hook and its limits, and the sweep itself, with the
subset of it each trigger runs.

---

## Rendering the wiki as a site

A wiki is written for agents and for whoever edits it, and **neither is a reader**. A project can
render its own wiki into a browsable static site — backlinks, graph, search, hover previews — and,
where several wikis reference each other, into links that actually cross between them.

**The project that owns the wiki builds its own site**, from the `site/` directory in the wiki's home
(`wiki/site/` by default). There is no central builder: a wiki always lives inside the repo that owns its subject matter, so anything
else means submoduling a subdirectory (git cannot) or handing a build host a credential to a repo
whose markdown is the only part it needs.

Two things worth knowing before reading further, because both come up as questions rather than as
tasks:

- **Do not gitignore the site directory wholesale.** `quartz.config.yaml` and its `.gitignore` are
  tracked; the renderer clone and the build output are not.
- **A cross-wiki reference is a prefixed markdown link** — `[conventions](otherwiki:meta/conventions.md)`
  — never a `[[wikilink]]`, for the placeholder reason in
  [Other wikis](#other-wikis-read-only-mounts). It stays inert in the graph and the editor, and becomes a
  real link only in the rendered site.

**Read `site.md` beside this file** before setting a site up, changing its configuration, running or
debugging a build, or making a cross-wiki reference resolve. It covers the pinned renderer, what git
tracks, build versus publish mode, the shadow check that holds the renderer and the graph to one answer, the
cross-wiki resolver and its registry, and the traps — including why a wiki whose home is `README.md`
serves a 404 at its root.
