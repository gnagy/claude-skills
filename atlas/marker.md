# The marker file

An `atlas.md` lets a directory declare its own place, so a catalogue does not have to work it out
from build files, IDE configuration and guesswork. One file, findable anywhere, readable by any
catalogue.

It is a declaration rather than a derivation, which makes it the file in this model most likely to
fill up with things a tool could have read for itself. The prohibitions below matter more than the
permissions.

## Two roles

The distinction exists because one thing reads this file: a walker descending a tree, deciding
whether to stop.

| `atlas:` | Meaning                                                                                          |
|----------|--------------------------------------------------------------------------------------------------|
| `zone`   | Claims the region. Resolution stops here and continues under this authority                      |
| `entry`  | Annotates a subtree without claiming it. Descent continues, and the enclosing zone still owns it |

Neither is the safe default. A zone is a boundary and should be rare. An entry is an annotation and
is the ordinary case.

**A zone names its authority, or claims it.** Either it says where the region is catalogued — naming
the catalogue instance, a wiki, or a person — or it says nothing and is itself the catalogue for the
region. Without that, the flag says *stop* but not *ask whom*, and resolution dead-ends. **Name the
instance, never the standard**: "catalogued by atlas" is the same dead end, one word longer. See
*Atlas is the standard, and a catalogue has its own name* in `SKILL.md`.

**A repo root is not automatically a zone.** Some repos are components of a larger thing and defer to
it. The flag lets them say so, instead of leaving a walker to conclude that a `.git` directory marks
a boundary.

If a directory fits neither role, **say so in the file and leave it unclassified.** A misfit you can
see is worth more than a wrong label that looks correct.

## Describing somewhere else

The target defaults to the directory the file sits in. Some things cannot hold a file at all: a repo
you do not own, a read-only share, a mounted volume, a project with no files on this machine. State
the target explicitly in those cases.

Two consequences follow.

- **A walker must not treat a describes-elsewhere entry as covering the directory it sits in.** The
  file is a pointer, not a description of its own neighbourhood.
- **A remote target goes stale silently.** Nothing local changes when the thing it describes moves or
  disappears, so record what you observed and the date you observed it. That is the same exception
  that applies wherever a fact is needed before the files exist.

## What goes in

- What this is, in the model's terms: a unit from the list in `SKILL.md`, an aggregation, or not a
  unit at all
- Membership, when the name does not make it obvious
- The backward pointer, when the thing exists to serve something in particular
- Pointers: the setup document, CI/CD, the wiki root and its prefix, lineage
- Lifecycle, when it is not live

## What must never

Remotes, sizes, commit dates, toolchains, dependency lists, branch names.

A tool can read every one of those, or they change without anyone editing this file. Written here
they outrank the real source the moment the two diverge, and a reader cannot tell a stale line from a
current one. **If a tool can read it, do not write it down.**

## An entry must earn itself

Write one only where the contents do not already say what the thing is. A source directory never
needs one. A folder of scanned documents whose filenames say nothing does.

The failure mode is one in every directory, most of them restating what a listing already shows. That
buries the few that carry real information, and it is how this file stops being read.

## Examples

A zone that delegates to its own documentation:

```markdown
---
atlas: zone
---
# acme-platform

Project and workspace. Catalogued by its own wiki at `docs/wiki/`.

Setup: `docs/getting-started.md`, which names its prerequisites and proceeds in phases.
Deploys to production directly. There is no staging environment.
```

An entry covering neighbours it cannot write into:

```markdown
---
atlas: entry
describes: ./
---
# upstream clones

Five checkouts of other people's repositories, kept here to read and build against.
The role is vendor: none of this is ours, all of it is re-cloneable, and none of it
needs backing up. Local modifications, where they exist, are listed in `PATCHES.md`.
```

An entry for material that cannot hold its own:

```markdown
---
atlas: entry
describes: /Volumes/archive/scans
observed: 2026-08-29
---
# scanned reference documents

On a read-only share, so this file lives here instead of there. Roughly 350 GB of
scanned manuals. The filenames are serial numbers and say nothing about the contents.
No upstream to re-fetch from, so this is irreplaceable.
```

## Walking a tree

1. Descend. At each directory, look for an `atlas.md`.
2. On a **zone**, record the delegation and its authority, then stop descending. Everything below
   belongs to that authority.
3. On an **entry**, fold it into what you know about the region and keep descending.
4. On a **describes-elsewhere** file, record it against the stated target and treat the directory it
   sits in as if the file were not there.
5. With **no marker**, infer as usual. Most directories will never have one, and a tree with no
   markers at all must still be walkable.
