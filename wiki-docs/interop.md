# Other wikis: read-only mounts and messaging

Reference file for the `wiki-docs` skill. Read it when this project mounts another project's wiki, or
when you need to send a correction to one. `SKILL.md` beside this file covers the wiki itself.

**Which wikis are mounted, what each is authoritative for, and when to consult which is project
knowledge.** It belongs in the wiki's own `meta/interop.md`, not here. Read that first.

## What a mount is

A project may mount other projects' wikis as additional read-only servers, conventionally named after
the project. Each mount adds five read tools; `--allow-writes` adds seven more, so a mount
without it has no write surface at all rather than an unused one.

`workspace_info` reports `root` and `allowWrites`, so confirm which server you are talking to
before writing, rather than inferring it from the server's name. To add a mount, see `setup.md`
beside this file.

### Which way a mount points

**Mounts run one way: down the dependency, never back up.** The project that already depends on the
other is the one that mounts it — a build consuming another repo's output, a migration targeting
another repo's schema, a script shelling into a sibling path. The depended-on project must never
mount its dependent: it has to build and be worked on with no sibling checkout present.

Two wikis mounting each other is a cycle. The symptom is a path to repo B in the committed config of
a repo that should not know B exists — and it usually arrives as "mirroring their setup", which is
not a reason.

**Correspondence is symmetric; mounts are not.** Both sides can have an inbox and use the same
message convention. Messages never needed a mount to work, so an asymmetric mount costs the exchange
nothing.

Where neither project depends on the other, prefer no mount over two: read the sibling checkout with
ordinary file tools and treat its absence as normal.

## Rules

1. **Never edit another project's notes.** The read-only mount is the guarantee; don't route around it
   with Write/Edit. If something there is wrong, send a message (below), or go and work in that repo,
   where its conventions and `CLAUDE.md` load with you.
2. **Copy only with declared provenance.** Duplication is not the problem; silent duplication is. A
   copied identifier — a model number, a MAC, an address — is often the readable thing to do, but it
   must carry where it came from and when, so a later reader can tell a copy from an independent
   assertion and divergence is detectable rather than invisible. Copy identifiers, not judgements: a
   claim you cannot re-check is one to cite, not restate.
3. **Foreign front matter is a foreign vocabulary** — the same field name can carry different values
   in each wiki. Read the other wiki's `meta/conventions.md` before interpreting it.

   Where wikis are meant to share a vocabulary, don't restate it in each — **declare a base and its
   deviations**, the same pattern OKF conformance uses (an OKF extension key; extensions are legal
   and consumers must preserve them):

   ```yaml
   # meta/conventions.md front matter
   derived-from: <wiki>:meta/conventions.md
   ```

   and document only what this wiki does differently. Prose conventions decay, so put the parts that
   matter into shared **front-matter schemas** (`.remark/*.schema.json`): a shared vocabulary that
   fails a check is worth more than one that agrees in two documents until it doesn't.

   **`derived-from` inherits mistakes as readily as vocabularies**, so fixing a convention upstream is
   only half the job — message the wikis that derive from it. One misleading sentence about a project's
   tooling reached a second wiki purely by inheritance, cost real work there, and was still sitting in
   the wiki it came from after the derived copy had been fixed. Two wikis carrying the same wrong
   sentence is not two mistakes; it is one, propagating.
4. **Wikilinks do not cross wikis.** See below.
5. **Two wikis that reached the same answer independently are evidence; two that disagree are an open
   question.** Record the disagreement — don't quietly pick a side. Independent corroboration is the
   main reason to keep the wikis separate at all.

## Cross-wiki references

**Settled, and `site.md` beside this file owns it.** A cross-wiki reference is an ordinary markdown
link whose destination carries a wiki prefix instead of a scheme —
`[conventions](otherwiki:meta/conventions.md)` — inert in the graph and in the editor, resolved to a
real URL at build time by the `awt-cross-wiki` transformer. Read *Cross-wiki references* in `site.md`
before writing one, for the registry, the prefix rules and the resolver's semantics.

Three rules matter wherever the reference is written, site or no site:

> **Never write one as a `[[wikilink]]`.** Any unresolved `[[…]]` is a placeholder, so it lands in the
> placeholder list permanently and poisons the one signal that means "note worth writing". A prefixed
> link carries a scheme, so the graph treats it as external and never counts it.

- **Always the full path within the target wiki, never a bare stem.** Basenames collide across wikis —
  `index.md`, `meta/conventions.md` and `meta/scope.md` exist in every one — so the reference must
  carry both the wiki's prefix and the path.
- **In front matter the same prefixed string is idiomatic**, and nothing resolves it either way:
  `resource: homeit:implementation/hardware/electron-chassis.md`.

A wiki with no site still writes references this way. They do not resolve until something renders
them, but they are the form that will, and they cost nothing meanwhile.

**What is still genuinely missing is backlinks *across* wikis.** Resolution is solved, and so is
anchor checking — `awt-headings` publishes each page's anchors beside the content index, so a
reference carrying `#a-heading` into a wiki that emits it is verified at build time and warns when the
heading is gone. But a reference into another wiki is still invisible *to that wiki*: nothing there
knows it is depended upon, and `awt check` deliberately treats a schemed destination as external. That
is the remaining argument for tooling beyond plain read-only mounts.

## Finding another wiki's inbox

**The receiver declares its own inbox; senders never assume a path.** Each wiki that accepts messages
carries `meta/interop.md` — read it through a mount if you have one, otherwise straight from the
sibling checkout; it is plain markdown either way:

| Section  | Declares                                                                           |
|----------|------------------------------------------------------------------------------------|
| Inbound  | Where this wiki's inbox is, what it is authoritative for, what messages it accepts |
| Outbound | Which other wikis this one mounts, and when to consult each                        |

**Outbound applies only to a wiki that actually mounts something.** A wiki that mounts nothing says
so, with the reason, so the absence reads as a decision rather than an oversight.

**No `meta/interop.md` means no inbox — do not write to that repo.** Accepting messages is an opt-in
its owner declares, not a default a sender may assume.

## Sending a message to another wiki

Rule 1 says don't edit their notes; this is the sanctioned alternative. The receiving project has an
inbox directory **outside its wiki root** — conventionally `docs/wiki-inbox/`, beside `docs/wiki/`,
but always as declared in its `meta/interop.md`. The sender writes a file there directly with Write.
That is the **only** path in another project's repo any session may write.

**Outside the wiki root is deliberate, on two counts.** Nothing indexes the inbox, so messages
don't distort note counts, orphans or placeholders — and a message cannot quietly start being read as
a fact. The receiver has to *convert* it into a real note before it counts as knowledge.

**The inbox must not be gitignored.** An unhandled message is an untracked file, which is what makes
it appear in `git status` next time that repo is opened. Git is the notifier, not the transport:
nothing needs committing or pushing for a message to arrive, and the index is recomputed per call, so
mounted wikis reflect each other's uncommitted edits live.

Filename `YYYY-MM-DD-<sender>-<slug>.md`:

```markdown
---
from: <sending wiki>
date: <YYYY-MM-DD>
about: <uri in the receiving wiki, if it concerns an existing note>
evidence: <wiki>:<uri>          # where the reasoning lives — a pointer, never a copy
---

What is claimed, what would settle it, and what you want back — if anything.
```

**Four fields, and no message type.** An earlier version fixed a `kind` vocabulary —
`correction | contribution | verification-request | drift-notice` — and it did not survive contact:
across fifteen real messages senders reached for it four times, always for the same value, and
otherwise wrote their own. The classification never changed what a receiver did, nothing mechanical
reads these files, and a closed vocabulary no machine enforces is a schema you can only fail. **Say
what happened in the body, in your own words.**

**Resolving one:** fold it into a real note with its own `verified` entry, citing the message's
`evidence` in `sources`, then **delete the inbox file**. Deletion is the close, and the resulting note
is the record — don't build a `resolved/` archive, because the provenance belongs in the note.

**Re-read the file immediately before folding it, not when you triage it.** A message is a file in a
working tree that its sender is still editing, and they have no way to know you have started. One was
rewritten mid-fold in real traffic: the receiver had already folded the old version and written a
reply. So senders **amend a message in place rather than replacing it with a differently-named
file** — a rename strands whoever is mid-fold and leaves the superseded copy in history. Stage the
inbox by explicit path; a `git add` of the directory sweeps in whatever arrived while you worked.

**Record what you handled in `log.md`**, one line under the day's heading, and nowhere else. `log.md`
is already the wiki's chronological record (see *Open Knowledge Format* in `SKILL.md`), and the
question *"what has this inbox dealt with?"* gets asked at the start of every session. A tally in the
inbox `README.md` is the wrong answer — it was tried, and it needed hand-renumbering on every message
until it went stale.

### Replying

**Reply when the sender needs something back that only you can write** — a decision, a ruling on their
proposal, an answer they are waiting on. Write it into their inbox like any other message, with
`evidence` pointing at the note where the decision now lives, so they get something durable to cite
rather than a claim to copy.

**A reply is never itself replied to.** One round trip is the maximum, which is what keeps this from
becoming acknowledgement traffic — the failure mode a blanket "never reply" rule was originally
written to prevent. It prevented the useful half too: in one two-day exchange, five of six inbound
messages asked for a decision the sender could not make, and answering them was the only way the work
moved.

**When it is unclear whether an answer is wanted, reply.** An unwanted reply is one file the sender
deletes; a missing one leaves them blocked, and you cannot see that from here.
