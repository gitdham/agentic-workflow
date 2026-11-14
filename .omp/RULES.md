# Rules for every agent in this workspace

These rules reach every session, including subagents, which receive no `AGENTS.md` automatically.

- Before working in a repository, read its `AGENTS.md` and its `.memory/MEMORY.md` if present. Before running shell commands, read the workspace `.memory/MEMORY.md`: it records this workstation's shell and tooling pitfalls.
- Run every shell command through `rtk proxy`: inspection, Git, dependency installs, dev servers, builds, tests. Proxy each command of a chain separately, or run the whole script through one proxied shell. Native read and edit tools need no proxy. If `rtk proxy` cannot run a required command, report the blocker; never fall back to a bare command.
- Commit only when your assignment or the repository policy requires it, and only inside the repository whose files you changed; never stage a nested repository's files in the root repository. Never push `products/paint-reactive-2` or `products/irkar-demo` unless the user asks.
- Memory stays inside this workspace: never write to global or user-level memory stores, never install a memory plugin or enable a memory backend. Edit `.memory/` only when assigned; propose a learning as one line `LEARNING: ...` in your report.
