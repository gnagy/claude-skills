# The product view

A **feature is a permanent part of the product**. The map. Export. Search. It has no deadline, no size
and no completion, because products do not complete — they are changed, by campaigns, over and over.

Default directory `product/features/`, one note per feature; this project's layout is in its `meta/`
note.

## Who names a feature

**Users name the parts they can see.** They point at the map and call it a feature; that is the name,
and it is better than anything triage would invent, because it is the name the requester will use
again in six months.

**Engineers name the parts users cannot see.** The CI pipeline, the auth layer, the data platform are
equally real parts of the product and equally permanent; they simply have nobody outside to name them.

Two naming authorities, one axis. Visibility is an **attribute of a feature, not a separate axis** —
nothing treats internal features differently, and splitting them means every rollup looks in two
places for no gain. Where the distinction matters is navigation, which is what a Map of Content is for.

## The list is an inventory, not a planning output

The feature list exists **before** any request arrives and describes what the product currently has.
Refinement reads from it and occasionally appends to it; it does not produce it.

That gives a health test: **the feature list does not grow with request volume.** Dozens, not
hundreds, stable for months at a time. A list gaining entries every refinement round is being used to
group work again, which is the campaign's job.

Two rules follow:

- **A name that describes a change is a category error, not a bad name.** `chart-refactor` is a
  campaign. `charts` is a feature.
- **Assign a feature to rejected work too** — see [intake.md](./intake.md).

## Feature notes are living

One note per feature, describing what it does **now**. Rewritten in place, never versioned into
variants, carrying no history of its own.

| Holds                                      | Does not hold                      |
|--------------------------------------------|------------------------------------|
| What the feature does today, for whom      | What it used to do                 |
| Its current constraints and known gaps     | Why each change was made           |
| The campaigns that shaped it, as citations | The campaigns' reasoning, restated |

**History lives in the campaign notes**, which are frozen and accumulate. *What did the map do in
2025?* is answered by reading the campaigns that touched it, not by a history section here. That
division is what earns the freezing on one side and the rewriting on the other.

A feature note that is wrong is simply corrected. It needs none of the frozen side's annotations.

## They are load-bearing

After fifty closed campaigns the feature notes are the only readable account of what the product does
— everything else is a pile of frozen records that has to be replayed to be understood. If they rot,
the knowledge base becomes archaeology.

So the discipline that matters most is keeping them current:

- **In the same change that makes it true.** Not as a later step: a note is cheap while the change is
  in front of you and expensive a week later, when whoever knew has moved on. Any trigger firing after
  the work lands competes with the next piece of work, and loses.
- **Verified at campaign close.** A closed campaign whose cited feature notes have not been touched
  since its last enlistment is a detectable defect — see [campaigns.md](./campaigns.md).

**Nothing is promoted.** A campaign note never becomes a feature note. Durable statements are *written
out* to the living notes while the campaign is open, and the campaign keeps its own account. Text may
move out of a campaign note only while it is open; after close the record stands and the living note
is written instead.

The feature note is usually the **smallest part of a write-out**, because it is a hub over the rest:
design that outlives the campaign, as-built detail and business facts learned along the way go
wherever the project keeps those.

## What counts as part of the product yet

A proof of concept nobody can reach has not changed the product, and writing it into a feature note
churns the one account that is supposed to be readable.

The test is not how settled the code is — **it is whether users can reach it**, the same test a
structural dependency uses. *Not expected to change any more* is the wrong bar, and it is the same
mistake as closing a campaign when its scope is complete: that moment does not arrive. Reachability
is a fact about the world rather than a prediction, so it can be answered on the day.

Behind a flag, the change that makes it true is the **enablement**, not the merge.

**Provisional is content, not a reason to omit.** *A proof of concept, on for one partner, may be
pulled* is a good sentence in a note whose job is what the feature does today and for whom. When it
settles the paragraph is rewritten; when it is withdrawn the paragraph is deleted.

## A coarser level, if the project has one

Some projects need a grouping above the feature — the level a cost is carried against and a rollup is
read at. It sits on the **same axis** as the feature and shares its properties: permanent, never
scheduled, never closed.

**Add it only when a rollup is wanted coarser than the feature**, and reuse a grouping the project
already has rather than inventing a third one. Where the thing users name is also the thing you would
budget against, feature and the coarser level are the same row, and the second level is structure for
its own sake. Whether this project has one, and which grouping carries it, is choice 2 in
[adoption.md](./adoption.md).
