---
name: atlas
description: Orientation across projects, workspaces, repos and checkouts — what a directory is, what it is a member of, and what depends on it. The catalogue model, the `atlas.md` marker file, and the cleanup and bootstrap workflows. Runs on `/atlas`.
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

## Units

| Unit          | What it is                                                                                            |
|---------------|-------------------------------------------------------------------------------------------------------|
| **project**   | A named body of work. Nestable. May span workspaces, machines and roots, and may have no files at all |
| **workspace** | A place work happens. See below                                                                       |
| **repo**      | An identity, usually a remote. Independent of where it is checked out                                 |
| **checkout**  | A repo in a directory on one machine. The thing you move, delete, back up                             |
| **material**  | Files in no repo, described by nothing else                                                           |

**`repo` and `checkout` are separate on purpose.** One is identity and one is placement, and the same
repo can be checked out twice on one machine.

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

The same rule recognises members cheaply: **a thing whose own docs say "read the parent first" is a
member of the parent, not a project in its own right.**

## Workspace

A grouping of resources someone decides to work on as a unit. It stays subjective on purpose, because
the useful cases vary too much to pin down without excluding one: a local tree of checkouts, a CI
pipeline, a cloud workspace an agent is given.

- **Started, not detected.** A user or an agent decides to open one. It is not a property you
  discover on disk, and no derivation from the files has ever been right on a real tree.
- **Children are not validated.** A member may be tightly coupled to its siblings, or checked out
  only to read alongside them. Both are legitimate, and no rule tells them apart.

**Recognition is needed only when cleaning up an existing tree**, and a heuristic is enough there
because a person confirms the answer. Three signals, and they answer different questions:

- **IDE or agent configuration sitting at a level** — `.idea/`, `.vscode/`, an `AGENTS.md` — says
  *where* a workspace is. It is the most reliable signal and it is silent about membership.
- **A module list inside that configuration** names *some* members. Evidence, not truth: on one real
  tree it named seven of twenty siblings and left out several that belonged.
- **Build coupling**, where checkouts bind to each other by relative path, is the strongest
  membership signal and still incomplete — a checkout kept only to read alongside the others has
  none.

## How far a project describes itself

This decides whether the catalogue links or describes, and you can see every rung on disk.

| Rung              | What is there                                                          | What the catalogue does                                                            |
|-------------------|------------------------------------------------------------------------|------------------------------------------------------------------------------------|
| **Nothing**       | No README, no docs                                                     | The catalogue is the only possible description                                     |
| **Prose**         | A README, a `CLAUDE.md`, an `AGENTS.md`                                | Link, and describe what the prose leaves out                                       |
| **Organised**     | A knowledge base with its own conventions, often a wiki. No `atlas.md` | Link. Worth reading, but the catalogue still has to work out what the project *is* |
| **Declared**      | An `atlas.md` at the root, usually delegating into the docs above      | Record the pointer. The project says what it is, in these terms                    |
| **Can hand over** | A followable document that stands a workspace up                       | Link and delegate                                                                  |

**The rungs are what a project gives you, not what it owes you.** No project needs a wiki, and plenty
of good ones stop at prose.

**The step that changes the catalogue's job is *Declared*.** A wiki with no `atlas.md` is a useful
source and nothing more: an agent reads it, and whoever walks the tree still has to decide what the
project is and what belongs to it. A marker at the root says that outright, in the model's own terms,
and points at the wiki for the rest — so the catalogue records a pointer instead of a description.
The two are complementary rather than competing, which is why a project usually ends up with both.

**Grade the top rung on the handover document, not on process notes in general.** A project can
document how work is done and still leave an agent with no way to start, and that failure stays
invisible until someone tries.

**A defined process does not follow the directory tree.** A project defines its own and inherits
nothing from its parent, so a nested project can sit two rungs above the one containing it.

---

## Derive, don't transcribe

Never record what a tool can read from the files: remotes, commit dates, sizes, dependency lists. Two
exceptions:

- **A thing with no files here.** An idea, an archived project, something on a share you cannot read.
  Nothing exists to derive from, so you write it by hand.
- **Facts needed before the files exist.** An agent deciding what to clone wants the cost before
  cloning, and a remote does not tell you the size. Record those with the date you measured them.

## The catalogue points, the project instructs

Setup and process live **in the project**, in documents an agent can follow. The catalogue records
that the project exists, where it is, and that it documents itself.

A catalogue that explains how to build something has taken on a copy of a build file, and that copy
drifts within a week. The clone commands, the branch a sibling has to sit on, the toolchain: all of it
belongs to the project. The catalogue says which project to go and read.

Where a project documents nothing, the catalogue describes it. That is the one exception, and it is
why `material` is a unit.

## The marker file

A tree can declare its own place with an `atlas.md`, so a catalogue does not have to infer it. **Read
`marker.md` beside this file** before writing one, or before walking a tree that has them.

---

## Workflows

**Cleaning up an existing tree.** Walk it and ask of each directory: is this a unit, and which one.
Three answers do most of the work.

- **A member of something else.** Its own docs defer to a parent.
- **An aggregation.** Real, but it does not own everything under it.
- **Not a unit at all.** A container that accumulated, whose children are several unrelated things.

**Check the rung before reaching for the third.** A directory with no README and no docs sits at the
*Nothing* rung, where the catalogue is the only possible description — so it is a unit nobody has
described, not a directory that is not a thing. The two look identical from a listing, and the
not-a-unit verdict is the satisfying one, which is why it gets reached for wrongly.

A catalogue that cannot say "this folder is not a thing" invents a project that does not exist.
Record answers as you go, because the tree is large and you will not recover the reasoning later.

**Bootstrapping a workspace.** Given a task and a project's catalogue, list what to clone, install and
obtain, and nothing more. This test fails loudly: the catalogue and the project's own instructions
either produce a working environment or they do not.
