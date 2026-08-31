---
name: unslop
description: Use this skill to strip the AI tells out of any prose — "unslop this", "this reads like AI wrote it", "make it sound human" — and before shipping a README, a doc, a wiki note, a PR description or a commit message. A catalog of the patterns that mark machine-written text, what to replace each with, and the wikilink hazards that make editing a wiki note different from editing a file.
---

# Unslop

Edit text to remove the patterns that mark it as machine-written, and put a voice back in.

This skill owns the **slop catalog**. Other skills point here rather than restating it —
`technical-writing` owns document structure and sentence craft and defers to this file for the
pattern list.

## Process

1. Scan for the patterns below.
2. Rewrite. Preserve meaning, match the intended tone.
3. Add voice (next section).
4. Self-audit: *"what makes this obviously AI generated?"* Fix what is left.
5. If the target is a wiki note, run the [wiki checks](#editing-a-wiki-note) before and after.

## Adding voice

Removing patterns is half the job. Sterile, voiceless writing is just as obvious as flowery writing.

- **Have opinions.** React to the facts instead of listing pros and cons at equal weight.
- **Vary rhythm.** Short sentences. Then longer ones that take their time and carry a fact with its
  condition attached. Mix them on purpose.
- **Acknowledge complexity.** "Impressive, and also kind of unsettling" beats "impressive".
- **Use "I" when it fits.** First person is not unprofessional.
- **Let some mess in.** Perfect structure looks machine-made.
- **Be specific.** Not "this is concerning" but "there is something unsettling about agents churning
  away at 3am".

> **Voice is mode-dependent.** A reference table, an API listing or a wiki note carrying
> `type: reference` stays dry on purpose: describe, do not persuade. The voice rules apply to prose
> that explains, argues, or introduces. Applying them to a lookup table is its own kind of slop.

## Patterns to detect and fix

### Content

1. **Puffery.** "pivotal moment", "testament to", "evolving landscape", "setting the stage for",
   "indelible mark", "deeply rooted". Cut it, state what happened.
2. **Name-dropping.** Listing outlets, companies or tools without context. Pick one, say what it did.
3. **Superficial -ing phrases.** "highlighting…", "ensuring…", "reflecting…", "showcasing…",
   "fostering…". Delete, or expand into a real claim with a source.
4. **Promotional language.** "nestled", "vibrant", "breathtaking", "groundbreaking", "renowned",
   "stunning", "must-visit". Describe neutrally.
5. **Vague attributions.** "Experts believe", "Industry reports suggest", "Some critics argue". Name
   the source or delete the sentence.
6. **Formulaic challenges.** "Despite challenges, X continues to thrive." Replace with the facts.

### Language

7. **AI vocabulary.** Additionally, crucial, delve, enduring, enhance, fostering, garner, interplay,
   intricate, landscape (abstract), pivotal, showcase, tapestry (abstract), testament, underscore,
   vibrant. Use the plain word.
8. **Fancy ways to say "is".** "serves as", "stands as", "boasts", "features". Say "is" or "has".
9. **"Not just X, but Y."** State the point directly.
10. **Rule of three.** Forcing ideas into groups of three. Use the number there actually is.
11. **Synonym cycling.** Protagonist, main character, central figure, hero in one paragraph. Pick one
    and repeat it. In technical prose this is worse than repetitive — two names read as two things.
12. **False ranges.** "from X to Y" where X and Y sit on no shared scale. List the items.

### Style

13. **Em dash discipline.** An em dash per paragraph is voice. Three is a tell, and a dash joining
    two independent clauses is usually a sentence that wanted to end. Never swap one for parentheses
    to dodge the rule — that trades one tell for another. Do not convert dashes wholesale across a
    document you are only lightly editing; the churn is bigger than the fix.
14. **Colon overuse.** A colon before a list or an example is fine. As a mid-sentence connector it
    adds nothing: *"If you're coming from traditional automation: instead of registering handlers,
    you describe conditions"* says more without the comparison framing —
    *"Describing when the scheduler should fire works best as plain English."*
15. **Boldface overuse.** Don't bold every proper noun or acronym. Bold marks the claim of a
    paragraph, not its nouns.
16. **Inline-header lists.** The tell is a bold label and colon that restates the line:
    "**Performance:** Performance improved…". Convert those to prose. A bold lead-in that ends in a
    period, names the item, and is followed by genuinely new detail ("**Schema in TypeScript.**
    Tables live in one file.") is fine, and is the house pattern.
17. **Title case headings.** Use sentence case.
18. **Decorative emojis.** Remove from headings and bullets.
19. **Curly quotes.** Replace with straight quotes. Watch for smart-quote substitution sneaking back
    in through copy-paste.

### Communication artifacts

20. **Chatbot phrases.** "I hope this helps!", "Let me know if…", "Of course!", "Certainly!", "Found
    the smoking gun!" Remove.
21. **Cutoff disclaimers.** "While specific details are limited…" Find the source or drop the claim.
22. **Sycophantic tone.** "Great question! You're absolutely right!" Respond directly.

### Filler

23. **Filler phrases.** "In order to" is "To". "Due to the fact that" is "Because". "It is important
    to note that" is nothing.
24. **Excessive hedging.** "could potentially possibly be argued that it might" is "may".
25. **Generic conclusions.** "The future looks bright." State the specific plan, or stop.

### Jargon

26. **Abstract metaphor nouns.** Substrate, wedge, vector, locus, vantage, nexus, primitive (as a
    noun), harness (as a metaphor), surface (as in "API surface"), bedrock, scaffolding (as a
    metaphor), modality, paradigm, gold-plating, ratchet (as a metaphor), evacuate (for moving code),
    endgame, north star, flywheel. Each has a plainer concrete word: "substrate" is "base", "wedge
    in" is "add", "vector" is "way", "gold-plating" is "more than the job needs", "ratchet" is the
    mechanism's real name or "a limit that only tightens", "evacuate" is "move out", "endgame" is
    "the last phase". Pick the concrete word.

    When you retire a new one, add it to this list with its replacement. This catalog is the record.

### Plain speech

27. **Say what it does, not how it feels.** "the database stays close at hand", "SQL you can read",
    "types that follow your schema" all name a feeling. The fix names the mechanism or a number:
    "`.toSQL()` returns the exact string sent to the database", "a column rename fails the build".
    Ask what the sentence tells the reader to do or know, then write that. If you cannot restate it
    as an instruction, a fact or a number, cut it. One more check: a sentence that could appear
    unchanged in another project's docs says nothing about this one.
28. **Shorten or split dense sentences.** If the reader has to backtrack to parse it, break it in
    two or drop a clause. One idea per sentence.
29. **Active voice.** Catch "is/are/was/were + past participle" and name the actor: "queries are
    validated" becomes "the compiler validates queries". Passive is fine when the actor is unknown or
    genuinely beside the point.
30. **Cut adverbs, or use a stronger verb.** "runs quickly" is "is fast", or the number.
    "significantly improves" is the measured delta. An adverb propping up a weak verb means the verb
    is wrong.
31. **Prefer the plain word.** "utilize" is "use", "leverage" is "use", "facilitate" is "help",
    "numerous" is "many", "in the event that" is "if". The fancier synonym is rarely clearer.

## Editing a wiki note

A wiki note is a node in a link graph, not a document. **Load `wiki-docs` before editing one** — it
owns the storage mechanics and the wikilink rules, and the wiki's own `meta/conventions.md` wins
wherever it disagrees with this file. Five hazards belong to *this* skill, because they are caused by
rewriting prose:

> **Cutting a sentence can cut an edge.** Rule 23 says delete the sentence that does no work. If that
> sentence carries the only `[[wikilink]]` to a note, the deletion removes a graph edge — a backlink
> disappears, or a note becomes an orphan — and the text diff looks like a tidy-up. Before deleting
> any sentence containing `[[…]]`, check whether that link exists elsewhere in the note. If it does
> not, keep the link and rewrite around it.

- **Never rewrite inside `[[ ]]`.** The stem is an identifier, not prose. A stem that reads badly is
  a `move_resource` job, never an edit — renaming it by hand breaks every inbound link silently.
- **Never reach for `[[stem|label]]` to fix grammar.** Labels are banned in `wiki-docs` because a
  rename rewrites the target and keeps the stale label. Put the phrase in prose and the link beside
  it: `Dashboards are tier 2 ([[two-tier-state]])`.
- **Leave cross-wiki links alone.** `[conventions](otherwiki:meta/conventions.md)` is deliberate and
  looks broken. Converting one to a `[[wikilink]]` poisons the placeholder list permanently.
- **Front matter is prose too.** `title` and `description` collect slop like any sentence —
  "This note explores the evolving landscape of…". `description` is one sentence saying what the note
  is about; `title` matches the H1 exactly, so change both or neither.
- **Recorded uncertainty is not hedging.** Rule 24 kills "could potentially possibly". It does not
  touch *"these two sources disagree; unverified"*, which is a fact about the state of knowledge and
  is what a wiki is for. Cut the hedge, keep the open question.

Do not hand-fix tables, escaping or curly quotes in a wiki: run `awt fmt` (see `wiki-docs`).
After a bulk pass, verify the graph rather than the text — `foam lint` and a `grep -c '\[\['` count
before and after. `wiki-docs` has the commands.

## Lineage

Adapted from the `unslop` skill in **pstack**, by poteto (Lauren Tan) — MIT, cloned from
[Benjamin-Connelly/cursor-plugins](https://github.com/Benjamin-Connelly/cursor-plugins). Changed
here: the blanket em-dash ban became rule 13's discipline, since em dashes are house voice; voice is
scoped by document mode; and the wiki section is new.
