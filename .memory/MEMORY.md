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

## Web Server DAT with blank `localaddress` binds `0.0.0.0`

- **Finding:** TouchDesigner's Web Server DAT listens on all interfaces when `localaddress` is blank.
- **Conditions:** the touchdesigner-mcp bridge (`mpc_webserver`, port 9981) on TD 2025.33070.
- **Evidence:** `netstat` showed `0.0.0.0:9981` owned by `TouchDesigner.exe`; the bridge has no auth, no Origin/Host check, and its exec endpoint runs arbitrary Python. Upgrading the bridge to 1.6.1 does not fix this.
- **Implication:** keep the bridge loopback-only: the project-owned Execute DAT `/project1/mcp_loopback_guard` in `products/paint-reactive-2` sets `localaddress = '127.0.0.1'` (its `onCreate` runs before the bridge's first bind, so there is no `0.0.0.0` window), and the firewall rule `Block TD MCP inbound` (TCP 9981, 9982) is the backstop.

## Pulsing Web Server DAT `restart` inside an MCP request locks up TD's HTTP handling

- **Finding:** a `restart` pulse issued through the MCP bridge leaves TouchDesigner's HTTP handling hung.
- **Conditions:** pulsing the `restart` parameter of the Web Server DAT while serving an MCP request.
- **Evidence:** TD stopped answering and had to be force-killed.
- **Implication:** never pulse `restart` via MCP; setting `localaddress` alone rebinds.

## TouchDesigner default saves increment and maintain a link file

- **Finding:** With incrementing/link saves enabled, path-less `project.save()` writes a numbered `.N.toe`, updates the un-numbered `.toe` as an identical copy, and moves the previous numbered file to `Backup/`; opening a path loads that exact file.
- **Conditions:** TD 2025.33070 with `general.inc=2` and `general.link=1` (defaults).
- **Evidence:** throwaway-copy experiment on 2026-11-08; matching SHA-256 hashes for the latest numbered and un-numbered files. An explicit path started a separate filename family without updating the existing link.
- **Implication:** use path-less saves and open the un-numbered project file for latest work.

## External COMP edits need external-Tox saving

- **Finding:** Edits inside a COMP with a non-empty External Tox path and `enableexternaltox` on are lost on reopen after plain `project.save()` unless the external `.tox` is saved; a blank path keeps the COMP embedded in the `.toe` even when `enableexternaltox` is on, so plain saves persist it.
- **Conditions:** TD 2025.33070, only for a COMP that actually loads an external `.tox`. In paint-reactive-2 that is only the vendored `mcp_webserver_base` bridge, which must not be edited; every project-owned COMP has a blank External Tox path, and `/project1/visual` has `enableexternaltox` off.
- **Evidence:** a probe DAT edit in an external COMP was lost after plain `project.save()` but survived `project.save(saveExternalToxs=True)`; `COMP.save(...)` was also verified. The embedded `/project1/visual` edits survive a path-less save: the `.63` save (SHA-256 `F5AEA5D718F0860E18AD9526AFE038ABC8E621DECFD2B675F30D481760991448`, expanded and checked by a cleanup worker) contains `Dripoffset`/`Dripscale`, the 4 × 50 `drip_seeds_chop`, and the field's `uDrips` array binding.
- **Implication:** use path-less `project.save(saveExternalToxs=True)` or `COMP.save(...)` only for edits inside a COMP with an external `.tox`; for paint-reactive-2 use plain path-less `project.save()`.

## toeexpand `.toc` files must keep LF line endings

- **Finding:** `toecollapse` fails on an expanded project whose `.toc` file has CRLF line endings.
- **Conditions:** editing an expanded `.toe` tree before collapsing it.
- **Evidence:** collapse failed until the `.toc` was written with LF.
- **Implication:** write `.toc` files with LF line endings when editing expanded projects.

## Non-Commercial TouchDesigner caps portrait TOP output at 720×1280

- **Finding:** TouchDesigner Non-Commercial caps TOP resolution at 1280 px per side; 9:16 portrait output therefore maxes out at 720×1280.
- **Conditions:** designing portrait outputs on this machine without a Commercial/Pro licence.
- **Evidence:** the planner/worker found 1080×1920 impossible under this licence; performance was measured at 60.0 fps at 720×1280 on Intel Iris Xe.
- **Implication:** design portrait outputs at 720×1280 unless a Commercial/Pro licence is installed.

## TouchDesigner frame counters read 60 fps even when frames are dropped

- **Finding:** counters based on `absTime`, `cookAbsFrame`, or the timeline frame count track timeline time, not rendered frames, so an FPS readout built on them stays at 60 while the project renders slower; the Perform CHOP counts only while it is pulled or displayed, and in Perform Mode an Execute DAT pulls it once per frame at end of frame.
- **Conditions:** reporting render rate in TD 2025.33070, including the overlay metrics in `products/paint-reactive-2`.
- **Evidence:** paint-reactive-2's counter read 60 fps while the project actually rendered only about 37-45 frames/s; on 2026-11-10 the Perform CHOP measured 59.9 rendered frames/s in Perform Mode versus 55.1 in Designer Mode with the network editor open.
- **Implication:** never report 60 fps from a timeline counter; report rendered frames/s from a Perform CHOP plus the frames actually rendered per second, and expect Designer Mode with the editor open to be slower than Perform Mode.

## Locked Windows desktop swallows TouchDesigner cursor hover

- **Finding:** while the Windows session is locked, `SetCursorPos` still moves the pointer but TD receives no rollover/hover events because LockApp/LogonUI sits in front; a `LockApp` process can stay alive even when the desktop is visible.
- **Conditions:** automated real-cursor hover tests of TD panels on this workstation.
- **Evidence:** the hover test of paint-reactive-2 failed while the session was locked and passed when unlocked.
- **Implication:** run real-cursor tests only on an unlocked session with no other TD instance open, and check the active window rather than only whether a `LockApp` process exists.

## TouchDesigner DAT-synced files are written with CRLF

- **Finding:** DATs synced to files from TD on Windows write CRLF line endings and can also carry a BOM, which violates the repositories' LF rule.
- **Conditions:** saving a synced DAT in a repository that requires LF.
- **Evidence:** 9 shaders ended up CRLF and one carried a BOM in `products/paint-reactive-2` (best-practices research finding).
- **Implication:** rewrite the synced file to LF and strip the BOM after each sync, before lint or commit checks.

## TouchDesigner `numpyArray()` returns rows bottom-to-top

- **Finding:** `TOP.numpyArray()` yields image rows from the bottom upward, so y is flipped relative to normal image/array indexing.
- **Conditions:** measuring pixel direction, such as rotation direction, from a TD TOP's numpy array.
- **Evidence:** the rotation-direction measurement on paint-reactive-2 matched only after flipping the y axis.
- **Implication:** invert y (or flip the array vertically) when deriving direction from a `numpyArray()` in TD.

## Diagnostic Info DATs with a viewer cook every frame and pull their target

- **Finding:** an Info DAT whose viewer is on and that is not passive cooks every frame and pulls its target; when the target sits inside a Feedback TOP cycle, TD can lock into a `Cook dependency loop` retry and keep cooking the chain after the output window is closed.
- **Conditions:** diagnostic Info DATs watching nodes inside a feedback cycle, TD 2025.33070.
- **Evidence:** paint-reactive-2 had 12 diagnostic Info DATs; the cook loop added 743 cooks in 12.1 s. Setting them passive with viewers off gave 0 cooks when idle and 10 open/close cycles without warnings.
- **Implication:** make diagnostic Info DATs passive with their viewer off, and inspect nodes inside a cycle through an Out TOP outside it.

## TouchDesigner stops rendering when its output window is fully covered

- **Finding:** TD stops rendering when another window completely covers the output window, so FPS measured in that state is not valid.
- **Conditions:** measuring render rate with the output window obscured by other windows.
- **Evidence:** worker observation on paint-reactive-2, 2026-11-10.
- **Implication:** keep the output window fully visible and uncovered while measuring FPS.

## `toeexpand.exe` ignores the destination directory argument

- **Finding:** `toeexpand.exe` ignores a destination-directory argument and expands the archive beside the input file.
- **Conditions:** expanding a `.toe` on this workstation, including when a separate target directory is passed.
- **Evidence:** during verification of the `.63` save on 2026-11-10 a worker found `*.toe.dir`/`*.toc` created in the project folder instead of the requested target, and cleaned them up.
- **Implication:** copy the `.toe` out of the repository and expand that copy; otherwise `*.toe.dir`/`*.toc` appear in the project folder.
