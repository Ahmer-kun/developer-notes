# Git Cheatsheet

Commands worth remembering. For the "why", see the linked notes.

## Everyday

| Command | What it does | Example |
|---|---|---|
| `git status` | shows staged, unstaged, untracked | `git status -sb` (short + branch) |
| `git add` | stage changes | `git add -p` (choose hunks interactively) |
| `git commit` | record staged changes | `git commit -m "Fix login redirect"` |
| `git diff` | unstaged changes | `git diff --staged` for staged ones |
| `git log` | history | `git log --oneline --graph --all` |
| `git switch` | change branch | `git switch -c new-branch` to create |
| `git pull` | fetch + integrate | `git pull --rebase` |
| `git push` | send commits | `git push -u origin my-branch` sets upstream |
| `git fetch` | download without changing your branch | `git fetch --prune` |

## Inspecting

| Command | What it does | Example |
|---|---|---|
| `git show` | one commit's changes | `git show HEAD~1` |
| `git blame` | who last changed each line | `git blame -L 10,20 file.js` |
| `git log -p` | history with diffs | `git log -p -- path/to/file` |
| `git log -S` | commits that added/removed a string | `git log -S"parseToken"` |
| `git bisect` | binary search for the commit that broke something | `git bisect start`, then `good` / `bad` |

## Branching

| Command | What it does | Example |
|---|---|---|
| `git branch` | list / delete branches | `git branch -d old-branch` |
| `git merge` | merge another branch into the current one | `git merge feature` |
| `git rebase` | replay commits onto another base | see [rebase](git-rebase.md) |
| `git cherry-pick` | apply one commit onto the current branch | `git cherry-pick abc1234` |

`cherry-pick` copies the change as a new commit. Cherry-picking a lot between long-lived branches tends to cause duplicate-change conflicts later.

## Stash

| Command | What it does |
|---|---|
| `git stash` | shelve tracked changes |
| `git stash -u` | include untracked files |
| `git stash list` | see stashes |
| `git stash pop` | apply latest and remove it |
| `git stash apply` | apply, keep in list |

## Undo

| Command | What it does |
|---|---|
| `git restore file` | discard working directory changes to a file |
| `git restore --staged file` | unstage |
| `git reset --soft HEAD~1` | undo commit, keep staged |
| `git revert <sha>` | new commit that reverses one |
| `git reflog` | find where you were |

Details: [undoing changes](git-undoing-changes.md), [recovering lost work](git-recovering-lost-work.md).

## Merge conflicts

```
<<<<<<< HEAD
your side
=======
their side
>>>>>>> feature
```

1. Edit the file to the intended final content and delete the marker lines.
2. `git add file`
3. `git commit` (merge) or `git rebase --continue` (rebase).

`git merge --abort` / `git rebase --abort` returns to before the operation.

`git status` lists files with unresolved conflicts ("both modified").

## Config worth setting once

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main
git config --global pull.rebase true      # optional: pull with rebase by default
```

## Remember

- `git status` before doing anything scary.
- Commit or stash before experimenting.
- Never rewrite shared history.
