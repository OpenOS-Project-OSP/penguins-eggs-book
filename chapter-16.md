# Chapter 16

# RISC-V and the MuseBook port

The `all-features` branch adds `riscv64` as a supported architecture. This
chapter documents RISC-V support in penguins-eggs and the ongoing port to the
MuseBook (Spacemit K1) single-board computer.

---

## RISC-V support in penguins-eggs

RISC-V (`riscv64`) is supported alongside `i386`, `amd64`, and `arm64`. The
main addition for RISC-V (and ARM) targets is the `--dtbdir` flag, which
specifies the path to Device Tree Blob (DTB) files.

### Device Tree Blobs

On ARM and RISC-V systems, the bootloader needs DTB files to describe the
hardware to the kernel. When producing an ISO for these targets, pass the
directory containing the relevant `.dtb` files:

```
sudo eggs produce --dtbdir /path/to/dtbs
```

For example, on a Raspberry Pi 4:

```
sudo eggs produce --dtbdir /usr/lib/linux-image-6.6.0-rpi4/broadcom
```

On a MuseBook (Spacemit K1):

```
sudo eggs produce --dtbdir /boot/dtbs/6.6.63/spacemit
```

eggs copies the DTB files into the ISO's boot structure so the bootloader can
find them at runtime.

### Android RISC-V

`eggs android produce` also supports `riscv64` for Android environments running
on RISC-V hardware. ISO mode is supported for this architecture:

```
sudo eggs android produce --mode=iso --arch=riscv64
```

---

## The MuseBook (Spacemit K1) port

The MuseBook is a RISC-V laptop powered by the Spacemit K1 chip, running
Bianbu Linux. The `musebook/` directory in the `all-features` branch documents
the ongoing work to port penguins-eggs to this hardware.

### Hardware overview

| Component | Details |
|-----------|---------|
| SoC | Spacemit K1 (8-core RISC-V, RV64GCVB) |
| OS | Bianbu Linux (Debian-based) |
| Architecture | riscv64 |
| Boot firmware | U-Boot |

### Partition layout

The MuseBook uses a 6-partition GPT layout required by the Bianbu bootloader:

| Partition | Filesystem | Contents |
|-----------|-----------|---------|
| 1 | raw | Bootloader header (sector 0: `boot_header_sector0.bin`) |
| 2 | raw | SPL (`spl.bin` at sector 2048) |
| 3 | raw | U-Boot (`uboot.bin` at sector 4096) |
| 4 | raw | U-Boot environment (`env.bin` at sector 256) |
| 5 | ext4 | Boot filesystem (`bootfs`): kernel, DTB, extlinux.conf, `env_k1-x.txt` |
| 6 | ext4 | Root filesystem (`rootfs`): `/live/filesystem.squashfs` |

The bootloader components must be injected at specific sector offsets using
atomic writes, not standard partition tools.

### Bootloader injection

```bash
# SDC signature at sector 0
dd if=boot_header_sector0.bin of=/dev/sdX bs=512 seek=0 conv=notrunc

# U-Boot environment at sector 256
dd if=env.bin of=/dev/sdX bs=512 seek=256 conv=notrunc

# SPL at sector 2048
dd if=spl.bin of=/dev/sdX bs=512 seek=2048 conv=notrunc

# U-Boot at sector 4096
dd if=uboot.bin of=/dev/sdX bs=512 seek=4096 conv=notrunc
```

The bootloader binaries (`spl.bin`, `uboot.bin`, `env.bin`,
`boot_header_sector0.bin`) are included in `musebook/` and were extracted from
a working Bianbu SSD installation.

### Boot configuration

U-Boot on the MuseBook reads `env_k1-x.txt` from partition 5 (bootfs). The
file sets the kernel path, DTB path, and boot arguments:

```
kernel=/boot/vmlinuz-6.6.63
dtb=/boot/dtbs/6.6.63/spacemit/k1-x_MUSE-Book.dtb
bootargs=earlycon=sbi clk_ignore_unused swiotlb=65536 console=tty1 loglevel=8
```

The live system's `filesystem.squashfs` is placed on partition 6 (rootfs) at
`/live/filesystem.squashfs`.

### Current status and known issues

The port has successfully replicated the original Bianbu 6-partition layout and
the MuseBook recognizes the SD card and starts U-Boot (the Bianbu logo
appears). However, a reboot loop occurs after the logo screen.

**Observed behaviour:**
- The Bianbu logo appears, the system waits a few seconds, then clears the
  screen and reboots
- U-Boot appears to ignore `extlinux.conf` and `env_k1-x.txt` placed on
  partition 5
- No kernel logs appear before the reset, even with `earlycon=sbi` in bootargs

**Open questions:**
1. Does the stock U-Boot for the MuseBook have a hardcoded preference for the
   config file location? It may only look at partition 1 even though partition 5
   is the designated bootfs in the original layout.
2. Is a compiled `.scr` script (rather than a plain text `env_k1-x.txt`)
   required to trigger loading of partition 5 files?
3. Why does the kernel fail to initialize the console even with `earlycon=sbi`
   enabled?

**Debugging without a serial console** is the main challenge. The MuseBook's
built-in display is the only output available during early boot.

### Useful references

- [Bianbu multiboot guide](https://www.workswithriscv.guide/wiki/hardware/K1/bianbu-multiboot.html)
- [penguins-eggs repository](https://github.com/pieroproietti/penguins-eggs)
- [ISO downloads](https://sourceforge.net/projects/penguins-eggs/files/Isos/)

### Contributing

If you have experience with U-Boot on the Spacemit K1 or similar RISC-V
hardware, contributions to the MuseBook port are welcome. The key areas where
help is needed:

- Determining the correct U-Boot config file location and format for the K1
- Debugging early boot without a serial console
- Testing the 6-partition layout with different bootloader configurations

Open issues and discussion on the
[penguins-eggs repository](https://github.com/pieroproietti/penguins-eggs).
