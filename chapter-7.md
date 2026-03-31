![](/media/chapter-7/chapter-7.jpg)

# Chapter 7

# Producing ISO image file

The `eggs produce` command creates a live image of your system without including
personal data. It captures the entire OS setup — applications, configurations,
and system files — into a bootable ISO that can be distributed or used for
installation.

## Step 1: cleaning the system

### Cleaning with eggs

```
sudo eggs tools clean
```

This removes system logs, apt caches, and other unnecessary files. Run
`eggs tools clean --help` for the full flag list.

### Cleaning with BleachBit

BleachBit is an open-source system cleaning tool that removes temporary files,
browser history, cache, and other data. Install it from your distribution's
repository, then run:

```
sudo bleachbit
```

Or in command-line mode:

```
sudo bleachbit --list
sudo bleachbit --clean apt.clean
```

## Step 2: preparing the skel folder

```
sudo eggs tools skel
```

This updates `/etc/skel/` with configuration files from the current user's home
directory. New users created from the live ISO will start with this
configuration.

To copy from a specific user:

```
sudo eggs tools skel --user david
```

## Step 3: prepare ISO for offline installation

```
sudo eggs tools yolk
```

This configures Calamares or krill to install packages without an internet
connection. Downloaded packages are stored in `/var/local/yolk`.

## Step 4: eggs produce

### Basic usage

```
sudo eggs produce
```

Get the full flag list:

```
eggs produce --help
```

### Compression flags

| Flag | Compression | Use case |
|------|-------------|----------|
| *(default)* | zstd level 3 | Fast builds, general use |
| `--pendrive` (`-p`) | zstd -b 1M -Xcompression-level 15 | USB drives |
| `--standard` (`-f`) | xz -b 1M | Smaller ISOs |
| `--max` (`-m`) | xz -Xbcj ... | Maximum compression for distribution |

### Clone and encryption flags

| Flag | Description |
|------|-------------|
| `--clone` (`-c`) | Include user data and accounts in the live system |
| `--homecrypt` (`-k`) | Save user data in an encrypted LUKS volume within the ISO |
| `--fullcrypt` (`-f`) | Full system encryption (Debian trixie/excalibur only) |

### Naming and branding flags

| Flag | Description |
|------|-------------|
| `--basename <value>` | Base name for the ISO file |
| `--prefix <value>` (`-P`) | Prefix for the ISO file name |
| `--theme <value>` | Theme for the live CD and Calamares branding |
| `--addons <value>...` | Addons to include: `adapt`, `ichoice`, `pve`, `rsupport` |

### Behaviour flags

| Flag | Description |
|------|-------------|
| `--nointeractive` (`-n`) | No user interaction |
| `--script` (`-s`) | Script mode: generate scripts to manage the ISO build |
| `--release` | Remove penguins-eggs, Calamares, and dependencies after installation |
| `--unsecure` (`-u`) | Include `/root` contents in the live system |
| `--yolk` (`-y`) | Force yolk renewal |
| `--static` | Do not rebuild the exclude list |
| `--udf` (`-U`) | Use UDF format (genisoimage), breaks the 4.7 GB ISO limit |
| `--excludes <value>...` | Exclusion profiles: `static`, `homes`, `home` |
| `--links <value>...` | Desktop links to include |
| `--noicon` (`-N`) | Do not place the eggs icon on the desktop |

### Recovery flags

| Flag | Description |
|------|-------------|
| `--recovery` | Layer penguins-recovery tools onto the produced ISO |
| `--recovery-gui=<profile>` | GUI profile for recovery: `minimal`, `touch`, or `full` |
| `--recovery-rescapp` | Include the rescapp GUI rescue wizard |

Examples:

```
sudo eggs produce --recovery
sudo eggs produce --recovery --recovery-gui=minimal
sudo eggs produce --recovery --recovery-gui=full --recovery-rescapp
```

The `--recovery` flag downloads
[penguins-recovery](https://github.com/Interested-Deving-1896/penguins-recovery)
if not already present and runs the adapter script to inject recovery tools into
the ISO. The output file is named `<original-name>-recovery.iso`.

### ChromiumOS flavour flags

These flags apply only when building on a ChromiumOS-based system:

| Flag | Description |
|------|-------------|
| `--cros-flavour=<flavour>` | Browser flavour: `chromium`, `thorium`, `brave`, `vanadium`, `bromite`, `cromite`, `custom` |
| `--cros-browser-repo=<url>` | Custom browser repo URL (required when `--cros-flavour=custom`) |

Examples:

```
sudo eggs produce --cros-flavour=thorium
sudo eggs produce --cros-flavour=brave
sudo eggs produce --cros-flavour=custom --cros-browser-repo=https://github.com/user/fork
```

### ARM and RISC-V flag

| Flag | Description |
|------|-------------|
| `--dtbdir=<path>` | Path to Device Tree Blobs (DTB) directory |

Example:

```
sudo eggs produce --dtbdir /usr/lib/linux-image-6.6.0/broadcom
```

Required for ARM and RISC-V targets where the bootloader needs DTB files to
initialize hardware. See Chapter 16 for RISC-V details.

### Distribution and security flags

| Flag | Description |
|------|-------------|
| `--lfs` | Track the produced ISO in git-LFS after build |
| `--ipfs` | Publish the produced ISO to IPFS via brig after build |
| `--snapshot` | Create BTRFS snapshots before and after the build |
| `--sbom` | Generate a Software Bill of Materials (SBOM) via syft |

Examples:

```
sudo eggs produce --lfs
sudo eggs produce --ipfs
sudo eggs produce --snapshot
sudo eggs produce --sbom
```

The `--sbom` flag requires [syft](https://github.com/anchore/syft) to be
installed. It generates an SBOM file alongside the ISO documenting all packages
included in the image.

See Chapter 15 for full documentation of `eggs lfs`, `eggs ipfs`, and `eggs st`.

### Common examples

```
sudo eggs produce                                    # fast compression
sudo eggs produce --max                              # max compression
sudo eggs produce --pendrive                         # pendrive-optimized
sudo eggs produce --clone                            # include user data
sudo eggs produce --homecrypt                        # encrypted home clone
sudo eggs produce --basename=colibri
sudo eggs produce --theme lastos
sudo eggs produce --excludes static
sudo eggs produce --excludes homes                   # exclude /home/*
sudo eggs produce --excludes home                    # exclude ~/*
sudo eggs produce --recovery --recovery-gui=minimal  # with recovery tools
sudo eggs produce --sbom                             # with SBOM
sudo eggs produce --lfs                              # track in git-LFS
sudo eggs produce --dtbdir /path/to/dtbs             # ARM/RISC-V
```

## The produced ISO

The ISO is placed in the `snapshot_dir` defined in `eggs.yaml` (default:
`/home/eggs/`). The file name follows the pattern:

```
<prefix><basename>-<arch>.iso
```

For example: `egg-of_debian-bookworm-amd64.iso`

The directory also contains:

- A `.md5` checksum file (if `make_md5sum: true` in `eggs.yaml`)
- An SBOM file (if `--sbom` was used)

## Grub boot menu

The produced ISO includes a GRUB boot menu. The default menu entries allow
booting the live system, running the installer, and (if `--recovery` was used)
accessing recovery tools.

Default GRUB boot menu:

![](media/image100.png)

![image101.jpg](media/image101.jpg)
