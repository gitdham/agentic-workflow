# Multiwave delivery

Adapted from meta7 (r17x). Read this when a task runs the `medium` or `full` delivery profile.

The idea: research, design, review, and implementation are separate waves. Each wave produces an artifact, the next wave reads only that artifact, and agents hold every gate between waves. The user gives intent at the start and reviews results at the end, not in between.

## Profiles

The planner picks one per task and records it in the contract. A user instruction overrides the choice.

| Profile | Use when | Waves |
|---|---|---|
| `light` | small change, direction clear | contract, implementation, close (the protocol in `.omp/agents/planner.md`) |
| `medium` | new feature in an existing product | W4 → W5 → W6 → W7 |
| `full` | new product, new subsystem, or many unknowns | W1 → W7 |

## Waves

All artifacts go to `plans/<task>/` unless stated. The planner spawns every wave agent itself; wave agents do not spawn agents.

Every agent receives `.omp/RULES.md`, which tells it to read the target repository's `AGENTS.md` and memory; the planner still names the repository in each brief. W2 and W5 reviewers get only their input file as task material, so the information boundary holds.

| Wave | Agents | Reads only | Writes |
|---|---|---|---|
| W1 Explore | ≥3 `scout`, parallel, one per distinct open question | the intent, repo docs and memory, sources it finds | `w1-<slug>.md` (recorded by the planner from the scout's return) |
| W2 Cross-examine | one `reviewer` per W1 file, parallel | its one W1 file | `w2-<slug>.md` |
| W3 Research synthesis | planner | W1 and W2 files | `research.md` |
| W4 Design | planner | `research.md`, repo guidance | `contract.md` |
| W5 Pre-implementation review | ≥2 `reviewer`, parallel, one per review area | `contract.md` only | `w5-<area>.md` |
| W6 Implementation | one `worker` per slice, sequential within a repository | the slice brief | code, tests, local commit per accepted slice |
| W7 Close | planner and one worker | everything above | promoted docs, `REVIEW.md` packets, report |

### W1 Explore

Each scout gets one question and reports what is known (with source), unknown, and contested. Questions come from the intent, the repository docs, memory, and `INBOX.md` items for that repository.

### W2 Cross-examine

The reviewer labels every factual claim `VERIFIED` (primary source), `UNVERIFIED`, or `DISPUTED` (with the correction and source), lists coverage gaps, and gives `PASS` or `FAIL`. A pass with notes is a `FAIL`. On `FAIL` the planner re-runs that scout once with the gaps; a second `FAIL` carries the gaps into `research.md` as open questions.

### W3 Research synthesis

`research.md` groups findings by theme: findings with sources, open questions, constraints. Disputed claims appear only in corrected form.

### W4 Design

The contract from `.omp/agents/planner.md` (six questions), plus:

- **Laws**: invariants stated as "for every X, if P then Q", each mapped to a machine check in the product's own language (property-based test or equivalent). Bend proofs are deferred until work moves to a Linux host; do not run Bend on this workstation.
- **Surfaces**: for a product a person looks at or listens to, name each surface, the successful flow, its void states (empty, loading, partial, error, denied), and what it needs to exist.
- **Slices**: implementation units in dependency order, each with its acceptance.

### W5 Pre-implementation review

Each reviewer takes one area (for example interfaces, failure behavior, hardware boundary, test strategy) and asks: can a worker act on this without guessing? Findings are `BLOCKER`, `GAP`, `WEAK`, or `OK`. Any `BLOCKER` fails the gate:

- design contradiction → revise `contract.md`, re-run W5;
- missing knowledge → add W1 scouts for that gap only, then W2, W3, W4, W5.

At most two back-arcs per task. A third failure stops the run: the planner records a held decision with a recommendation for `INBOX.md` and reports `BLOCKED`.

### W6 Implementation

Slices run in contract order through the normal brief, worker, and acceptance loop of `.omp/agents/planner.md`, including its two-round correction limit. Law checks are part of each slice's acceptance. Each accepted slice is committed locally in the product repository; never pushed.

### W7 Close

A worker promotes `research.md` and `contract.md` into the product repository's `docs/` when they are worth keeping. The planner returns `REVIEW.md` packets, `optimistic` defaults, and `LEARNING:` lines with its verdict.

## Run log

`plans/<task>/run.log` is append-only; never rewrite a line. One line per event:

    <ISO-8601 time> <node> <status> <note>

- Nodes: `W1.<slug>`, `W2.<slug>`, `W3`, `W4`, `W5.<area>`, `W6.<slice>`, `W7`.
- Statuses: `pending`, `running`, `done`, `failed`, `blocked`, `decision`, `steer`.
- The last line for a node is its state.
- The planner appends on every spawn, return, decision, and steer.
- On resume, any node whose last line is `running` and has no live agent goes back to `pending` and is re-spawned.

## No questions inside a run

A planner on `medium` or `full` does not ask the user anything. Ambiguity → the conservative reading, logged as a `decision` line. Only decisions in the held category of `docs/review-flow.md` stop work, and only the work that depends on them; the rest of the run continues.

## Steering with follow-up notes

When the user sends a note about a running task, Main forwards it to that task's planner. The planner logs a `steer` line and re-runs from the earliest wave the note invalidates; waves before it are kept:

| Note is about | Re-run from |
|---|---|
| a new question or unknown area | W1, new scouts only |
| a fact or correction | W3 |
| a design decision or constraint | W4 |
| implementation only | W6, affected slices |

## Failure handling

A failed node is retried once with the same brief. The same failure twice → the node is `blocked` and escalated with evidence. A node is never marked done without its artifact.

## Pilot

`products/kobar` runs every task on the `full` profile while this branch is under trial. After the pilot, compare against earlier tasks: agents spawned, back-arcs, defects caught in W2 and W5, correction rounds in W6, and items that reached `REVIEW.md`.
