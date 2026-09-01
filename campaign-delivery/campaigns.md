# Campaigns: opening, running, closing

A **campaign** is a named, bounded body of work on the product: gather requirements under it, ship
something, get feedback, ship again, and eventually decide it is finished. It is the unit of handover,
design, budgeting and release, and the primary object of the whole process — the thing that moves
while the product view sits still.

**Bounded does not mean fixed.** A campaign's scope moves the whole time it is open. What makes it
bounded is that it *ends*.

Default directory `campaigns/`; this project's name is in its `meta/` note.

## Where campaigns come from

|              | **Product-originated**                | **Technically-originated**                |
|--------------|---------------------------------------|-------------------------------------------|
| Arrives from | Refinement — requests bundled up      | Engineering — cleanup, migration, upgrade |
| Sponsor      | The requester, or an internal sponsor | The dev lead or architect                 |
| Justified by | User value                            | Cost or risk avoided                      |
| Touches      | User-named features                   | Often only internal ones                  |

**The campaign does not belong to the product side.** Refinement is one supplier of campaigns, not
their owner — which is why a technical cleanup is a campaign in exactly the same sense, with the same
sponsor rule, the same appetite and the same dependencies.

**Every campaign has a sponsor**, whichever door it came through. Nobody asks for a dependency
upgrade, so without the rule an unsponsored guess travels the whole pipeline carrying invented
priority.

## Enlisting

Material is refined in the scratch area — see [intake.md](./intake.md) — and **enlisted** into a
campaign when there is enough information and enough interest to work on it under that name.

Enlisting is **one explicit decision, made over and over**. The first enlistment opens the campaign;
every later one grows it. There is no separate act of creation, and no batch handed over once and for
all.

| Enlisting **does**                                       | Enlisting does **not**                       |
|----------------------------------------------------------|----------------------------------------------|
| Fix the campaign's **identity** — a stable, citable name | Fix its scope. Scope moves until close       |
| Move material out of the scratch area into the campaign  | Commit to an unchanging appetite             |
| Make the work bookable against something                 | Mean the campaign will only proceed forwards |

**Nothing outside the scratch area may cite a campaign before its first enlistment.** Until then there
is no stable name to cite, and material still in refinement may be split, merged or dropped.

**A campaign existing does not capture everything about it.** Related ideas keep arriving and are
refined like anything else; enlisting each one is a separate decision. That is what stops an open
campaign quietly absorbing every adjacent thought.

## Three states, and nothing finer

| State     | Means                                                            |
|-----------|------------------------------------------------------------------|
| `forming` | In the scratch area. No stable name yet, and nothing may cite it |
| `open`    | Named. Work happening, scope moving, material still enlisting    |
| `closed`  | Done spending on it — delivered, abandoned, or handed on         |

A finer ladder would claim a progression that does not happen. Inside an open campaign the work loops:
requirements, a proof of concept, feedback, delivery, new ideas, delivery again.

The field carrying this is separate from the note's own `status` — choice 3 in
[adoption.md](./adoption.md).

## The appetite

Roughly how much is worth spending before the campaign is reconsidered. Set when it opens, revised
openly.

It is a **bet, not a budget line**. Tracking it to the hour turns it back into an estimate, and
uncertainty here is expected rather than a defect.

**Weigh it against the capacity that is genuinely free, not against headcount.** Most throughput is
not campaign work — routine fixes, migrations, partner requests, data corrections carry no campaign at
all, and that share has been measured at roughly two-thirds of issues created on a real board. An
appetite assuming the whole team is available promises a team that does not exist. Because routine
work is exactly the work with no *why* coordinate, the share is a query — see
[tracker.md](./tracker.md).

## Dependencies

Two kinds, separated by one test: **can a timeboxed spike substitute for it?**

|                       | **Informational**               | **Structural**                    |
|-----------------------|---------------------------------|-----------------------------------|
| B needs from A        | An answer or a decision         | Working software, users can reach |
| Satisfied when        | The answer exists — often early | The capability is available       |
| Blocks B's            | Design                          | Release, usually the build too    |
| If A is abandoned     | Buy the answer with a spike     | B is dead, or absorbs A's scope   |
| Spike can substitute? | **Yes**                         | **No**                            |

**A structural dependency points at product state, never at a campaign.** What B needs is *the map
supports date ranges*, satisfied by whichever release delivers it. A campaign ships in increments,
keeps taking on new material and ends by judgment — so "wait for campaign A" is both too coarse and
never cleanly satisfied. Wait for the capability.

Where feature flags are in use, a capability is available when it is **enabled**, not when it is
deployed.

Describe dependencies in prose in the campaign note. **Structural dependencies can cycle** — A cannot
ship without B, B cannot ship without A — and a cycle there is an unschedulable state that has to be
spotted rather than prevented.

### Foundational campaigns

A campaign many others structurally depend on. Worked ahead of them, and worth hunting for
deliberately, because two disguises are common:

- **The primary ask is often a cross-cutting technical decision.** *"The data must be copyable"*
  sounds like one item among several; it determines whether every chart is an image, a table or an
  export.
- **A repeated presentation detail is one shared mechanism.** The same small thing appearing across
  many asks is one decision, after which each of them is small.

## The campaign note

Usually **one note**: what has been enlisted so far, what is explicitly out, its appetite, its
sponsor, its dependencies in prose, the features it touches, where its material came from, and the
roadmap items it serves.

Not a folder of tickets, not a specification to be executed literally, and not a fixed brief. Material
keeps being enlisted while it is open, so what is being built is answered again each time.

**It accumulates while open** — enlistments appended and dated, never rewritten away — and freezes at
close. It gains subfolders only once it holds a cluster worth separating; those names are choice 6 in
[adoption.md](./adoption.md).

A frozen note is never edited but can be **annotated**: `superseded` when a later campaign reverses
its decision, `archived` when it stops being worth reading. The supersession is itself part of the
record.

## Breaking it down

Enlisted material becomes stories, each carrying the three coordinates. Sort them by kind of work,
because it drives both who picks one up and how it is estimated:

| Kind                      | Character                                | Estimation                  |
|---------------------------|------------------------------------------|-----------------------------|
| **Spike**                 | Produces an answer, not working software | Timebox, do not size        |
| **Data or model change**  | Schema, migration, capture rules         | Sizable                     |
| **Query or aggregation**  | Read-side computation over existing data | Sizable                     |
| **UI**                    | Screens, components, presentation        | Sizable                     |
| **Export or integration** | Leaving or entering the system           | Sizable, often foundational |

Two signals worth acting on: a campaign that is **entirely one kind** is often a single task wearing a
campaign's name, and one with **two unrelated data or model changes** has usually taken on material
that belongs elsewhere.

**Investigations disguised as build work.** Some items look like something to build and are really a
question about existing data or behaviour. They belong in a spike that produces an answer — and the
answer may turn out to be a bug rather than a change. Sizing one as build work is how a two-hour query
becomes a two-week ticket.

## Sequencing

1. Spikes and known gaps before whatever depends on their answers.
2. Foundational campaigns before the campaigns they make cheap.
3. Anything blocked on an external answer or another system is sequenced later **with the blocker
   named**, never held silently.
4. A structural dependency clears when the capability is **available to users** — behind a flag, that
   is enablement rather than deployment.

## Shipping

Campaigns ship in increments, and shipping is not the end of one. A first increment may be a proof of
concept whose whole purpose is to attract feedback, which arrives in the scratch area and is enlisted
in turn.

A campaign is not a release and a release is not a campaign: one campaign lands across many releases,
and one release can carry work from several.

## Writing out

**The durable statements a campaign produces are written out to the living notes as it goes**, in the
same change that makes each one true — never promoted, never left in a lump for the end. What counts
as true yet, and which sentence belongs on which side, are in [features.md](./features.md) and the
tense test in [SKILL.md](./SKILL.md).

They land in more than one place. What the product now does goes to the feature notes; design that
outlives the campaign, as-built detail and business facts learned along the way go to wherever the
project keeps them.

**A spike's answer is written out when the answer exists.** It changes no software, so no shipping
trigger will ever fire for it, and a finding left in the campaign note freezes there. Writing it down
is what *done* means for a spike.

**The campaign note carries a dated list of what it wrote out and where.** That is what makes the
check at close mechanical: read the list, confirm each entry was actually touched.

## Closing

**Close a campaign when further work would be better tracked under a new name.** Not when its scope is
complete — that never happens, because scope grows for as long as anyone is interested.

So closing is a **judgment, and openly a subjective one**. `closed` means *finished, abandoned, or
handed to a successor campaign*, never *succeeded*. A campaign that failed still closes, with the
reason recorded; a record that quietly drops its failures invites repeating them.

**A campaign is not closed until everything it wrote out is current.** The dated write-out list names
each living note it changed, so that is the definition of done, and it is checkable rather than a
memory exercise.

**Review long-open campaigns deliberately.** A subjectively-bounded campaign that nobody closes
becomes exactly the thing campaigns exist to avoid — an effort that accumulates scope and never
finishes. The closing convention is the only defence, and it works only if someone applies it.
