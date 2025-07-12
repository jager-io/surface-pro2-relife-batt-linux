
# 🖥️ Arch Linux on Microsoft Surface Laptop 2

Welcome to the documentation for turning a discontinued Surface Laptop 2 into a modern, fast, and functional Arch Linux machine. This guide captures the full installation journey—from bare-metal prep and driver injection to wireless fixes, desktop setup, and aesthetic polish.

> 💡 Built from scratch, debugged offline, and revived with a secondhand battery in Shanghai.

---

## 📌 Highlights

* ✅ Dual-stage USB boot and manual partitioning
* ✅ Offline Arch install with no network tools
* ✅ Custom `linux-surface` kernel integration
* ✅ Wi-Fi fix for Marvell 88W8897 (manual firmware injection)
* ✅ Full KDE Plasma desktop with power tools
* ✅ Hardware-level recovery and personalization

---

## 🔧 Hardware Specs

| Component | Info                           |
| --------- | ------------------------------ |
| Model     | Microsoft Surface Laptop 2     |
| CPU       | Intel Core i5-8250U            |
| RAM       | 8GB                            |
| Storage   | 256GB SSD                      |
| Screen    | 13.5” PixelSense Touch Display |
| Wi-Fi     | Marvell AVASTAR 88W8897        |

---

## 🚀 Installation Overview

**Stage 1: Prepare Bootable USB**

* ISO downloaded from [archlinux.org](https://archlinux.org/download/)
* Used Rufus with `dd` mode to create a bootable Arch HDD
* Disabled Secure Boot in Surface UEFI

**Stage 2: Partitioning & Base Install**

* Used `cfdisk` to manually delete partitions and create:

  * 512MB EFI System partition
  * Remaining root partition (ext4)
* Filesystem formatting:

  ```bash
  mkfs.fat -F32 /dev/nvme0n1p1
  mkfs.ext4 /dev/nvme0n1p2
  ```
* Mounted drives and ran `pacstrap` to install Arch

**Stage 3: Bootloader**

* Installed `systemd-boot`
* Created custom boot entry with proper `PARTUUID`
* Set defaults and timeout in `/boot/loader/loader.conf`

---

## 📡 Wi-Fi Fix: Marvell 88W8897

The internal Wi-Fi did not function post-install. The fix required:

1. Downloading `pcie8897_uapsta.bin` from `linux-firmware` Git
2. Transferring it to `/lib/firmware/mrvl/` via USB
3. Rebuilding `initramfs`:

   ```bash
   mkinitcpio -g /boot/initramfs-linux-surface.img -k $(uname -r)
   ```
4. Enabling NetworkManager and connecting via `nmtui`

✅ Result: Wi-Fi operational through internal chipset.

---

## 🧱 KDE Plasma + Applications

```bash
sudo pacman -S plasma kde-applications sddm
sudo systemctl enable sddm
reboot
```

Plasma desktop is now the primary GUI, with full SDDM login and KDE tools available.

---

## 🔋 Battery Replacement Log

* 🛒 Replacement battery purchased from Taobao: ¥118 CNY (\~\$16 USD)
* 🛠️ Installation service by local technician: ¥500 CNY (\~\$69 USD)

Instead of paying Microsoft’s \$250–\$400 service cost, a trusted neighborhood repair shop performed a successful battery swap. This choice not only saved money but preserved control over the device’s future.

---

## 🧼 Aesthetics & Restoration

After battery replacement:

* Old stickers removed using isopropyl alcohol
* Surface chassis cleaned and refreshed
* New decals applied for a sharp, minimal look

👉 [View the cleaned Surface](IMAGES/clean.jpeg)
👉 [View new stickers](IMAGES/NEW.jpeg)

---

## 🧭 Files and Structure

```
├── README.md            # Main project documentation
├── NOTES.md             # Terminal logs, errors, and fixes
├── AESTHETICS.md        # Visual polish and mods
├── TODO.md              # Remaining setup and post-install tasks
└── IMAGES/              # Screenshots and photos
```

---

## 🔮 What's Next

* Surface Control tools from AUR
* VPN integration (WireGuard or Clash)
* Desktop ricing with Kvantum and custom icons
* System hardening and backup strategy

---

## 🧠 Philosophy

This project isn’t just about installing Arch—it’s about ownership, transparency, and reviving hardware that still has value. Every error solved and every workaround documented is part of the story. No bloat. No cloud lock-in. Just a powerful machine running on one’s own terms.


