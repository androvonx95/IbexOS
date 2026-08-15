# IBEX OS

> **Almost Rust. GNU-free Linux.**

<p align="center">
  <img src="https://github.com/user-attachments/assets/a2c1cefd-ee6a-4828-9cf5-5f10e8dddccd" width="700" height="500" alt="image">
</p>

IbexOS is a hyper-minimalist, bare-metal Linux distribution engineered entirely from scratch. Built on a pure `musl` libc and BusyBox userland, it strips away `glibc` bloat and legacy dependencies to provide a hyper-efficient, independent operating environment.

This OS is designed to be the ultimate foundation for embedded IoT architectures, Decentralized Physical Infrastructure Networks (DePIN), custom GPU hardware rigs, and systems where standard distributions carry too much overhead.

## Core Architecture

* **GNU-Free Userland:** Pure `musl` libc integration with a lightweight BusyBox environment.
* **Live In-Memory Boot:** Custom `initramfs` leveraging an optimized read-only SquashFS combined with an in-memory OverlayFS.
* **Native UEFI Bare-Metal Installer:** Dynamic disk partitioning (GPT/ESP/ext4), automated hardware UUID mapping, and pure native Limine bootloader injection.
* **Security & Privilege:** Configured with robust SUID privilege escalation and standard multi-user separation natively out of the box.

## Current Milestones & Roadmap

IbexOS is currently booting independently on bare metal. We are actively seeking maintainers and contributors for the next phase of core infrastructure:

* [x] Custom Linux Kernel & Initramfs handoff
* [x] OverlayFS / SquashFS Live Environment
* [x] Interactive UEFI Bare-Metal Installer
* [x] **Package Management:** Native `apk` integration successfully ported for dynamic dependency resolution.
* [x] **Service/Init Manager:** Integrated OpenRC for fast, dependency-aware process management.
* [ ] **Networking Stack:** Moving beyond DHCP to full `wpa_supplicant`/`iwd` integration.
* [ ] **Display Server:** Implementation of a Wayland-based graphical stack.

## 🚀 Installation Guide

IbexOS provides a stateless Live ISO with a built-in, automated deployment tool. You can install IbexOS natively on bare metal or inside a virtual machine (e.g., QEMU, VirtualBox).

### 1. Boot the Live Environment

Download the latest `ibexos-live.iso` from the Releases page.

* **Virtual Machine:** Attach the ISO as a CD-ROM drive and boot using UEFI firmware (OVMF).
* **Bare Metal:** Flash the ISO to a USB drive using tools like `Rufus` or `dd` and boot your machine in UEFI mode.

### 2. Run the Installer

Once the Live Environment finishes booting, you will be dropped into a root shell (`~ #`). To begin the installation, simply run the deployment tool:

```bash
ibex-install
```

### 3. Follow the Prompts

The installer will orchestrate the partitioning, formatting, and bootstrapping automatically. You will be prompted to provide:

* **Target Disk:** The drive you want to install IbexOS on (e.g., `/dev/vda`, `/dev/nvme0n1`, `/dev/sda`). *Warning: This drive will be completely wiped.*
* **System Hostname:** A single word to identify your machine on a network.
* **New Username:** Your standard user account name.
* **Password:** You will be prompted to enter and verify a secure password for your new user.

### 4. Reboot into IbexOS

When the installer outputs `[+] Deployment complete.`, the operating system is permanently installed on your drive.

Power off the live environment:

```bash
poweroff -f
```

Remove the installation media, and boot directly into your new, persistent IbexOS environment!

## Build Instructions

To build the toolchain and master the IbexOS Live ISO, you will need an Arch Linux host environment.

### Dependencies

Ensure you have the following installed on your host:

```bash
sudo pacman -S base-devel squashfs-tools xorriso qemu-full
```

### Mastering the ISO

Run the following commands in the root of the repository to bake the filesystem, set core permissions, and package the Live CD:

```bash
# Set critical SUID bits for privilege escalation
sudo chmod 4755 target-root/bin/busybox
sudo chmod 4755 target-root/bin/su 2>/dev/null || true

# Ensure base filesystem structure
sudo mkdir -p target-root/home

# Bake the SquashFS (ZSTD compressed)
sudo rm -f iso_build/boot/rootfs.squashfs
sudo mksquashfs target-root/ iso_build/boot/rootfs.squashfs -comp zstd -noappend

# Master the Limine UEFI Bootable ISO
xorriso -as mkisofs -b boot/limine-bios-cd.bin \
        -no-emul-boot -boot-load-size 4 -boot-info-table \
        --efi-boot boot/limine-uefi-cd.bin \
        -efi-boot-part --efi-boot-image --protective-msdos-label \
        iso_build -o ibexos-live.iso
```

### Testing the Build

You can test the Live ISO and run the interactive installer in QEMU:

```bash
# Create a blank target drive
qemu-img create -f qcow2 target.qcow2 10G

# Boot the Live Environment
qemu-system-x86_64 \
    -drive if=pflash,format=raw,readonly=on,file=/usr/share/edk2/x64/OVMF_CODE.4m.fd \
    -drive if=pflash,format=raw,file=/tmp/ovmf_vars.fd \
    -drive file=target.qcow2,format=qcow2,if=virtio \
    -cdrom ibexos-live.iso \
    -m 2G -serial stdio
```

## Contributing

Pull requests are welcome. For major architectural changes (like package manager proposals), please open an issue first to discuss what you would like to change.

## Author

**androvonx95**

