# Workspace memory

Cross-repository and workflow learnings. Format: finding, conditions, evidence, implication.

## `rtk proxy bash` on this workstation runs WSL

- **Finding:** On this Windows machine `rtk proxy bash` starts a shell inside WSL (Ubuntu), not a Windows shell.
- **Conditions:** any command routed through `rtk proxy bash`, including `bash -lc`.
- **Evidence:** a runaway WSL job (~7 cores; `vmmemWSL` ~60% of machine CPU) made the machine slow, and recovery required `wsl --shutdown`; the user forbids WSL for agent work for that resource cost.
- **Implication:** run Windows binaries through `rtk proxy cmd /c ...` or a `.ps1` via `rtk proxy powershell ... -File ...`; do not use WSL for agent commands.

## WSL-side git reports false CRLF modifications

- **Finding:** Git run from WSL over a Windows checkout reports whole-file CRLF changes that do not exist on disk.
- **Conditions:** repository checked out by Windows Git, inspected or staged from a WSL shell.
- **Evidence:** whole-file CRLF diffs with no content change; the same status from Windows Git is clean.
- **Implication:** always use the Windows Git through `rtk proxy git ...`.

## `rtk proxy cmd /c` mangles backslashes

- **Finding:** Backslashes inside a quoted command passed to `rtk proxy cmd /c` are altered or dropped.
- **Conditions:** Windows paths or `\`-heavy arguments inside the quoted command string.
- **Evidence:** quoted paths failed to resolve after proxying.
- **Implication:** use forward slashes, or put the logic in a `.bat`/`.ps1` and run the file.

## `rtk proxy powershell -Command` mangles `$` and `$env:`

- **Finding:** PowerShell interpolation, `$_`, and `$env:` are altered or dropped when passed through `rtk proxy` as an inline `-Command` string.
- **Conditions:** this workstation's `rtk proxy` wrapper with `powershell -Command "..."`.
- **Evidence:** a mangled `$env:TEMP` once created junk files in a repository root; `$env:LOCALAPPDATA` failed to expand.
- **Implication:** put logic in a `.ps1` and run `rtk proxy powershell -NoProfile -ExecutionPolicy Bypass -File <path>`; pass the `-File` path and any relative script path with forward slashes, because backslash paths are stripped.

## Default Python is a bare python.org 3.14

- **Finding:** `python` resolves to a per-user python.org Python 3.14.10 with no third-party packages; the Store `python3` alias is a stub, while KiCad, Espressif, and TouchDesigner each ship their own Python 3.11 with PIL/numpy.
- **Conditions:** running Python from a normal agent shell.
- **Evidence:** Pillow, numpy, and PyMuPDF imports fail in the default interpreter; the 3.11 bundles resolve them.
- **Implication:** do not assume third-party packages; ask before installing into a venv, and target a product's bundled interpreter only when it is the intended runtime.

## CPU readings need the right counters

- **Finding:** `Get-Process` and `\Process(*)` counters read 0 for protected processes and for `vmmemWSL`.
- **Conditions:** measuring machine or per-process CPU on this Windows host.
- **Evidence:** such counters stayed at 0 during the runaway WSL load while the machine was saturated.
- **Implication:** use `\Processor(_Total)\% Processor Time` for machine CPU and `Win32_PerfRawData_PerfProc_Process` deltas for per-process CPU, and state machine CPU next to every performance number.
