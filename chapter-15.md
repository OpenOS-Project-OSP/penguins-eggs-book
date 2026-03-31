# Chapter 15

# Distribution and security

The `all-features` branch adds three commands for distributing ISOs and
verifying their integrity: `eggs lfs`, `eggs ipfs`, and `eggs st`. The
`--sbom` flag on `eggs produce` generates a Software Bill of Materials.

---

## eggs lfs — git-LFS tracking

`eggs lfs` manages [git-LFS](https://git-lfs.com/) tracking for produced ISOs.
git-LFS stores large binary files (like ISOs) outside the main git repository
while keeping lightweight pointers in the tree, making it practical to version
and distribute ISOs via git hosting services.

### Prerequisites

git-LFS must be installed:

```
sudo apt install git-lfs    # Debian/Ubuntu
sudo pacman -S git-lfs      # Arch
```

### Commands

```
eggs lfs <action> [path]
```

| Action | Description |
|--------|-------------|
| `track <iso-path>` | Add the ISO to git-LFS tracking and commit |
| `list [directory]` | List LFS-tracked files in a directory (default: `/home/eggs`) |
| `setup` | Configure git-LFS for the eggs output directory |
| `enable` | Enable LFS tracking globally for eggs |
| `disable` | Disable LFS tracking |

### Flags

| Flag | Description |
|------|-------------|
| `--remote <name>` | git remote name (default: `origin`) |
| `--server <url>` | LFS server URL |
| `--verbose` (`-v`) | Verbose output |

### Examples

```
# Track a produced ISO in git-LFS
sudo eggs lfs track /home/eggs/egg-debian-amd64.iso

# List all LFS-tracked ISOs
eggs lfs list /home/eggs

# Set up LFS with a custom server
sudo eggs lfs setup --server https://lfs.example.com

# Enable LFS tracking for all future eggs produce runs
sudo eggs lfs enable
```

### Integration with eggs produce

The `--lfs` flag on `eggs produce` automatically tracks the produced ISO in
git-LFS after the build completes:

```
sudo eggs produce --lfs
```

---

## eggs ipfs — IPFS distribution via brig

`eggs ipfs` manages decentralized distribution of ISOs via
[IPFS](https://ipfs.tech/) using [brig](https://github.com/sahib/brig) as the
backend. brig is a distributed, encrypted file synchronization tool built on
IPFS that provides a familiar filesystem interface.

### Prerequisites

brig must be installed. See [github.com/sahib/brig](https://github.com/sahib/brig)
for installation instructions.

### Commands

```
eggs ipfs <action> [path]
```

| Action | Description |
|--------|-------------|
| `publish <iso-path>` | Publish an ISO to IPFS and return its CID |
| `list` | List all ISOs published to IPFS |
| `get <brig-path>` | Download an ISO from IPFS |
| `gateway` | Print the IPFS gateway URL for published ISOs |
| `history [brig-path]` | Show publish history |
| `enable` | Enable IPFS publishing for eggs |
| `disable` | Disable IPFS publishing |
| `setup` | Configure brig for use with eggs |

### Flags

| Flag | Short | Description |
|------|-------|-------------|
| `--dest <path>` | `-d` | Destination path for `get` |
| `--backend <name>` | | IPFS backend: `brig`, `raw-ipfs`, `ipgit` |
| `--remote <name>` | | brig remote name |
| `--verbose` | `-v` | Verbose output |

### Examples

```
# Publish an ISO to IPFS
sudo eggs ipfs publish /home/eggs/egg-debian-amd64.iso

# List published ISOs
eggs ipfs list

# Download an ISO from IPFS
eggs ipfs get /isos/egg-debian-amd64.iso --dest /tmp/

# Show the gateway URL
eggs ipfs gateway

# View publish history
eggs ipfs history /isos/egg-debian-amd64.iso
```

After publishing, the command prints the CID and (if configured) a gateway URL
for sharing:

```
Published: CID=bafybeig...
Size: 1024.3 MB
Gateway: https://ipfs.io/ipfs/bafybeig...
```

### Integration with eggs produce

The `--ipfs` flag on `eggs produce` automatically publishes the produced ISO to
IPFS after the build completes:

```
sudo eggs produce --ipfs
```

---

## eggs st — System Transparency

`eggs st` produces [System Transparency](https://system-transparency.org/)
compatible boot artifacts from eggs ISOs. System Transparency is a security
framework that provides cryptographically signed OS packages, enabling
verifiable boot chains for servers and appliances.

### How it works

System Transparency packages an OS into a signed bundle containing:

- A kernel and initrd extracted from the ISO
- A descriptor JSON file with metadata and hashes
- An Ed25519 signature over the descriptor

Firmware that implements System Transparency verifies the signature before
booting, ensuring the OS has not been tampered with.

### Commands

```
eggs st <action> [path]
```

| Action | Description |
|--------|-------------|
| `package <iso-path>` | Create an ST-compatible OS package from an ISO |
| `keygen` | Generate an Ed25519 key pair for signing |
| `verify <descriptor-path>` | Verify the signature on an ST package |
| `extract <iso-path>` | Extract kernel and initrd from an ISO |

### Flags

| Flag | Description |
|------|-------------|
| `--label <name>` | OS package label |
| `--key <path>` | Ed25519 private key for signing |
| `--pubkey <path>` | Ed25519 public key for verification |
| `--output <dir>` (`-o`) | Output directory |
| `--verbose` (`-v`) | Verbose output |

### Examples

```
# Generate a key pair
eggs st keygen --output /etc/penguins-eggs.d/st-keys/

# Package an ISO as an ST bundle
sudo eggs st package /home/eggs/egg-debian-amd64.iso \
  --label my-distro \
  --key /etc/penguins-eggs.d/st-keys/key.pem

# Verify a package
eggs st verify /tmp/st-output/descriptor.json \
  --pubkey /etc/penguins-eggs.d/st-keys/key.pub

# Extract kernel and initrd from an ISO
sudo eggs st extract /home/eggs/egg-debian-amd64.iso \
  --output /tmp/components
```

After packaging, the output directory contains:

```
/tmp/st-my-distro/
├── bundle.tar.gz       # ST OS package bundle
├── descriptor.json     # Metadata and hashes
└── descriptor.json.sig # Ed25519 signature
```

---

## --sbom: Software Bill of Materials

The `--sbom` flag on `eggs produce` generates a Software Bill of Materials
(SBOM) for the produced ISO using [syft](https://github.com/anchore/syft).

### Prerequisites

syft must be installed:

```
curl -sSfL https://raw.githubusercontent.com/anchore/syft/main/install.sh | sh -s -- -b /usr/local/bin
```

### Usage

```
sudo eggs produce --sbom
```

The SBOM file is placed alongside the ISO in the output directory:

```
/home/eggs/
├── egg-of_debian-bookworm-amd64.iso
└── egg-of_debian-bookworm-amd64.sbom.json
```

The SBOM is in CycloneDX JSON format and lists all packages included in the
ISO with their names, versions, licenses, and checksums. This is useful for:

- License compliance auditing
- Vulnerability scanning (feed the SBOM to tools like
  [grype](https://github.com/anchore/grype))
- Supply chain transparency

### Combining flags

Distribution and security flags can be combined with each other and with other
`eggs produce` flags:

```
# Produce, generate SBOM, track in LFS, and publish to IPFS
sudo eggs produce --sbom --lfs --ipfs

# Produce with max compression, SBOM, and ST package
sudo eggs produce --max --sbom
sudo eggs st package /home/eggs/egg-of_debian-bookworm-amd64.iso \
  --label debian-bookworm \
  --key /etc/penguins-eggs.d/st-keys/key.pem
```

---

## BTRFS snapshots

The `--snapshot` flag on `eggs produce` creates BTRFS snapshots of the system
before and after the build. This requires the root filesystem to be on BTRFS.

```
sudo eggs produce --snapshot
```

Snapshots are labelled `pre-eggs-produce-<timestamp>` and
`post-eggs-produce-<timestamp>`. They can be used to roll back the system if
the build process causes unexpected changes, or to compare the system state
before and after.
