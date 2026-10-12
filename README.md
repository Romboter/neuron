# Neuron

**An open-source alternative to Razer Synapse for Windows.**

Neuron controls supported Razer mice and keyboards directly over USB HID, without a Razer account, vendor SDK, or kernel driver. It includes a desktop app for configuring devices and a command-line tool for scripting.

It's a beta project maintained by one developer. Some features are tested on real hardware; others are experimental. Check [the project status](docs/STATUS.md) before assuming a feature or device will work.

<p align="center"><img src="docs/media/hero-desktop.webp" alt="Neuron desktop app" width="100%"></p>

## At a glance

| | |
|---|---|
| **OS** | Windows: app and CLI. Linux: builds from source, but hardware control is unverified and no release package is available. macOS is not supported. |
| **Tested hardware** | Razer Naga V2 Pro and BlackWidow Chroma V2. Other Razer devices may need protocol verification. |
| **Interface** | Desktop app, system tray, and CLI |
| **Configuration** | Local TOML files |
| **Network requirements** | No account, cloud service, or telemetry |
| **License** | Mostly GPL-3.0-or-later with a linking exception; certain research modules use separate terms. See [LICENSE.md](LICENSE.md). |

## What can it do?

- **Device settings:** DPI, polling rate, brightness, battery information, lift-off distance, and supported onboard settings. Availability varies by device.
- **Button remapping:** Bind device buttons and other inputs to keystrokes, actions, profile changes, or macros.
- **Profiles:** Save device settings, lighting, and bindings together, and switch profiles based on the active application.
- **Lighting:** Use firmware effects, custom effects, gradients, and per-key patterns on compatible devices.
- **Macros:** Run Python macros from bindings, with a restricted capability mode by default and an explicitly opt-in unrestricted mode.
- **Gestures and controls:** Mouse-drawn gestures, radial menus, and desktop/window actions.
- **Audio controls:** Bind microphone mute, gain, and default audio-device switching.
- **CLI:** Inspect devices, change settings, back up device state, and automate configuration.

Several of these systems are still under development. For details and limitations, see the [feature guide](docs/GDD.md) and [current status](docs/STATUS.md).

<p align="center"><img src="docs/media/binds-direct.webp" alt="Bindings editor" width="100%"></p>

## Install

### Windows

Download the installer or portable ZIP from [Releases](https://github.com/worflor/neuron/releases).

- **Installer:** Installs the app and CLI under `%LOCALAPPDATA%\Programs\Neuron`.
- **Portable ZIP:** Extract to a writable folder and run `neuron-app.exe`.

Windows builds are currently **unsigned**, so SmartScreen may warn you. Verify the download against the release's `SHA256SUMS.txt`. Packages include a `SOURCE.txt` file identifying their source commit and build method.

The native Chroma broker may require elevation during installation or when first used from a portable copy. The tray app and macros otherwise run without elevated privileges.

### Build from source

Neuron is a Rust workspace. To build on a supported development system:

```sh
cargo build --release
```

This produces the CLI (`neuron`) and GUI (`neuron-app`) binaries in `target/release` (with `.exe` extensions on Windows).

On Windows, run the project's validation script:

```powershell
.\validate.ps1
```

Linux builds are available from source, but the GUI and CLI have not been tested with connected Razer hardware on Linux. There is no packaged Linux release yet.

If the GUI fails to render on a VM, over Remote Desktop, or with an older graphics driver, try setting `NEURON_RENDERER=software`.

## Common CLI commands

```sh
neuron list                          # Find recognized devices
neuron feel dpi 1600                 # Set DPI
neuron feel stages 800 1600 3200     # Set DPI stages
neuron lighting effect fire          # Preview effect bytes (dry run)
neuron backup                        # Back up supported device state
neuron import prof.synapse3 --apply  # Import a Synapse export
neuron run --safe                     # Observe input without arming actions
neuron macro add lift my.py          # Register a Python macro
```

See the [CLI reference](docs/reference/CLI.md) for all commands and options. Device writes depend on supported hardware and may be refused when verification is missing.

## Configuration

Neuron stores its settings in a shared *run root* used by both the GUI and CLI.

| Installation | Default location |
|---|---|
| Windows installer / source build | `%LOCALAPPDATA%\neuron` |
| Linux source build | `$XDG_DATA_HOME/neuron` (usually `~/.local/share/neuron`) |
| Portable ZIP containing `portable.flag` | The executable's folder |
| Custom | Set `NEURON_RUN_DIR` to override the default |

Config files are plain TOML. An existing older configuration beside the executable is copied to the new location on first launch, rather than moved.

## Device support and safety

Neuron talks to Razer hardware using HID feature reports. Protocol support has been developed through USB captures, comparisons with [OpenRazer](https://github.com/openrazer/openrazer), and testing on actual devices.

**Hardware-verified does not mean universally supported.** The primary test devices are a Naga V2 Pro and BlackWidow Chroma V2. Some writes are available only when the device returns values that confirm the change; experimental writes are gated or disabled. Some features have no known working command yet.

For input and macros:

- Input injection starts behind an arm/disarm gate. `--safe` keeps it disarmed.
- Device writes are checked by reading settings back where supported.
- Python macros have a default bounded mode, but **this is not a security sandbox**. A macro explicitly marked `# neuron: raw` can execute unrestricted Python with your user account's permissions.

The [status document](docs/STATUS.md) lists supported, experimental, and missing capabilities. The [protocol ledger](crates/neuron-core/protocol/ledger.toml) tracks evidence behind hardware commands.

## Known limitations

- Windows is the primary supported platform. Linux hardware control has not been verified, and macOS support is not implemented.
- Razer compatibility is device-specific; there is no guarantee that every Razer model works.
- Experimental Logitech HID++ support is currently read-only discovery, without hardware verification.
- Neuron imports supported plaintext Synapse exports, not encrypted Synapse cloud-account data.
- Not every device feature can be written safely or persisted to onboard memory.
- The project is in beta and has been tested mostly on the maintainer's own equipment.

For Linux users needing established Razer support today, see [OpenRazer](https://github.com/openrazer/openrazer). For multi-brand RGB control, see [OpenRGB](https://openrgb.org).

## Contributing

Bug reports, device captures, testing, and code contributions are welcome.

Start with [CONTRIBUTING.md](CONTRIBUTING.md), then read [AGENTS.md](AGENTS.md) for repository layout and validation rules. Work that's ready for contributors is listed on the [project board](https://github.com/users/worflor/projects/1).

Before submitting changes, run:

```sh
cargo test --workspace
```

Tests must not arm input or access live devices. Hardware writes must verify their results or remain gated. Please avoid running `cargo fmt --all` over the entire codebase; the repository currently uses hand-formatted source and doing so produces unrelated changes.

Contributions are subject to the licenses and [contributor agreement](LICENSES/CONTRIBUTOR-AGREEMENT-1.0.md).

## Documentation

- [Project status and verified features](docs/STATUS.md)
- [Feature guide](docs/GDD.md)
- [Technical design](docs/TDD.md)
- [CLI reference](docs/reference/CLI.md)
- [Protocol host integrations](docs/reference/PROTOCOL-HOST.md)
- [Troubleshooting and bug reports](skills/neuron-lazy-update/issues.md)
- [Agent-assisted installation and updates](skills/neuron-lazy-update/SKILL.md)
- [Python macro development](skills/neuron-macros/SKILL.md)
- [Licensing details](LICENSE.md)

---

Neuron is developed by Woflo Labs, the publishing name used by its individual maintainer. This fork retains the original project's attribution and documentation links.
