# Wiring a project to this skill, and keeping it wired

This file is for the agent setting a project up to use `wiki-docs`, or bringing a project up to date
with a new release of it. The rest of the skill is for the agent doing wiki work.

**The problem it solves is drift in one direction only.** This skill moves; a project's `CLAUDE.md`
and `meta/` notes do not move with it. Every sentence a project copies out of here is correct on the
day it is written and unfalsifiable afterwards — it still reads as local law long after the mechanic
it describes has changed. So a project gets a **pointer that has no content to go stale**, and
everything else stays here.

## What a wired project has

1. A **marked block in `CLAUDE.md`** — the mandate to use this skill, and the rule against restating
   it. Fixed text, replaced wholesale, never edited in place.
2. A **guard hook**, for the sessions that will not read the mandate.
3. **`meta/` notes carrying choices only** — vocabularies, layout, the navigation axis, what belongs
   in the wiki. No mechanics.

## 1. The `CLAUDE.md` block

Paste this under whatever heading the project uses for the wiki. **Everything between the markers is
verbatim** — do not adapt it, do not add project detail inside it, do not summarise it. Project detail
goes above the markers, where the project owns it:

```markdown
## The wiki

`wiki/notes/` is this project's wiki. <!-- the root, and any other project fact, lives out here -->

<!-- wiki-docs:begin — managed by the wiki-docs skill; replace wholesale, never edit in place -->

**All wiki work goes through the `wiki-docs` skill** — reading, searching, editing, restructuring,
renaming, bulk operations, and any script or shell command that touches those files. Load it before
the first wiki file is opened, not after something looks wrong.

**The skill owns the mechanics** — tools and their flags, the wikilink and formatting hazards, health
checks, OKF, cross-wiki links. **Do not restate any of it in the files that say how to work this
wiki** — this one, the wiki's `meta/` notes, `index.md` and `log.md`. A local copy of a mechanic goes
stale silently and the next agent reads it as law. Local files carry this project's *choices*; for
everything else they point at the skill by section name.

**Nothing except the tool is evidence about the tool.** Not a `meta/` note, not your own system
prompt, not a habit from another project. Check `awt --help` before concluding it cannot do something.

<!-- wiki-docs:end -->
```

The markers are the whole mechanism: they make re-adoption a replacement rather than a judgement
call, and they make an in-place edit visible as one. **If you find the block edited, the edit is the
bug** — replace it and move whatever the edit was trying to say to the project-owned side.

**A formatter reflowing the block is not an edit.** The blank lines inside the markers are there
because a markdown formatter puts them there — `mdfmt` rewrites the tight form into this one and then
leaves it alone — so what is above is already a fixed point rather than something a project's own
formatting pass will fight. Wrap width still differs between projects, so **the rule is about the
words, not the bytes**: if whitespace is the only difference, nothing is wrong. If more than
whitespace differs, or you cannot tell, replace the block instead of adjudicating it — replacement is
wholesale and costs nothing, which is why it is the default and the comparison is only a shortcut.

## 2. The guard hook

The mandate above is in context every turn and still gets skipped — most reliably by a session that
never framed its work as *"editing the wiki"* at all, which is what a multi-day restructure looks like
from the inside. A `PreToolUse` hook catches those, because it fires on the path rather than on the
agent's framing.

**The script is managed, exactly like the block above: replaced wholesale, never edited in place.**
`WIKI_ROOT` is the one line a project owns. Nothing propagates a change to it — this is a file copied
into each project, with no version in it and nothing reading this skill at runtime — so a project
takes a fix only when step 5 runs, which is why step 5 replaces rather than inspects. A project that
genuinely needs different matching keeps its own and **says in the file that it is a deviation, and
why**, the same as step 3's third row.

`.claude/hooks/wiki-docs-guard.sh`, `chmod +x`:

```bash
#!/usr/bin/env bash
# Nudge an agent into the wiki-docs skill the first time it touches the wiki.
#
# MANAGED BY THE wiki-docs SKILL — DO NOT EDIT. A new release replaces this file wholesale,
# so an edit here is lost silently on the next adoption sweep, and until then it is a copy
# nothing propagates a fix to. Project-specific guarding belongs in a SEPARATE hook beside
# this one, registered as its own entry in .claude/settings.json: that keeps this file a
# straight copy the skill can overwrite with no merge and no judgement call. WIKI_ROOT below
# is the one line a project owns.
#
# Fails open by design: anything wrong here lets the tool call through.
set -u

WIKI_ROOT="wiki/notes"   # this project's notes: <rootDir>/notes, rootDir from awt.config.mjs

command -v jq >/dev/null 2>&1 || exit 0

input=$(cat)
tool=$(jq -r '.tool_name // ""' <<<"$input" 2>/dev/null)
session=$(jq -r '.session_id // "nosession"' <<<"$input" 2>/dev/null)

case "$tool" in
  Write|Edit|MultiEdit|NotebookEdit) target=$(jq -r '.tool_input.file_path // ""' <<<"$input" 2>/dev/null) ;;
  Bash)                              target=$(jq -r '.tool_input.command // ""'   <<<"$input" 2>/dev/null) ;;
  *) exit 0 ;;
esac

# Match WIKI_ROOT as a whole path, not a prefix: `docs/wiki-inbox/` is a sibling, and it is
# the one path in another repo the interop protocol says to write to. The trailing space folds
# "target ends with the root" into "root followed by whitespace".
case "$target " in
  *"$WIKI_ROOT"/*|*"$WIKI_ROOT"[[:space:]]*|*"$WIKI_ROOT"\"*|*"$WIKI_ROOT"\'*) ;;
  *) exit 0 ;;
esac

marker="${TMPDIR:-/tmp}/wiki-docs-guard-${session}"
[[ -e "$marker" ]] && exit 0
: > "$marker" 2>/dev/null || exit 0

jq -n '{
  hookSpecificOutput: {
    hookEventName: "PreToolUse",
    permissionDecision: "deny",
    permissionDecisionReason: "That path is the project wiki, and the wiki-docs skill governs it. Load the skill, then retry — this guard fires once per session and will not block you again."
  }
}'
exit 0
```

`.claude/settings.json` — project-level, so it is checked in and every session gets it:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write|Edit|MultiEdit|NotebookEdit|Bash",
        "hooks": [
          { "type": "command", "command": "${CLAUDE_PROJECT_DIR}/.claude/hooks/wiki-docs-guard.sh" }
        ]
      }
    ]
  }
}
```

What it does and does not do, because both matter:

- **`deny` is the mechanism, not the intent.** A PreToolUse hook's `permissionDecisionReason` is the
  one output the model reliably reads — `systemMessage` goes to the debug log. So the guard denies
  once to deliver a sentence, then writes a marker and stays out of the way. It is a nudge with teeth,
  not a lock: an agent that ignores the reason and retries gets through.
- **It covers `Bash` because the bypass is `Bash`.** A harness that tells an agent to prefer `sed` and
  heredocs over `Write` routes straight around a Write-only guard. The Bash check is a text test on
  the command, so it catches `cat > wiki/notes/x.md` and misses a script that builds the path at
  runtime.
- **The root is matched as a whole path, not a prefix, and that is not fussiness.** On the old layout
  `docs/wiki-inbox/` began with `docs/wiki`, so a substring test fired on the conventional inbox —
  the one path in another project's repo this protocol permits writing to. The denial is not the
  damage, since the guard fails open. **A false positive spends the session's only fire**, and the
  real wiki edit later in that same session then passes unguarded, silently. `wiki/notes` has no such
  sibling, and the rule stays because the next layout may. Anchoring on the root with a trailing
  slash would also stop a collision and would lose `awt fmt wiki/notes`, a genuine bulk write with
  nothing after the root.
- **`WIKI_ROOT` is the one copy of the layout outside `awt.config.mjs`, and it is a copy on purpose.**
  A shell hook cannot read a JavaScript config, so the path is restated here and nowhere else. It
  moves when `rootDir` moves, and §4 is where that happens.
- **It fails open at every step** — no `jq`, unreadable JSON, unwritable marker. A guard that blocks
  wiki work because it is itself broken is worse than no guard.
- **The marker is per session id, in `TMPDIR`.** Nothing to clean up, and nothing lands in the repo.

## 3. The sweep — after a release, after authoring, and on request

It is mechanical on purpose: the parts that get missed are the ones that need judgement, so this
converts as many of them as possible into a grep. Five things run it, and not all of them run all of
it:

| Trigger                                                                           | Steps         |
|-----------------------------------------------------------------------------------|---------------|
| A new version of the skill is installed                                           | All, then §4  |
| Asked whether the project is still in step with the skills it uses — any phrasing | All           |
| **You have written or rewritten a `meta/` note, or the `CLAUDE.md` block**        | 2, 3, 4, 6    |
| **Content has been extracted out of this wiki into a skill**                      | 3, 4, 6       |
| **Asked to deduplicate the wiki, or to resolve contradictions in it**             | 1, 2, 3, 4, 6 |

**One of these is a push, two are pulls, and two are things you did yourself.** A release landing
pushes the sweep at a project, and that is the only one with an event behind it. The other four have
none, which is why they were the ones missing.

**The pull side has no fixed wording, so match on intent.** Nobody asks for "the adoption sweep". They
ask whether `meta/` is still right, whether a note still holds, why two files disagree, or for the
duplication in here to be cleaned up. **Treat any request to reconcile the project's local files
against the skills it loads as this sweep**, however it is phrased, and do not wait for a release to
justify running it — a project drifts from a skill that never changed, because the project moved.

**The skills in scope are all of them, not this one.** A `meta/` note can copy or contradict any skill
the project loads, and the failure is identical in each: a local sentence that was right once, that
nothing updates, and that the next agent reads as law.

**The authoring trigger is the one that matters most, because it is the only one that runs while the
drift is being made.** The first two describe *discovering* drift that already exists. Authoring
creates it: the managed block is in context every turn saying not to restate the mechanics, and an
agent writing a set of notes applies that from memory, sentence by sentence, with nothing checking it.
In the run this trigger came out of, the user named restated mechanics in four separate messages
before they were gone, and the sweep afterwards still found one more — a `meta/` note stating a
cross-wiki rule that had already moved out of the note it pointed at. Authoring is nothing but
judgement, so it was exactly the moment this file did not cover.

**Deduplicating the wiki, or reconciling something that contradicts itself, *is* this sweep asked for
by another name** — and it is the trigger most easily missed, because nothing has changed and nothing
looks like adoption. Left to itself such a pass compares notes against each other, and that is the
wrong axis: the copy and the contradiction that matter are between a note and a file outside the
wiki, which a wiki-scoped comparison cannot see however carefully it is done. **Read the skills the
project loads before comparing any two notes**, starting with this one.

**A contradiction between a note and a skill is not a tie to break on the merits.** Step 3's table
decides it by subject, and it decides it whichever side is better written, longer, or more recently
touched — a `meta/` note is simply not an authority about a mechanic, and a skill is simply not an
authority about this project's choices. Resolving one on which text reads more convincingly is how the
stale copy wins, and the note it beat is the one that was right.

**Under the authoring trigger, step 2 is a verification rather than a replacement.** Check the block
still says what §1 says — the words, not the bytes, for the formatting reason given there. A session
that has just rewritten `CLAUDE.md` by hand is the likeliest one to have edited inside the markers,
which §1 calls the bug. Anything beyond whitespace, or any doubt, and you do the replacement.

Steps 1 and 5 belong to a release: you cannot diff against a version you never read, and the guard
hook's `WIKI_ROOT` only moves when the project's layout does.

1. **Read the skill first**, all of it: `SKILL.md`, plus `setup.md`, `interop.md`, `site.md` and this
   file where they apply. Not a skim for what changed — you cannot diff against a version you never
   read.

2. **Replace the marked block** in `CLAUDE.md` with the current one above, wholesale. If the project
   had no block, add it and say so.

3. **Sweep local files for restated mechanics.** From the project root:

   ```shell
   grep -rniE 'awt|mdfmt|remark|foam|wikilink|\[\[|okf|front.?matter|placeholder|orphan|dead.?end|split_by_heading|rename_tag|build_listing|allow-writes|workspace_info|quartz' \
     CLAUDE.md AGENTS.md README.md wiki/notes/index.md wiki/notes/log.md wiki/notes/meta/ wiki/site/README.md .claude/ 2>/dev/null
   ```

   **`index.md` and `log.md` are in the list because they are the gap the rule's wording left.** Both
   are OKF reserved names, so they exist in every wiki, and both carry prose *about* the wiki — a home
   page explaining what a health check reports satisfies "not in `meta/`" while breaking the point of
   it. Add any other file that tells an agent how to work here: a repo-root marker file, an
   `AGENTS.md`, whatever this project uses.

   **That pattern is this skill's vocabulary, and on either pull trigger it is too narrow.** List what
   the project actually loads and add each skill's own terms and filenames to it.

   Classify **every** hit as one of three things, and act:

   | Hit                                                                     | Do                                                      |
   |-------------------------------------------------------------------------|---------------------------------------------------------|
   | A **choice** — this project's vocabularies, layout, axis, own commands  | Keep                                                    |
   | A **mechanic** — a tool, a flag, a hazard, a format this skill defines  | Delete, leave a pointer by section name                 |
   | A **declared deviation** — this project genuinely differs               | Keep, and make it say *that it is a deviation, and why* |
   | **Subject matter** — a note whose topic *is* a tool this project builds | Keep                                                    |

   The last two are the ones that get mislabelled, in both directions. A project pinning an older
   toolbox is a deviation worth stating; a project describing how `awt` works because someone once
   found it useful is a copy. And a note telling you how to use a tool is a copy, while a note
   arguing what that tool should become is the project's own work — a wiki whose subject is its own
   tooling is full of the second kind.

4. **Sweep for mechanics that have gone stale.** Any local sentence naming a command, flag, tool or
   field that **no longer appears anywhere in this skill** is either a deviation (step 3, row three)
   or a leftover. Leftovers are the ones that do damage — they are confidently wrong rather than
   merely redundant.

   **One standing rewrite the test above cannot reach: a local file naming any formatter other than
   `awt fmt` as the command for *this wiki* is stale, and the replacement is `awt fmt`.** It escapes
   the test because the other name is still a live command elsewhere — `mdfmt` formats a README
   perfectly well and `markdown-remark` still documents it — so the hit reads as current rather than
   as a leftover. Rewrite it anyway, including where the note says both work. See *Markdown tooling*
   in `SKILL.md` for why identical output is not the point.

5. **Replace the guard hook** with the script in §2, wholesale, carrying this project's `WIKI_ROOT`
   across — then check it is executable and that `WIKI_ROOT` is `<rootDir>/notes` for the `rootDir`
   in `awt.config.mjs` (`wiki/notes` when unwritten). Add it if it is missing, and say so. If the
   project is still on the old layout, `WIKI_ROOT` is `docs/wiki` and §4 is the next thing to run.

   **Replacement rather than inspection, because this hook's failures are silent.** It fails open by
   design, so a stale one guards nothing and says nothing about it; there is no error to notice and
   no version to compare. Reading it to decide whether it looks current costs more than overwriting
   it. Leave it alone only where the file declares itself a deviation.

6. **Report, do not silently rewrite.** Deleting a restated mechanic is a documentation fix and needs
   no permission. **Changing a `meta/` vocabulary is a project decision** — front-matter values,
   folder layout, the navigation axis, what belongs in the wiki. Raise those with the user, with what
   you would change and why, and leave them alone until answered.

**Adoption is not a wiki edit.** It touches `CLAUDE.md`, `.claude/` and `meta/` prose — no notes are
created, renamed or moved, so nothing here needs the write tools. Run `awt check` afterwards anyway
if you touched `meta/`, since a deleted paragraph can take a `[[link]]` with it.

## 4. Moving a wiki to the home layout

**Run this once, on a project whose wiki is still `docs/wiki/` beside `site/`.** How to tell: `awt
check --json` from the project root reports a `notesDir` ending in `docs/wiki`, and the first `awt`
command of a session prints one line saying the project *is laid out the old way*. A project already
reporting `wiki/notes` is done and this section does not apply.

What the move is for is in *The layout* in `setup.md`: one home, named once, every other directory a
fixed name inside it, and no second copy of the path anywhere. The old shape keeps working
indefinitely — the toolbox reads it as legacy — so this is not urgent; it is worth doing because
every file that restated the path is one the sweep above has to police, and afterwards there is one.

**It is a wiki edit and a repo edit at once, and neither the write tools nor `mv` are the right tool
for the first half.** The notes move as a directory, with `git mv`, so their history follows and
every `[[wikilink]]` inside is untouched — links are relative to the notes root, and the root is what
moves. Nothing inside the notes changes.

Do it in this order, from the project root, on a clean working tree:

1. **Read what names the old paths.** Before moving anything:

   ```shell
   grep -rnE 'docs/wiki|\bsite/|\.remark|wiki-inbox' \
     awt.config.mjs .mcp.json CLAUDE.md README.md AGENTS.md bin/ .claude/ site/README.md \
     site/quartz.config.yaml docs/wiki/meta/ docs/wiki/index.md docs/wiki/log.md 2>/dev/null
   ```

   Every hit is either a path to rewrite in step 5 or prose to rewrite in step 6. Keep the list.

2. **Move the directories.** The four moves, in one commit later:

   ```shell
   mkdir -p wiki
   git mv docs/wiki       wiki/notes
   git mv site            wiki/site         # if the project has one
   git mv .remark         wiki/schemas      # if it has schemas
   git mv docs/wiki-inbox wiki/inbox        # if it exists — it may not, between messages
   ```

   `git mv` carries untracked files inside `site/` along — the Quartz clone, `public/`, the plugin
   symlinks — so nothing has to be rebuilt. The symlinks `awt bootstrap-quartz` made are absolute or
   relative within `site/` and survive the move; the one from the clone back to
   `quartz.config.yaml` is relative and survives too. Run `awt bootstrap-quartz` afterwards anyway
   if anything about the site looks off; it is idempotent.

3. **Rewrite `awt.config.mjs`.** Three things:

   - `rootDir` — leave it unwritten if the home is `./wiki`; write it only for a different home.
   - Schema paths — `./.remark/x.json` becomes `./wiki/schemas/x.json`.
   - **Schema globs — relative to the notes now.** `docs/wiki/meta/**/*.md` becomes `meta/**/*.md`.
     A glob still spelling out `wiki/notes/` matches nothing and says nothing, because the layout
     anchors globs to the notes directory; this is the one edit that fails silently if skipped.

   Then `awt fmt --check` from the project root and confirm the schemas fire: introduce a deliberate
   bad `status` in one note, see it reported, revert it.

4. **Shrink `.mcp.json`.** The entry becomes `["mcp", "--allow-writes"]` — no `--workspace`, no path.
   A session has to be restarted to pick that up; until then the running server still points at
   `docs/wiki`, which no longer exists, and every MCP tool call fails. Use `awt` from a shell for the
   rest of this section.

5. **Replace the guard hook with `WIKI_ROOT="wiki/notes"`** — §2's script wholesale, as step 5 of the
   sweep says, carrying the new root across. This is the one copy of the path that remains, and it
   moves here or it guards nothing.

6. **Rewrite the prose that named the old paths** — the list from step 1. `CLAUDE.md`'s line above
   the managed block, the README, `wiki/site/README.md`, `meta/interop.md`'s inbox declaration, any
   `bin/` wrapper that `cd`s into `site/`. The `meta/` notes are the ones to read carefully: a note
   that says *the schemas live in `.remark/`* was a choice when written and is now a stale path.
   Reduce each to the choice it records, or delete it and point at *The layout* in `setup.md`.

7. **Tell the wikis that depend on this one.** A registry in another project's
   `site/quartz.config.yaml` names this wiki's build output by relative path —
   `../../this-project/site/public/static/contentIndex.json` — and now has to say
   `../../../this-project/wiki/site/public/…` from its own new home, or `../../this-project/wiki/site/…`
   from an old one. The resolver warns rather than fails on a missing index, so a stale registry
   shows up as every cross-wiki link into this wiki going dead with a warning in that project's build
   log, not as an error. Send that project a message (see `interop.md`) or fix it if it is yours.

8. **Verify, then commit as one change.**

   ```shell
   awt check                         # from the project root: notesDir is wiki/notes, no legacy line
   awt fmt --check                   # formatting and the schemas, from the project root
   awt serve                         # if there is a site: it builds, and the shadow is clean
   git status                        # nothing untracked that used to be ignored
   ```

   Then the sweep's step 3 grep once more over the new paths — a `docs/wiki` that survives is either
   history, which is fine in `log.md`, or a stale instruction, which is not.

**What does not change:** the notes themselves, every wikilink, every rendered URL, the site's
`quartz.config.yaml` (its plugin sources and index path are relative to `site/`, which moved as a
unit), the pin, and the port. **What stays behind on purpose:** `.mcp.json`, `.claude/`, and the
managed block — the agent harness's files, at the repo root where the harness looks for them.
