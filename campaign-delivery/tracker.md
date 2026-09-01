# The wiki and the tracker

Neither system owns the roadmap. **The wiki owns intent, decisions and the description of the product;
the tracker owns scheduled work and what shipped.** They meet at the campaign.

|                    | **Wiki**                                 | **Tracker**                            |
|--------------------|------------------------------------------|----------------------------------------|
| Holds              | Roadmap, campaigns, features             | Stories, releases                      |
| Unit               | The **campaign** — what was committed to | The **story** — a few days of change   |
| Time               | Readiness, no dates                      | Sprints, releases, dates               |
| Admits uncertainty | Yes — `forming` is a legitimate state    | No — an unstarted ticket looks stalled |

Which tracker, and how the fields below map onto it, is choice 5 in [adoption.md](./adoption.md).

## The gate

**Nothing enters the tracker before it is enlisted into a campaign.**

Material still in refinement has no home and no stable name, so there is nothing to book it against
and nothing certain to survive the week. Putting it in anyway creates a container that looks like
committed work and is not — and an empty container is indistinguishable from a stalled one, so the
board reports as failure what is really an idea nobody has committed to.

The corollary: **an unstarted container representing intent belongs back in the scratch area**, as a
roadmap item or unenlisted material, with the shell closed.

## Do not align the two taxonomies

The wiki groups by **intent** — what is wanted, who wants it, what it is worth. The tracker groups by
**delivery** — what ships together, what shares a codebase area, what a team can take on.

The same work therefore lands in differently-shaped buckets on each side, and the mapping between them
is legitimately many-to-many. That is not drift to be corrected; it is the two systems doing different
jobs. Forcing them into one shape breaks whichever is made to move:

- Copying the wiki's structure into the tracker produces empty containers.
- Reorganising the wiki to match the tracker's areas produces a plan no stakeholder recognises.

**Keep both, and invest in the join.**

## The join is the three coordinates

Every story carries **why**, **where** and **when**, and each needs a field that already exists rather
than a new one. What each coordinate requires of the field:

| Coordinate                        | Needs a field that is                                | Jira example |
|-----------------------------------|------------------------------------------------------|--------------|
| **Why** — campaign                | Multi-valued, cheap to add, closes with the campaign | Label        |
| **Where** — feature               | Multi-valued — a story can touch more than one       | Label        |
| **Where** — coarser level, if any | Multi-valued, permanent, exists for rollup           | Component    |
| **When** — release                | Dated, closes, is what actually ships                | Fix version  |

Two consequences follow:

- **Nothing that never closes should be a container.** A permanent grouping is a tag, not an epic; the
  thing that closes is a campaign, which is multi-valued rather than a parent.
- **Roll-up to the roadmap happens in the wiki**, where the item-to-campaign citation already lives.
  No roadmap id needs to exist in the tracker at all.

## What the discipline buys

Applying three fields per story is only worth doing if something reads them. Three things do:

- **Release notes become a query** — take a release, group by feature, annotate by campaign.
- **Campaign progress is a query**, working whether or not the stories sit under any common container.
- **Run-the-business share is a query** — everything with no campaign coordinate. That is the number
  an appetite has to be weighed against, and it is measured rather than guessed.

These mechanisms are free and routinely unused: on one real board, no labels, components or fix
versions on 183 issues created in a year.

## Where the story text lives

The tracker holds the story. The **reasoning** behind it stays in the campaign note, and the
**description of what the product now does** goes to the feature note — see
[features.md](./features.md). A ticket description that is the only account of a decision is a
decision that will be lost when the board is archived.
