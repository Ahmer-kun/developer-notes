# Undoing Changes in Git: restore, reset, revert

Three commands, three different jobs. Confusion mostly comes from `reset` doing several things depending on flags.

## Quick map

| Situation | Command |
|---|---|
| Discard changes to a file in the working directory | `git restore file` |
| Unstage a file (keep the changes) | `git restore --staged file` |
| Undo last commit, keep the changes staged | `git reset --soft HEAD~1` |
| Undo last commit, keep the changes unstaged | `git reset HEAD~1` (same as `--mixed`) |
| Undo last commit and throw the changes away | `git reset --hard HEAD~1` |
| Undo a commit that's already pushed/shared | `git revert <commit>` |

## restore

```bash
git restore file.js
```

Overwrites `file.js` in the working directory with the version from the index (staged version, or last commit if nothing is staged). **Uncommitted changes to that file are gone** and can't be recovered by Git.

```bash
git restore --staged file.js
```

Removes the file from the staging area. The file itself keeps your edits.

```bash
git restore --source=HEAD~2 file.js
```

Gets the version of the file from two commits ago into your working directory.

## reset

`reset` moves the current branch to another commit. The flag decides what happens to the index and working directory.

| Flag | Branch pointer | Index (staged) | Working directory |
|---|---|---|---|
| `--soft` | moved | unchanged | unchanged |
| `--mixed` (default) | moved | reset to that commit | unchanged |
| `--hard` | moved | reset | **reset (changes lost)** |

```bash
git reset --soft HEAD~1    # e.g. redo the commit message or combine commits
git reset --hard origin/main   # make local branch identical to the remote one
```

`reset` rewrites history. Fine on local, unpushed commits. On shared branches it causes trouble.

## revert

```bash
git revert abc1234
```

Creates a **new commit** that applies the opposite of `abc1234`. History is preserved, so it's the safe choice for commits that other people already have.

Reverting a merge commit needs to know which parent to keep:

```bash
git revert -m 1 <merge-commit>
```

`-m 1` means "keep the first parent" (usually the branch you merged into).

## Common mistakes

- Running `git reset --hard` with uncommitted work. Those changes have never been in a commit, so reflog can't help.
- Using `reset` and then force-pushing a shared branch.
- Thinking `git checkout -- file` and `git restore file` differ. They do the same job here; `restore` is the newer, clearer command.
- Reverting a merge and later re-merging the same branch: Git considers those changes already applied. You may need to revert the revert.

## If you went too far

Committed work is usually recoverable: see [recovering lost work](git-recovering-lost-work.md).

## Remember

- `restore` = files. `reset` = move the branch. `revert` = new commit that undoes.
- `--hard` is the one that destroys uncommitted changes.
- Shared history: revert. Local history: reset is fine.

## Related

- [Git rebase](git-rebase.md)
- [Git cheatsheet](git-cheatsheet.md)

## References

- Pro Git book (git-scm.com/book), chapter on Reset Demystified
- `git help restore`, `git help reset`, `git help revert`
