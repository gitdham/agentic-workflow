# Delegation and delivery protocol

Who owns what, how Main hands work to a planner, and how results are reported. Read it when routing
work, writing a hand-off, or reporting a result.

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
- Main MUST NOT dispatch a `worker` directly; even trivial changes go to a task-scoped `planner` (profile `light`).

### 2. Hand off to the planner

- Before dispatch, Main records each affected repository's `git rev-parse HEAD` as its base SHA using this minimal Git coordination fact and gives it to the planner. A new repository has no base SHA, so the planner reviews the relevant tree. For workspace-only work, use the root repository's base SHA.
- The planner's contract, briefs, acceptance, and correction loop are defined in `.omp/agents/planner.md`; the worker contract and `SIGNAL` values in `.omp/agents/worker.md`. Main reads them only to interpret a report.
- The planner escalates to Main after two failed rounds on one slice, on a disproved assumption that changes outcome or scope, and on decisions in the held category. Main handles those with the user.

### 3. Memory, plans, and reporting

- After planner acceptance, Main applies the Repository memory rules as a decision and accountability step. Main assigns any memory or dashboard mutation to an authorized worker; the planner validates meaningful changes. Main does not make these edits itself.
- Create dedicated acceptance-evidence artifacts only when genuinely required by repository policy or explicitly requested; otherwise include observed results in the final report. Main authorizes and scopes any required artifact; an assigned worker creates or updates it, while the planner coordinates and validates as appropriate. Do not alter unrelated or pre-existing artifacts.
- The planner's verdict is `ACCEPT`, `REJECT`, `BLOCKED`, or, after the design waves of `medium` and `full`, `HANDOFF`, with the report defined in `.omp/agents/planner.md`; a worker signal is not a verdict. On `HANDOFF`, Main assigns a fresh planner for implementation with the returned `contract.md` path. Main moves review packets and `optimistic` defaults into `REVIEW.md` through an assigned worker.
- The final report states what changed per affected repository or workspace area, which actor ran which checks and their actual results, any failed or corrected attempts, blockers and unverified work, and decisions needed. List worker-reported changed files without implying every file was personally inspected. Mention that planner-led validation is not independent design critique when relevant.
