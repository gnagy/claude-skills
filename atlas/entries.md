# Catalog entries

One entry per resource. An entry is a markdown note in the catalog's wiki: front matter with a
few fixed fields, then prose. The prose says what the thing is and anything the fields cannot
carry. Keep it short; the entry points at the resource's own documentation, it does not repeat it.

## Two kinds of entry

**Identity entries** describe something that is the same everywhere: a **project** or a **repo**.
They carry no path.

**Placement entries** describe something at a path: an **environment**, a **workspace** or
**material**. A path is true in one environment only, so every placement entry lives in that
environment's section of the catalog and nowhere else. The identity entries are what the sections
have in common.

## The fields

| Field         | On                                   | Answers                                                                                        |
|---------------|--------------------------------------|------------------------------------------------------------------------------------------------|
| `type`        | every entry                          | Which kind of thing this is                                                                    |
| `location`    | repo, environment, workspace, material | Where it is: a URL for a repo, a path for the rest                                             |
| `project`     | project, workspace                   | Which project this is part of: the parent of a subproject, or the project a workspace places   |
| `checkout-of` | workspace                            | Which repo this directory is a clone of. Present, the workspace is a checkout                  |
| `authority`   | project                              | The shared catalog whose facts these are, when the project has one                             |
| `observed`    | every placement                      | The date the thing was last looked at, `YYYY-MM-DD`; what the entry says was true then         |

Each field answers one question. **Containment is never written**: which workspace holds which
is read from the paths in a section, never from a field.

## Where entries go

Identity entries sit outside every section; each environment has one section, a folder named for
it, holding its placements, with the environment entry at the section's head. The default:

```text
index.md
projects/<project>.md            identity
repos/<repo>.md                  identity
environments/<name>/<name>.md    the environment entry
environments/<name>/…            its workspaces and material
```

The environment entry is its section's own page: it lists what the section holds, the folder has
no `index.md` beside it, and a link to it carries the folder path, `[[environments/<name>/<name>]]`,
since a bare stem does not reach it. The catalog may lay the identity entries out differently;
what the skill fixes is that no placement sits outside a section and no identity entry sits
inside one. Fields name identity
entries by filename, so those names are unique across the catalog. Placement entries are named by
nothing, since a placement is never pointed at, so the same filename may recur in two sections;
link one, if ever, with its folder in the link.

## The shape of an entry

```markdown
---
type: workspace
location: ~/work/acme/api
checkout-of: github-acme-api
---
```

## Identity entries

### project

A named body of work, and the main organizing entry. It is not local: a project has no
`location`, and where it is in a given environment is answered by the workspaces there that name
it. Projects can nest: a subproject is a project entry whose `project` names its parent, and
nothing else marks it as one. The nesting is kept on the project entries so that every
environment reads the same project tree, whether or not it holds a workspace for any of it.

`authority` names the shared catalog when the project's facts are kept in a project repository
rather than here; see *Who owns which facts* in `SKILL.md`. The prose says what the project is
and which document to read next.

### repo

A version control repository, typically git: `location` is its URL. It is a shared resource, so
it carries nothing about one user or one environment. A repo can exist with no remote copy; it
still has a version history, and the entry says that it exists in one place only.

## Placement entries

### environment

One place where paths resolve: a host, a VM, a container, a sandbox with its own filesystem. Two
contexts are one environment when a path recorded in one opens the same files in the other, so a
git worktree is another workspace in the same environment, not a new one.

Each environment has its own checkout of the catalog and its own section in it, holding every
placement that exists there. `location` is that checkout's path, so another environment can say
where the catalog is over there. A session learns which environment it is in from a local-only
agent file at the catalog checkout, `CLAUDE.local.md` for Claude Code, naming the environment;
*Finding the catalog* and *Instruction files in an environment* in `SKILL.md` have the rule and
the file's shape. An environment that is thrown away takes its
section with it; one that is kept costs nothing.

### workspace

A directory where work happens, for agents, IDEs and other tools. `location` is its path. It may
be a remote share, as long as it is mounted into the environment's filesystem.

`project` names the project this workspace is a placement of, and is written **only where the
answer changes**: on the outermost workspace of a project, and on a nested workspace that belongs
to a subproject. A nested workspace never repeats its parent's project; a workspace with no
`project` and no cataloged workspace above it belongs to no project, which is a plain grouping.

**With `checkout-of` a workspace is a checkout**: one clone of a repo, in this environment. When
the working copy has several remotes, which repo it is a checkout of is your call; `origin` is
only a default. There is no separate checkout type, because the one fact that makes a workspace a
checkout is the field. So the simplest case, cloning a repo and working in it, is one entry; a
project that is one repo is that entry with `project` beside `checkout-of`; a submodule is a
checkout inside a checkout; a monorepo subdirectory assigned to a subproject is a workspace inside
a checkout, with `project` and no `checkout-of`.

A placement dies with its directory. The repo and project entries it named outlive it. Every
placement carries `observed`, the date it was last looked at: a placement is a claim about a disk
that changes without anyone editing the entry, and the date is what tells a reader how far to trust
it.

### material

Files that matter to a project but live in no repository: data directories, documents, scans, a
read-only share. `location` is its path. The prose says what it holds, how to read it, and what
may be done with it; where the repo beside it cannot describe it, because it is gitignored, this
entry is the only description.

### unknown

Something the user or an agent added to the catalog by explicit request, not while charting, but
whose type and details are yet to be determined. Ignore it when looking for information. List
them to the user when they ask, so they can inspect and catalog them.
