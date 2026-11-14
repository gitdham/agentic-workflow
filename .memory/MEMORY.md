# Workspace memory

Learnings about this workspace's own setup: roles, rules, and how the agent runtime loads them. Machine-specific and product-specific learnings do not belong here. Format: finding, conditions, evidence, implication.

## Subagents get `.omp/RULES.md` but never `AGENTS.md`

- **Finding:** a subagent started from a session receives the workspace `.omp/RULES.md` and its own agent definition, and no `AGENTS.md`. `RULES.md` is read when a session starts, so a session opened before the file existed does not carry it.
- **Conditions:** omp subagents (`worker`, `scout`, `reviewer`) started from a session whose cwd is the workspace root.
- **Evidence:** in a session started before `RULES.md` existed, a `worker` reported no shell-proxy rule and no `AGENTS.md`; in a new session the same question returned the rule quoted from the generic-rules block.
- **Implication:** put anything every agent must follow in `.omp/RULES.md` and anything role-specific in that role's agent file; restart the session after editing `RULES.md`.

## `scout` has no shell

- **Finding:** `scout` agents cannot run commands, so a research question that needs an experiment is answered from reading and web sources only.
- **Conditions:** W1 of a multiwave run.
- **Evidence:** a run log recorded "scout had no shell; did not run tests" for two of three scouts, and the planner wrote a probe itself to fill the gap.
- **Implication:** route questions that need an experiment to an agent with a shell, not to a scout, so the planner does not do the research it later designs from.

## Bundled `reviewer` follows the brief, not the intended boundary

- **Finding:** the bundled `reviewer` is a code-review agent; used for document review, it also opened and probed source code when the brief did not forbid it.
- **Conditions:** W2 and W5 reviews whose brief says "read only <file>".
- **Evidence:** the W5 interfaces reviewer in a multiwave run reported a probe of the current source, which exposed a defect the contract alone could not show.
- **Implication:** state the intended read boundary explicitly in each reviewer brief, including whether probing existing code is allowed.
