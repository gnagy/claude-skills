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

`docs/wiki/` is this project's wiki. <!-- the root, and any other project fact, lives out here -->

<!-- wiki-docs:begin — managed by the wiki-docs skill; replace wholesale, never edit in place -->
**All wiki work goes through the `wiki-docs` skill** — reading, searching, editing, restructuring,
renaming, bulk operations, and any script or shell command that touches those files. Load it before
the first wiki file is opened, not after something looks wrong.

**The skill owns the mechanics** — tools and their flags, the wikilink and formatting hazards, health
checks, OKF, cross-wiki links. **Do not restate any of it here or in the wiki's `meta/` notes.** A
local copy of a mechanic goes stale silently and the next agent reads it as law. Local files carry
this project's *choices*; for everything else they point at the skill by section name.

**Nothing except the tool is evidence about the tool.** Not a `meta/` note, not your own system
prompt, not a habit from another project. Check `awt --help` before concluding it cannot do something.
<!-- wiki-docs:end -->
```

The markers are the whole mechanism: they make re-adoption a replacement rather than a judgement
call, and they make an in-place edit visible as one. **If you find the block edited, the edit is the
bug** — replace it and move whatever the edit was trying to say to the project-owned side.

## 2. The guard hook

The mandate above is in context every turn and still gets skipped — most reliably by a session that
never framed its work as *"editing the wiki"* at all, which is what a multi-day restructure looks like
from the inside. A `PreToolUse` hook catches those, because it fires on the path rather than on the
agent's framing.

`.claude/hooks/wiki-docs-guard.sh`, `chmod +x`:

```bash
#!/usr/bin/env bash
# Nudge an agent into the wiki-docs skill the first time it touches the wiki.
# Fails open by design: anything wrong here lets the tool call through.
set -u

WIKI_ROOT="docs/wiki"   # this project's choice — keep it in step with .mcp.json

command -v jq >/dev/null 2>&1 || exit 0

input=$(cat)
tool=$(jq -r '.tool_name // ""' <<<"$input" 2>/dev/null)
session=$(jq -r '.session_id // "nosession"' <<<"$input" 2>/dev/null)

case "$tool" in
  Write|Edit|MultiEdit|NotebookEdit) target=$(jq -r '.tool_input.file_path // ""' <<<"$input" 2>/dev/null) ;;
  Bash)                              target=$(jq -r '.tool_input.command // ""'   <<<"$input" 2>/dev/null) ;;
  *) exit 0 ;;
esac

[[ "$target" == *"$WIKI_ROOT"* ]] || exit 0

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
  heredocs over `Write` routes straight around a Write-only guard. The Bash check is a substring test
  on the command, so it catches `cat > docs/wiki/x.md` and misses a script that builds the path at
  runtime.
- **It fails open at every step** — no `jq`, unreadable JSON, unwritable marker. A guard that blocks
  wiki work because it is itself broken is worse than no guard.
- **The marker is per session id, in `TMPDIR`.** Nothing to clean up, and nothing lands in the repo.

## 3. Adopting a new release

Run this when a new version of the skill is installed, or when asked to bring the project up to date
with it. It is mechanical on purpose — the parts that get missed are the ones that need judgement, so
this converts as many of them as possible into a grep.

1. **Read the skill first**, all of it: `SKILL.md`, plus `setup.md`, `interop.md`, `site.md` and this
   file where they apply. Not a skim for what changed — you cannot diff against a version you never
   read.

2. **Replace the marked block** in `CLAUDE.md` with the current one above, wholesale. If the project
   had no block, add it and say so.

3. **Sweep local files for restated mechanics.** From the project root:

   ```shell
   grep -rniE 'awt|mdfmt|remark|foam|wikilink|\[\[|okf|front.?matter|placeholder|orphan|dead.?end|split_by_heading|rename_tag|build_listing|allow-writes|workspace_info|quartz' \
     CLAUDE.md docs/wiki/meta/ .claude/ 2>/dev/null
   ```

   Classify **every** hit as one of three things, and act:

   | Hit                                                                    | Do                                                      |
   |------------------------------------------------------------------------|---------------------------------------------------------|
   | A **choice** — this project's vocabularies, layout, axis, own commands | Keep                                                    |
   | A **mechanic** — a tool, a flag, a hazard, a format this skill defines | Delete, leave a pointer by section name                 |
   | A **declared deviation** — this project genuinely differs              | Keep, and make it say *that it is a deviation, and why* |

   The third is the one that gets mislabelled in both directions. A project pinning an older toolbox
   is a deviation worth stating; a project describing how `awt` works because someone once found it
   useful is a copy.

4. **Sweep for mechanics that have gone stale.** Any local sentence naming a command, flag, tool or
   field that **no longer appears anywhere in this skill** is either a deviation (step 3, row three)
   or a leftover. Leftovers are the ones that do damage — they are confidently wrong rather than
   merely redundant.

5. **Check the guard hook** exists, is executable, and that its `WIKI_ROOT` still matches
   `.mcp.json`.

6. **Report, do not silently rewrite.** Deleting a restated mechanic is a documentation fix and needs
   no permission. **Changing a `meta/` vocabulary is a project decision** — front-matter values,
   folder layout, the navigation axis, what belongs in the wiki. Raise those with the user, with what
   you would change and why, and leave them alone until answered.

**Adoption is not a wiki edit.** It touches `CLAUDE.md`, `.claude/` and `meta/` prose — no notes are
created, renamed or moved, so nothing here needs the write tools. Run `awt check` afterwards anyway
if you touched `meta/`, since a deleted paragraph can take a `[[link]]` with it.
