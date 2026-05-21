# rpiv-mono — Agent Guidance

General guidance for AI agents working on the rpiv-mono fork (`hasundue/rpiv-mono`).

## Git Branches

### Remotes

| Remote | URL |
|---|---|
| `origin` | `https://github.com/hasundue/rpiv-mono.git` (fork) |
| `upstream` | `https://github.com/juicesharp/rpiv-mono.git` (canonical) |

### Main branch

`main` is a consumer branch. It tracks `origin/main` and may include breaking changes against upstream. Sync from upstream via rebase:

```
git pull --rebase upstream main
git push origin main
```

### Branch naming

Use a conventional prefix followed by a kebab-case description:

| Prefix | When to use |
|---|---|
| `feat-` | A new feature that can be PR'd to upstream |
| `fix-` | A bug fix that can be PR'd to upstream |
| `chore-` | Maintenance, refactoring, or tooling that can be PR'd to upstream |
| `fork-*` | Any of the above, but **not** mergeable to upstream (depends on unmerged work) |

A branch is non-mergeable (gets a prefix under `fork-`) **when it depends on another branch that is not yet in upstream/main**. Determine this mechanically:

```
# If the base of this branch has commits not in upstream/main, it gets fork-...
git merge-base --is-ancestor upstream/main <base-branch> || fork-relevant-prefix
```

This rule is transitive — if branch A depends on a `fork-*` branch, then A automatically qualifies as `fork-` too.

### Starting new work

Before creating a branch, choose the right base:

| Branch type | Base from | Reason |
|---|---|---|
| `feat-*`, `fix-*`, `chore-*` | `upstream/main` | Your work must be clean for upstream PRs; `origin/main` may contain fork-only breaking changes |
| `fork-*` | Its parent branch | Depends on unmerged work in the parent |

Create the branch and push it to origin with upstream tracking:

```bash
git fetch upstream main
git checkout -b feat-description upstream/main
git push -u origin feat-description
```

### Keep branches current

Rebase your branch to keep it current with upstream. Since rebase rewrites history, push with `--force-with-lease` to avoid accidentally overwriting others' work.

**Conventional branches** (`feat-*`, `fix-*`, `chore-*`) — rebase against upstream/main:

```bash
git fetch upstream main
git switch <branch-name>
git rebase upstream/main
git push --force-with-lease origin <branch-name>
```

**`fork-*` branches** — rebase against the parent branch instead:

```bash
git fetch origin <parent-branch>
git switch <branch-name>
git rebase <parent-branch>
git push --force-with-lease origin <branch-name>
```

If the parent is `main`, fetch from `upstream/main` just like a conventional branch.

### Branch lifecycle

- Add features and fixes atomically — one concern per branch, orthogonal when possible.
- Delete your branch once it has been merged upstream (or merged into `main` and confirmed working).
- Always try to keep branches mergeable to upstream. When that isn't possible, the `fork-` prefix distinguishes them clearly.
