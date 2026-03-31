# Chapter 13

# Android image production

The `all-features` branch adds `eggs android`, a subcommand tree for producing
bootable Android images from a running Android environment. This enables
creating distributable Android ISOs, raw disk images, Waydroid container
snapshots, and OTA flashable zips using the same eggs workflow.

## Prerequisites

`eggs android produce` must be run from within an Android environment. It
detects the environment automatically. To check whether your environment is
supported:

```
eggs android status
```

This prints the detected Android variant, version, SDK level, architecture,
source type, and whether GApps are present.

## Supported environments

| Environment | Source type | Notes |
|---|---|---|
| Android-x86 | native | x86/x86_64 ISOs |
| BlissOS | native | x86_64 ISOs |
| Waydroid | waydroid-container | Waydroid snapshots |
| Generic Android (ARM) | native | Raw disk images |
| RISC-V Android | native | ISO (riscv64) |

## Output modes

| Mode | Output | Supported architectures |
|------|--------|------------------------|
| `iso` | Bootable ISO with GRUB/syslinux | x86, x86_64, riscv64 |
| `raw-img` | Raw disk image | arm64-v8a, armeabi-v7a |
| `waydroid` | Waydroid container snapshot | any |
| `ota` | OTA flashable zip | any |
| `auto` | Detected from environment | any |

In `auto` mode (the default), eggs selects the output mode based on the
detected environment:

- Waydroid containers → `waydroid`
- Architectures that support ISO (x86, x86_64, riscv64) → `iso`
- All other architectures → `raw-img`

## eggs android produce

### Basic usage

```
sudo eggs android produce
```

### Flags

| Flag | Short | Description |
|------|-------|-------------|
| `--mode=<mode>` | `-m` | Output mode: `auto`, `iso`, `raw-img`, `waydroid`, `ota` (default: `auto`) |
| `--arch=<arch>` | `-a` | Target architecture: `x86_64`, `x86`, `arm64-v8a`, `armeabi-v7a`, `riscv64` (auto-detected if omitted) |
| `--compression=<algo>` | `-c` | Compression for `system.sfs`: `gzip`, `lz4`, `xz`, `zstd` (default: `zstd`) |
| `--output=<path>` | `-o` | Output file path |
| `--verbose` | `-v` | Verbose output |

### Examples

```
# Auto-detect mode and architecture
sudo eggs android produce

# Force ISO mode with zstd compression
sudo eggs android produce --mode=iso --compression=zstd

# Waydroid container snapshot
sudo eggs android produce --mode=waydroid

# OTA flashable zip
sudo eggs android produce --mode=ota

# Specify output path
sudo eggs android produce --output=/tmp/android.iso

# Explicit architecture
sudo eggs android produce --mode=iso --arch=x86_64
```

## eggs android status

Prints a summary of the detected Android environment:

```
eggs android status
```

Example output:

```
Detected: Android-x86 (BlissOS 16)
  Android 13 (SDK 33)
  Arch: x86_64
  Source: native
  GApps: yes
```

If no Android environment is detected, the command explains what was checked
and why detection failed.

## ISO mode details

When producing an ISO (`--mode=iso`), eggs:

1. Detects the kernel and initrd from the running Android system
2. Locates the system image and vendor image
3. Compresses the system image into `system.sfs` using the selected algorithm
4. Builds an ISO structure with GRUB (UEFI) and syslinux (BIOS) bootloaders
5. Packs everything into a bootable ISO

The resulting ISO can be written to a USB drive with `dd` or Etcher and booted
on compatible hardware.

## Waydroid mode details

When producing a Waydroid snapshot (`--mode=waydroid`), eggs creates a
compressed archive of the Waydroid container state. This snapshot can be
restored on another machine running Waydroid.

## OTA mode details

When producing an OTA zip (`--mode=ota`), eggs generates a flashable zip
compatible with standard Android recovery environments (TWRP, etc.). The zip
includes OTA metadata for version tracking.

## Architecture detection

eggs reads Android system properties (`build.prop`) to detect the primary ABI
and whether the system is multilib (supports both 32-bit and 64-bit). The
detected architecture determines the default output mode and the bootloader
selection.

| Architecture | ISO supported | Default mode |
|---|---|---|
| x86_64 | yes | iso |
| x86 | yes | iso |
| riscv64 | yes | iso |
| arm64-v8a | no | raw-img |
| armeabi-v7a | no | raw-img |
