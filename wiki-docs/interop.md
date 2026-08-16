# Other wikis: read-only mounts and messaging

Reference file for the `wiki-docs` skill. Read it when this project mounts another project's wiki, or
when you need to send a correction to one. `SKILL.md` beside this file covers the wiki itself.

**Which wikis are mounted, what each is authoritative for, and when to consult which is project
knowledge.** It belongs in the wiki's own `meta/interop.md`, not here. Read that first.

## What a mount is

A project may mount other projects' wikis as additional read-only foam servers, conventionally named
`foam-<project>`. Each mount adds 18 read tools; `--allow-writes` adds exactly 7 more, so a mount
without it has no write surface at all rather than an unused one.

`get_workspace_info` reports `root_dir` and `read_only`, so confirm which server you are talking to
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
4. **Wikilinks do not cross wikis.** See below.
5. **Two wikis that reached the same answer independently are evidence; two that disagree are an open
   question.** Record the disagreement — don't quietly pick a side. Independent corroboration is the
   main reason to keep the wikis separate at all.

## Cross-wiki references — unresolved

There is no good way to reference a note in another wiki. Foam, Obsidian and every other tool resolve
links **within** a workspace only, and a `<wiki>:<uri>` convention would be a scheme no tool
understands — a real cost, for a real gap. **Treat this as an open question in the project's wiki, not
as a settled convention.**

> **What not to do:** never write a cross-wiki reference as a `[[wikilink]]`. Foam registers any
> unresolved `[[…]]` as a placeholder, so it lands in `get_placeholders` permanently and poisons the
> one signal that means "note worth writing".

Until it is settled:

- **In front matter a prefixed string is already idiomatic** and low-risk, because nothing resolves
  `sources` entries anyway: `resource: homeit:implementation/hardware/electron-chassis.md`.
- **In prose, name the wiki and the note in plain text.** It doesn't resolve, but it doesn't pretend to.

Basenames collide across wikis — `index.md`, `meta/conventions.md` and `meta/scope.md` exist in every
one — so a cross-wiki reference must always carry the wiki's name, never a bare note stem.

If this becomes load-bearing it is the strongest argument for custom tooling over plain read-only
mounts: resolution, backlinks and link-checking across wikis are exactly what no existing tool provides.

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

**Outside the wiki root is deliberate, on two counts.** Foam never indexes the inbox, so messages
don't distort note counts, orphans or placeholders — and a message cannot quietly start being read as
a fact. The receiver has to *convert* it into a real note before it counts as knowledge.

**The inbox must not be gitignored.** An unhandled message is an untracked file, which is what makes
it appear in `git status` next time that repo is opened. Git is the notifier, not the transport:
nothing needs committing or pushing for a message to arrive, and foam-cli watches the filesystem, so
mounted wikis reflect each other's uncommitted edits live.

Filename `YYYY-MM-DD-<sender>-<slug>.md`:

```markdown
---
from: <sending wiki>
date: <YYYY-MM-DD>
kind: correction | contribution | verification-request | drift-notice
about: <uri in the receiving wiki, if it concerns an existing note>
evidence: <wiki>:<uri>          # where the reasoning lives — a pointer, never a copy
---

What is claimed, and what would settle it.
```

`kind` tells the receiver what is being asked. `correction` — the sender can disprove something you
record. `contribution` — the sender learned a fact your wiki owns. `verification-request` — the sender
depends on a claim of yours and wants it checked. `drift-notice` — the sender changed something that
invalidates a claim of yours.

**Resolving one:** fold it into a real note with its own `verified` entry, citing the message's
`evidence` in `sources`, then **delete the inbox file**. Deletion is the close, and the resulting note
is the record — don't build a `resolved/` archive, because the provenance belongs in the note.

No acknowledgement flows back, and none is needed: only one side ever writes each state transition. If
the sender wants to track that it asked, that is an entry in the sender's own open questions.
