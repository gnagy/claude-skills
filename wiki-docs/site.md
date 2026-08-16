# Rendering a wiki as a site

A Foam wiki is written for agents and for whoever edits it, and **neither is a reader**. Rendering it
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

**Do not gitignore `site/` wholesale.** Two of its four entries are the site; the other two are
machine-local and rebuildable:

| Path                      | Tracked | What                                                              |
|---------------------------|---------|-------------------------------------------------------------------|
| `site/quartz.config.yaml` | **yes** | The whole configuration — the only file worth reviewing           |
| `site/quartz.pin`         | **yes** | One line: the Quartz commit this project builds against           |
| `site/README.md`          | **yes** | Project specifics — the port, the registry, how to run it         |
| `site/.gitignore`         | **yes** | Ignores the two below, and says why                               |
| `site/.quartz-src/`       | no      | The Quartz clone, its `node_modules`, and a symlink to the config |
| `site/public/`            | no      | Build output                                                      |

```gitignore
# Quartz itself is cloned at a pinned commit by the build, never vendored.
# Nothing of it belongs in this repo.
.quartz-src/

# Build output. Machine-local and rebuildable; in publish mode the last one is
# the release record, which is still not something git should carry.
public/
```

That file **is** the statement of what is transient — keep the rationale in its comments rather than
restating the split anywhere else, where the two can disagree.

Anything project-specific — the dev port, which wikis this one references, how to run the build — goes
in that tracked `site/README.md`, the same way project-specific wiki rules go in `meta/` notes rather
than into this skill. **A project owns its own site**: the pin, the port and the registry are its
decisions, and it must be able to build with no other checkout of yours present.

## First-time setup

Everything below comes from **`quartz-wiki-tools`**, pinned by tag. Nothing is copied into the
project: a per-repo copy of a shared script is drift waiting to happen.

```shell
echo <pinned-sha> > site/quartz.pin
npx github:gnagy/quartz-wiki-tools#v0.3.0 bootstrap-quartz
```

That clones Quartz at the pinned commit, symlinks the config into the clone, and installs. Idempotent
— re-running after editing `quartz.pin` is how a Quartz bump is applied.

**Quartz publishes no usable release**, which is why a project pins a commit SHA rather than a tag:
the newest GitHub release is from 2023, and the lone `v5.0.0` tag is hundreds of commits behind the
`v5` branch and does not build at all — its plugin installer clones repos that ship no `dist/`, then
crashes before writing the index it needs. Re-run the link graph check on every bump, because a branch
is not a release.

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
# from site/.quartz-src
node quartz/bootstrap-cli.mjs build -d ../../docs/wiki -o ../public --serve --port 8101 --wsPort 3101
```

Pass `-d` explicitly rather than relying on the `content` symlink `quartz create` leaves behind — that
symlink lives inside the untracked clone and is not reproducible from the repo.

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

Give each wiki on a machine its own port pair, and record it in `site/README.md`: cross-wiki links in
build mode point at a dependency's **dev** base URL, so both wikis have to be servable at once.

## The link graph check

**Foam owns the link graph. Quartz is a renderer that must be verified to agree with it, before any
customisation.** They are two independent implementations of the same graph, and that overlap is the
only one — so it is checkable, and a mismatch means the rendered site is a plausible lie.

The criterion is a **set comparison**, not "zero broken links": the links the renderer failed to
resolve must match `foam list placeholders` exactly, in both directions. A wiki with 17 placeholders
should render exactly those as broken and nothing else. A check that asserted zero would be wrong for
every wiki that has a backlog.

Three things a real check needs, each learned by it being wrong first:

- **Resolve links against the filesystem**, using the host's own rule — try `$uri`, then `$uri.html`,
  then `$uri/index.html`. That catches a dead link the renderer failed to *mark*, which reading the
  `broken` class alone cannot.
- **Links to placeholders are supposed to be unresolved.** Count them separately; only an unresolved
  link *outside* the placeholder set is a defect.
- **A destination carrying a scheme is external** — including an unresolved cross-wiki reference like
  `otherwiki:meta/conventions.md`. Those are the resolver's to report; this check owns one wiki's graph.

Foam resolves by **basename**, so it can report a placeholder for a name that several files carry while
the renderer resolves the relative link pointing at one of them perfectly well. Worth surfacing, not
worth failing on.

> **A note's basename must not equal its parent folder's name.** Quartz collapses `foo/foo.md` into
> that folder's index page — slug `…/foo/index` — so every `[[foo]]` resolves to a URL that never
> exists, while `foam lint` reports a perfect graph. Unique basenames are **not** sufficient.

Run the check on every Quartz bump, and after any bulk edit. An obligation described in prose is not
one anybody can discharge — keep it as an executable script, and keep it in `bin/` rather than `site/`,
because what it verifies is the *wiki's* graph and the site is only the instrument.

## Cross-wiki references

Only relevant when a wiki references another one. Rendering a single wiki needs none of this.

A cross-wiki reference is an ordinary markdown link whose destination carries a **wiki prefix** instead
of a scheme:

```markdown
See [conventions](shelton-dios:meta/conventions.md).
```

**Never a `[[wikilink]]`** — Foam registers any unresolved `[[…]]` as a placeholder, so every
cross-wiki reference would land permanently in `get_placeholders` and poison the one signal that means
*a note worth writing*. A prefixed link is external to Foam, an ordinary destination to remark and
`mdfmt`, and a resolvable reference to the renderer. Verified: a wiki with prefixed references still
reports zero placeholders and zero lint errors, and `mdfmt` leaves the destinations untouched.

Two rules that are the inverse of the intra-wiki ones. **Always a full path within the target wiki,
never a bare stem** — uniqueness is guaranteed inside one wiki and guaranteed absent across several.
And **the prefix is a registry key, not a repo name**: it is typed in every reference and read in every
URL, so it may be shorter than the repo it points at.

The resolver is a Quartz transformer, configured **before `crawl-links`**, which would otherwise treat
the prefixed destination as a relative path inside the wiki being built:

```yaml
  - source: "github:gnagy/quartz-wiki-resolver#<tag>"
    enabled: true
    order: 55
    options:
      self: this-wiki-prefix
      registry:
        other-wiki:
          dev: http://localhost:8101
          buildIndex: ../../other-wiki/site/public/static/contentIndex.json
```

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

**Not covered:** anchors are unverifiable, because the content index carries no heading list; backlinks
do not cross wikis; and the registry is duplicated per repo.

## Traps

- **A wiki whose home is `README.md` serves a 404 at `/`.** The renderer slugs it to `readme`, and
  nothing lands on the root — which is the first URL anybody opens. Foam accepts either name for the
  home note, so this only shows up once the wiki is rendered. Fix it with a redirect rather than a
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
  cross-wiki links with unresolved ones. Restart the server afterwards.
