# 🧾 Lazarus Build Log

Arch Linux + linux-surface on Microsoft Surface Laptop 2

> Status: **Stable boot**, **Wi-Fi functional**, **KDE Plasma active**

---

## 🔧 Phase 1: Bootloader & Arch Install

### Resources

* [linux-surface GitHub](https://github.com/linux-surface/linux-surface)
* [Arch USB creation guide](https://wiki.archlinux.org/title/USB_flash_installation_medium)
* [Arch ISO download](https://archlinux.org/download/)
* [Disable Secure Boot](https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/disabling-secure-boot?view=windows-11)

### Steps Taken

1. Downloaded Arch ISO via torrent.
2. Used **Rufus** to burn ISO to USB:

   * Version: `rufus-4.9.exe (Windows x64)`
3. Entered Surface UEFI:

   * Volume Up + Power
   * Boot Configuration → Moved **USB Storage** to the top
4. Disabled **Secure Boot**.
5. Booted into Arch successfully.

---

## 📶 Wi-Fi Temporarily Skipped (to be fixed later)

Used `iwctl` initially to try connecting — but the Marvell wireless card was unsupported at this stage.

---

## ⚠️ ERROR001: Partitioning Failure

### Issue

```
failed to commit transaction (not enough free disk space)
```

### Fix

Repartitioned using `cfdisk`:

```bash
cfdisk /dev/nvme0n1
```

* Deleted all partitions
* Created:

  * 512M → EFI System
  * Rest → Linux Filesystem (ext4)

### Then Formatted

```bash
mkfs.fat -F32 /dev/nvme0n1p1
mkfs.ext4 /dev/nvme0n1p2
```

### Mounted

```bash
mount /dev/nvme0n1p2 /mnt
mkdir /mnt/boot
mount /dev/nvme0n1p1 /mnt/boot
```

### Confirmed

```bash
df -h /mnt
# Showed 1–2G used, 220G+ free
```

---

## 📦 Bootstrap Installation

```bash
pacstrap /mnt base linux linux-firmware nano networkmanager
```

✅ SSD and filesystems confirmed working.

---

## 🏗️ Base Config

```bash
genfstab -U /mnt >> /mnt/etc/fstab
arch-chroot /mnt
```

```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systohc
```

```bash
echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

```bash
echo "lazarus" > /etc/hostname
systemctl enable NetworkManager
passwd
```

---

## 🧹 Bootloader (systemd-boot)

```bash
bootctl install
```

### `arch.conf`

```bash
nano /boot/loader/entries/arch.conf
```

```ini
title   Arch Linux
linux   /vmlinuz-linux
initrd  /initramfs-linux.img
options root=PARTUUID=<copied-from-blkid> rw
```

### `loader.conf`

```bash
nano /boot/loader/loader.conf
```

```ini
default arch
timeout 3
editor no
```

Rebooted, unplugged USB — system booted into Arch.

---

## 💻 Phase 2: Surface Kernel & Wi-Fi

### Problem

* Marvell 88W8897 Wi-Fi chip not supported out of the box
* `ip link` showed only `lo`
* No tethering, dongles, or Ethernet available

---

## 🛠️ Wi-Fi Fix: Marvell 88W8897 Firmware

1. Download firmware from:

   ```
   https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/plain/mrvl/pcie8897_uapsta.bin
   ```
2. Transfer via USB:

```bash
mkdir -p /lib/firmware/mrvl
cp /mnt/usb/pcie8897_uapsta.bin /lib/firmware/mrvl/
```

3. Rebuild initramfs (no preset):

```bash
uname -r  # → 6.15.4-arch2-1

mkinitcpio -g /boot/initramfs-linux-surface.img -k 6.15.4-arch2-1
```

4. Reboot → `ip link` showed:

```
wlp2s0 <BROADCAST, MULTICAST, UP>
```

5. Installed NetworkManager:

```bash
pacman -Sy networkmanager
systemctl enable NetworkManager
systemctl start NetworkManager
nmtui
```

✅ Connected successfully via `nmtui`.

---

## 🧱 Post-WiFi Setup: KDE Install & Userland

### System Setup

```bash
pacman -S sudo git base-devel nano
EDITOR=nano visudo  # → Uncomment %wheel ALL=(ALL:ALL) ALL
useradd -m -G wheel j*****r
passwd j*****r
```

Switched to `j*****r`, tested `sudo`.

---

## 🖥️ Desktop Environment

```bash
sudo pacman -S plasma kde-applications sddm
```

All groups installed using defaults.

### Finalize Desktop

```bash
sudo systemctl enable sddm
reboot
```

✅ KDE Plasma active on reboot.

---

## 🔜 Next Steps

1. Install `yay` AUR helper (VPN/mirrors still unstable).
2. Install:

   ```bash
   yay -S surface-control surface-control-daemon
   ```
3. Polish: ricing, theme configs, dotfiles.
4. Add remaining firmware, Surface-specific modules.
5. Harden system, publish full `INSTALL.md`, `notes/`, and screenshots to GitHub.

---
