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
| `feat/` | A new feature that can be PR'd to upstream |
| `fix/` | A bug fix that can be PR'd to upstream |
| `chore/` | Maintenance, refactoring, or tooling that can be PR'd to upstream |
| `fork/feat/` | A new feature **not** mergeable to upstream (depends on unmerged work) |
| `fork/fix/` | A bug fix **not** mergeable to upstream (depends on unmerged work) |
| `fork/chore/` | Maintenance, refactoring, or tooling **not** mergeable to upstream (depends on unmerged work) |

A branch is non-mergeable (gets a prefix under `fork/`) **when it depends on another branch that is not yet in upstream/main**. Determine this mechanically:

```
# If the base of this branch has commits not in upstream/main, it gets fork/...
git merge-base --is-ancestor upstream/main <base-branch> || fork/relevant-prefix
```

This rule is transitive — if branch A depends on `fork/feat/B`, then A automatically qualifies as `fork/` too.

### Keep branches current

Before starting work on a conventional branch (`feat/`, `fix/`, `chore/`), rebase it against upstream/main:

```
git rebase upstream/main
```

For `fork/*` branches, rebase against your parent branch instead (the branch it depends on):

```
git rebase <parent-branch>
```

If the parent is `main`, rebase against `upstream/main` just like a conventional branch.

### Branch lifecycle

- Add features and fixes atomically — one concern per branch, orthogonal when possible.
- Delete your branch once it has been merged upstream (or merged into `main` and confirmed working).
- Always try to keep branches mergeable to upstream. When that isn't possible, the `fork/*` prefix distinguishes them clearly.
