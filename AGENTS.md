# Agent orchestration workspace

This directory is an orchestration workspace containing the root repository and, optionally, repositories nested anywhere beneath the workspace root. The user gives high-level intentions. **Main is the active control-plane orchestrator/liaison, not a hands-on developer**: it owns user intent, routing, scope and ownership constraints, dependencies, single-writer coordination, escalation, decisions about control-artifact updates, and the final report. Main actively steers concurrent agent work and user communication rather than relaying every technical message. A task-scoped planner translates intent into an executable contract and technical plan, directly delegates implementation, and owns the correction and acceptance loop; workers implement, mutate assigned files, and run assigned checks.

## Layout

- `AGENTS.md`: workspace guidance.
- `INBOX.md`: decisions that need the user. It is not memory.
- `.memory/MEMORY.md`: root-level memory for cross-repository and workflow learnings.
- `.omp/agents/planner.md`: task-scoped planner definition.
- `plans/`: optional gitignored in-flight planner output.
- `LOCAL.md`: optional Git-excluded machine-specific notes and repository inventory. Main and planners read it for routing when present; never track it.

Nested repositories live wherever the user places them under the workspace root. Each is its own Git repository with its own `AGENTS.md` and is kept out of the root repository through local `.git/info/exclude` rules, not tracked configuration.

## Orchestrator protocol

Main is the active control-plane orchestrator/liaison, not a hands-on developer or pair programmer. It actively steers concurrent agent work and user communication.

Main may read `LOCAL.md`, `INBOX.md`, workspace and repository `AGENTS.md` and `.memory/MEMORY.md` files for routing; agent reports, transcripts, and task status; and only minimal Git coordination facts (`git rev-parse HEAD` and `git status --short`). It may assign and steer agents; own user intent, routing, outcome and ownership contracts, dependencies, single-writer coordination, and escalation; handle genuine user scope decisions; decide whether memory or dashboard updates are needed; and synthesize the final report.

By default, Main MUST NOT explore source or code to solve technical tasks, author the task-specific contract, implementation/design plan, or worker brief, edit repository files (including code, docs, plans, memory, or dashboards), install dependencies, run apps, services, migrations, tests, builds, linters, formatters, technical smoke checks, browser verification, or debugging, or stage or commit. Delegate planning and technical design to the planner; workers perform assigned repository mutations and checks; planners validate acceptance.

Only an explicit user request for a specifically named, non-development administrative control-plane decision or communication permits Main to perform that action directly. This narrow exception does not permit repository file mutation, technical execution, or broader action; all repository changes remain delegated.

1. At session start, Main reads `INBOX.md`, root guidance, relevant root memory, and `LOCAL.md` if present for routing. Main discovers affected repositories from the user's request, the filesystem, and `LOCAL.md` when present. It may also read agent reports, transcripts, task status, and the minimal Git facts above.
2. Assign a task-scoped planner when work needs planning. Keep that planner for the task's full delivery loop, not as a standing repository lead. The planner MUST read each affected repository's `AGENTS.md` first, then its `.memory/MEMORY.md` if present. Workspace-only tasks use root guidance. The planner authors the task-specific contract and design and directly delegates technical work.
3. Enforce the single-writer rule: only one implementation writer may own an affected repository or working tree at a time, including across separate tasks. The planner waits for the current worker to finish before dispatching a correction worker and does not run destructive checks while a writer is active. Never let two workers edit the same repository or working tree at once.
4. Report to the user what changed per affected repository or workspace area, who ran which checks and their results, what remains unverified, and any human decision needed. Main consumes worker and planner evidence; it does not execute or repeat technical checks.

## Rules

- Memory stays inside this workspace. NEVER write to global or user-level memory stores, and NEVER install a memory plugin or enable a memory backend.
- When a delegated executor is assigned or required to commit, it commits inside the affected repository's own Git repository. An executor changing files in a nested repository never stages or commits those files in the root repository. This does not authorize Main to stage or commit.

## Command Execution: Always Use RTK Proxy

- Run every shell command through `rtk proxy`. This includes inspection, Git, dependency installation, development servers, builds, tests, and automated verification commands.
- Do not run bare commands or silently bypass the proxy.
- For command chains, proxy each command separately, or run the entire script through a proxied shell.
- If `rtk proxy` is unavailable or cannot execute a required command, report the blocker and ask how to proceed instead of falling back to an unproxied command.
- Native file-reading and editing tools do not require a shell command.

Examples:

    rtk proxy git status --short
    rtk proxy ls -la
    rtk proxy npm test
    rtk proxy npm run build && rtk proxy npm test
    rtk proxy bash -lc 'npm run build && npm test'

The examples illustrate command wrapping; they do not prescribe a package manager or technology stack.

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
| Worker | `task` | `@task` role | implement a slice and perform assigned repository mutations/checks; choose internal HOW using repository conventions |

Main sets user intent and outcome/ownership constraints; the planner authors task-specific WHAT, WHY, scope, invariants, technical plan, and acceptance; the worker owns internal implementation choices. The planner does not implement: workers mutate files. Planner acceptance checks do not grant permission to edit implementation. Planner-led validation determines whether agreed outcomes are demonstrated; it is not independent design critique. Main consumes the evidence and does not perform edits or checks. Use an additional independent review only when specifically requested for a concrete need.

### 1. Understand intent

- Restate the user's goal as outcome, affected repositories or workspace areas, constraints, and observable completion. If readings materially change the outcome, escalate the scope question to the user; otherwise use the conservative reading and state it.
- Discover affected repositories from the user's request, the filesystem, and `LOCAL.md` when present. Read each affected repository's `.memory/MEMORY.md` if present. Note dependencies and sequence them; workspace-only work uses root guidance.
- Scope planners around work and its acceptance. One planner can cover several repositories; independent work MAY use separate planners, subject to single-writer coordination.
- Truly trivial changes with no design choice, such as a typo or rename with named targets, MAY bypass the planner only through direct Main-to-worker delegation with explicit outcome and acceptance checks. The worker executes and reports those checks; any correction is delegated to a fresh worker. Main uses reported evidence and never performs or repeats acceptance checks.

### 2. Establish the contract

- Before dispatch, Main records each affected repository's `git rev-parse HEAD` as its base SHA using this minimal Git coordination fact and gives it to the planner. A new repository has no base SHA, so the planner reviews the relevant tree. For workspace-only work, use the root repository's base SHA.
- The planner reads relevant guidance, source, tests, memory, and existing conventions. It maps the work through six questions: (1) things and states, (2) happy path, (3) failure behavior, (4) dependencies and ordering, (5) data boundaries, and (6) proof of behavior.
- The planner's contract covers outcome, WHY, scope and non-goals, invariants, necessary shared/public interfaces, acceptance cases and expected results, assumptions, and dependencies.
- Keep planner context for the full task. A plan is an intermediate contract, not a reason to stop when the assignment includes delivery.
- Do not require the planner to prescribe every internal type, helper, algorithm, retry policy, or exact implementation recipe. The worker chooses internal HOW using repository conventions while honoring the contract.

### 3. Brief and dispatch workers

- The planner writes and checks each brief, then directly calls `task`; Main does not relay implementation dispatches. Use a fresh worker for every slice and every correction attempt. Do not dispatch a second writer until the previous writer has actually finished.
- Pass the worker the relevant slice and WHY from the planner's contract. Every brief includes affected repository or workspace paths, files and non-goals, base SHAs, and these concise sections:
  - `# Target`: affected repositories or workspace areas, files with paths, and non-goals.
  - `# Why`: the design reason, grounded in the planner's map, so the worker can handle edge cases.
  - `# Change`: observable obligations and actual shared/public constraints, not a prescription for every internal detail.
  - `# Acceptance`: starts with one sentence describing observable correct behavior, then runnable required checks and expected results. Include at least one check that would fail if required behavior were missing.
  - `# Report`: signal, files changed, actual commands/results, blockers, and commit SHA if committed or `not committed`.
- Before dispatch, the planner checks that a worker can identify the outcome, WHY, boundaries, and observable acceptance without an open design question. The planner resolves technical ambiguity; Main handles genuine scope decisions.
- Shared worker context carries relevant workspace and repository `AGENTS.md` paths, the single-writer rule, worker contract, and requirements not to add placeholders, weaken or delete tests, or take unrequested scope.

**Worker contract (include in each worker context).**

- Implement the required outcome and acceptance. Choose internal implementation details using repository conventions.
- Do not change required outcomes, shared/public constraints, scope, or acceptance criteria without planner agreement. Report a contradiction or missing prerequisite rather than guessing.
- Run assigned local checks and report actual results. Do not claim an unrun check passed. Main never launches or executes a live technical smoke check; any required technical smoke check is assigned to a worker or planner according to the contract, and that actor reports its evidence.
- Do not weaken or delete tests, change data or constants to force a pass, add placeholders, or take unrequested scope.
- Report files changed and commit SHA if a commit was made, otherwise `not committed`; a commit is not required unless the task or repository policy requires it. These commit instructions apply only to delegated executors; Main must not stage or commit.
- The final message MUST begin with exactly one line `SIGNAL: <X>` as its first characters, where `<X>` is exactly one of the five signals in the table below; then give commands, exit codes and key output, changed files, blockers, and commit SHA or `not committed`.

| Signal | Meaning |
|---|---|
| `VERIFICATION_PASSED` | The worker implemented the slice and all checks assigned to that worker ran and passed. This informs the planner but is not a task acceptance verdict. |
| `VERIFICATION_FAILED` | The worker implemented the slice and assigned checks ran but still fail after attempted fixes. Include failing output. |
| `IMPLEMENTATION_COMPLETE` | The worker implemented the slice, but an assigned check could not be run. State which check and why; any required check still unrun blocks acceptance. |
| `IMPLEMENTATION_BLOCKED` | The worker cannot proceed, such as from a missing dependency or broken environment. State what was tried. |
| `NEEDS_COORDINATOR_INPUT` | A real ambiguity or contradiction prevents implementation. State the question and recommended default; the planner resolves technical contract issues and escalates scope decisions. |

### 4. Validate acceptance

- After the worker finishes, the planner uses the report to understand the delivered outcome and assigned-check results, then runs required acceptance and smoke checks against the contract. Worker claims alone are not a pass. Compare actual commands, exit codes, and output with expected results.
- Any required acceptance failure rejects the implementation. Any required check that is unrun or could not run blocks acceptance (`ACCEPTANCE_UNVERIFIED`), never `ACCEPT`.
- The planner runs the contract's required acceptance; do not automatically repeat every repository suite for each worker. Workers run only local checks assigned to them. Main consumes reported evidence and never executes or repeats checks.
- Run only agreed acceptance checks by default; do not add routine per-file or diff audits or other checks. Targeted source inspection is permitted to a planner or worker to diagnose a failed acceptance or concrete issue, or verify an explicit source-level or repository constraint in the contract; it is not a blanket gate. There is no mandatory separate validator, independent audit, risk-by-risk mutation package, or Main check rerun. Do not represent planner-led validation as independent review.

### 5. Correct and escalate

- On rejection, the planner gives a fresh worker the specific defect, expected behavior, and observed evidence. Keep outcome and acceptance criteria unchanged; the worker chooses the correction's internal HOW.
- A missing or invalid signal/report gets one request for a valid report and counts as a failed round. Each failed round on a slice is bounded; after two failed rounds on the same slice, stop delegating and escalate to Main with evidence and a recommended next action.
- If an assumption is disproved, record the evidence and impact. The planner MAY revise the contract to preserve user intent but must not weaken criteria merely to pass a candidate. Outcome or scope changes go to Main for user handling.
- A missing capability or unrun required acceptance is a blocker, not a pass. The planner reports the exact blocker and what remains unverified.

### 6. Memory, plans, and reporting

- After planner acceptance, Main applies the Repository memory rules as a decision and accountability step. Main assigns any memory or dashboard mutation to an authorized worker; the planner validates meaningful changes. Main does not make these edits itself.
- Create dedicated acceptance-evidence artifacts only when genuinely required by repository policy or explicitly requested; otherwise include observed results in the final report. Main authorizes and scopes any required artifact; an assigned worker creates or updates it, while the planner coordinates and validates as appropriate. Do not alter unrelated or pre-existing artifacts.
- The planner's verdict is `ACCEPT`, `REJECT`, or `BLOCKED`. `ACCEPT` requires every required acceptance check to pass and demonstrate the contract outcomes; a worker signal is not a verdict.
- The final report states what changed per affected repository or workspace area, which actor ran which checks and their actual results, any failed or corrected attempts, blockers and unverified work, and decisions needed. List worker-reported changed files without implying every file was personally inspected. Mention that planner-led validation is not independent design critique when relevant.
