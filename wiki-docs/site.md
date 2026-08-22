# Rendering a wiki as a site

A wiki is written for agents and for whoever edits it, and **neither is a reader**. Rendering it
produces a browsable site with backlinks, a graph, search and hover previews — and, once more than one
wiki is involved, links that cross between them.

Read this when asked to render, build, serve or publish a wiki, when a project is deciding what in
`site/` belongs in git, or when a reference to another wiki needs to become a real link.

The renderer is [Quartz](https://quartz.jzhao.xyz/) v5. It reads the wiki directory, resolves
`[[wikilinks]]` natively, and emits static HTML — nothing of it runs at serve time.

## The project that owns the wiki builds its own site

There is no central builder, and that is not an omission. A wiki always lives at `docs/wiki/` inside
the repo that owns its subject matter, so any other arrangement means either submoduling a
subdirectory — git cannot — or handing a build host a credential to a repo whose markdown is the only
part it needs. Building in place needs no cross-repo credential at all.

**A site is opt-in.** Plenty of wikis are only ever read through the MCP server and an editor; a wiki
gets a site when someone wants to read it in a browser, or when it needs to resolve references into
another wiki. Building is also separate from publishing — see [Build and publish are different
modes](#build-and-publish-are-different-modes).

## What lives in `site/`, and what git tracks

**Do not gitignore `site/` wholesale.** Four of its entries are the site and belong in the repo; the
other three are machine-local, rebuildable, and about 300 MB:

| Path                      | Tracked | What                                                              |
|---------------------------|---------|-------------------------------------------------------------------|
| `site/quartz.config.yaml` | **yes** | The whole configuration — the only file worth reviewing           |
| `site/quartz.pin`         | **yes** | One line: the Quartz commit this project builds against           |
| `site/README.md`          | **yes** | Project specifics — the port, the registry, how to run it         |
| `site/.gitignore`         | **yes** | Ignores the three below, and says why                             |
| `site/.quartz-src/`       | no      | The Quartz clone, its `node_modules`, and a symlink to the config |
| `site/public/`            | no      | Build output                                                      |
| `site/awt-*`              | no      | Symlinks to this machine's Quartz plugin install                  |
| `site/.awt-index.json`    | no      | The index the plugins read, re-emitted before every build         |

```gitignore
# Quartz itself is cloned at the commit in quartz.pin, never vendored — see
# README.md here. Nothing of it belongs in this repo, including its node_modules
# (~250 MB, entirely separate from the project's own).
.quartz-src/

# Build output. Disposable: rebuild it with `awt serve`.
public/

# Symlinks to the Quartz plugins installed on this machine, created by
# `awt bootstrap-quartz`, and the index they read. Machine-local by definition,
# and regenerated rather than authored.
awt-links
awt-cross-wiki
awt-headings
awt-folder-notes
.awt-index.json
```

**`awt bootstrap-quartz` writes exactly that file** when `site/.gitignore` does not already exist,
before it clones anything — so the tree is never both huge and untracked. It never overwrites one that
does: the project owns the file from then on. Commit it.

When the file *does* exist, it says which of those lines are missing and leaves the editing to you —
which is the case that bites a project adopting `awt` with a `.gitignore` from older tooling, because
none of the `awt-*` entries are in it and the first `git add -A` commits four symlinks and an index.

That file **is** the statement of what is transient — keep the rationale in its comments rather than
restating the split anywhere else, where the two can disagree.

Anything project-specific — the dev port, which wikis this one references, how to run the build — goes
in that tracked `site/README.md`, the same way project-specific wiki rules go in `meta/` notes rather
than into this skill. **A project owns its own site**: the pin, the port and the registry are its
decisions, and it must be able to build with no other checkout of yours present.

> **Record the invariant, not the counts.** *"This wiki has 10 placeholders at 19 call sites, and all
> 19 are expected"* is falsified by deleting one stale `[[link]]` in an unrelated note, and it cannot
> be refreshed by hand: the graph's placeholder list and the rendered check count different things — 10
> placeholders across 21 reference lines versus 19 call sites, for one real wiki in one state — so the
> only way to restate the numbers is to run the check and copy from it, which is what the sentence
> claimed to be documenting. Write the property instead: *every unresolved link is expected to be a
> placeholder; the check reports the counts.* Same rule as a wiki note not carrying counts of the
> wiki's own contents, one file further out.

## First-time setup

Everything below is a subcommand of **`awt`**, installed once per machine and on `PATH`. **A project
never depends on the tool's git repository**, and no file in it names a tool version. If the command
is missing, the tooling is not installed on this machine.

> **Installing it on a machine that hasn't got it** (check with `awt --version`). It is **not on
> npm**: from a checkout of `agent-wiki-toolbox`, `bin/install` copies it to
> `~/.local/lib/agent-wiki-toolbox` and puts `awt` on `~/.local/bin`. That is the only step that
> exposes a change, so re-run it after editing the toolbox — and then re-run `awt bootstrap-quartz` in
> each project, because Quartz never re-resolves a plugin it has already installed. Quartz itself
> needs **Node ≥ 22**, a version above what the rest of this tooling requires.

```shell
echo <pinned-sha> > site/quartz.pin
awt bootstrap-quartz
```

That clones Quartz at the pinned commit, symlinks the config into the clone, symlinks the toolbox's
Quartz plugins beside it, clears the plugin cache and installs. Idempotent — re-run it to apply a
`quartz.pin` bump or to pick up a newly installed toolbox. `awt --version` reports which installation
is actually running.

`awt bootstrap-quartz`, `awt serve` and `awt publish` run from **anywhere inside the project** — they walk up for
the nearest `site/quartz.config.yaml` and print the project they found. Pass `--site` / `--wiki` only
to point at something other than the project you are standing in; those resolve against the current
directory.

> **`awt check` does not walk up.** It reads `$AWT_WORKSPACE`, else the current directory — so from a
> repo root `awt check` reads the whole repo rather than the wiki, and reports every stray markdown
> file outside it. Pass `-w docs/wiki`, or export `AWT_WORKSPACE=docs/wiki`.

**It writes about 300 MB into `site/`**, nearly all of it Quartz's own `node_modules` — which is why
it writes `site/.gitignore` first if the project has none yet, per
[What lives in `site/`](#what-lives-in-site-and-what-git-tracks).

**A passing build is not evidence the setup is complete.** A plugin symlink is only load-bearing when
the config sources a plugin from it, so a wiki configuring none of them builds and passes with the
symlinks missing entirely — observed on a 289-page site whose `site/` had been populated from another
checkout. Run `awt bootstrap-quartz` because the project has not run it, not because something broke.

> **Quartz never re-resolves an installed plugin.** If `.quartz/plugins/<name>` exists it returns early
> without comparing what is installed against what is configured, so changing a plugin source is a
> silent no-op until that directory goes — the build succeeds and quietly keeps the old code. Clearing
> it is why re-running `awt bootstrap-quartz` is the one act meaning "take the current tooling".

**Quartz publishes no usable release**, which is why a project pins a commit SHA rather than a tag:
the newest GitHub release is from 2023, and the lone `v5.0.0` tag is hundreds of commits behind the
`v5` branch and does not build at all — its plugin installer clones repos that ship no `dist/`, then
crashes before writing the index it needs. Read the shadow's line on every bump, because a branch is
not a release and the shadow is what notices the renderer's slug rules moving.

The config is symlinked **into** the clone because Quartz reads `quartz.config.yaml` *and*
`./package.json` from the current working directory — so the working directory has to be Quartz's own,
while the config still lives in `site/` where it is tracked and reviewable.

Copy an existing project's `quartz.config.yaml` as the starting point and change three things: the
`pageTitle`, the `baseUrl` port, and the `note-properties` field list — **that last one is per wiki**,
because front-matter vocabularies differ between wikis and a field named there that the wiki does not
carry simply renders nothing.

**None of this touches the project's own `node_modules`.** Quartz installs into
`site/.quartz-src/node_modules`, which has its own `package.json`, so npm never walks up to the project
root — which matters in a repo that already has a frontend build of its own.

## Build and publish are different modes

| Mode        | What it does                                        | URLs               |
|-------------|-----------------------------------------------------|--------------------|
| **Build**   | Generates HTML and serves it locally, hot-reloading | Local, disposable  |
| **Publish** | Makes a site reachable from other machines          | A global namespace |

```shell
awt serve       # build mode, hot-reloading, on the project's configured port
awt publish     # publish mode, into site/release
```

**`awt serve` emits `site/.awt-index.json` and then runs the build**, in that order and in one
command, because the two must not come apart: the shadow below compares the rendered pages against
that index, so a build started any other way compares against whatever was last written. Since any
edit invalidates it, that is a failure reported on a wiki where nothing is wrong. The shadow now
refuses a missing or out-of-date index rather than comparing against one.

It passes `-d` explicitly rather than relying on the `content` symlink `quartz create` leaves behind
— that symlink lives inside the untracked clone and is not reproducible from the repo.

> ### Read a site with `--serve`, always
>
> Not a convenience — two independent reasons, both of which bite silently.
>
> Quartz emits **extensionless URLs**, so opening `public/` in a browser, or serving it without a
> fallback, 404s every internal link. A static server needs one rule — *try `$uri.html` before
> returning 404* — and the dev server has it built in.
>
> And a build **without** `--serve` is *publish* mode. Quartz exposes `argv.serve` to transformers, so
> that flag is what tells the cross-wiki resolver whether to emit local or published URLs. Run a plain
> `build` to "just generate the HTML" and every cross-wiki reference takes the publish branch — in a
> project with no published host, that leaves them unresolved.
>
> `awt serve` always passes it and `awt publish` never does, which is the point of having both rather
> than one command with a flag to forget.

### Choosing the ports

Give each wiki on a machine **its own port pair**, and put it in the project's `awt.config.mjs` so
`awt serve` takes it and nobody retypes it:

```js
export default {serve: {port: 8101}}   // the hot-reload socket derives as port + 100
```

Two rules, and both are about collisions rather than taste.

**Every wiki on the machine gets a distinct port.** Cross-wiki links in build mode point at a
dependency's **dev** base URL, so both wikis have to be servable at once — a shared port means the
dependency's links resolve to whichever server happens to be up. Record the assignment in
`site/README.md` as well, because that is where someone looks before picking the next one.

**Start at 8100 and go up.** Everything below is somebody's default — 3000, 4200, 5173, 8000, 8080 —
so a wiki parked there collides with whatever the project itself runs, intermittently and only for
whoever has both up, which is the hardest kind of collision to diagnose. `awt serve` defaults to 8100
for that reason but **will use any port you configure**: this is a convention for whoever sets the
project up, not a rule the tool enforces, because which range is free is a fact about the machine and
not about the toolbox.

So a machine's assignment reads 8101, 8102, 8103 as wikis are added — sockets 8201, 8202, 8203,
derived, staying a port apart with them. Existing projects below the floor are not worth churning;
apply it to the next one.

> ### Confirm what is serving before believing a page
>
> **A held port fails loudly, and then the failure gets swallowed.** Quartz prints its success line
> *before* the failure:
>
> ```
> Emitted 617 files to `../public` in 36s
> Started a Quartz server listening at http://localhost:8101
> Port 8101 is already in use. Try a different port with --port <number>
> ```
>
> The bare command **exits 1**, honestly. But trimming that output — `| tail`, `| head`, the default
> reflex for a build this chatty — makes the shell report the *last* stage's status, so the 1 becomes a
> 0 and nothing flags it. Measured both ways on one run: bare `1`, piped `0`.
>
> ```shell
> node quartz/bootstrap-cli.mjs build … --serve --port 8101 | tail -2
> echo ${PIPESTATUS[0]}    # bash — the build's own status, not tail's
> echo $pipestatus[1]      # zsh
> set -o pipefail          # or make the pipeline fail by itself, in either shell
> ```
>
> **Read it immediately.** Any command in between — an `echo`, a `[` test — resets it, and you get a
> confident `0` that means nothing.
>
> **And the port still answers regardless**, `HTTP 200`, from whichever server got there first. If that
> one was started from the same `-d`, it is hot-reloading the same wiki, so it serves your
> *uncommitted* edits: live, current, correct-looking, and not your build. This is how a site gets
> reported as verified when nothing was verified.
>
> Ask who owns the port rather than whether it answers:
>
> ```shell
> lsof -nP -iTCP:8101 -sTCP:LISTEN
> ps -o lstart=,command= -p <pid>     # started before your build = not your server
> ```
>
> Same class as the box above, one level up: that one is about reading stale *HTML*, this is about
> reading a stale *server*, and this one is worse because nothing about the page looks old.

## Two checks, and neither is the other

**The toolbox owns the link graph. Quartz is a renderer that must be verified to agree with it,
before any customisation.** They are two independent implementations of the same resolution rule, and
that overlap is the only one — so it is checkable, and a mismatch means the rendered site is a
plausible lie.

**The graph on disk** is `awt check -w docs/wiki`: broken links, ambiguous links, missing anchors, and
placeholders reported separately. Run it after any bulk edit. It reads files and builds nothing, so it
is safe to run while a dev server is up.

> A **placeholder is the backlog signal and never fails it** — a `[[note]]` nobody has written yet is
> what every working wiki has, and failing on those would mean no wiki with a backlog could pass. An
> **ambiguous** link does fail: a stem matching two notes is a question for the author, not something
> a tool gets to guess at.

**The rendered site** is the `awt-links` shadow, a Quartz plugin that runs inside every build and
compares what the renderer resolved against the index emitted moments before. Nothing to run by hand.
Configure it with `failOnDisagreement: true` and a disagreement stops the build.

```yaml
  - source: "../awt-links"
    enabled: true
    options:
      index: ./.awt-index.json
      failOnDisagreement: true
```

**Do not read a clean first run as a result.** What the shadow exists to catch is Quartz's slug
semantics moving underneath the toolbox, and that happens when the pinned SHA moves — so a `quartz.pin`
bump is the run whose output is worth reading, and an unchanged rebuild proves nothing.

Two differences are **deliberate**, and the shadow models both rather than reporting them:

- **An ambiguous link.** The toolbox calls two matches an error; Quartz resolves only on a unique
  match and otherwise falls through *silently* to a root-relative slug — usually a 404, and
  occasionally a real page, which is what `[[README]]` does in a wiki holding four of them. The shadow
  cannot predict which, so it excludes that landing slug in both directions.
- **A cross-wiki reference.** A destination carrying a prefix is external to the graph and the
  resolver's to report — see [Cross-wiki references](#cross-wiki-references).

> **A note's basename must not equal its parent folder's name.** Quartz collapses `foo/foo.md` into
> that folder's index page — slug `…/foo/index` — so `[[foo]]` resolves to a URL that never exists,
> while the graph reports a perfect wiki. Unique basenames are **not** sufficient.
>
> A note sitting **beside** a folder of its own name — `campaigns/v2.md` next to `campaigns/v2/` — is
> the opposite case and is handled: without help the generated folder listing takes that URL and the
> note is shadowed by it, silently, with every breadcrumb pointing at the listing. `awt-folder-notes`
> serves the note as the folder's landing page instead, at the address `awt check` computes and the
> index publishes. Where *two* notes claim one landing page, `awt check` reports it rather than
> picking.

## Cross-wiki references

Only relevant when a wiki references another one. Rendering a single wiki needs none of this.

A cross-wiki reference is an ordinary markdown link whose destination carries a **wiki prefix** instead
of a scheme:

```markdown
See [conventions](shelton-dios:meta/conventions.md).
```

**Never a `[[wikilink]]`** — any unresolved `[[…]]` is a placeholder, so every cross-wiki reference
would land permanently in the placeholder list and poison the one signal that means *a note worth
writing*. A prefixed link is external to the graph, an ordinary destination to `awt fmt`, and a
resolvable reference to the renderer. Verified: a wiki with prefixed references still reports zero
placeholders and zero errors, and the formatter leaves the destinations untouched.

Two rules that are the inverse of the intra-wiki ones. **Always a full path within the target wiki,
never a bare stem** — uniqueness is guaranteed inside one wiki and guaranteed absent across several.
And **the prefix is a registry key, not a repo name**: it is typed in every reference and read in every
URL, so it may be shorter than the repo it points at.

The resolver is a Quartz transformer, configured **before `crawl-links`**, which would otherwise treat
the prefixed destination as a relative path inside the wiki being built. **The source is the gitignored
symlink `awt bootstrap-quartz` creates**, never a git URL: a project depends on the machine's install,
not on the tool's repository, and a git source would walk straight into the no-`dist/` trap in
[Traps](#traps) — installing fine while leaving every cross-wiki reference silently unresolved.

```yaml
  - source: "../awt-cross-wiki"
    enabled: true
    order: 55
    options:
      self: this-wiki-prefix
      registry:
        other-wiki:
          dev: http://localhost:8101
          buildIndex: ../../other-wiki/site/public/static/contentIndex.json
          published: https://wiki.example.com
          publishedIndex: ../../other-wiki/site/release/static/contentIndex.json
```

`dev`/`buildIndex` are used in build mode and `published`/`publishedIndex` in publish mode, chosen
from `argv.serve` rather than from a flag — so a published site cannot end up full of localhost URLs
because somebody forgot one.

It **reads the slug from the target wiki's own `contentIndex.json`**, never computes it. That index
carries `filePath` beside `slug`, so resolution is a lookup of the URL that site actually serves —
recomputing another wiki's path rules is a second implementation free to drift, and it fails by
rendering a link that points nowhere. Existence checking falls out of the lookup for free, and answers
the question a link actually asks: not *does this note exist* but *is this page served*.

Everything unresolvable is a **warning, never a build failure** — unknown prefix, unreadable index,
missing path. A wiki has to build before the wikis it depends on have ever been built, or nothing
bootstraps.

**Wire the resolver in the direction the mount already runs.** If wiki A mounts wiki B read-only and B
deliberately does not mount back — because B is a product that must build with no sibling checkout
present — then only A gets a registry, and B's config gains no path pointing at a sibling. Mirroring
the wiring "for symmetry" reintroduces exactly the dependency the mount rule exists to prevent.

A reference naming the wiki currently being built becomes an ordinary internal link instead of an
absolute URL, so it keeps hover previews, backlinks and in-page navigation. (A new backlink appears
only after a full rebuild — an incremental one does not re-emit the target page.)

**Anchors are checkable, one plugin further on.** `contentIndex.json` carries no heading list, so a
reference like `other:meta/scope.md#a-heading` used to resolve to the page and say nothing about the
heading. `awt-headings` emits `awtHeadings.json` beside the content index, and the resolver reads it
from the target wiki: an anchor that wiki does not publish **warns**, never fails, by the same rule as
everything else here.

```yaml
  - source: "../awt-headings"
    enabled: true
    options:
      index: ./.awt-index.json
```

Both wikis need it — the one being referenced emits the file, the one referencing it reads. A target
built without it is silent rather than noisy: it has not said its anchors are unknown, it has said
nothing, and warning about every anchor into it would drown the ones that mean something.

**Still not covered:** backlinks do not cross wikis, and the registry is duplicated per repo.

## Traps

- **A wiki whose home is `README.md` serves a 404 at `/`.** The renderer slugs it to `readme`, and
  nothing lands on the root — which is the first URL anybody opens. The graph accepts either name for
  the home note, so this only shows up once the wiki is rendered. Fix it with a redirect rather than a
  rename: `aliases: [index]` in that note's front matter emits an `index` page pointing at it, leaving
  every existing `[[README]]` link intact. Renaming to `index.md` also works but is the bigger change
  — OKF reserves `index.md` as front-matter-free, so a home note carrying classification would have to
  shed it.
- **A plugin that works as a local path can still fail from its tag.** The two source forms take
  different code paths: a local path is **symlinked** and loaded as-is, while a git source is
  **cloned**, then `npm install --ignore-scripts` and — whenever the clone ships no `dist/` —
  `npm run build` are run inside it. A plain-JavaScript plugin with no build step therefore installs
  perfectly during development and fails from git with `Missing script: "build"`. The install error is
  visible, but the *consequence* is not: the build continues, the plugin is simply absent, and every
  cross-wiki reference is silently left unresolved. **Always rebuild from the tag before relying on
  it**, and give a no-build plugin a no-op `build` script.
- **Never point plain remark at a wiki.** Without `remark-wiki-link` it escapes every `[[wikilink]]`,
  which still renders while every backlink and placeholder silently disappears. See the main skill file.
- **`remove-draft` keys on a `draft:` field, not on `status: draft`.** A wiki using `status` loses
  nothing to it — but check, rather than assume, because the failure is notes vanishing with no error.
- **`UnlistedPages`, `EncryptedPages` and `remove-draft` make "exists" and "is served" diverge.** That
  divergence is why cross-wiki checking reads the content index rather than the target's markdown.
- **Running a non-`--serve` build over a directory a dev server is serving** replaces its resolved
  cross-wiki links with unresolved ones. `awt publish` builds into a staging directory and renames it
  into place, and refuses to build over `site/public` for exactly this reason; a hand-rolled `build`
  is what to avoid.
- **Editing the toolbox working copy exposes nothing.** `bin/install` is the step that does, and a
  project takes new Quartz plugins only by re-running `awt bootstrap-quartz` after it. Two steps, both
  easy to skip, and skipping either leaves the build running the old code with no sign of it.
