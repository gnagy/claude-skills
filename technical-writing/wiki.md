# Writing a wiki note

Read this before writing or reviewing a note in a Foam wiki. The four layers in `SKILL.md` still
apply. What changes is that a note is a **node in a link graph** with a controlled vocabulary, a
scope rule, and a rule against duplicating its own sources.

**Load `wiki-docs` first.** It owns the storage mechanics: the foam-wiki tools, front matter, the
wikilink hazards, the health checks. The wiki's own `meta/conventions.md` and `meta/scope.md` own
*this* project's vocabularies and what belongs at all, and they win wherever they disagree with this
file. `unslop` owns the slop catalog and its own wiki hazards. This file only covers what changes
about the *writing* when the target is a note.

## Diátaxis picks the mode; the wiki's axes classify the note

**Do not add a `diataxis` field, and do not stretch an existing axis to mean mode.** Inventing front
matter is forbidden by `wiki-docs`, and the two systems are answering different questions:

| Question                            | Answered by                                                  |
|-------------------------------------|--------------------------------------------------------------|
| What kind of writing is this?       | Diátaxis — a thinking tool, invisible in the file            |
| What form does this wiki accept?    | `type` — the project's closed vocabulary                     |
| Where in the lifecycle does it sit? | Usually an `area` or folder axis: analysis, design, as-built |
| What subject is it about?           | Usually a `topic` or `tags` axis, cutting across the folders |

A wiki's `type` vocabulary is usually narrower than Diátaxis and drawn on a different line. Read the
vocabulary, pick the value it defines, and let Diátaxis decide only how the prose is written. Two
recurring shapes:

- **A "reference" `type` is not always Diátaxis reference.** In most wikis it means "an inventory of
  external facts that will age", which is a narrower thing than "facts for lookup".
- **A decision record is explanation.** It exists to answer a why question, and opinion is allowed in
  it — that is the one mode where it is.

## Which modes belong in a wiki at all

Most notes are **explanation**. A wiki holds the narrative knowledge: how the pieces fit, the
decisions behind them, the open questions. Each note's title should tolerate an implicit "About…".

**Reference** notes belong when the facts are external and would otherwise be re-derived: a tool
landscape, a field list, an option comparison. Two constraints from `wiki-docs` bite here, and both
outrank the Diátaxis instinct to be complete:

- **Never paste the authoritative source.** No config contents, no data rows, no copied tables from
  another project. Link the path and explain the shape and the reasoning. A reference note is
  complete about *shape and consequence*, never about content the source already holds.
- **Facts that age get their vintage.** Say when they were checked and against what. "No hedging"
  means no hedged claims, not pretending a snapshot is permanent.

**How-to** rarely belongs. A procedure someone runs belongs in the repo that runs it — a README, a
script's `--help`, or a skill. A note explaining *why* the procedure is shaped that way belongs in
the wiki, and links to it. **Tutorials** do not belong at all.

## Where the layers and the wiki rules collide

Four conflicts come up repeatedly. The wiki rule wins each time.

> **"State facts with no hedging" vs. "record open questions."** Reference mode says be sure. The
> wiki says a note that reads *"these two sources disagree; unverified"* beats one that quietly picks
> a side. These do not actually conflict: an open question is a **fact about the state of knowledge**,
> stated plainly. Write it as a question with what is known on each side, never as a claim wearing a
> hedge. Cut "may possibly be somewhat"; keep "unverified".

- **"One document, one mode" vs. "one concept per note."** The wiki rule is stricter, and it usually
  explains the mode problem: a note that turns into reference halfway through has picked up a second
  concept. Split it and link the two, rather than sectioning the mixture.
- **"Call each thing by one name" is the glossary.** In a wiki this is not a style preference — it is
  the graph. Two names for one thing become two notes eventually. When you find a term that is not
  settled, that is a decision to put to the user, not one to fix silently while editing; `/grill-with-wiki`
  is the tool for settling it.
- **"Say where the link goes" vs. bare wikilinks.** Google style wants link text that describes the
  destination. `wiki-docs` bans `[[stem|label]]`, so the link text is fixed as the note's own stem.
  The sentence has to carry the context instead: *"the two axes stay orthogonal ([[wiki-conventions]])"*,
  not *"see [[wiki-conventions]] for more"*.

## Front matter is writing

- **`title` is a noun phrase that names the note, and the H1 repeats it exactly.** Change one, change
  both. Sentence case.
- **`description` is one sentence saying what the note is about.** Not that it exists ("This note
  explores…"), not what the reader will learn. It appears in search results, in listings and in the
  index, so it is read far more often than the note.
- **The index entry is writing too.** A new note that never reaches `index.md` is an orphan, and its
  one-line summary is held to the same standard as the description — the same voice, not a second
  paraphrase that drifts from it.

## Checks before you finish

1. Does the note carry exactly one concept, and does its `type` value come from the project's
   vocabulary?
2. Does `title` match the H1, and does `description` say what the note is about in one sentence?
3. Is every claim about another file linked to that file rather than copied from it?
4. Does every term in the note match the wiki's existing name for that thing?
5. Are open questions written as questions, and hedges cut?
6. Is the note reachable — an `index.md` entry or a link from a note that has one?
7. Did the edit change the link graph? Run the `foam lint` and placeholder checks from `wiki-docs`.
   A rewritten paragraph that dropped a link shows up nowhere else.
8. Run `mdfmt` rather than hand-aligning tables (see `markdown-remark`).
