# The marker file

An `atlas.md` lets a directory declare its own place, so a catalogue does not have to work it out
from build files, IDE configuration and guesswork. One file, findable anywhere, readable by any
catalogue.

The prohibitions below matter more than the permissions.

## Front matter

Three fields, and each has one reader.

```yaml
---
atlas: project
spec: https://github.com/gnagy/claude-skills/tree/HEAD/atlas
authority: git@github.com:acme/acme-project.git
---
```

| Field       | Says                                                                                             | Read by                           |
|-------------|--------------------------------------------------------------------------------------------------|-----------------------------------|
| `atlas`     | What this directory is — one value from the closed list below                                    | Whoever is reading the file       |
| `spec`      | Which format this is written in. The same URL in every marker                                    | An agent that has never seen one  |
| `authority` | Who owns the facts about this region — a clonable catalogue, or `self`. **Absent is meaningful** | A walker deciding whether to stop |

**`atlas` takes one value:**

| Value                                             | Means                                        |
|---------------------------------------------------|----------------------------------------------|
| `project` · `workspace` · `checkout` · `material` | The unit it is, as `SKILL.md` defines them   |
| `unknown`                                         | Nobody has worked out what this is yet       |
| `aggregation`                                     | A container holding children it does not own |

`repo` never appears here: a directory holds a checkout, and the repo is the identity behind it.

**`unknown` is an answer, and on a first pass it is usually the right one.** It says somebody looked
and could not tell, which is a different fact from an absent marker, and it is the only one of these
values a walk may write on its own — see *Unknown is an answer* in `walking.md`.

**`aggregation` is a verdict, and it comes from a person.** It says a directory is real and does not
own what is under it, which needs to know what the directory is *for* — and nothing on disk carries
that. Written from a walk alone it is a guess wearing the same clothes as the rest of the file.

**There is no value for "not a thing at all".** Where a walk decides a directory is not a unit,
that goes in the catalogue entry for whatever contains it, with the reason — see *Keeping a
catalogue* in `SKILL.md`.

**`spec` is the same URL in every marker.** It is the way in for an agent that has never seen one.

**`authority` decides descent, and absent means the enclosing authority still owns this.** That is the
ordinary case — most markers annotate without claiming anything. `self` says this thing is its own
catalogue. A repository reference says the region is catalogued there, and is clonable so that an
agent starting cold can go and read it.

**A repo root is not automatically an authority.** Some repos are components of a larger thing and
defer to it. Leaving the field out lets them say so, instead of leaving a walker to conclude that a
`.git` directory marks a boundary.

If a directory fits none of the `atlas` values, **write `unknown` and say in the body what did not
fit.** A misfit you can see is worth more than a wrong label that looks correct, and a misfit with a
value is worth more than one buried in prose, because it can be counted and come back to.

## Describing somewhere else

The target defaults to the directory the file sits in. Some things cannot hold a file at all: a repo
you do not own, a read-only share, a mounted volume, a project with no files on this machine. Add
`describes:` with the target and `observed:` with the date in those cases.

Two consequences follow.

- **A walker must not treat a describes-elsewhere marker as covering the directory it sits in.** The
  file is a pointer, not a description of its own neighbourhood.
- **A remote target goes stale silently.** Nothing local changes when the thing it describes moves or
  disappears, which is what `observed:` is for.

## The body

Four parts, in this order, and any of them may be empty.

1. **An abstract.** Two sentences: what this is, and what larger thing it is part of.
2. **Where the real documentation is.** The project's own README, `CLAUDE.md`, wiki or setup
   document. *The catalogue points, the project instructs* — so this file never explains how to build
   anything, it says which document does.
3. **What is directly below**, one level and no deeper. Each child gets a line: what it is, and
   whether it is a member. Deeper levels carry their own markers, which is what keeps this file short
   and the tree walkable one step at a time.
4. **Anything the name does not make obvious** — the backward pointer where the thing exists to serve
   something in particular, and lifecycle where it is not live.

**Prefer a reference to a restatement everywhere.** A marker that summarises the document it points at
has taken on a copy that ages separately from its source.

## What must never

Remotes, sizes, commit dates, toolchains, dependency lists, branch names — **for anything that is
here.** A tool can read every one of those, or they change without anyone editing this file. Written
here they outrank the real source the moment the two diverge, and a reader cannot tell a stale line
from a current one.

**The exception is a thing that is absent.** Where to fetch something that is not on disk cannot be
derived from a disk that does not have it, and an agent starting from a bare tree needs exactly that.
So: **name where to get what is missing, and say nothing about the origin of what is present.** This
is *Derive, don't transcribe*'s second exception, and `authority` is the case that matters most —
clone it and the catalogue holds the rest.

## A project root carries one; below it, a marker earns itself

A project that adopts atlas puts a marker at its root, whatever its documentation looks like: what
this is, who owns the facts, and what is directly below are never in a listing. Below a root, write
one only where the contents do not already say what the thing is, and expect to write few. Being
asked for one is what starts it, in both cases.

**`atlas.md` and `atlas/` are different things, and a repo hosting a catalogue has both:**

| Path       | Holds                                                                   |
|------------|-------------------------------------------------------------------------|
| `atlas.md` | The marker. What *this* directory is, for whoever has just landed in it |
| `atlas/`   | The catalogue this repo hosts — what it records about a wider region    |

`authority: self` is what says the catalogue is here rather than elsewhere, and `atlas/index.md` is
its entry point. A repo that hosts no catalogue has the marker and no directory — which is most of
them.

A source directory never needs one. A folder of scanned documents whose filenames say nothing does.
The failure mode is one in every directory, restating what a listing already shows.

## Examples

A project that catalogues itself:

```markdown
---
atlas: project
spec: https://github.com/gnagy/claude-skills/tree/HEAD/atlas
authority: self
---
# acme-platform

The platform and the workspace it is worked in. Catalogued by its own wiki at `docs/wiki/`.

Setup is `docs/getting-started.md`, which names its prerequisites and proceeds in phases.
```

A directory holding one machine's part of a larger engagement:

```markdown
---
atlas: project
spec: https://github.com/gnagy/claude-skills/tree/HEAD/atlas
authority: git@github.com:acme/acme-project.git
---
# Acme

A client engagement. This directory is where it sits on this machine; the rest is on other
people's machines and in what the client runs.

`platform/` documents itself, wiki included — read that, not this.

Directly below:

- `platform/` — the workspace; its own `atlas.md` covers what is in it
- `vendor/` — upstream clones, kept to read and build against. Not ours, all re-cloneable
- `scratch/`, `screenshots/` — material
```

Material that cannot hold its own marker:

```markdown
---
atlas: material
spec: https://github.com/gnagy/claude-skills/tree/HEAD/atlas
describes: /Volumes/archive/scans
observed: 2026-08-29
---
# scanned reference documents

On a read-only share, so this file lives here instead of there. Roughly 350 GB of scanned
manuals. The filenames are serial numbers and say nothing about the contents. No upstream to
re-fetch from, so this is irreplaceable.
```

## Walking a tree that has them

How markers change a descent is in `walking.md` beside this file, with the rest of the walk.
