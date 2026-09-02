---
name: atlas
description: Use this skill when a question runs past one repository — what a directory is, which project it belongs to, what depends on it, what a working copy is a checkout of, or what has to be cloned before a task can run. Also when a tree contains an `atlas.md` marker, or when asked to walk a tree, or to write or keep a catalogue of one. Covers the units and relations, finding and reading a catalogue, keeping one, the marker format, and walking an uncatalogued tree. Runs on `/atlas` as well.
---

# Atlas

A **catalogue** records what a tree of repositories cannot say about itself: which directories
belong together, what each one is for, and what depends on it from outside. **Atlas** is the model
catalogues are written in, so one person's catalogue is readable by someone else's agent. An
`atlas.md` **marker** lets a directory declare its own place. `README.md` beside this file says
what atlas is for; this file says what to do.

**Reading is free; writing is asked for.** Answer from a catalogue or a marker whenever the
question comes up. Write an `atlas.md` only when someone asks for one, and start a walk only when a
person starts it. Noticing that a tree would be easier to work in with markers is worth one
sentence to whoever owns it, not a marker.

## Finding the answer

From the directory in question:

1. Look for an `atlas.md` here, then in each parent directory upward. Stop at the first one found.
2. If it has no `authority:`, read it and keep going up: it describes this place, and the catalogue
   that owns the region is further up.
3. If `authority: self`, the catalogue is in this repository, starting at `atlas/index.md`. If
   `authority:` names a repository, the catalogue is there; open or clone it and start at its
   `atlas/index.md`.
4. Find the entry for the directory, or for its nearest ancestor that has one, and answer from it.
   If the catalogue does not say, say so. Do not infer membership from the tree.
5. With no marker anywhere above, there is no catalogue for this tree. Answer from what is on disk
   and say it is a reading, not a record.

Example: in `~/work/acme/platform/api/` the nearest marker is `~/work/acme/atlas.md`, with
`authority: git@github.com:acme/acme-project.git`. Open that repository's `atlas/index.md`, find
the entry for `platform`, and read what `api/` is from there.

## Units

| Unit          | What it is                                                                                            |
|---------------|-------------------------------------------------------------------------------------------------------|
| **project**   | A named body of work. Nestable. May span workspaces, machines and roots, and may have no files at all |
| **workspace** | A place work happens. See below                                                                       |
| **repo**      | An identity, usually a remote. Independent of where it is checked out                                 |
| **checkout**  | A repo in a directory on one machine. The thing you move, delete, back up                             |
| **material**  | Files in no repo, described by nothing else                                                           |

**A directory holds a placement, never an identity.** A `repo` appears only in a catalogue. A
directory containing a working copy is a `checkout`, including when that working copy is the whole
of the project's work; its marker body names the project, and the catalogue records `member-of`.
`project` on a directory means the project's presence on this machine where that presence is not
one working copy: a client engagement directory holding several checkouts and material.

Three things are **not** units:

- **Third-party is a role, not a type.** The role belongs to the relation, never to the repo.
- **A README, a `CLAUDE.md` or a wiki is an attribute.** See *How far a project describes itself*.
- **A toolchain is declared in the repo.** Record where the declaration is, never what it says.

## Relations

| Relation      | Joins                                                         | Where it comes from                                      |
|---------------|---------------------------------------------------------------|----------------------------------------------------------|
| `contains`    | A directory to what is under it                               | The filesystem. Derived, never recorded                  |
| `member-of`   | A repo, checkout, material or project to the project it is in | Hand-written. The highest-value fact in a catalogue      |
| `depends-on`  | A project or repo to a project or repo it needs               | Hand-written. Crosses projects, never implies membership |
| `checkout-of` | A checkout to the repo it is a working copy of                | Hand-written. One-way                                    |

**A project owns a unit when the project's people decide its fate**: whether it is kept, changed,
moved or deleted. A vendor clone kept to read alongside is contained and not owned; a fork that is
patched and maintained is owned. `member-of` records ownership. `contains` never implies it.

**Aggregation is a directory holding children it does not own**, and it is normal. A library one
project depends on is a member of its own project and a dependency of the other, never a member
of it. **A thing whose own docs say "read the parent first" is a member of the parent**, not a
project in its own right.

**`checkout-of` runs one way.** A checkout names its repo; a repo record never names a checkout,
and carries no path, branch, directory or link down to one. `origin` is not the authoritative
answer to what a working copy is: a checkout can carry several remotes, and which repo it
principally is, is a judgement someone records.

## Workspace

A grouping of resources someone decides to work on as a unit: a local tree of checkouts, a CI
pipeline, a cloud workspace an agent is given. **Started, not detected**: a person or an agent opens
one, and nothing on disk defines it. **Children are not validated**: a member may be coupled to
its siblings or checked out only to read alongside them.

Recognising one in an existing tree is a heuristic a person confirms. Three signals:

- **IDE or agent configuration at a level** (`.idea/`, `.vscode/`, an `AGENTS.md`) says *where* a
  workspace is. Most reliable, and silent about membership.
- **A module list inside that configuration** names *some* members. Evidence, not truth.
- **Build coupling by relative path** is the strongest membership signal and still incomplete.

## How far a project describes itself

Two questions, asked separately.

**How well does it document itself?** This decides whether the catalogue links or describes.

| Rung          | What is there                                           | What the catalogue does                      |
|---------------|---------------------------------------------------------|----------------------------------------------|
| **Nothing**   | No README, no docs                                      | The catalogue is the only description        |
| **Prose**     | A README, a `CLAUDE.md`, an `AGENTS.md`                 | Link, and describe what the prose leaves out |
| **Organised** | A knowledge base with its own conventions, often a wiki | Link. The catalogue still says what it *is*  |

**Does it declare itself?** An `atlas.md` at its root says what it is in the model's terms and
points at its documentation. Declared: record the pointer. Not declared: the catalogue decides.

**Check that a document names the thing it sits in.** A README about a different repository grades
as *Prose* and misleads everyone after you. Where it does not, grade *Nothing*, describe the thing
yourself, and record that the document is wrong. A nested project's documentation is its own; it
inherits nothing from the project containing it.

## Derive, don't transcribe

Never record what a tool can read from the files: remotes, commit dates, sizes, dependency lists.
Two exceptions:

- **A thing with no files here.** An idea, an archived project, something on a share you cannot
  read. Write it by hand.
- **Facts needed before the files exist.** What to clone and how large it is, for an agent deciding
  whether to. Record those with the date you measured them.

## The catalogue points, the project instructs

Setup and process live **in the project**, in documents an agent can follow. The catalogue records
that the project exists, where it is, which project it belongs to, and which document to read next.
It never explains how to build anything. Where a project documents nothing, the catalogue describes
it; that is the one exception, and why `material` is a unit.

## Local rules

A catalogue may add rules for its own tree: *every `data/` beside a checkout here is gitignored
working material*, *anything under this root not in the module list is held, not owned*. Adding
none is the ordinary case. **Apply the model's rules first, then the local ones**; where they
disagree, say which you think is wrong and stop. **Local rules classify; they never add a value,
redefine `workspace` or add a relation.** Where no value fits, write `unknown` and say in the body
what did not fit. Where the additions live is the catalogue's business.

## Entries

**One unit, one entry.** A repo and its checkout are two entries with two names: an identity after
its owner and remote, a placement after its directory.

**Every entry records when it was last looked at.** Its subject is elsewhere and goes stale with
nothing in the file changing.

**An entry is in one of four states, and all four are useful:**

| State         | What it says                                    |
|---------------|-------------------------------------------------|
| No entry      | Nobody has looked, or a parent's entry names it |
| `unknown`     | Somebody looked and could not tell              |
| A unit type   | Somebody decided what it is                     |
| `aggregation` | Somebody decided it holds what it does not own  |

Where a parent's entry already names a child and says what it is, the child needs no entry. Where
somebody looked and found nothing to add, write the entry anyway: *looked and found nothing* and
*nobody looked* are different facts. `aggregation` needs to know what the directory is *for*, which
is not on disk; from a listing alone, `unknown` is the honest record.

**Never write a word that is only true from where you are standing.** *Ours*, *the client's*,
*third-party*, *upstream of us*: name the owner instead.

**Where a catalogue exists, keep the markers thin.** A marker must stand alone; a catalogue holds
what several markers would otherwise repeat.

**The entry format is not fixed yet.** In an existing catalogue, follow its conventions. Starting
one, keep to the rules above and record the shape you chose in the catalogue's own conventions.

## Keeping a catalogue

A tree is walked once; a catalogue is kept for as long as the work lasts. Everything here is done
when asked.

- **Add**, when something lands the catalogue does not have. One unit, one entry.
- **Correct a type.** A type is a claim: a container that turns out to have a purpose was a
  workspace, not an aggregation.
- **Resolve an `unknown`.** Ask whoever put the directory there what it was for. That answer is
  not on any disk and stops existing when they forget.
- **Write down a rule you have applied twice**, with the catalogue's conventions.
- **Retire.** A placement record dies with its directory; an identity record outlives it.

**Two checks settle coverage:**

- **Every entry's location is distinct.** Two entries for one directory means identity and
  placement were conflated.
- **Everything in the region has an entry, or is named in a parent's entry with a reason.** A
  directory that is not a unit at all goes here, named by its parent as not a thing, and why. It
  never gets an entry of its own.

**Record decisions; measure separately.** Counts and sizes are true on the day and stale after;
they go wherever the catalogue keeps its chronology, dated.

**When the tree and the catalogue disagree:** on what exists, the tree wins, so re-observe and
correct the entry. On what things mean, membership, dependency, which identity is primary, the
catalogue wins, because the tree records none of it.

## The marker file

**Read `marker.md` beside this file** before writing an `atlas.md` or walking a tree that has them.

## Walking a tree

**Read `walking.md` beside this file.** It is started by a person, never volunteered.
