# Git Worktree Notes

## The command

```sh
git worktree add ../toydb-reference main
```

## What it does, piece by piece

- `git worktree` — subcommand for managing multiple **working directories** attached to one
  repository (one `.git` database, several checked-out folders).
- `add` — create a new working directory.
- `../toydb-reference` — where to put it: a sibling folder next to the current repo, e.g. if this
  repo lives at `~/dev/toydb-rust`, the new one appears at `~/dev/toydb-reference`.
- `main` — which branch/commit to check out into that new folder.

After running it, `~/dev/toydb-reference` contains a full checkout of `main` — every file, fully
buildable — sharing the same underlying object database (commits, blobs) as the original repo. No
data is duplicated on disk beyond the working files themselves; it's not a clone.

## Why use it instead of...

- **...just `git checkout main` in the same folder**: that would blow away your in-progress build
  branch's working files. Worktrees let both branches be checked out *simultaneously*, in two
  folders, so you can have your from-scratch build open in one editor window and the finished
  `main` code open in another, side by side.
- **...`git clone` into a second folder**: a clone duplicates the whole repo (all objects) and is
  a fully independent repo — fetches/pulls don't share state. A worktree stays linked to the
  original repo: one `git fetch` updates refs visible from both, and there's no duplicated object
  storage.

## Day-to-day use here

- Build branch (this repo, current folder): where you write your from-scratch implementation,
  stage by stage.
- Reference worktree (`../toydb-reference`, `main`): read-only in practice — check it *after*
  attempting a stage yourself, never before, so you're not just transcribing it.

Useful commands:

```sh
git worktree list          # show all worktrees and which branch each has checked out
git worktree remove ../toydb-reference   # clean up when no longer needed
```

Since `main` doesn't change, the reference worktree needs no maintenance — check it out once and
leave it.
