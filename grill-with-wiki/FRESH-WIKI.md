# Grilling a wiki into existence

When the wiki has no `meta/` notes, the first session is about the wiki itself. There is nothing to
challenge a term against yet, so the interview builds the thing it will later challenge against.

Use `wiki-docs` to create the skeleton, then grill out three things — and only these three.

## 1. The primary navigation axis

Every wiki needs one subject axis orthogonal to its folders, so a note has an obvious home and related
notes cluster.

**Teach the shape; let the project pick the axis.** DDD bounded contexts are the documented default
for application-shaped projects and earn their keep there. Dropped on a homelab or a research
workspace they read as costume — *"the bounded context of DNS"*. Layer, host-role, component or plain
topic may fit better.

Grill for the axis that gives every note an obvious home, then write the chosen values down as a
closed vocabulary in `meta/conventions.md`.

## 2. The term table

Start with a **single `glossary.md`**. Promote a term to its own note only once it outgrows a couple
of sentences, leaving a link behind.

Per-term notes from the start is more churn than a grilling session sustains, and a placeholder link
costs nothing — `get_placeholders` turns it into a backlog.

## 3. Where decisions land

Settle this once, here, rather than improvising per decision later. A register note holding a table of
related decisions, one note per decision, or inline in the note each decision affects — any of them
works, and the cost of not choosing is that you get all three.

If the project uses a note-maturity `status` field, keep decision lifecycle in a separate field.
Overloading one field with `draft|stable|deprecated` and `proposed|accepted|superseded` loses both.

---

**Stop after the skeleton and agree it with the user before writing many notes.** Layout and taxonomy
are far cheaper to settle before there are fifty notes to migrate.

Then run the real grilling session on the first actual subject. The vocabulary will need adjusting —
that is the model getting its first honest test, not a failure of the bootstrap.
