# Walking a tree

Working out what is in a tree nobody has catalogued, so a catalogue can be written from it. **Read
`SKILL.md` beside this file first**: a walk produces answers in its units, relations and rungs. A
tree is walked once and kept afterwards; **a walk is started by a person, never volunteered.**

## What a walk says about each directory

Descend, and give each directory one of three answers:

- **The unit it is.** A checkout, a project's presence, a workspace, material.
- **A member of something else.** Its own docs defer to a parent.
- **Unknown.** You could not tell.

A fourth, **`aggregation`**, is a person's verdict and not a walk's to give. A directory that is not
a unit at all gets no answer here; it is named in its parent's entry with the reason.

Record answers as you go. The tree is large and the reasoning does not survive the session.

## Unknown is an answer

**Say `unknown` and move on.** Do not investigate or weigh signals; the resolution is somebody
else's. Three things to check before writing it, all cheap:

1. **The catalogue's local rules.** A rule that already answers it is a decision, not a guess.
2. **The rung.** A directory with no README sits at *Nothing*, where the catalogue is the only
   description, so naming the unit is an ordinary answer.
3. **The workspace signals** in `SKILL.md`, which say where a workspace is and nothing about
   membership.

## What the tree tells you, and what it does not

- **A `.git` directory marks a checkout.** Read `git remote -v` for its identities, all of them.
  Which one it principally is, is a judgement to record, never `origin` by default.
- **A nested `.git`, a submodule or a worktree is its own checkout**, contained by the one above and
  not a member of it unless something says so.
- **Follow a symlink once and record where it went.** Do not descend into the target twice.
- **Gitignored directories are still in the tree.** `git status` does not list them; the reverse
  diff below is what finds them.
- **Mounted or read-only paths** get a describes-elsewhere marker in the catalogue's repo, not a
  file at the target.
- **Commits "not on a remote-tracking ref" are not unpushed work.** The count includes stashes and
  stale refs.

## Markers change the descent

1. At each directory, look for an `atlas.md`.
2. With **no `authority`**, fold the marker into what you know and keep descending.
3. With an **`authority`**, record the delegation and stop. `self` means the region catalogues
   itself; a repository reference means the catalogue is there.
4. On a **describes-elsewhere** marker, record it against its stated target and treat the directory
   it sits in as if the file were not there.
5. With **no marker**, infer as usual. A tree with no markers must still be walkable.

## Closing the walk

**A walk does not verify itself.** Descending finds what it descends into, and what it missed
leaves no gap to report. So close by going the other way: diff a listing of the tree against the
finished catalogue, and account for every path on one side that is absent from the other. Check at
the same time that each document names the thing it sits in.

**Write an entry even where there is nothing to add.** *Looked and found nothing* and *nobody
looked* are different facts, and a catalogue that cannot tell them apart cannot be diffed.

**Hand over two lists**: what was classified, and what could not be. The second is a decision queue
for whoever owns the tree, and one question resolves most of it: what is this directory for?
