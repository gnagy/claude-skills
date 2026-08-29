# atlas

You have most likely arrived here from a `spec:` line in an `atlas.md`. This directory defines that
file's format, and the model behind it.

## Reading an `atlas.md`

An `atlas.md` describes the directory it sits in, for whoever has just landed there with no other
context. Three fields carry it:

- **`atlas:`** — what the directory is. One of `project`, `workspace`, `checkout`, `material`,
  `aggregation` (a container holding children it does not own), or `none` (not a unit at all).
- **`spec:`** — where the format is defined. The URL that brought you here, and the same in every
  marker.
- **`authority:`** — who owns the facts about this region. A clonable repository means the region is
  catalogued there; `self` means the thing catalogues itself; **absent means the enclosing authority
  still owns it**, which is the ordinary case.

Two more appear only when a marker describes somewhere else: `describes:` names the target path, and
`observed:` the date it was last looked at. **A marker with `describes:` says nothing about the
directory it sits in** — it is a pointer, not a description of its own neighbourhood.

If you are walking a tree: keep descending past a marker with no `authority`, and stop at one that
has it. Everything below an authority belongs to that authority.

The body under the front matter says what the directory is and what is directly below it, one level
deep. It never explains how to build anything — that belongs to the project's own documentation, and
the marker points at it.

**`marker.md` beside this file is the full specification**, including what a marker must never
contain and worked examples.

## What atlas is

A model for describing what is on disk, so that work can start without re-explaining it. It exists
for the questions that run past one repository: what is this directory, which project does this
checkout belong to, what else depends on it, what has to be cloned and installed before a task can
run at all.

A walk of the tree answers some of that. It is slow, part of it is guesswork someone has to confirm,
and the answer is thrown away at the end of the session — and the most useful facts are the ones
somebody decided rather than the ones a tool can read. A **catalogue** is where those get recorded:
markdown notes holding membership, placement and dependency, and nothing derivable from the files. An
`atlas.md` is how a directory declares its own place, so a catalogue does not have to infer it.

The model prescribes a small vocabulary and one file format, and stops there. It does not name your
catalogue, your repositories or your directories, and a project that adopts it keeps its own
documentation, its own conventions and its own build.

## What is in this directory

| File        | What it is                                                                |
|-------------|---------------------------------------------------------------------------|
| `SKILL.md`  | The model — units, relations, workspaces, and the workflows that use them |
| `marker.md` | The `atlas.md` specification, and how to walk a tree that has them        |

## Status

**Alpha, and honest about it.** The model was derived from reading real directory trees and has been
exercised once, against a single client tree. No catalogue exists yet, and no `atlas.md` has been
committed anywhere. The format has already changed once in response to that first walk, and it will
change again.

**If you are writing markers now, write few.** A field renamed after thirty of them exist is thirty
migrations, some in repositories you cannot write to.

## Using it as a Claude Code skill

`atlas` is a [Claude Code](https://claude.com/claude-code) skill in
[gnagy/claude-skills](https://github.com/gnagy/claude-skills). It is manual-only — it never loads
itself, and runs on `/atlas`. To install it from a clone of that repository:

```shell
npx skills add "$PWD" --skill atlas --agent claude-code --agent universal -g -y
```

Nothing else in this directory depends on Claude Code. `marker.md` is a file format any agent or
person can read, and the model in `SKILL.md` is prose.
