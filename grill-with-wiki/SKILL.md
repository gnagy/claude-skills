---
name: grill-with-wiki
description: Use this skill on any "grill" trigger — "grill me on this", "grill this design". A relentless one-question-at-a-time interview that challenges the project's own terminology as it goes, capturing terms, decisions, and open questions into the project wiki as they surface.
---

# Grill with wiki

A relentless interview that sharpens a plan or design **and leaves a paper trail in the project's
wiki**.

**Load `wiki-docs` first.** It owns the storage mechanics — the foam-wiki tools, front matter, the
lookup recipe, the wikilink hazards, the health checks. The wiki's own `meta/` notes own *this*
project's layout, vocabularies and capture practice, and win wherever they disagree with this file.
What lives here is the interview and what it produces.

## The interview

Interview the user relentlessly about every aspect of the subject until you reach shared
understanding. Walk each branch of the decision tree, resolving dependencies between decisions one at
a time.

- **One question at a time**, waiting for the answer before the next. Several at once produces vague
  answers to all of them.
- **Attach your recommended answer to every question.** It gives the user something to push against.
- **Look facts up; ask decisions.** Anything findable in the wiki, the code, the config or a tool,
  find it. The decisions are the user's — put each one to them and wait.
- **Load the glossary and the navigation axis before the first question.** You cannot challenge a term
  against a glossary you have not read. If the wiki has no `meta/` notes yet, the first session is a
  different job — see [FRESH-WIKI.md](./FRESH-WIKI.md).
- **Build only once the user confirms** you have reached shared understanding. Capturing what the
  interview settles is not building.

## Sharpen the model as you go

Five behaviours, all active throughout the interview:

**Challenge against the glossary.** When a term conflicts with the wiki's existing language, say so
immediately: *"The glossary defines Site as the whole station and reserves Location for the geocoded
point — you seem to mean the point. Which is it?"*

**Sharpen fuzzy language.** When a term is vague or overloaded, propose the precise canonical one.
*"You're saying 'account' — do you mean the Customer or the User? Those are different things."*

**Stress-test with concrete scenarios.** Invent specific edge cases that force the boundaries between
concepts to be precise. Abstract agreement hides disagreement; a scenario surfaces it.

**Cross-reference against the source.** When the user states how something works, check the code,
schema or config. Surface contradictions: *"The entity cancels whole Orders, but you just said partial
cancellation is possible — which is right?"* The source wins over a note, and a note found stale gets
fixed as part of the session.

**Watch the navigation axis.** If the subject fits no existing value of the wiki's primary axis
(`domain`, `topic`, `layer` — whatever `meta/conventions.md` fixed), that is a finding. Either the
note belongs somewhere you have not considered, or the vocabulary needs extending — and extending a
controlled vocabulary is a decision to put to the user, not one to make silently.

## Capture inline

**Write each outcome down the moment it crystallises.** Batching is how a session ends with a great
conversation and nothing in the wiki.

**Follow the grain of the wiki.** Before writing the first outcome, work out how this wiki already
records these things and match it — the form is the project's choice, not this skill's. Read what it
*does*, not only what `meta/` says it does: a `type` value registered in the conventions but never
used is not yet a practice.

| Outcome                         | Lands in                                                                          |
|---------------------------------|-----------------------------------------------------------------------------------|
| A term pinned down              | The glossary — with the adjudication and the date, so a ruling reads as a ruling  |
| A question resolved             | The topic's open-question register, marked resolved *in place* with the reasoning |
| A question needing someone else | That register, plus whatever cross-wiki index collects them                       |
| A decision with rationale       | Wherever this wiki puts decisions — a design note, a register, or an ADR          |
| A new concept                   | Its own note, in the area the conventions assign it                               |
| Something you could not settle  | A new open question. Recording it *is* an outcome                                 |

> **Resolved questions stay where they were asked.** Marking one resolved in place keeps the fact that
> it was ever contested — which is most of the value. Keep the reasoning and link the note that now
> carries it.

Two rules the storage layer does not cover:

- **A glossary is a glossary.** Not a spec, not a scratchpad, not somewhere to park implementation
  detail because it had no other home.
- **Offer a standalone decision record sparingly** — only when all three hold: hard to reverse,
  surprising without context, and the result of a real trade-off. Otherwise the decision belongs
  inline in the note it affects. Most do.

## Close the session

Report three things: the notes you touched, each decision and where its rationale now lives, and every
question left open. Then run the graph check — a session that created or renamed notes changed the
link graph, and a broken graph does not show up in a text diff (`wiki-docs` has the commands).

## Lineage

The interview and the modelling behaviours are adapted from
[mattpocock/skills](https://github.com/mattpocock/skills) (MIT) — `grilling`, `domain-modeling` and
their `grill-with-docs` wrapper, flattened into one skill. Their fixed storage (`CONTEXT.md`,
`CONTEXT-MAP.md`, numbered `docs/adr/`) is replaced by whatever the project's wiki already uses.
