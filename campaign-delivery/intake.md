# Intake: from an arriving request to an enlisted one

The scratch area is a **standing staging area, not a queue that drains**. Requests arrive, expand
earlier requests, get re-read against what has since been learned, and accumulate. Refinement runs
over it continuously and never terminates. Nothing forces it onto a calendar.

**Material arrives whether or not a campaign for it exists.** An idea about an open campaign lands
here like any other, is refined like any other, and is enlisted into that campaign only by a separate
decision. An open campaign does not swallow everything adjacent to it.

Default directory `intake/`; this project's name is in its `meta/` note.

## Capture

*This section stands alone. It assumes nothing about campaigns and applies to any project that takes
requests from people.*

Transcribe a request **verbatim** — its original wording, its original nesting, dated, with a stable
id per item.

**Never edit a captured ask.** All movement happens downstream, so the difference between what was
asked and what was built stays inspectable a year later. A change to something already captured goes
**beside it as a new dated entry**, never into the original.

Captures are frozen on arrival and kept forever, alongside the living notes rather than in the scratch
area — they are evidence, not working material. Default `product/requests/`.

Three things sit close together here and behave completely differently:

|                        | Stable?               | Leaves?                           |
|------------------------|-----------------------|-----------------------------------|
| A **roadmap item**     | Permanently revisable | Never — it is cited, not promoted |
| **Scratch material**   | Unstable              | Always — enlisted, or dropped     |
| A **captured request** | Frozen on arrival     | Never                             |

## Assign, do not invent, the feature

Every item gets a feature from the product view, **which already exists** — see
[features.md](./features.md). This is **assignment, not grouping**: the map is there, users named it,
and the only question is which part of the product an ask lands on. The failure mode is
misassignment, not misgrouping.

Assign one to rejected items too. Knowing that three declined asks all landed on the same feature is
exactly what is wanted when someone asks again.

Occasionally an ask genuinely names something the product does not have, and the inventory gains an
entry — rarely, or the list is being used to group work.

## The data feasibility check

*This section stands alone. It applies to any request for a number, a report or a chart, in any
project.*

A request assumes its data exists — reliably, and for the whole period being asked about. Check the
assumption **before writing anything that depends on it**, not when the query returns nonsense.

| Check           | Question                                                                        |
|-----------------|---------------------------------------------------------------------------------|
| **Existence**   | Is this captured at all, or inferred from something adjacent?                   |
| **Consistency** | Captured the same way by everyone, every time — or does it depend on diligence? |
| **Coverage**    | Does it exist for the whole period, including before the field was added?       |
| **Granularity** | Recorded at the level needed — per task? per site? per minute?                  |

Check against the **actual tables**, not against what the data model says should be there.

**Every "no" is work that precedes the visible change**, and often its own campaign. A gap found here
is cheap; the same gap found after something ships is a credibility loss with the person who asked.

This is one instance of a general move: **buy certainty with a spike** — a timeboxed question
producing an answer rather than software. Data feasibility is the most common question worth buying,
not the only one.

## Bundling and churn

Material clusters as it is refined, and the clusters move. Split them when their threads stop sharing
a source, a counterparty or a definition of done; merge them when they turn out to be one thing; pull
them apart again when a feasibility answer changes the picture.

All of it happens in the scratch area, where nothing is stable and nothing may be cited.

## The gate

**Nothing is enlisted without three things:**

- **A sponsor** who wants it and will answer for it.
- **Rough scope**, including what is explicitly out.
- **A one-sentence acceptance** — what makes the person who wanted it say it is done.

The same gate applies to the first item enlisted into a new campaign and to the fiftieth into an open
one. What differs is only that the first fixes the campaign's name — see
[campaigns.md](./campaigns.md).

Where the ask came from outside, **the requester is the sponsor and the priority is theirs**, never
inferred. Where it did not, a sponsor has to be found, and failing to find one is itself the answer.

**Acceptance is real usage.** For anything reporting on real data, acceptance is one real period of
real data, checked by the requester against their own records — never a synthetic screenshot.
Requests of that kind usually exist because numbers were already untrustworthy, so *the chart renders*
is not the bar.

## Answering the requester

Where someone asked, they are owed a reply: what is being done, what is not and why, and what went to
the roadmap instead.

- Items closed as out of scope are closed **visibly, with a reason**.
- Questions go back as **one batched list on a timebox**, not a trickle over a week.
- **"Not now" is a real answer.** An ask worth doing but not soon becomes a roadmap item and stops
  being an open promise — more honest than silence, more useful than a refusal.

## The roadmap

The roadmap lives in the scratch area, with everything else that might not be true. It is the one note
there that is permanent rather than transient, and the one that never graduates.

Stakeholders want to see where the product is going a year out; reality never matches the plan. Both
are permanent conditions, so the roadmap is **a statement of intent, not a queue of future work**.

Two levels: a **theme** is a stable bucket answering *what kind of investment is this*; an **item** is
one direction we may invest in. Both are freely revisable, because neither claims to record anything.

### Items are never promoted into campaigns

The failure mode is treating the roadmap as the campaign backlog. Then every campaign that does not
match an item makes the roadmap wrong, keeping it true becomes a chore, and within a year it is
abandoned or quietly re-dated into fiction.

The join is **citational**: a campaign forms in refinement and *cites* whichever items it serves.
Many-to-many, and either side may be empty — a campaign serving no item is normal, and most technical
campaigns do exactly that.

### Items have no state

A dream does not progress. It has not been specified, nobody is working on it, and giving it a
readiness ladder manufactures the appearance of a pipeline where there is only a wish — the same
pretence as an empty container in a tracker.

An item exists until it is dropped. Its only signal is **how many campaigns cite it**. Readiness
belongs to campaigns, where things actually move.

### What a progress report says

Not a percentage. For each item: the campaigns opened against it, and how many have released.

> *Item 7 — four campaigns opened against it, three released, one still open.*
> *Item 12 — no campaigns in fourteen months.*

**The second line is the valuable one.** It says a direction is not happening, out loud, at a point
where someone can fund it or drop it. A roadmap that can produce that line is worth maintaining; one
that only gets re-dated is not.

### Working rules

1. **Every item carries a short description**, written at capture: what it concretely covers, what
   makes it non-obvious, what it is blocked on. A register of one-line titles has to be re-derived
   every planning round; one with descriptions can be read cold.
2. **Dropped items stay, with the reason.** A register without its rejections repeats them.
3. **No dates, no sizes, no priorities.** Those belong to campaigns, which are the things that get
   committed to.
