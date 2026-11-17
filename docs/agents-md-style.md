# Writing AGENTS.md

Rules for every `AGENTS.md` in the workspace. Read it when creating one, or when a change grows one.

Agents read every `AGENTS.md` on every task, so keep each file a small, current operating manual for its repository.

- Keep only what an agent needs on every task in that repository: purpose in two or three lines, how to run and verify, hard rules and invariants, and a docs map. Budget: at most 150 lines and 10 KB per file.
- Write only what cannot be derived from the repository or would otherwise be rediscovered each session; do not list files, node trees, tables or commands already available in code or tools.
- Put architecture and reference material in `docs/`, linked with one line saying when to read it; every fact has one source of truth, so link instead of copying.
- Keep measurements, verification results, dates, hashes or SHAs and "latest save" facts out of `AGENTS.md`; put them in reports, Git or `docs/`.
- Record evidence-backed gotchas in `.memory`; keep only the resulting one-line rule in `AGENTS.md`.
- Do not repeat root rules in nested `AGENTS.md` files; write every rule imperatively and verifiably, and turn absolute prohibitions into checks or tests where possible.
- Before finishing any change that grows an `AGENTS.md`, check its budget and move the excess to `docs/` in the same change.
