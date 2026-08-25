---
title: "Convert Ubuntu 24.04 from BIOS to EFI on VMware"
categories: [Linux, VMware]
tags: [ubuntu, uefi, bios, grub, vmware]
date: 2026-08-24 11:51:00 +0330
image:
    path: assets/posts/images/convert_ubuntu_2404_from_BIOS_to_EFI.png
    alt: convert_ubuntu_2404_from_BIOS_to_EFI
categories: [Linux, Virtualization]
tags:
  - linux
  - vmware
author: arman
toc: true
published: true
---
I had an Ubuntu 24.04 VM template running in BIOS mode on VMware vCenter. Instead of reinstalling Ubuntu, I converted the existing installation to UEFI.

The original disk looked like this:

```text
/dev/sda
├── sda1   1M     bios_grub
└── sda2   80G    LVM
```

The disk had plenty of unused space after `sda2`, so I created a 512 MB EFI System Partition.

## 1. Fix GPT if the disk was expanded

```bash
sudo parted /dev/sda
```

If you get:

```text
Warning: Not all of the space available to /dev/sda appears to be used
```

choose:

```text
Fix
```

## 2. Create the EFI partition

```bash
sudo parted /dev/sda
```

```text
mkpart EFI fat32 80.5GB 81GB
set 3 esp on
quit
```

Format it:

```bash
sudo mkfs.fat -F32 /dev/sda3
```

## 3. Mount the EFI partition

```bash
sudo mkdir -p /boot/efi
sudo mount /dev/sda3 /boot/efi
```

## 4. Install UEFI GRUB

```bash
sudo apt install grub-efi-amd64 shim-signed
```

```bash
sudo grub-install \
  --target=x86_64-efi \
  --efi-directory=/boot/efi \
  --bootloader-id=ubuntu \
  --recheck
```

```bash
sudo update-grub
```

## 5. Add EFI to `/etc/fstab`

Get the UUID:

```bash
sudo blkid /dev/sda3
```

Add:

```fstab
UUID=XXXX-XXXX  /boot/efi  vfat  umask=0077  0  1
```

Test it:

```bash
sudo umount /boot/efi
sudo mount -a
findmnt /boot/efi
```

## 6. Change VMware firmware

Shut down the VM and change:

**vCenter → VM Settings → VM Options → Boot Options → Firmware**

from:

```text
BIOS
```

to:

```text
EFI
```

Boot the VM.

Verify:

```bash
test -d /sys/firmware/efi && echo "UEFI" || echo "BIOS"
```

Expected:

```text
UEFI
```

## Final layout

```text
/dev/sda
├── sda1   1M       bios_grub
├── sda2   80G      LVM
└── sda3   512M     EFI System Partition
```

The existing Ubuntu installation, LVM, `/boot`, and data remain unchanged. No reinstall is required.

> Always take a snapshot or backup before modifying the partition table or bootloader.
