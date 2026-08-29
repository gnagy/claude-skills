---
name: atlas
description: Orientation across projects, workspaces, repos and checkouts — what a directory is, what it is a member of, and what depends on it. The catalogue model, the `atlas.md` marker file, and the cleanup, placement and bootstrap workflows. Runs on `/atlas`.
disable-model-invocation: true
---

# Atlas

For people and agents working across more than one repository, most often the two together. An agent
uses a catalogue to explain a tree back to the person who owns it, and to work through a task in one
unattended. Both need the same things first: what is here, what it belongs to, and what depends on
it.

Walking the tree gets you a long way, and this skill says how. It is also slow, part of it is
guesswork someone has to confirm, and the answer is thrown away at the end of the session. The parts
a walk does not recover are the ones somebody decided: which siblings belong together, what a
directory was for, and what depends on it from outside.

A **catalogue** is where those answers get recorded, so nobody works them out twice — markdown notes
holding membership, placement and dependency, and nothing a tool can already read for itself.
**Atlas** is the model they are written in, so one person's catalogue is readable by someone else's
agent.

---

## Orientation

**An orientation is what you need to know before starting, and nothing more**: which project this is
part of, where the rest of it lives, what has to exist before the task can run. Answering that is
what a catalogue is for.

It is assembled for one task and thrown away. **A catalogue holds facts; an orientation is a reading
of them for one purpose**, so a catalogue never stores one. A briefing written in advance is a copy
that goes stale — *Derive, don't transcribe* below, pointed at prose instead of at remotes.

The case that shows the point: someone has an idea and half an hour, and wants a workspace standing
up to work on it with an agent. What they should have to say is the idea. Which checkouts, which
project owns them, what has to be installed and what to read first are all orientation, and a
catalogue that cannot produce them has saved nobody anything.

**Bootstrapping and placing a new thing both end in an orientation.** Cleaning up a tree does not —
it is how a catalogue gets good enough to answer.

## Atlas is the standard, and a catalogue has its own name

**Atlas is this model** — the units, the relations, the delegation mechanics and the `atlas.md`
marker. It is not the name of anybody's catalogue.

**A catalogue is an instance of the model, and takes a name of its own.** That name is what a zone's
authority names, so "catalogued by atlas" says nothing: it names the standard, not the catalogue that
holds the answer.

---

## What belongs here, and what belongs to a catalogue

One question sorts it: **would the sentence still be true on someone else's machine?** True anywhere,
it belongs to this file. True only here — which roots are in scope, where a delegation points, what
is a member of what — it belongs to the catalogue.

**A catalogue note that restates a mechanic from this file is not a second authority. It is a copy
that goes stale silently**, because nothing updates it and the next reader takes it as law. Refer to
this skill by section name instead, so the reference survives an edit inside that section.

## The catalogue points, the project instructs

Setup and process live **in the project**, in documents an agent can follow. The catalogue records
that the project exists, where it is, and that it documents itself.

A catalogue that explains how to build something has taken on a copy of a build file, and that copy
drifts within a week. The clone commands, the branch a sibling has to sit on, the toolchain: all of it
belongs to the project. The catalogue says which project to go and read.

Where a project documents nothing, the catalogue describes it. That is the one exception, and it is
why `material` is a unit.

## Derive, don't transcribe

Never record what a tool can read from the files: remotes, commit dates, sizes, dependency lists. Two
exceptions:

- **A thing with no files here.** An idea, an archived project, something on a share you cannot read.
  Nothing exists to derive from, so you write the entry by hand.
- **Facts needed before the files exist.** An agent deciding what to clone wants the cost before
  cloning, and a remote does not tell you the size. Record those with the date you measured them.

---

## Delegation

A catalogue hands a region to another catalogue, and the mechanism borrows from DNS zones:

- **One authority per region.** No fact has two owners.
- **Delegation is a pointer** — that a region is catalogued elsewhere, and by whom. Nothing else, so
  nothing can drift.
- **A unit belongs to exactly one catalogue**, which is what makes a subtree detachable.

**A delegate catalogue is a project that catalogues itself.** "Sub-catalogue" and "a second
catalogue" are one mechanism at two scales, so the boundary is a question of self-description rather
than of size. Catalogue down to the level where a project can speak for itself, and point from there.

**Delegation points down only.** A catalogue has to work with no parent catalogue present, and must
never point back at one.

The hierarchy covers containment and nothing else. A repo checked out twice, a workspace that uses a
vendor repo: those are references, they cross catalogue boundaries, and a reference has to work from
both ends or it is invisible from one of them.

---

## Units

| Unit                   | What it is                                                                                            |
|------------------------|-------------------------------------------------------------------------------------------------------|
| **project**            | A named body of work. Nestable. May span workspaces, machines and roots, and may have no files at all |
| **workspace**          | A place work happens. See below                                                                       |
| **repo**               | An identity, usually a remote. Independent of where it is checked out                                 |
| **checkout**           | A repo in a directory on one machine. The thing you move, delete, back up                             |
| **material**           | Files in no repo, described by nothing else                                                           |
| **external system**    | A service the work depends on that is not on disk at all                                              |
| **credential pointer** | What is needed, what kind, and who grants it: an account, a group, a VPN, a key. **Never the value**  |

**`repo` and `checkout` are separate on purpose.** One is identity and one is placement, and the same
repo can be checked out twice on one machine.

**A unit type earns structure only when something reads it.** Prose until then. Closed front-matter
fields when entries have to be queried. A vocabulary invented for entries that do not exist yet
produces a taxonomy nobody can fit real things into.

Three things are deliberately **not** units:

- **Third-party is a role, not a type.** The same upstream repo is vendor material in one workspace
  and the subject of work in another. The role belongs to the relation, not to the repo.
- **A README, a `CLAUDE.md` or a wiki is an attribute.** It says the thing documents itself, and
  where. Graded rather than binary; see *How far a project describes itself*.
- **A toolchain is declared in the repo.** Record where the declaration is, never what it says.

## Relations

Three relations, and conflating them is what makes a catalogue lie.

| Relation     | Shape | Where it comes from                                            |
|--------------|-------|----------------------------------------------------------------|
| `contains`   | tree  | The filesystem. Always derivable, and uninteresting on its own |
| `member-of`  | graph | Hand-written. The highest-value fact in the catalogue          |
| `depends-on` | graph | Crosses projects, and **never** implies membership             |

**Aggregation is a container holding children it does not own.** A folder can supply `contains`
without `member-of`, and that is normal. A library a project depends on is a member of its own
project and a dependency of the other one, never a member of it. **A catalogue that reads containment
as membership invents projects that do not exist.**

**Lineage is written down or it does not exist.** A fork's origin and a library's parent project are
not derivable — those are genuinely different strings, and a checkout often records no upstream
remote at all. `forked-from` between repos, `extracted-from` between projects, and only where it
matters.

## Direction of description

A project can document **what it exists for, what hosts it, what it came out of**. Those are
singular, stable, and part of its own identity.

A project cannot document **who uses it**. That list grows, and it is incomplete the moment it is
written. Only a catalogue can hold it, and "what else consumes this, is it safe to move" is one of
the questions a catalogue exists to answer.

The same rule recognises members cheaply: **a thing whose own docs say "read the parent first" is a
member of the parent, not a project in its own right.**

---

## Workspace

A grouping of resources someone decides to work on as a unit. It stays subjective on purpose, because
the useful cases vary too much to pin down without excluding one: a local tree of checkouts, a CI
pipeline, a cloud workspace an agent is given. Each is set up for a purpose and thrown away
afterwards.

- **Started, not detected.** A user or an agent decides to open one. It is not a property you
  discover on disk, and every attempt to derive it — from an agent instruction file, an IDE project,
  build coupling — was wrong on real trees.
- **Children are not validated.** A member may be tightly coupled to its siblings, or checked out
  only to read alongside them. Both are legitimate, and no rule tells them apart.
- **Recognition is needed only when cleaning up an existing tree**, and a heuristic is enough there,
  because a person confirms the answer.

## How far a project describes itself

This decides whether the catalogue links or describes, and you can see every rung on disk.

| Rung              | What is there                                        | What the catalogue does                        |
|-------------------|------------------------------------------------------|------------------------------------------------|
| **Nothing**       | No README, no docs                                   | The catalogue is the only possible description |
| **Prose**         | A README, a `CLAUDE.md`, an `AGENTS.md`              | Link, and describe what the prose leaves out   |
| **A wiki**        | A structured knowledge base with its own conventions | Link. The project owns its own vocabulary      |
| **Can hand over** | A followable document that stands a workspace up     | Link and delegate                              |

**Grade the top rung on the handover document, not on process notes in general.** A project can
document how work is done and still leave an agent with no way to start, and that failure stays
invisible until someone tries. The rung goes to a document that names its prerequisites, proceeds in
verifiable steps, and says which decisions to ask about rather than guess.

**A defined process does not follow the directory tree.** A project defines its own and inherits
nothing from its parent, so a nested project can sit two rungs above the one containing it.

## The marker file

A tree can declare its own place with an `atlas.md`, so a catalogue does not have to infer it. **Read
`marker.md` beside this file** before writing one, or before walking a tree that has them.

---

## Workflows

**Cleaning up an existing tree.** Walk it and ask of each directory: is this a unit, and which one.
Three answers do most of the work.

- **Not a unit at all.** A container that accumulated, whose children are several unrelated things.
- **An aggregation.** Real, but it does not own everything under it.
- **A member of something else.** Its own docs defer to a parent.

A catalogue that cannot say "this folder is not a thing" invents a project that does not exist.
Record answers as you go, because the tree is large and you will not recover the reasoning later.

**Placing a new thing.** Which project is it a member of, which catalogue owns that region, and does
it need a workspace of its own or does it join one.

**Bootstrapping a workspace.** Given a task and a project's catalogue, list what to clone, install and
obtain, and nothing more. The target may be a new machine, a clean cloud instance, or a person
joining. This test fails loudly: the catalogue and the project's own instructions either produce a
working environment or they do not.
