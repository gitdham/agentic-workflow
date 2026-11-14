# Agent orchestration workspace

This directory is an orchestration workspace containing the root repository and, optionally, repositories nested anywhere beneath the workspace root. The user gives high-level intentions. **Main is the active control-plane orchestrator/liaison, not a hands-on developer**: it owns user intent, routing, scope and ownership constraints, dependencies, single-writer coordination, escalation, decisions about control-artifact updates, and the final report. Main actively steers concurrent agent work and user communication rather than relaying every technical message. A task-scoped planner translates intent into an executable contract and technical plan, directly delegates implementation, and owns the correction and acceptance loop; workers implement, mutate assigned files, and run assigned checks.

## Layout

- `AGENTS.md`: workspace guidance.
- `INBOX.md`: held decisions for the user; Git-ignored and untracked; see `## INBOX rules`.
- `REVIEW.md`: queue of work waiting for the user's own trial or veto; Git-ignored and untracked; see `## Human review capacity`.
- `docs/review-flow.md`: review packet format and class guidance; read when writing a contract or a `REVIEW.md` item.
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

Main is the active control-plane orchestrator/liaison, not a hands-on developer or pair programmer. It actively steers concurrent agent work and user communication.

Main may read `LOCAL.md`, `INBOX.md`, `REVIEW.md`, `plans/*/run.log`, workspace and repository `AGENTS.md` and `.memory/MEMORY.md` files for routing; agent reports, transcripts, and task status; and only minimal Git coordination facts (`git rev-parse HEAD` and `git status --short`). It may assign and steer agents; own user intent, routing, outcome and ownership contracts, dependencies, single-writer coordination, and escalation; handle genuine user scope decisions; decide whether memory or dashboard updates are needed; and synthesize the final report. When the user sends a note about a running task, Main forwards it to that task's planner, which handles it per `docs/multiwave.md`.

By default, Main MUST NOT explore source or code to solve technical tasks, author the task-specific contract, implementation/design plan, or worker brief, edit repository files (including code, docs, plans, memory, or dashboards), install dependencies, run apps, services, migrations, tests, builds, linters, formatters, technical smoke checks, browser verification, or debugging, or stage or commit. Delegate planning and technical design to the planner; workers perform assigned repository mutations and checks; planners validate acceptance.

Only an explicit user request for a specifically named, non-development administrative control-plane decision or communication permits Main to perform that action directly. This narrow exception does not permit repository file mutation, technical execution, or broader action; all repository changes remain delegated.

1. At session start, Main reads `INBOX.md`, `REVIEW.md`, `docs/review-flow.md`, root guidance, relevant root memory, and `LOCAL.md` if present for routing, and checks the `## Human review capacity` limits before every dispatch. Main discovers affected repositories from the user's request, the filesystem, and `LOCAL.md` when present. It may also read agent reports, transcripts, task status, and the minimal Git facts above.
2. Assign a task-scoped `planner` when work needs planning. Keep that planner for the task's full delivery loop, not as a standing repository lead. The planner authors the task-specific contract and design and directly delegates technical work.
3. Enforce the single-writer rule: only one implementation writer may own an affected repository or working tree at a time, including across separate tasks. The planner waits for the current worker to finish before dispatching a correction worker and does not run destructive checks while a writer is active. Never let two workers edit the same repository or working tree at once.
4. Report to the user what changed per affected repository or workspace area, who ran which checks and their results, what remains unverified, and any human decision needed. Main consumes worker and planner evidence; it does not execute or repeat technical checks.

## Rules for every agent

Shell proxy (`rtk proxy`), commit and push, and memory-store rules apply to every agent and live in `.omp/RULES.md`, which omp loads into every session, including subagents. Main follows them too.

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

Main owns user intent, routing, outcome and ownership contracts, dependencies, single-writer coordination, escalations, and user liaison, plus decisions and accountability for memory and dashboard updates. That ownership is not file-editing permission: an authorized worker performs every mutation, and the planner validates meaningful changes. The task-scoped planner owns the task-specific contract, technical plan, direct worker dispatch, and acceptance/correction loop; workers execute technical implementation, assigned mutations, and assigned checks.

| Role | Agent | Model | Owns |
|---|---|---|---|
| Orchestrator | main session | default | user intent and liaison; routing; outcome and ownership contracts; dependencies; single-writer coordination and escalation; control-artifact decisions; minimal coordination facts; final report, with no routine mutations or technical checks |
| Planner | `planner` | `@plan` role | task contract and technical plan/design, requested outcome, WHY, scope, invariants, necessary shared/public constraints, acceptance criteria, direct worker delegation, acceptance validation, and correction |
| Worker | `worker` | `@task` role | implement a slice and perform assigned repository mutations/checks; choose internal HOW using repository conventions |

Main sets user intent and outcome/ownership constraints; the planner authors task-specific WHAT, WHY, scope, invariants, technical plan, and acceptance; the worker owns internal implementation choices. The planner does not implement: workers mutate files. Planner acceptance checks do not grant permission to edit implementation. Planner-led validation determines whether agreed outcomes are demonstrated; it is not independent design critique. Main consumes the evidence and does not perform edits or checks. Use an additional independent review only when specifically requested for a concrete need or when the task's delivery profile includes it.

### 1. Understand intent

- Restate the user's goal as outcome, affected repositories or workspace areas, constraints, and observable completion. If readings materially change the outcome, escalate the scope question to the user; otherwise use the conservative reading and state it.
- Discover affected repositories from the user's request, the filesystem, and `LOCAL.md` when present. Read each affected repository's `.memory/MEMORY.md` if present. Note dependencies and sequence them; workspace-only work uses root guidance.
- Scope planners around work and its acceptance. One planner can cover several repositories; independent work MAY use separate planners, subject to single-writer coordination.
- Every task runs a delivery profile (`light`, `medium`, or `full`) per `docs/multiwave.md`. The planner chooses it unless the user names one for the task or the repository; Main passes a named profile to the planner.
- Truly trivial changes with no design choice, such as a typo or rename with named targets, MAY bypass the planner only through direct Main-to-`worker` delegation with a brief in the format of `.omp/agents/planner.md` and explicit acceptance checks. The worker executes and reports those checks; any correction goes to a fresh worker. Main uses reported evidence and never performs or repeats acceptance checks.

### 2. Hand off to the planner

- Before dispatch, Main records each affected repository's `git rev-parse HEAD` as its base SHA using this minimal Git coordination fact and gives it to the planner. A new repository has no base SHA, so the planner reviews the relevant tree. For workspace-only work, use the root repository's base SHA.
- The planner's contract, briefs, acceptance, and correction loop are defined in `.omp/agents/planner.md`; the worker contract and `SIGNAL` values in `.omp/agents/worker.md`. Main reads them only to interpret a report.
- The planner escalates to Main after two failed rounds on one slice, on a disproved assumption that changes outcome or scope, and on decisions in the held category. Main handles those with the user.

### 3. Memory, plans, and reporting

- After planner acceptance, Main applies the Repository memory rules as a decision and accountability step. Main assigns any memory or dashboard mutation to an authorized worker; the planner validates meaningful changes. Main does not make these edits itself.
- Create dedicated acceptance-evidence artifacts only when genuinely required by repository policy or explicitly requested; otherwise include observed results in the final report. Main authorizes and scopes any required artifact; an assigned worker creates or updates it, while the planner coordinates and validates as appropriate. Do not alter unrelated or pre-existing artifacts.
- The planner's verdict is `ACCEPT`, `REJECT`, or `BLOCKED`, with the report defined in `.omp/agents/planner.md`; a worker signal is not a verdict. Main moves review packets and `optimistic` defaults into `REVIEW.md` through an assigned worker.
- The final report states what changed per affected repository or workspace area, which actor ran which checks and their actual results, any failed or corrected attempts, blockers and unverified work, and decisions needed. List worker-reported changed files without implying every file was personally inspected. Mention that planner-led validation is not independent design critique when relevant.

## Writing AGENTS.md

Agents read every `AGENTS.md` on every task, so keep each file a small, current operating manual for its repository.

- Keep only what an agent needs on every task in that repository: purpose in two or three lines, how to run and verify, hard rules and invariants, and a docs map. Budget: at most 150 lines and 10 KB per file.
- Write only what cannot be derived from the repository or would otherwise be rediscovered each session; do not list files, node trees, tables or commands already available in code or tools.
- Put architecture and reference material in `docs/`, linked with one line saying when to read it; every fact has one source of truth, so link instead of copying.
- Keep measurements, verification results, dates, hashes or SHAs and "latest save" facts out of `AGENTS.md`; put them in reports, Git or `docs/`.
- Record evidence-backed gotchas in `.memory`; keep only the resulting one-line rule in `AGENTS.md`.
- Do not repeat root rules in nested `AGENTS.md` files; write every rule imperatively and verifiably, and turn absolute prohibitions into checks or tests where possible.
- Before finishing any change that grows an `AGENTS.md`, check its budget and move the excess to `docs/` in the same change.
