![](/media/chapter-1/chapter-1.jpg)

# Chapter 1
# What is penguins' eggs?

eggs is a versatile command-line tool that transforms your existing Linux
installation into a redistributable live ISO image. It supports Debian, Devuan,
Ubuntu and their derivatives (Xubuntu, Kubuntu, Linux Mint, and more), as well
as Arch Linux and Manjaro.

With penguins' eggs, you can create an installable live ISO of your system,
complete with all installed applications and configurations. The tool also
supports offline installation, scripting mode for automated ISO management,
and full visual customization of the live environment and Calamares installer
via [penguins-wardrobe](https://github.com/pieroproietti/penguins-wardrobe).

The `all-features` branch expands penguins-eggs from a single remastering CLI
into a full ecosystem: an AI assistant, a unified GUI, Android image production,
ChromiumOS flavour support, RISC-V architecture, decentralized ISO distribution,
security tooling, and a suite of companion tools for recovery, factory reset,
immutable Linux, and kernel management.

# Why penguins' eggs?

eggs gives Linux users a single tool to capture, customize, and distribute their
system. Whether you want a clean redistributable ISO, a full clone with user
data, or a crypted backup, eggs handles it. It supports a wide range of
distributions and architectures, and its plugin and integration ecosystem means
it can be extended to fit specialized workflows.

# Top penguins-eggs features

## Fast and efficient

eggs uses `livefs` for instant acquisition of the live system rather than
copying the entire filesystem. By default it uses `zstd level 3` compression,
which produces ISOs significantly faster than traditional `xz`-based tools while
keeping file sizes reasonable.

## Multiple compression algorithms

eggs supports several compression algorithms to suit different use cases:

- **zstd level 3** (default) — fast builds, good compression ratio
- **zstd pendrive** (`--pendrive`) — `zstd -b 1M -Xcompression-level 15`, optimized for USB drives
- **xz standard** (`--standard`) — `xz -b 1M`, smaller ISOs at the cost of build time
- **xz max** (`--max`) — `xz -Xbcj ...`, maximum compression for distribution

## Clone and crypted clone

eggs provides several options for including user data in the ISO:

- **`--clone`** — preserves user data and accounts in the live system
- **`--homecrypt`** (`-k`) — user data and accounts are saved in an encrypted LUKS volume within the ISO
- **`--fullcrypt`** (`-f`) — full system encryption (available on Debian trixie/excalibur)

## Cuckoo and PXE boot

The `cuckoo` command starts a PXE boot server from the live system, making the
live environment accessible to all computers on the local network. This enables
network-wide deployment without physical media.

## TUI and GUI installers

eggs ships its own TUI installer called `krill`, useful when no graphical
environment is available. For desktop systems, eggs configures the
[Calamares](https://calamares.io/) GUI installer automatically.

## Wardrobe

The Wardrobe system lets you organize and apply customizations — costumes,
accessories, and themes — to tailor the live ISO and Calamares branding to
specific needs. See Chapter 8 for the full Wardrobe guide.

## eggs love — the simplest way to get an egg

`eggs love` is a one-command workflow that reads a `love.yaml` configuration
file and runs the full sequence of eggs commands automatically. It is the
fastest path from a running system to a finished ISO:

```
sudo eggs love
```

Optional flags mirror those of `eggs produce`:

```
sudo eggs love --clone
sudo eggs love --homecrypt
sudo eggs love --fullcrypt
sudo eggs love --dtbdir /path/to/dtbs
```

## Recovery tools integration

The `--recovery` flag on `eggs produce` layers
[penguins-recovery](https://github.com/Interested-Deving-1896/penguins-recovery)
tools onto the produced ISO, turning it into a rescue medium without a separate
build step:

```
sudo eggs produce --recovery
sudo eggs produce --recovery --recovery-gui=minimal
sudo eggs produce --recovery --recovery-rescapp
```

GUI profiles are `minimal`, `touch`, and `full`. The `--recovery-rescapp` flag
adds the rescapp graphical rescue wizard.

## ChromiumOS flavour support

When building on a ChromiumOS-based system, the `--cros-flavour` flag selects
the browser to embed in the ISO:

```
sudo eggs produce --cros-flavour=thorium
sudo eggs produce --cros-flavour=brave
sudo eggs produce --cros-flavour=chromium
sudo eggs produce --cros-flavour=vanadium
sudo eggs produce --cros-flavour=bromite
sudo eggs produce --cros-flavour=cromite
sudo eggs produce --cros-flavour=custom --cros-browser-repo=https://github.com/user/fork
```

## Android image production

`eggs android produce` creates bootable Android images from a running Android
environment. It auto-detects the environment and selects the appropriate output
mode:

| Mode | Output | Architectures |
|------|--------|---------------|
| `iso` | Bootable ISO | x86, x86_64, riscv64 |
| `raw-img` | Raw disk image | arm64-v8a, armeabi-v7a |
| `waydroid` | Waydroid container snapshot | any |
| `ota` | OTA flashable zip | any |

```
sudo eggs android produce
sudo eggs android produce --mode=iso --compression=zstd
sudo eggs android produce --mode=waydroid
sudo eggs android status
```

See Chapter 13 for the full Android guide.

## AI assistant (eggs-ai)

[eggs-ai](https://github.com/Interested-Deving-1896/eggs-ai) is an AI agent
that understands penguins-eggs commands, configurations, and workflows. It
provides diagnostics, guided ISO building, config generation, Calamares
assistance, and general Q&A. It supports 7 LLM providers including local Ollama
for fully offline operation, and exposes an MCP server for integration with
Cursor, Claude Desktop, and other AI agents. See Chapter 11.

## Unified GUI (eggs-gui)

[eggs-gui](https://github.com/Interested-Deving-1896/eggs-gui) is a unified
graphical frontend for penguins-eggs with three interface options:

- **BubbleTea TUI** (Go) — terminal power users and SSH sessions
- **NodeGUI desktop** (Qt6/TypeScript) — native desktop application
- **NiceGUI web** (Python) — browser-based remote access

All frontends share a single Go daemon backend. See Chapter 12.

## Distribution and security tooling

The `all-features` branch adds three commands for ISO distribution and security:

- **`eggs lfs`** — manages git-LFS tracking for produced ISOs (track, list, setup, enable, disable)
- **`eggs ipfs`** — publishes ISOs to IPFS via [brig](https://github.com/sahib/brig) (publish, list, get, gateway, history)
- **`eggs st`** — produces [System Transparency](https://system-transparency.org/) compatible boot artifacts with Ed25519 signing (package, keygen, verify, extract)

The `--sbom` flag on `eggs produce` generates a Software Bill of Materials for
the produced ISO using [syft](https://github.com/anchore/syft).

See Chapter 15 for full details.

## Companion ecosystem

The `all-features` branch integrates four companion tools:

- **[penguins-recovery](https://github.com/Interested-Deving-1896/penguins-recovery)** — unified rescue toolkit supporting 6 distro families
- **[penguins-powerwash](https://github.com/Interested-Deving-1896/penguins-powerwash)** — factory reset with soft/medium/hard/sysprep/hardware modes
- **[penguins-immutable-framework](https://github.com/Interested-Deving-1896/penguins-immutable-framework)** — immutable Linux framework (abroot, ashos, frzr, akshara, btrfs-dwarfs backends)
- **[penguins-kernel-manager](https://github.com/Interested-Deving-1896/penguins-kernel-manager)** — full kernel lifecycle management across all major distributions

See Chapter 14 for the full integrations guide.

## Supporting multiple distributions

eggs supports the following distributions and their major derivatives:

| Base | Distributions |
|------|--------------|
| Arch | Arch Linux, EndeavourOS, Garuda, Manjaro, BigLinux, CachyOS |
| Debian | Debian, Devuan, Ubuntu, Linux Mint, LMDE, MX Linux, Zorin, Pop!_OS, elementary |
| Ubuntu | Kubuntu, Xubuntu, Lubuntu, Ubuntu MATE, KDE neon |

Additional derivatives can be added by extending the distro detection logic.

## Supported hardware architectures

eggs supports the following CPU architectures:

| Architecture | Description |
|---|---|
| `i386` | 32-bit x86, legacy PCs |
| `amd64` | 64-bit x86, standard PCs and servers |
| `arm64` | 64-bit ARM, Raspberry Pi 4/5 and other SBCs |
| `riscv64` | RISC-V, e.g. MuseBook (Spacemit K1) |

For ARM and RISC-V targets, the `--dtbdir` flag specifies the path to Device
Tree Blob (DTB) files required for hardware initialization:

```
sudo eggs produce --dtbdir /path/to/dtbs
```

See Chapter 16 for the RISC-V / MuseBook port.

## Repository integrity

eggs uses only the original distribution's packages and makes no modifications
to repository lists, preserving the integrity and authenticity of the base
system.

# More information

- [CHANGELOG](https://github.com/pieroproietti/penguins-eggs/blob/master/CHANGELOG.md)
- [penguins-eggs repository](https://github.com/pieroproietti/penguins-eggs)
- [penguins-eggs website](https://penguins-eggs.net)

![](media/chapter-1/penguins-eggs-net.png)
