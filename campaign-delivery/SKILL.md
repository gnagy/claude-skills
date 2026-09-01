---
name: campaign-delivery
description: Use this skill when a project runs campaign-based delivery — capturing an incoming request, refining it, enlisting material into a campaign, opening or closing a campaign, breaking one into stories, keeping the feature notes current, or wiring a project up to work this way. Triggers include "open a campaign", "enlist this", "close the campaign", "capture this request", "which feature does this land on", "what should we work on next", "update the roadmap". Covers the four delivery axes, the three coordinates every story carries, the scratch/living/frozen write disciplines, and the wiki-tracker boundary. This is one specific process rather than a general SDLC: it carries a fit test, and a project that fails it should not adopt it.
---

# Campaign delivery

How work gets from a request to a release, in a product that is changed continuously by people who
keep asking for things.

**This is one process, not the process.** It suits a specific situation and fails outside it. Before
applying any of it to a project that has not already adopted it, read [Does this
fit](#does-this-fit) — and if the project has adopted it, its own `meta/` note records the choices it
made, which this file does not know.

## Load alongside

| Skill             | Owns                                                                                                                |
|-------------------|---------------------------------------------------------------------------------------------------------------------|
| `wiki-docs`       | Storage mechanics — the toolbox, front matter, wikilink hazards, health checks. **Load it before touching a note.** |
| `grill-with-wiki` | The interview. Refining an ask, and running the adoption questions in [adoption.md](./adoption.md)                  |

This skill owns the process and nothing else. Where it needs a note written, `wiki-docs` says how.

## Does this fit

All five, or do not adopt it:

1. **A long-lived product**, changed over years — not a fixed-scope build with an end date.
2. **Requests arrive continuously from identifiable people** who expect an answer. A backlog the team
   wrote for itself is not this.
3. **A wiki somebody actually writes in.**
4. **A tracker that is not authoritative for intent.** It may hold stories and releases; if it also
   holds the roadmap and the plan of record, this process fights it and loses.
5. **Someone with the authority to close a campaign on judgment** — and to say out loud that a
   direction failed.

Any one of these disqualifies it:

- Work against a signed specification with fixed scope. Scope moving until close is the core mechanic
  here, and a contract that forbids it forbids the process.
- A library or internal tool with no stakeholder stream. There is nobody to refine for, and the
  feature inventory degenerates into a table of contents.
- Dates committed above the release. This deliberately produces no dates and no sizes on the intent
  axis; a project that must forecast a year out will fill the gap with fiction.

**The parts are load-bearing on each other**, so half-adopting is worse than not adopting. Campaigns
without the write disciplines give you a frozen record nobody can read. Refinement without the enlist
gate gives you a tracker full of empty containers. If only part of it fits, take the vocabulary and
none of the rules.

## Who owns what

| Subject                                                                                                                                                      | Authority                                  |
|--------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------|
| **The model** — the axes, the coordinates, the write disciplines, the gate, what closing means                                                               | This skill and the files beside it         |
| **The choices** — directory names, field names, whether a coarse structural level exists, the tracker and its field mapping, appetite and sponsor vocabulary | The project's `meta/` note and `CLAUDE.md` |

The eight choices a project makes are listed in [adoption.md](./adoption.md). **That list is closed.**
A project that needs to change something outside it has either found a gap in the model — say so, and
it changes here for every project — or it is forking, and it stops claiming to run this process.

**A `meta/` note restating the model is not a second authority, it is a stale copy.** It was written
against some earlier version of this file, nothing updates it, and the next agent reads it as local
law. Delete it and leave a pointer. Changing a *choice* is the opposite: that is the project's, and
yours to ask about rather than edit.

**Refer, never quote.** Point at this skill by name and section heading, so the reference survives an
edit inside that section.

## The model

### Feature and campaign are different kinds of noun

|          | **Feature**                  | **Campaign**                            |
|----------|------------------------------|-----------------------------------------|
| Is       | A part of the product        | A bounded change to the product         |
| Lifetime | Permanent, until removed     | Scope, deadline and budget, then closed |
| Named by | Users, where they can see it | Us                                      |
| Answers  | *Where in the product?*      | *Which push produced this?*             |
| Example  | The map                      | The 2026 map improvements               |

Neither contains the other, and the proof is that the relation runs **many-to-many in both
directions**: a campaign spans several features, a feature is changed by many campaigns over its life.

The consequence that gets broken most often: **a feature is never planned, scheduled, sized or
completed.** Campaigns are. A feature is the *place* work lands.

### The four axes

| Axis          | Objects                                 | Nature                                    | Wrong if                        |
|---------------|-----------------------------------------|-------------------------------------------|---------------------------------|
| **Structure** | Feature, and optionally a coarser level | Permanent                                 | It misdescribes the product     |
| **Flow**      | Campaign                                | Bounded — it ends, though its scope moves | It misrecords what we committed |
| **Time**      | Release                                 | An event, not a container                 | It misrecords what shipped      |
| **Intent**    | Theme, roadmap item                     | Aspirational, freely revisable            | —                               |

The empty cell is deliberate. **Intent is the only axis where being wrong is free**, because it never
claimed to record anything. Everything else records something that happened, which is why nothing
load-bearing may hang off the roadmap.

**None of the four is a level of another.** The recurring planning failure is forcing them into one
hierarchy — a chain from theme down to story that nothing in reality obeys.

### The three coordinates

Every unit of change — a story, a ticket, a commit — carries three:

| Coordinate | Axis     | Answers                  | Cardinality |
|------------|----------|--------------------------|-------------|
| **Why**    | Campaign | Which push produced this | Zero or one |
| **Where**  | Feature  | What part of the product | One or more |
| **When**   | Release  | What deploy carried it   | Exactly one |

**Why is the optional one, and its absence carries meaning.** Work below the size of a campaign — a
routine fix, a data correction — carries none, and *no campaign* means *routine*. That makes the
run-the-business share of throughput a query rather than an annual guess. See
[tracker.md](./tracker.md).

### The three write disciplines

Three kinds of note, differing by **what you are allowed to do to the file** — which is what an agent
most needs to know before touching one. Default directory names shown; this project's names are in its
`meta/` note.

| Role    | Default      | Discipline                          | Authoritative?               |
|---------|--------------|-------------------------------------|------------------------------|
| Scratch | `intake/`    | Rewritten freely, never drains      | **No.** Thinking in progress |
| Living  | `product/`   | Rewritten on change                 | **Yes** — what exists now    |
| Frozen  | `campaigns/` | Appends while open, frozen at close | **Yes** — what happened      |

**Nothing in the scratch area may be cited.** Everything there might not be true. That line is what
stops unshaped material being quoted as fact, and it is stronger than permanent-versus-transient.

Unversioned throughout: no `-v2.md`, no archive copies. Git owns history.

### The tense test

A campaign note and a feature note describe the same work in two tenses, and the tense decides which
file a sentence goes in:

| Sentence                                                          | Belongs to                |
|-------------------------------------------------------------------|---------------------------|
| *We chose one primary per site, because the feed only names one.* | The campaign — frozen     |
| *A site has one primary.*                                         | The product view — living |

The same fact living on both sides is correct: the campaign's copy is dated and says why, the living
copy is current and says what. **A campaign note written in the present tense about the product is
misfiled**, and the misfiling matters because the campaign will freeze and take the description with
it.

## Where to go next

| The moment                                                                          | Read                           |
|-------------------------------------------------------------------------------------|--------------------------------|
| A request just arrived, or material needs refining, or something is ready to enlist | [intake.md](./intake.md)       |
| Naming, writing or correcting the description of the product                        | [features.md](./features.md)   |
| Opening, running, sizing, sequencing or closing a campaign                          | [campaigns.md](./campaigns.md) |
| Anything touching the tracker — stories, labels, releases, queries                  | [tracker.md](./tracker.md)     |
| Setting a project up, or retrofitting one that copied the model into `meta/`        | [adoption.md](./adoption.md)   |

## The five that get broken

Load-bearing, and each one fails silently:

1. **Nothing enters the tracker before it is enlisted into a campaign** — [tracker.md](./tracker.md).
2. **Nothing outside the scratch area may cite a campaign before its first enlistment**, because
   until then there is no stable name — [campaigns.md](./campaigns.md).
3. **A captured request is never edited.** Changes go beside it, dated — [intake.md](./intake.md).
4. **Durable statements are written out to the living notes in the same change that makes them
   true**, never in a lump at the end, never promoted — [features.md](./features.md).
5. **A campaign closes on judgment, not on completed scope**, and not until its write-out list checks
   out — [campaigns.md](./campaigns.md).
