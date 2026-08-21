---
name: bro
description: Use this skill on "bro", "in plain English", "say that again like a human", "I have no idea what you just said". Restates your own previous message with the jargon stripped out, as one person talking to another. It rewrites the message, never a file.
disable-model-invocation: true
---

# Bro

Restate your last message. Stop using jargon and speak coherently. Say it more simply and more
briefly, like one human talking to another.

That is the whole skill. The rest is what it must not turn into.

- **It rewrites the message, not the work.** No file, no wiki note, no commit message changes because
  of `/bro`. If the plain version is genuinely better than what is written down, say so and offer to
  put it there — as a separate step, with the usual skill for that target (`wiki-docs` for a note).
- **It is not a summary.** Same content, plainer words. Dropping half the message is a different
  request, and dropping the half the user needed is how this fails.
- **Real names survive.** File paths, symbols, commands and flags are the plain words here. Replace
  the metaphor, keep `mdfmt --check`. "The formatter run that fails CI" is jargon removal;
  "the checking thing" is information loss.
- **Say the thing you were dancing around.** Most jargon in a message is hiding an uncomfortable
  answer: it did not work, it is slower than expected, you are guessing. Plain language means the
  guess is labelled a guess.

If the message was already plain, say that instead of padding it out into a longer one.

## Lineage

Adapted from the `bro` skill in **pstack**, by poteto (Lauren Tan) — MIT, cloned from
[Benjamin-Connelly/cursor-plugins](https://github.com/Benjamin-Connelly/cursor-plugins). The
original is one sentence; the additions here are the boundaries, so a plain restatement stays a
restatement and never becomes an edit or a summary.
