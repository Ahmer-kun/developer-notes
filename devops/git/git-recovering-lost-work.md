# Recovering Lost Work in Git

If something was ever **committed** (or stashed), it's very often still recoverable. Uncommitted, never-staged changes are the exception.

## First: don't panic, don't run gc

Git doesn't delete unreachable commits right away. They stay in the object database until garbage collection removes them, which by default happens well after the reflog entries expire.

## reflog

The reflog records where `HEAD` (and each branch) pointed over time, locally.

```bash
git reflog
```

Sample output shape:

```
a1b2c3d HEAD@{0}: reset: moving to HEAD~3
9f8e7d6 HEAD@{1}: commit: add validation
5c4b3a2 HEAD@{2}: commit: add form
```

Find the entry from before the mistake, then:

```bash
git branch rescue 9f8e7d6      # safest: makes a branch pointing there
# or
git reset --hard 9f8e7d6       # move the current branch back (discards current working changes)
```

Making a `rescue` branch first is low-risk: you can inspect it and then decide.

Reflog is **local to your clone** and entries expire (by default around 90 days for reachable entries and 30 days for unreachable ones; configurable).

## Common cases

| What happened | Try |
|---|---|
| `git reset --hard` removed commits | `git reflog`, then branch/reset to the old commit |
| Bad rebase | `git reflog`, or `git reset --hard ORIG_HEAD` right after |
| Deleted a branch | `git reflog` for its last commit, `git branch name <sha>` |
| Amended commit lost content | old version is in reflog |
| Dropped a stash | `git fsck --unreachable` then look for commit objects, or check terminal output for the hash |
| Detached HEAD, switched away | reflog shows the commit; make a branch |

Recovering a dropped stash: `git fsck --no-reflog | grep commit` lists dangling commits; inspect them with `git show <sha>` and apply with `git stash apply <sha>`. (Works when the objects haven't been garbage-collected.)

## What can't be recovered

- Changes that were never added or committed and were removed by `git restore`, `git checkout -- file`, or `git reset --hard`. Git never stored them. Check editor local history or backups.
- Untracked files deleted by `git clean -fd`.

Habit: before risky operations, commit or stash. A `wip` commit is cheap and can be cleaned up later.

## Common mistakes

- Force-pushing before checking what's on the remote.
- Assuming a commit is gone because it no longer shows in `git log`. `log` shows what's reachable from the current branch only.
- Running `git gc --prune=now` or clearing reflog manually while trying to "clean up".

## Practical notes

- `git log --all --oneline --graph` shows all branches, sometimes enough to find "lost" commits still on another ref.
- `git fsck --lost-found` finds dangling objects.
- On the remote side, hosting platforms may keep old refs a while, but don't count on it.

## Remember

- Committed = almost always recoverable. Uncommitted = maybe not.
- `git reflog` first; make a branch before resetting.
- Local reflog only.

## Related

- [Undoing changes](git-undoing-changes.md)
- [Git rebase](git-rebase.md)

## References

- Pro Git: Maintenance and Data Recovery
- `git help reflog`, `git help fsck`
