---
name: atlas
description: Use this skill when orienting across projects, workspaces, repos and checkouts — what a directory is, which project it is a member of, what depends on it, or what has to be cloned and installed before a task can run. Also when a tree contains an `atlas.md` marker file, or when asked to walk a tree and work out what is in it. Covers the catalogue model, the marker format, and the cleanup and bootstrap workflows. Runs on `/atlas` as well.
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

**Reading this is free; acting on it is not.** Answer from it — what a directory is, what it belongs
to, what depends on it — whenever the question comes up. **Writing is asked for, never volunteered:**
an `atlas.md` is written when someone asks for one, and a walk of a tree is started by a person.
Noticing that a tree would be easier to work in with markers in it is worth one sentence to whoever
owns the tree; it is not a reason to start writing them. The catalogue records what somebody decided,
so a marker nobody asked for records a decision nobody made.

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

Four relations, and conflating them is what makes a catalogue lie.

| Relation      | Shape | Where it comes from                                               |
|---------------|-------|-------------------------------------------------------------------|
| `contains`    | tree  | The filesystem. Always derivable, and uninteresting on its own    |
| `member-of`   | graph | Hand-written. The highest-value fact in the catalogue             |
| `depends-on`  | graph | Crosses projects, and **never** implies membership                |
| `checkout-of` | graph | A checkout to the repos it places. Hand-written, and one-way only |

**Aggregation is a container holding children it does not own.** A folder can supply `contains`
without `member-of`, and that is normal. A library a project depends on is a member of its own
project and a dependency of the other one, never a member of it. **A catalogue that reads containment
as membership invents projects that do not exist.**

The same rule recognises members cheaply: **a thing whose own docs say "read the parent first" is a
member of the parent, not a project in its own right.**

**`checkout-of` joins the two units the other three cannot.** A checkout is not `member-of` its repo
and does not `depend-on` it — one is identity and the other placement, which is the whole reason they
are separate units, so reusing either relation destroys the distinction it was made for. The name is
the plain one on purpose: `clone-of` is git's word for one operation, and *clone* already means a
mirror or a backup copy in the same conversation.

**It is a list, and its order says which identity is the primary one.** A working copy can carry
several remotes and track a branch belonging to one that is not its `origin`, so **`origin` is not
the authoritative answer to what a directory is**. Which repo a checkout principally is, is a
judgement someone makes; git does not hold it, and recording it is exactly the sort of thing a
catalogue exists for.

**It runs one way, and the reason is that its two ends live in different catalogues.** A repo is an
identity, so its record belongs somewhere shared — the repo itself, or a catalogue read by the people
who own it. A checkout is one person's directory on one machine, so its record belongs to whatever
catalogue governs the workspace it was checked out into. **A repo record naming a checkout is a
pointer out of the shared document into somebody's private one**, which is the direction the model
forbids everywhere else. So a checkout names its repos, nothing points back, and a repo record is a
dead end in the graph rather than a defect to fix.

## Workspace

A grouping of resources someone decides to work on as a unit. It stays subjective on purpose, because
the useful cases vary too much to pin down without excluding one: a local tree of checkouts, a CI
pipeline, a cloud workspace an agent is given.

- **Started, not detected.** A user or an agent decides to open one. It is not a property you
  discover on disk, and no derivation from the files is reliable — the signals below are heuristics
  a person confirms.
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

This decides whether the catalogue links or describes. Every rung is visible on disk, which is also
where the ladder stops being reliable — see below.

| Rung              | What is there                                                          | What the catalogue does                                                            |
|-------------------|------------------------------------------------------------------------|------------------------------------------------------------------------------------|
| **Nothing**       | No README, no docs                                                     | The catalogue is the only possible description                                     |
| **Prose**         | A README, a `CLAUDE.md`, an `AGENTS.md`                                | Link, and describe what the prose leaves out                                       |
| **Organised**     | A knowledge base with its own conventions, often a wiki. No `atlas.md` | Link. Worth reading, but the catalogue still has to work out what the project *is* |
| **Declared**      | An `atlas.md` at the root, usually delegating into the docs above      | Record the pointer. The project says what it is, in these terms                    |
| **Can hand over** | A followable document that stands a workspace up                       | Link and delegate                                                                  |

**Every rung measures presence. None of them measures truth.** A README describing a *different*
repository grades at *Prose*, passes every test in the table, and leaves its reader confidently
wrong — which is worse than *Nothing*, since nothing at least misleads nobody. No structural check
finds this, because what the ladder grades is whether a document is there.

**One cheap check catches most of it: does the document name the thing it sits in?** A README titled
after another repository, a `CLAUDE.md` describing a different application, a setup guide written for
a sibling — each visible in a first paragraph and invisible from a listing. That is as far as this
goes: auditing documentation against the code it describes is not a walk's job, and does not finish.

**Where it fails, drop the rung and say why.** Prose about something else gives the catalogue nothing
to link to, so grade it *Nothing* and describe the thing yourself — and record that the document is
wrong. That is a fact nothing on disk carries, and the next reader will otherwise trust it exactly as
this one did.

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

## A catalogue extends these rules

**The model's rules always apply. A catalogue may add to them, for its own tree.** The additions are
the calls somebody has already made there, written down so nobody makes them twice: *every `data/`
beside a checkout here is gitignored working material*, *anything under this root that is not in the
module list is held, not owned*, *a directory named `*-test` here is a spike workspace*.

**Adding none is the ordinary case.** Most trees are described by the units and rules here and want
nothing local at all, and a catalogue with no additions is not an unfinished one. Write a rule
because the same answer keeps being given, never to have a rules section.

**A local rule is a person's answer, recorded.** That is why it does not contradict *purpose is not
on disk* — the purpose was supplied by somebody, once, and the rule is where it went. It is also the
only legitimate way a walk arrives at `aggregation`: not by inferring the verdict, but by applying
one already given.

**Apply the model's rules first, then the local ones.** The model's answers are the ones another
person's agent will also produce; a local rule refines what is left over. Where the two disagree,
either the local rule is wrong or the model is missing something — say which you think it is and
stop, rather than silently preferring one.

**Local rules classify. They never change the vocabulary.** A catalogue may say which of the values a
directory takes, and may not invent a value, redefine `workspace`, or add a relation. A catalogue
that does has stopped being an instance of this model, and its entries stop being readable by anyone
else's agent — which is the whole reason the model is separate from it.

**A rule that keeps reappearing belongs here instead.** One catalogue writing a rule is local; two
catalogues writing the same one is a gap in the model, and the fix is to lift it into this file
rather than to keep it copied. The same goes the other way for the vocabulary: a catalogue that needs
a value this list does not have should say so and leave the directory `unknown` — the value arrives
by being added here, on the evidence of somewhere it was actually needed, never by an instance
inventing it locally.

**Where the additions live is the catalogue's business** — with its own conventions, wherever it
keeps them. A walker that has stopped at an `authority` already knows where to look.

## Writing a catalogue

**Two shapes go in it.** An **entry** describes one unit that lives somewhere else. A **note** is
true of the walked tree without being a unit in it — why a remote here is not a backup, why five
directories are all working data nobody can commit. A note is how a catalogue states something once
that a marker would have to repeat in every directory it applies to.

**One unit, one entry — so a repo and its checkout are two entries with two names.** Name an
identity after its owner and remote, a placement after the directory. A catalogue giving them one
name has conflated identity and placement in the first place a reader looks.

**Every entry records when it was last looked at.** Its subject is elsewhere, so it goes stale with
nothing in the file changing. That date is what tells a reader how much to trust the entry, and it is
the field a marker has no need of.

**An entry earns itself, the same way a marker does.** Where a parent entry already names a child and
the name says what it is, the child needs none. Where several siblings share the only fact worth
recording, record it once on the thing that holds them.

**A thin entry is not a missing one, though.** An entry saying nothing the repository does not
already say is still worth writing: *somebody looked and found nothing to add* and *nobody has
looked* are different facts, and only one of them needs following up. Losing that distinction is how
a gap in the catalogue becomes invisible.

**So an entry sits in one of four states, and all four are useful:**

| State         | What it says                                   |
|---------------|------------------------------------------------|
| No entry      | Nobody has looked                              |
| `unknown`     | Somebody looked and could not tell             |
| A unit type   | Somebody decided what it is                    |
| `aggregation` | Somebody decided it holds what it does not own |

**The last row belongs to a person.** Saying a container does not own its children needs to know what
the directory is *for*, which is not on disk. Where the answer came from a listing rather than from
somebody who knows, `unknown` is the honest record, and a catalogue is better for holding a hundred
of those than one confident mistake.

**A directory that is not a unit at all gets no entry.** There is no type for it, because an entry
for a non-thing is a unit record with no unit in it, filed where nobody looks. It goes in the entry
for whatever contains it, with the reason — which is where a reader meets it, and which the coverage
check below already relies on.

**Never write a word that is only true from where you are standing.** *Ours*, *the client's*,
*third-party*, *upstream of us* — each is measured from the reader's position, and a catalogue is
written to be read from somewhere else, where "ours" names the wrong party. **Name the owner
instead.** This is the failure mode no schema catches: it is a fact about words rather than about
structure, so an entry full of role words passes every structural check there is. Naming owners also
records what the role words hid — that two namespaces are one organisation, say — which nothing
derives.

**Identity records and placement records belong to different catalogues.** A repo's identity is true
from anywhere, so its record belongs somewhere shared — in the repo itself, or in a catalogue read by
the people who own it. Where it is checked out is true of one directory on one machine, so that
record belongs to whichever catalogue governs that workspace. **They are two documents with two
owners**, which is why a repo record carries no path, no local branch, no directory name and no link
down into a checkout: each of those would put one person's disk into the document everybody reads.
A role word in its prose is the same mistake one layer down and out of reach of any schema.

**Where a catalogue exists, keep the markers thin.** A marker has to stand alone for whoever lands on
it with no catalogue, so anything shared between markers is repeated in each of them in full. A
catalogue is under no such constraint and holds it once. Put in a marker what that directory's reader
needs, and let the catalogue carry the rest.

**A delegation is an entry too.** Where a region catalogues itself, record that it does, plus the
facts its own catalogue cannot hold — which for a shared one is usually all of placement.

---

## Managing a catalogue

**A tree is walked once and a catalogue is kept for as long as the work lasts**, so this is the job
that recurs. `walking.md` beside this file covers the other one.

### What a catalogue is made of

| Element   | What it is                                                        | Carries                                                         |
|-----------|-------------------------------------------------------------------|-----------------------------------------------------------------|
| **entry** | One unit, described from outside it                               | Its type, what locates or identifies it, and when it was seen   |
| **note**  | Something true of the region that is not a unit in it             | No location — it is about the tree, not about a place in it     |
| **index** | What the catalogue holds, and groupings its structure cannot make | Nothing of its own. Every line in it is answerable from entries |

**The entry types are the marker's `atlas:` values plus `repo`.** A marker sits in a directory, so it
can only ever describe a placement. A catalogue names identities too, and that one extra type is the
whole difference between what a tree can say about itself and what a catalogue says about it.

**An index is derived, so never let it hold the only copy of anything.** Its groupings — by owner, by
what is irreplaceable, by what is not the engagement's — are the useful part and the part that ages,
and each one is a claim the entries themselves should be able to settle. An index is also the first
file to end up holding two versions of a section, because it is the file every change touches.

### Working with one

**Reading is the common case, and it is free.** Answer from the catalogue whenever the question comes
up. Everything below is asked for.

- **Ask it.** What is this directory, what does it belong to, what depends on it, what has to be
  cloned and installed before this task can run.
- **Add**, when something lands that the catalogue does not have. One unit, one entry.
- **Re-observe.** An entry is a record of a look, and the date says which look. Re-observing is the
  operation that keeps it worth trusting, and the one nothing prompts you to run.
- **Change what a thing is.** A type is a claim and claims get corrected — a container that turns out
  to have a purpose was a workspace all along, not an aggregation.
- **Resolve an `unknown`.** These are the catalogue's own to-do list, and the only item on it that
  needs a person rather than another look at the disk. Ask whoever put the directory there what it
  was for; that answer is not recoverable any other way, and it stops being recoverable at all once
  they have forgotten.
- **Write down a rule you have applied twice.** The second time the same answer is given about the
  same kind of directory, it is a local rule and belongs with the catalogue's conventions, where the
  next walk can apply it instead of asking again.
- **Retire.** Placement records die with the directory; identity records outlive it. Deleting a
  checkout does not delete the repository it was a placement of.
- **Bootstrap.** Given a task and the catalogue, list what to clone, install and obtain, and nothing
  more. This one fails loudly: the catalogue and the project's own instructions either produce a
  working environment or they do not.

**The person is where purpose comes from.** Every question a catalogue exists to answer that a tool
cannot — what a directory is *for*, which siblings belong together, which identity a working copy
principally is, what must never be pushed where — is somebody's answer, not a reading. An agent
maintaining a catalogue alone will keep every derivable field current and quietly leave those blank.

### Checking it rather than asserting it

Coverage is the claim a catalogue most easily makes falsely, and three checks settle it:

- **Every entry's location is distinct.** Two entries for one directory means identity and placement
  got conflated somewhere.
- **Everything in the region either has an entry, or is named in a parent's entry with a reason.**
  Deliberate omissions are recorded, so an absence is a decision rather than a gap. **This is also
  where a directory that is not a unit goes** — named by its parent as not a thing, and why. A
  catalogue that cannot say that invents projects that do not exist; one that gives it an entry has
  made a unit out of the very thing it meant to deny.
- **Relations reciprocate where they should, and only there.** `checkout-of` in particular: carrying
  a remote to a repository is not being a placement of it.

**Record decisions, and measure separately.** How many branches have no upstream, how large a
directory is, how many entries exist — all of it is true on the day and stale after. The entry holds
what was decided; the count belongs wherever the catalogue keeps its chronology, with the date on it.

### When the tree and the catalogue disagree

- **On what exists, the tree wins.** Directories move, get renamed and get deleted, and an entry is a
  record of an observation. Re-observe and correct the entry.
- **On what things mean, the catalogue wins.** Membership, dependency, which identity is primary,
  what must not be pushed where — nothing in the tree records any of it, so a tree that says otherwise
  is not disagreeing, it is silent.
- **Markers are maintained from the catalogue, not the other way round.** The catalogue is where the
  reasoning lives; a marker is a deployment of the part of it that belongs in one directory. When a
  sweep changes the catalogue, the markers are what gets regenerated.

## The marker file

A tree can declare its own place with an `atlas.md`, so a catalogue does not have to infer it. **Read
`marker.md` beside this file** before writing one, or before walking a tree that has them.

## Walking a tree

Working out what is in a tree nobody has catalogued. **Read `walking.md` beside this file** — the
verdicts and their traps, how markers change the descent, why a walk cannot verify itself, and the
derivations that look complete and are wrong. **It is started by a person, never volunteered.**
