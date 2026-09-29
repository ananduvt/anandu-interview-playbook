# Git

## Common commands
`init` · `clone` · `status` · `add` · `diff` · `commit` · `reset` · `fetch` · `merge` · `push` · `pull` · `rebase` · `stash`

## Pull vs Rebase
- **`git pull`** — fetch + **merge**; creates a merge commit; preserves history. Safe for shared branches.
- **`git rebase`** — fetch + **reapply** your commits on top; **linear** history, new commit IDs.
  ```sh
  git fetch origin && git rebase origin/main
  ```
- **`git pull --rebase`** — shortcut (fetch + rebase).
- Rule: rebase **private/feature** branches to clean history; **never rebase** shared branches already pushed.

## Merge vs Rebase
| | `git merge` | `git rebase` |
|--|-------------|--------------|
| History | non-linear, merge commit | linear, rewritten |
| Commit IDs | preserved | new |
| Conflicts | resolved once | possibly per commit |
| Safety | safe on public branches | risky (needs force-push) |
| Use | integrate shared branches | clean a feature branch before merge |

## File rename
Git detects renames by content similarity. Use `git mv old new` (stages delete + add); or `git add new && git rm old`.

## Useful
- `git commit --amend --author="Name <email>"` — fix last commit's author.
- `git stash` / `git stash pop` — shelve WIP.
- `git cherry-pick <sha>` — apply a specific commit.
- `git reset --soft/--mixed/--hard` — move HEAD (hard discards changes — careful).
