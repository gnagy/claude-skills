# Walking a tree

Working out what is in a tree nobody has catalogued, so a catalogue can be written from it. **Read
`SKILL.md` beside this file first** — the units, the relations and the self-description rungs are
what a walk produces answers in.

**This is the rarer job.** A tree is walked once and maintained afterwards; see *Managing a
catalogue* in `SKILL.md` for the job that recurs. A walk is also the expensive half of the skill, and
**it is started by a person, never volunteered.**

## Deciding what each directory is

Descend, and ask of each directory: is this a unit, and which one. A walk has three answers.

- **The unit it is.** A project, a workspace, a checkout, material.
- **A member of something else.** Its own docs defer to a parent.
- **Unknown.** You could not tell.

There is a fourth — **an `aggregation`**, a real container that does not own what is under it — and
it is not a walk's to give. See below. A directory that is not a unit at all has no answer here: it
gets named in its parent's entry instead, with the reason.

Record answers as you go, because the tree is large and you will not recover the reasoning later.

## Unknown is an answer, not a failure

**Say `unknown` and move on.** Do not investigate, do not weigh signals, do not reason your way to a
verdict. A directory a walk cannot classify is cheap to record and cheap to resolve later, and the
resolution is somebody else's — so the walk's job there is to name it and keep descending.

**One thing is worth checking first: whether the catalogue already answers it.** A catalogue extends
these rules for its own tree — see *A catalogue extends these rules* in `SKILL.md` — and those
additions are somebody's decision written down, so applying one is not guessing. Try the model's
rules, then the local ones, then write `unknown`.

**Saying what a directory does *not* own needs to know what it is *for*, and that is not on disk.** A
shelf of clones nobody owns and a workspace where those clones are read and patched are the same
listing, and they imply opposite things: a shelf can be deleted and re-cloned, work in progress
cannot. No amount of looking closer separates them, which is why looking closer is not the
instruction.

**Check the rung, though, before treating a bare directory as a puzzle.** A directory with no README
and no docs sits at the *Nothing* rung, where the catalogue is the only possible description — so it
is a unit nobody has described, and saying which unit is an ordinary answer, not a hard one.

## Hand the unknowns over

**A walk's output is two lists, and the second one is the point.** What it classified, and what it
could not. The second list is a decision queue for whoever owns the tree, and presenting it is how a
walk finishes — not as an apology for what it failed to work out, but as the part that needed a
person all along.

**One question resolves most of them: what is this directory for?** Ask it of whoever put the
directory there. That answer exists in one place, is not on any disk, and stops existing when they
forget.

## Markers change the descent

1. Descend. At each directory, look for an `atlas.md`.
2. With **no `authority`**, fold the marker into what you know about the region and keep descending.
3. With an **`authority`**, record the delegation and stop descending. Everything below belongs to
   that authority — `self` means the thing catalogues itself, a reference means clone it and read it.
4. On a **describes-elsewhere** marker, record it against the stated target and treat the directory it
   sits in as if the file were not there.
5. With **no marker**, infer as usual. Most directories will never have one, and a tree with no
   markers at all must still be walkable.

`marker.md` beside this file is the format itself.

## A walk does not verify itself

**Nothing in a walk's own results says the set is short.** Descending finds what it descends into,
and the things it misses — a gitignored directory, a path behind a symlink, a sibling nobody
mentioned — leave no gap for it to report. One real walk found four of six directories of working
data and read as complete.

**So close a walk by going the other way.** Diff the finished catalogue against a listing of the
tree, and account for every path on one side that is absent from the other. This is the step that
finds what descent could not, and the moment to check that each document names the thing it sits in
— a README about a different repository grades as documented and passes every check made on the way
down. *How far a project describes itself* in `SKILL.md` has what to do when one does not.

**Absence of an entry is the signal, so it has to mean something.** Write an entry even where there
is nothing to add beyond what the repository already says: *somebody looked and found nothing* and
*nobody has looked* are different facts, and a catalogue that cannot tell them apart cannot be
diffed against anything.

## Some derivations look complete and are wrong

*Derive, don't transcribe* says not to record what a tool can read. It does not say every reading is
sound, and a narrow one is worse than none, because it arrives with the confidence of a fact:

- **A checkout's first remote is not its identity.** A working copy can carry several, and track a
  branch belonging to one that is not its `origin`. Reading `origin` alone inverted a lineage on a
  real tree and made a backed-up repository look irreplaceable.
- **Commits "not on a remote-tracking ref" are not unpushed work.** The count picks up stashes and
  stale refs, overstating what exists on one disk only.

Where a derivation can be narrow in this way, say what you read it from, or record the answer as a
judgement someone made rather than as a fact.
