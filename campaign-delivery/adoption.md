# Wiring a project to this process

This file is for the agent setting a project up, or retrofitting one that copied the model into its
own notes. The rest of the skill is for the agent doing the work.

**Check the fit test in [SKILL.md](./SKILL.md) first**, and say so out loud if the project fails it.
Adopting this where it does not fit is worse than leaving the project alone.

## The problem this solves

A project's `meta/` notes and `CLAUDE.md` do not move when this skill moves. Every sentence a project
copies out of here is correct on the day it is written and unfalsifiable afterwards — it still reads
as local law long after the mechanic it describes has changed. So a project gets a **pointer with no
content to go stale**, plus a record of **its own choices**, and nothing else.

## The eight choices

Run these as an interview — one question at a time, recommended answer attached, the
`grill-with-wiki` way. Everything here is genuinely the project's; everything not here is not.

| # | Choice                                                                   | Default                                          | Constraint the answer must satisfy                                                                                 |
|---|--------------------------------------------------------------------------|--------------------------------------------------|--------------------------------------------------------------------------------------------------------------------|
| 1 | Names for the scratch / living / frozen directories                      | `intake/` `product/` `campaigns/`                | Three distinct directories. The citability line falls between scratch and the other two                            |
| 2 | A structural level coarser than the feature?                             | None                                             | If yes it is on the structure axis, and it reuses a grouping the project already has rather than inventing a third |
| 3 | The campaign state field: name and values                                | `campaign_status`: `forming` / `open` / `closed` | Separate from the note's own `status`. Exactly three values                                                        |
| 4 | Where the three directories sit relative to the project's existing areas | Top level of the wiki                            | They must not merge with an area that has a different write discipline                                             |
| 5 | The tracker, and which field carries each coordinate                     | Jira: label, label, component, fix version       | Every coordinate lands on a field that already exists — see [tracker.md](./tracker.md)                             |
| 6 | Campaign subfolder names, once one earns them                            | `analysis/` `design/` `delivered/`               | Not `implementation/` — it collides with the living notes                                                          |
| 7 | Appetite units, and who may be a sponsor                                 | Prose; every campaign has one                    | Not tracked to the hour                                                                                            |
| 8 | Where durable design lives, if not in the feature notes                  | The feature notes                                | The feature note stays a hub, and the write-out list still names every destination                                 |

**The list is closed.** A project needing something outside it has either found a gap in the model —
report it, and it changes here for every project — or it is forking, and it stops claiming to run this
process. Recording an out-of-list deviation in `meta/` as though it were a choice is how a fork gets
mistaken for a configuration.

Two questions are **not** choices, though they get raised as if they were:

- *Do we have to keep the roadmap?* Yes, if you want the intent axis at all. A project may run
  without one, but then it has no answer to "where is this going" and no home for *not now* — say
  that plainly rather than pretending the axis is optional.
- *Can a campaign be a container in the tracker?* No. That is the gate, and it is the mechanic the
  whole boundary rests on.

## What a wired project has

1. A **marked block in `CLAUDE.md`** — the mandate, and the rule against restating the model.
2. **One `meta/` note carrying the eight answers**, and nothing else.
3. **The three directories**, each with a stub saying what discipline it carries.

No second guard hook: if the project already runs the `wiki-docs` guard, it fires on these paths
because they are inside the wiki. If it does not, add that one — see the `wiki-docs` skill,
"adoption".

## 1. The `CLAUDE.md` block

**Everything between the markers is verbatim.** Do not adapt it, do not add project detail inside it,
do not summarise it. Project detail goes above the markers, where the project owns it.

```markdown
## Delivery

<!-- project facts live out here: where the wiki is, which tracker, who the sponsors are -->

<!-- campaign-delivery:begin — managed by the campaign-delivery skill; replace wholesale, never edit in place -->
**This project runs campaign-based delivery, and the `campaign-delivery` skill owns it.** Load it
before capturing a request, refining anything, enlisting material, opening or closing a campaign,
breaking one into stories, or touching the feature notes.

**The skill owns the model** — the four axes, the three coordinates, the write disciplines, the
enlist gate, what closing means. **Do not restate any of it here or in the wiki's `meta/` notes.** A
local copy goes stale silently and the next agent reads it as law. The project's `meta/` note carries
its eight *choices*; for everything else it points at the skill by section name.

Three rules break most often and fail silently: nothing enters the tracker before it is enlisted;
nothing outside the scratch area cites a campaign before its first enlistment; durable statements are
written out to the living notes in the same change that makes them true.
<!-- campaign-delivery:end -->
```

The markers are the mechanism: they make re-adoption a replacement rather than a judgement call, and
they make an in-place edit visible as one. **If you find the block edited, the edit is the bug** —
replace it, and move whatever the edit was trying to say to the project-owned side.

## 2. The `meta/` note

One note, the eight answers, roughly forty lines. A template — replace the answers, keep the shape,
add nothing:

```markdown
# Delivery choices

This project runs campaign-based delivery. **The `campaign-delivery` skill owns the model**; this note
records only what this project chose. Nothing here restates a mechanic — if you find something that
does, delete it and point at the skill.

| # | Choice | This project |
|---|--------|--------------|
| 1 | Scratch / living / frozen directories | … |
| 2 | Coarser structural level | … |
| 3 | Campaign state field | … |
| 4 | Placement in the wiki | … |
| 5 | Tracker and field mapping | … |
| 6 | Campaign subfolder names | … |
| 7 | Appetite and sponsors | … |
| 8 | Where durable design lives | … |

## Why, where a choice is not obvious

<!-- one short paragraph per non-default answer. The reason, not the mechanic -->
```

**The "why" paragraphs earn their place**; the rest is a lookup table. A non-default answer with no
recorded reason gets re-litigated every six months.

## 3. The directories

Create the three, each with a stub naming its discipline and pointing at the skill. Then stop.

**Adoption creates no content.** The feature inventory is the project's, it is the prerequisite for
the first campaign, and building it is the first real work — see [features.md](./features.md) for the
health test that tells you whether it is being built as an inventory or as a plan.

## Retrofitting a project that copied the model

A project that adopted this process by *porting the notes* has the model living in its `meta/`, where
nothing will ever update it. The retrofit:

1. **Read its ported notes against this skill** and sort every section into *model* or *choice*.
2. **The model sections are deleted**, not merged. They are a stale copy by construction, however
   good they were on the day.
3. **The choices collapse into the one `meta/` note** above. Expect eight answers out of several
   hundred lines.
4. **A section that is neither** — a real practice this skill has no equivalent for — is the
   interesting output. It is either a gap in the model, which comes back here, or genuinely local
   content that belongs in an ordinary wiki note rather than in `meta/`.
5. **Fix the inbound links** before deleting anything, and check for citations from outside `meta/`.
   That is `wiki-docs` work, and its rename and health-check tooling is what makes it safe.

Report what you would delete before deleting it. A ported model is usually the most carefully written
thing in the wiki, and its author will want to see the mapping rather than the diff.
