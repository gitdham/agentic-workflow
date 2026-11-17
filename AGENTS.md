# Agent orchestration workspace

This directory is an orchestration workspace containing the root repository and, optionally, repositories nested anywhere beneath the workspace root. The user gives high-level intentions. **Main is the active control-plane orchestrator/liaison, not a hands-on developer**: it owns user intent, routing, scope and ownership constraints, dependencies, single-writer coordination, escalation, decisions about control-artifact updates, and the final report. Main actively steers concurrent agent work and user communication rather than relaying every technical message. A task-scoped planner translates intent into an executable contract and technical plan, directly delegates implementation, and owns the correction and acceptance loop; workers implement, mutate assigned files, and run assigned checks.

## Layout

- `AGENTS.md`: workspace guidance.
- `INBOX.md`: held decisions for the user; Git-ignored and untracked; see `## INBOX rules`.
- `REVIEW.md`: queue of work waiting for the user's own trial or veto; Git-ignored and untracked; see `## Human review capacity`.
- `docs/review-flow.md`: review packet format and class guidance; read when writing a contract or a `REVIEW.md` item.
- `docs/stacked-prs.md`: stacked pull request policy; read it before branching, stacking, or force-pushing a pull request branch.
- `docs/orchestrator.md`: the orchestrator protocol; read it when acting as Main.
- `docs/delegation.md`: delegation and delivery protocol; read it when routing work or handing a task to a planner.
- `docs/agents-md-style.md`: rules for writing and growing any `AGENTS.md`.
- `.memory/MEMORY.md`: root-level memory for cross-repository and workflow learnings.
- `.omp/RULES.md`: rules for every agent, loaded into every session including subagents (shell proxy, commits and pushes, memory).
- `.omp/agents/planner.md`: the planner and its full delivery protocol (contract, briefs, acceptance, correction, verdict).
- `.omp/agents/worker.md`: the worker, its contract, and the `SIGNAL` values.
- `plans/`: optional gitignored in-flight planner output; multiwave runs keep their artifacts and append-only `run.log` in `plans/<task>/`.
- `docs/multiwave.md`: delivery profiles and the multiwave pipeline; read when a task runs the `medium` or `full` profile.
- `LOCAL.md`: optional Git-excluded machine-specific notes and repository inventory. Main and planners read it for routing when present; never track it.

Nested repositories live wherever the user places them under the workspace root. Each is its own Git repository with its own `AGENTS.md` and is kept out of the root repository through local `.git/info/exclude` rules, not tracked configuration.

## INBOX rules

- `INBOX.md` holds only decisions held for the user. Each item is one question with options and a recommendation when known.
- Hold a decision only when it is in the held category of `docs/review-flow.md`. Every other decision is `optimistic`: take the recommendation and list it in `REVIEW.md`.
- Reports, status, history, verification detail, and items that merely wait on a user trigger do not belong in `INBOX.md`. Work waiting for the user's trial goes to `REVIEW.md`; the rest goes to the final report, a repository's `AGENTS.md` or docs, memory (only verified reusable learnings), or Git.
- Remove an item as soon as the user answers. If the answer matters later, record it in its proper place, not in `INBOX.md`.
- When nothing is held, `INBOX.md` says `No held decisions.`
- Keep items short; no long status lines. Facts must be evidence-backed; mark unverified ones.
- Only an assigned worker edits `INBOX.md`; Main decides the content. Single writer at a time.

## Human review capacity

The user's attention is the scarcest resource. Schedule work against the user's review queue, not against agent capacity.

- Every contract assigns each deliverable one review class, `machine`, `optimistic`, or `experience`, as defined in `docs/review-flow.md`.
- Limits: at most one unreviewed `experience` item per repository and two across the workspace. At a limit, Main MUST NOT dispatch work that builds on an unreviewed item or adds another `experience` item. It dispatches only independent `machine`/`optimistic` work, or work that makes a pending review cheaper (recordings, one-command launchers, checks derived from earlier user feedback). An explicit user instruction overrides a limit; Main states the queue first.
- Every `experience` item enters `REVIEW.md` as a review packet in the format of `docs/review-flow.md`. An item without a packet is not ready for the user.
- Order `REVIEW.md` by how much downstream work each item unblocks. At session start and in every final report, Main states the total estimated review minutes and the first item, not the whole list.
- Remove an item when the user responds. Feedback goes to a planner as a correction and, when it expresses a reusable preference, becomes a spec clause or automated check so the next round is caught before it reaches the user. A vetoed default is reverted.
- `REVIEW.md` follows the `INBOX.md` editing rule: an assigned worker edits, Main decides the content, single writer at a time.

## Orchestrator protocol

- Orchestrator protocol: see `docs/orchestrator.md`; read it when acting as Main.

## Rules for every agent

Shell proxy (`rtk proxy`), commit and push, and memory-store rules apply to every agent and live in `.omp/RULES.md`, which omp loads into every session, including subagents. Main follows them too.

- Do not stack pull requests by default; if a stack is unavoidable it is at most 3 PRs, registered as a native GitHub stack, and no agent force-pushes a pushed PR branch without the user's per-occasion approval; see `docs/stacked-prs.md`.

## Repository memory

Plain Markdown in `.memory/MEMORY.md` is shared across agents. Main owns memory policy and accountability: recall and reflection, and decisions about consolidation, promotion, and removal; Main does not edit memory. When an update is warranted, Main assigns the mutation to an authorized worker, and the planner validates meaningful memory changes. Planners and workers do not independently decide memory policy or make unassigned edits. A worker MAY propose a learning on one line, `LEARNING: ...`, in its report; Main decides whether to adopt it.

- Save only verified, non-obvious, reusable learnings a future session would otherwise rediscover: debugging discoveries, recurring pitfalls, failed approaches and why they failed, tooling quirks, hidden constraints, and integration gotchas. Verified means confirmed by observed execution or concrete reviewed evidence.
- Do not save task progress, check counts, build status, obvious or searchable repository facts, Git history, speculation, rules already in `AGENTS.md`, or architecture documented elsewhere. Never paste code that the repository already contains.
- Entry format: finding, applicable conditions, evidence, implication. Keep each entry short.
- Keep `MEMORY.md` small. Create topic files in `.memory/` only when a split is justified; link them from `MEMORY.md`.
- After planner acceptance of meaningful work, Main asks: "What did this task teach us that a future session would otherwise have to rediscover?" If nothing qualifies, no memory update is authorized. If something qualifies, Main decides and specifies the update, then delegates a worker to consolidate it with existing entries without duplicates; the planner validates the meaningful change.
- When Main decides a learning is mature enough to promote, or current evidence invalidates an entry, an authorized worker performs the move, update, or removal: a recurring workflow rule goes to `AGENTS.md`; an architecture decision goes to repository docs; a correctness invariant goes to a test or validation.
- Each nested repository MAY have its own `.memory/MEMORY.md`, committed in that repository by an assigned worker. Learnings specific to that repository go there. Cross-repository and workflow learnings go in the root `.memory/MEMORY.md`; Main owns the decision and an authorized worker performs any edit.

## Delegation and delivery protocol

- Delegation and delivery protocol: see `docs/delegation.md`; read it when routing work or handing a task to a planner.

## Writing AGENTS.md

- Writing `AGENTS.md`: see `docs/agents-md-style.md`; read it when creating, growing, or reviewing any `AGENTS.md`.
