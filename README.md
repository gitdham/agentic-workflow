# Agent Orchestration Workspace

This template is a workspace for coordinating agent-driven changes across one or more Git repositories.
It keeps orchestration guidance in the root while nested repositories retain their own history, guidance, and checks.

## Workflow

Work flows from **Main** to a task-scoped **planner**, then to implementation **workers**.
Main owns user liaison, routing, dependencies, writer coordination, escalation, and the final report.
The planner defines the contract and acceptance criteria, delegates workers, validates results, and manages corrections.
Workers edit assigned files and run assigned checks.

Only one writer may own a repository or working tree at a time, including across concurrent tasks.
Workers use explicit signals in their reports so the planner can distinguish success, failure, blockers, and questions.
Every shell command is run through `rtk proxy`; native file-reading and editing tools do not need shell wrapping.

## Use the template

1. Clone or copy this repository as your workspace root.
2. Place or clone repositories anywhere beneath the root.
3. Add every nested repository directory to the root repository's local `.git/info/exclude` file so it remains local:

   ```sh
   echo '/my-repo/' >> .git/info/exclude
   ```

4. Optionally create an untracked `LOCAL.md` containing machine-specific notes or a repository inventory for routing.
5. Point the agent runtime at root `AGENTS.md` and `.omp/agents/planner.md`.
6. Keep repository-specific guidance and optional memory inside each nested repository.

`rtk` is required on `PATH` because the workspace policy routes all shell commands through `rtk proxy`.
Read `AGENTS.md` for the complete role boundaries, delegation contract, signal meanings, acceptance loop, and memory policy.
