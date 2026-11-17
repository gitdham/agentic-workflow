# Stacked pull requests

Workspace standard for pull request chains whose branches depend on each other. Read it before slicing a task into branches, before stacking, and before any push that rewrites a pushed branch.

We merge with "Rebase and merge", which rewrites commit SHAs, so an unregistered chain breaks: the pull request above keeps a base that no longer exists, and repair attempts with merge commits and `-s ours` made it worse. GitHub's native stacked pull requests retarget and rebase the pull requests above a merged base on the server, for all three merge methods, and therefore remove that failure mode where they apply. That is why every chain must be registered as a stack.

## Default: do not stack

Slice work into independent slices: disjoint files, mergeable in any order. That is always cheaper than a stack.

Stack only on a real technical dependency, when a slice cannot be built or reviewed without the slice below it.

## If you stack

- Keep every branch in the same repository; stacks across forks are not supported.
- At most 3 pull requests per stack. More needs the user's approval: the limit is the user's review capacity, not tooling.
- One owning planner owns the whole stack, bottom to top. Never base a pull request on another planner's branch.
- Register every chain as a native GitHub stack: `gh stack submit` for new work, `gh stack link <bottom> ... <top>` for an existing chain. An unregistered chain of base branches is not allowed.

## Hygiene: keep every commit replayable

- No merge commit inside a pull request branch; synchronize with the trunk by rebase only. A merge commit disables GitHub's rebase-merge until the branch is rebased.
- Never `git merge -s ours` or `git merge -X ours` in a pull request branch. `-s ours` makes the pull request diff claim to delete files that main added, with no conflict and no warning.
- Use the GitHub "Update branch" button only in its "Update with rebase" mode. The default mode adds a merge commit.
- One logical change per commit, with a message that says what changed.
- No volatile numbers in docs: test counts, file counts, check counts. If unavoidable, they go in the last commit of the top branch only. Repeated edits to one number are what made stacked branches conflict.
- Leave `git rerere` enabled; `gh stack init` turns it on, and it reuses earlier conflict resolutions.

## Force pushes

There is no standing authorization. A force push, including `--force-with-lease`, to a branch that is already pushed and has a pull request is never automatic. The owning planner asks Main; Main asks the user once per occasion; the planner waits for the answer.

Before asking, prefer a procedure that needs no force push at all: when the bottom pull request of a registered stack merges, GitHub retargets and rebases the pull requests above it on the server, so no agent push is needed.

When a manual repair is unavoidable, report to Main the exact command, the branch, the expected current (old) remote SHA, and what the diff should look like afterwards, then wait.

Which `gh stack` commands push:

- `gh stack push`: per-branch `--force-with-lease` with atomic updates. Counts as a force push.
- `gh stack sync`: rebases, then pushes with `--force-with-lease`. Counts as a force push.
- `gh stack rebase`: fetches and rebases locally and does not push. No ask needed.
- The web "Rebase stack" button force-pushes every rebased branch on the server and produces unsigned commits. It is the user's action, never an agent's.

So `gh stack push` and `gh stack sync` need the per-occasion ask; `gh stack rebase` does not.

## Repair a chain that is unregistered, or a sync that aborts

1. `git fetch origin`
2. `git rebase --onto origin/<trunk> <old-tip-SHA-of-the-merged-base> <own-branch>`, with `--update-refs` when the stack is 3 high, so the branch in the middle moves too.
3. Push with `--force-with-lease` only after the ask of `## Force pushes` is answered.

Plain `git rebase origin/<trunk>` is enough after a merge commit or a rebase-merge of the base when nothing else changed. After a squash-merge, `--onto` is required. Never `--rebase-merges`: it leaves the old merge in place and does not restack.

Repairing by merge commit, by `-s ours`, or by opening a replacement pull request while a rebase is still possible is forbidden.

## Pull request body

Every pull request in a stack starts with this block:

    Stack: <n>/<total> (base: <branch>)
    Merge order: #a -> #b -> #c
    Depends on: #<pr>
    Owner: <task/planner>

GitHub shows its own stack map, so the block is the agent-readable fallback through `gh pr view`.

## How Main tracks stack state

Three read-only commands, nothing else:

- `gh pr list --repo <repo> --state open --json number,baseRefName,headRefName,mergeable`: the open chain, each base, and which pull requests conflict.
- `gh api repos/<owner>/<repo>/stacks`: registered stacks and their members. Each pull request also carries a `stack` object with `number`, `size`, `position`, and `base`.
- `git -C <repo> worktree list`: which worktree holds which branch, for single-writer enforcement.

## Pitfalls

- Run `gh stack rebase` or `gh stack sync` before asking the user to merge. After a manual force push, a stale stack can leave a pull request pointing at a merged branch, and `gh stack submit` does not repair that.
- Never force-push the bottom branch by hand while a stack is active.
- Do not check out one branch in two worktrees; rebase, sync, and modify refuse to run.
- Keep other worktrees clean and idle while a stack is rebased. The gh-stack lock coordinates gh-stack processes only, not other Git commands or editors.
- Prefer rebase or squash over a merge commit when merging a stack.

## Decision table

Native stacked pull requests are available on github.com; an older GitHub Enterprise Server repository may lack the feature.

| Case | Action |
|---|---|
| Native stacks available | `gh stack`; auto-rebase; manual repair last |
| Native stacks absent | Max 2 levels; independent slices; `--onto` repair |
| Open chain, unregistered | `gh stack link` after the height check |
