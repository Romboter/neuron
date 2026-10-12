# Agent instructions

Repository-level instructions for AI coding agents working on Neuron. Follow these rules before changing code. Read task-specific documentation as needed; don't load the entire documentation set by default.

## Project and architecture

Neuron is a Rust workspace containing a Windows-first Razer device-control app and CLI. It uses raw HID, a shared core, and a single **Trigger → Action** dispatch engine. Linux source builds are in development; macOS is not implemented. Experimental Logitech HID++ discovery is read-only and unverified on hardware.

**Architecture rules**
- Put portable behavior, types, and tests in `neuron-core`. Keep the GUI/CLI thin.
- Route new input sources and actions through the existing Trigger → Action engine. Do **not** build a second dispatch path.
- Keep device-specific protocol wiring in device definitions and the protocol ledger; don't scatter unverified hardware assumptions through application code.
- Keep runtime profile settings, lighting, and bindings together.

| Location | Purpose |
|---|---|
| `crates/neuron-core` | Engine, HID protocol, registry, input dispatch, lighting, profiles, macros, safety gates |
| `crates/neuron-app` | Slint GUI and platform runtime |
| `crates/neuron-cli` | CLI and headless daemon |
| `crates/neuron-host` | OpenRGB/Chroma integrations, ownership arbitration, signal bus |
| `crates/engram` | Gesture codec/research; separate license terms |
| `crates/neuron-testkit` | Mock transports and fault-injection test utilities |
| `crates/neuron-core/devices/` | Curated device definitions; these override runtime `devices/auto/` definitions |
| `crates/neuron-core/protocol/ledger.toml` | Evidence for HID opcodes and write safety |

## Non-negotiable safety rules

1. **Tests must never arm or inject input or contact real hardware.** Preserve the process-wide arm gate and the test-only deny-by-default transport policy. Only the live GUI or daemon may arm input.
2. **Verify every device write.** Use the matching read-back getter; treat mismatches as errors. Default writes to volatile (`NOSTORE`), except scroll-stage selection, which stores onboard by default. Never invent an opcode or report success based only on a successful send.
3. **Gate unverified writes.** New write capabilities remain behind `NEURON_*_WRITE` controls until supported by wire-capture evidence. When evidence is missing, refuse the operation.
4. **Maintain protocol evidence.** Add or update `crates/neuron-core/protocol/ledger.toml` whenever an opcode or hardware claim changes. Grades are `live / capture / agreed / single / derived`; include dated evidence and pin upstream factual citations to a commit SHA. `tests/protocol_ledger.rs` enforces coverage and gating.
5. **Match documentation to reality.** When support or verification changes, update `docs/STATUS.md` and the applicable documentation. Do not describe compiled or simulated behavior as hardware-verified.
6. **Get owner approval before changing HID interception, device write paths, or the Python bridge.** These are `risky` areas even for otherwise small tasks. Stop before implementing them if approval has not been given.

## Workflow

1. **Inspect:** Read the relevant implementation, tests, and local conventions before editing. Treat source and tests as authoritative; design documents can be stale.
2. **Scope:** Make the smallest change that solves the task. Preserve unrelated local changes. Do not refactor neighboring systems without cause.
3. **Implement:** Maintain existing public contracts, safety gates, protocol evidence, and license boundaries.
4. **Test:** Run relevant targeted checks, then repository validation when practical.
5. **Verify behavior:** A passing build or unit suite does not prove the GUI works or device writes succeed on hardware.
6. **Report:** State what changed, which commands ran, their results, any actual GUI/hardware/platform testing, and what remains unverified.

Do not publish releases, public announcements, or modify external accounts/configuration without explicit approval in the current session.

## Validation commands

The primary gate is `validate.ps1`; CI calls the same script.

```powershell
.\validate.ps1               # Workspace build and tests
.\validate.ps1 -Locked       # Lockfile-enforced CI-equivalent dependencies
.\validate.ps1 -Mode full    # Feature matrix, release build, safe ignored tests, Clippy
.\validate.ps1 -Mode lint    # Format/PowerShell parsing checks
```

On Linux, use PowerShell 7:

```bash
pwsh ./validate.ps1
```

Linux app builds require the relevant Slint, fontconfig, X11/Wayland, GTK/AppIndicator development libraries. Validation does not establish cross-platform runtime parity or prove physical device functionality.

**Build and editing hazards**
- **Never run `cargo fmt --all`.** There is no pinned rustfmt style; whole-tree formatting creates extensive unrelated changes. Match local formatting and keep diffs focused. Rustfmt is advisory.
- **Do not run concurrent builds using the same `CARGO_TARGET_DIR`.** Use separate target directories for parallel jobs. If incremental linkage fails with spurious `LNK2019 anon.*.llvm` errors, clear `target/debug/incremental`.
- **Preserve UTF-8.** Avoid PowerShell 5.1 `Get-Content` → `Set-Content` round trips for source files; use a UTF-8-preserving editor or .NET file IO with no-BOM UTF-8.
- **Close a running Neuron tray app before Windows release builds** so the executable is not locked.
- **Do not overwrite an existing dirty working tree** or discard another contributor's changes.

## Verification by change type

| Change | Required evidence |
|---|---|
| Core logic | Relevant tests and repository validation |
| GUI/Slint | Launch and inspect the affected UI states; add render smoke coverage in `apptest.rs` for conditional/hovered controls |
| Input injection/macros | Confirm safety gates and disarmed tests remain intact; live behavior only with explicit owner authorization |
| HID/device/lighting | Check protocol ledger and read-back tests; identify actual device model and measured behavior if hardware tested |
| Windows/Linux runtime | State the OS and runtime path actually tested; do not infer platform parity from compilation |
| Docs only | Verify claims, paths, links, and examples against source; code build may be unnecessary |

The SYSTEM diagnostics bench has nine read-only probes for real-device checks. Do not confuse safe read-only diagnostics with write validation. If hardware, platform, or UI access is unavailable, explicitly report **not tested**.

## Code, comments, and UI

**Comments:** Document constraints, protocol byte layouts, evidence, failure contracts, and non-obvious reasoning. Include `// SAFETY:` explanations for `unsafe` and keep required SPDX/license headers. Avoid line-by-line narration, conversational asides, and lengthy histories. Record debugging history in commits or issues rather than source comments.

**UI:** Put settings and feedback next to the feature they affect instead of introducing a separate dialog/page when the local surface can handle it. Preview effects on the actual subject when possible. Avoid instructional paragraphs, redundant captions, gratuitous animation, and unnecessary visual elements. Keep list/model row identity stable when updating live Slint data; replacing whole models can discard hover state or active text edits. Use `theme.slint` for motion conventions.

## Task-specific references

Read these when the task calls for them, rather than upfront:

| Task | Reference |
|---|---|
| Feature behavior and UX | [`docs/GDD.md`](docs/GDD.md) |
| System design and runtime | [`docs/TDD.md`](docs/TDD.md) |
| Current verification/support | [`docs/STATUS.md`](docs/STATUS.md) |
| Protocol host integration | [`docs/reference/PROTOCOL-HOST.md`](docs/reference/PROTOCOL-HOST.md) |
| CLI behavior | [`docs/reference/CLI.md`](docs/reference/CLI.md) |
| Contributor and licensing requirements | [`CONTRIBUTING.md`](CONTRIBUTING.md), [`LICENSE.md`](LICENSE.md) |
| Issue labels | [`.github/LABELS.md`](.github/LABELS.md), [`.github/labels.yml`](.github/labels.yml) |
| User installation/update support | [`skills/neuron-lazy-update/SKILL.md`](skills/neuron-lazy-update/SKILL.md) |
| Python macro development | [`skills/neuron-macros/SKILL.md`](skills/neuron-macros/SKILL.md) |
| Configuring binds, profiles, and lighting | [`skills/neuron-cli/SKILL.md`](skills/neuron-cli/SKILL.md) |
| Promotional video tasks | [`skills/neuron-promo/SKILL.md`](skills/neuron-promo/SKILL.md) |

`docs/TDD.md` and `docs/reference/PROTOCOL-HOST.md` were written with an LLM and may be stale. Check implementation before relying on architectural claims.

## Issues and pull requests

- Work on a focused branch (convention: `area/short-description`), based on the intended integration branch.
- Include a concise change summary, tests run and results, device/OS used for any live testing, and limitations.
- For issues, choose exactly one Tier-1 type label and one Tier-2 `area:*` label; Tier-3 flags are optional. Add `autonomously-derived` when an unattended agent session originated the issue.
- Separate observed evidence from hypotheses. Put reproduction details in the issue body, not just labels.
- Follow the PR contributor-agreement and license requirements in `CONTRIBUTING.md`. Do not check acceptance boxes on behalf of someone without authorization.
