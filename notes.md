Notes
### 2025-07-05 – Bootloader Setup
- Followed the "linux-surface" github page
- [https://github.com/linux-surface/linux-surface]
- *There is a warning about dish encryption, this can wait until ARCH is installed*
- then it  took me to [https://wiki.archlinux.org/title/USB_flash_installation_medium]
Then I downloaded arch on a portable HDD drive [https://archlinux.org/download/]
I used the torrent as recommended
Then I went here to figure out how to disable secure boot [https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/disabling-secure-boot?view=windows-11]
*I had a problem with the bitlocker, I forgot to burn the ISO file into the USB*
-go to [https://rufus.ie/en/] download the rufus-4.9.exe	Standard	Windows x64 version 

### 2025-07-06 
- Now I had to gb to redownload the ISO file [https://archlinux.org/download/]
- I watched this video on using rufus [https://www.youtube.com/watch?v=GvI0oyTsuxI&ab_channel=SmitsRefined]
- Then I followed this intstructions.........
- 
- Plug it into the Surface
-Enter Surface UEFI (Volume Up + Power)
-Go to Boot Configuration → drag USB Storage to the top
-Make sure it's your Arch USB (not Windows installer)
-Save and reboot

 - Then it installed arch and correctly borught up the boat loader
-first I connect to wifi 
-iwctl
-station wlan0 scan
-station wlan0 get-networks
-station wlan0 connect YourSSID
-exit
-----------------ERROR001----------------
- when trying to partition the disk and install the base system I encountered this error
- " "failed to commit transaction (not enough free disk space) Errors occured. no packages were upgraded"
- The solution was to wipe the partitions completely
"- 1. Run cfdisk

-cfdisk /dev/nvme0n1
You’ll see a partition layout.

2. Delete all existing partitions:
-Use arrow keys → highlight each partition (EFI, root, etc.)

-Select Delete

-Do this until it says “Free Space”

3. Create two new partitions:
-EFI partition:

-Type: EFI System

-Size: 512M

-Root partition:

-Use the rest of the space

-Type: Linux filesystem

-When done, select Write, confirm with yes, then Quit."

--- THEN THIS
-. Create fresh partitions

-cfdisk /dev/nvme0n1
-Create:

512M → EFI System

-Remaining (~238G) → Linux filesystem

Write and quit
---------------I forgot to format---------------------------
4. Format them   

-mkfs.fat -F32 /dev/nvme0n1p1
-mkfs.ext4 /dev/nvme0n1p2

5. Mount and double-check space

-mount /dev/nvme0n1p2 /mnt
-mkdir /mnt/boot
-mount /dev/nvme0n1p1 /mnt/boot

-df -h /mnt
✅ You should now see something like:

-Filesystem      Size  Used Avail Use% Mounted on
-/dev/nvme0n1p2  238G  1.6G  225G   1% /mnt

---now its time for Arch’s version of bootstrapping
-pacstrap /mnt base linux linux-firmware nano networkmanager
-✅ What This Means:
-Your SSD is working
-The partitions are correct
-Filesystem is clean
-You’ve successfully “bootstrapped” your Arch Linux install

--- Now Do These Next Steps:
1. Generate fstab

genfstab -U /mnt >> /mnt/etc/fstab
2. Chroot into your new system

arch-chroot /mnt
You’re now inside your real Arch system on the internal drive.

 ---Then Finish the Base Setup (ARCH BASE SETUP)---------------------------
Timezone

ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
hwclock --systohc
Locale

echo "en_US.UTF-8 UTF-8" >> /etc/locale.gen
locale-gen
echo "LANG=en_US.UTF-8" > /etc/locale.conf
Hostname

echo "lazarus" > /etc/hostname
Enable networking

systemctl enable NetworkManager
Set root password

passwd
--- Bootloader Setup (systemd-boot)
Perfect for Surface hardware.

Install systemd-boot

bootctl install
Create the boot entry

nano /boot/loader/entries/arch.conf
Paste this in:

title   Arch Linux
linux   /vmlinuz-linux
initrd  /initramfs-linux.img
options root=PARTUUID=XXXX rw
Get the PARTUUID:

blkid /dev/nvme0n1p2
Copy the full PARTUUID=... and paste it into that config.

---Finalize bootloader

nano /boot/loader/loader.conf
Paste:

default arch
timeout 3
editor no

---FINALLY
exit
reboot

---My arch successfully reboot, somewhow
I set the UEFI to "internal Storage" and unplugged the HDD drive. 

--- Arch is live 

 --- Add the Surface Linux Repo & Keys
I cannot add the surface linux repo because the WIFI is messed up 
RECAP: 
Understood. Here's the same write-up without tables, restructured for clean Markdown or GitHub use:

---

## 🧾 Workstation Build Log: Surface Laptop 2 (Arch Linux Install + Surface Kernel)

### Phase: Surface Kernel Setup & Hardware Enablement

**Objective:** Enable internal keyboard, touchpad, and Wi-Fi on a Microsoft Surface Laptop 2 running Arch Linux by installing the custom `linux-surface` kernel — without an internet connection.

---

### References Used

* [`linux-surface` README](https://github.com/linux-surface/linux-surface/blob/master/README.md)
* [`linux-surface` Installation & Setup (Arch)](https://github.com/linux-surface/linux-surface/wiki/Installation-and-Setup#Arch)
* [Arch Linux Installation Guide](https://wiki.archlinux.org/title/Installation_guide)

---

### Initial System State

* Arch Linux was installed and booting from disk on the Surface Laptop 2.
* Hostname set to **workstation**.
* systemd-boot configured and functional.
* Disk partitioned using GPT: 512MB EFI system partition and the rest as ext4 root.
* No internet access available due to:

  * No internal Wi-Fi recognition (kernel limitation)
  * No Ethernet dongle on hand
  * iPhone available, but not tethered
  * No Android phone (USB tethering unavailable)

---

### Problems Encountered and Solutions

* GPG key fetch failed due to offline status. Solved by manually downloading the `.key` file and importing it into `pacman-key`.
* Wi-Fi interface was not showing in `ip link` or `nmtui`. This was expected without the `linux-surface` kernel installed.
* Attempting to mount a USB stick failed with `fsconfig()` errors. The USB was formatted as NTFS, which Arch does not support natively without `ntfs-3g`.
* Tried installing `ntfs-3g` via `pacman`, but failed due to lack of network connectivity.
* The fallback approach was to manually download the kernel `.zst` files on another laptop and transfer them via USB.
* Windows did not allow formatting the USB drive as FAT32 (only NTFS and exFAT offered) due to drive size.
* Used `diskpart` to force a 1GB FAT32 partition:

  * `clean` wiped the disk
  * `convert mbr` initialized the disk
  * A 1GB primary partition was created
  * The FAT32 format command initially failed due to "no volume selected" — fixed by selecting the partition before formatting.
* After reformatting, the `.blob` files were renamed to `.pkg.tar.zst` and successfully copied to the USB.

---

### Manual Package Installation Process

1. On the second laptop, visited: `https://pkg.surfacelinux.com/arch/x86_64/`
2. Downloaded:

   * `linux-surface`
   * `linux-surface-headers`
   * `surface-aggregator-module-dkms`
3. Renamed each `.blob` file to `.pkg.tar.zst`
4. Used `diskpart` to reformat the USB drive with FAT32 (1GB)
5. On the Surface laptop (Arch system):

   * Mounted the USB:

     ```bash
     mount /dev/sda1 /mnt/usb
     ```
   * Copied files:

     ```bash
     cp /mnt/usb/*.pkg.tar.zst /root/
     cd /root
     pacman -U *.pkg.tar.zst
     ```

---

### Next Steps
### HOW TO FIX THE WIFI 

# 🩺 Wi-Fi Fix Recap: Surface Laptop 2 (Marvell 88W8897)

## 🔍 Problem

After installing Arch and the `linux-surface` kernel, no network interfaces appeared (`ip link` only showed `lo`).
Running `lspci` revealed:

```
02:00.0 Ethernet controller: Marvell 88W8897 [AVASTAR] 802.11 Wireless
```

This Wi-Fi chip **is not supported out of the box**, even with `linux-firmware`. Manual installation was required.

---

## 🛠️ Solution Steps

### 1. Manually download required firmware

On a second laptop with internet access:

* Navigate to:

  ```
  https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/plain/mrvl/pcie8897_uapsta.bin
  ```
* Save the file to a USB stick

### 2. Transfer firmware to Lazarus

On the Arch system:

```bash
mkdir -p /mnt/usb
mount /dev/sda1 /mnt/usb
mkdir -p /lib/firmware/mrvl
cp /mnt/usb/pcie8897_uapsta.bin /lib/firmware/mrvl/
```

### 3. Rebuild initramfs (no preset available)

Since the `linux-surface` preset was missing:

```bash
uname -r  # output: 6.15.4-arch2-1

mkinitcpio -g /boot/initramfs-linux-surface.img -k 6.15.4-arch2-1
```

### 4. Reboot and confirm device

```bash
reboot
ip link
```

✅ Result:

```
wlp2s0 <BROADCAST, MULTICAST, UP, ...>
```

Wi-Fi interface now visible.

### 5. Install and activate NetworkManager

```bash
pacman -Sy networkmanager
systemctl enable NetworkManager
systemctl start NetworkManager
nmtui
```

Used `nmtui` to connect to Wi-Fi — confirmed connectivity with `ping`.

---

## ✅ Status: Wi-Fi functional

Lazarus now has wireless connectivity through the Marvell 88W8897 chipset using a manual firmware injection. No external dongles or tethering required.

###   Wi-Fi restored via manual Marvell firmware install (88W8897)
   ```
Absolutely — here’s a concise, no-fluff progress update to append to your Lazarus GitHub project:

---

### 🧱 Post-WiFi Setup Progress: KDE Install & Userland Configuration

Following successful Wi-Fi restoration using the Marvell 88W8897 firmware injection, focus shifted to building out the core Arch environment and preparing Lazarus for daily driver use.

### ✅ System Configuration Steps

* Installed `sudo` and configured `visudo` to allow `wheel` group access
* Created the `j***r` user and added to `wheel` group
* Confirmed working `sudo` privileges from non-root account
* Installed essential system tools: `nano`, `git`, `base-devel`
* Attempted but postponed Surface-specific tools (AUR access requires further networking/VPN setup)
* Proceeded to full desktop environment setup using KDE Plasma

### ✅ Desktop Environment Installation (in progress)

The following packages were selected:

```bash
sudo pacman -S plasma kde-applications sddm
```

All package groups were accepted at default. Installation is currently running and may take time due to network speed and package size.

---

## 🔜 Next Command

Once installation is complete:

```bash
sudo systemctl enable sddm
reboot
```

---




## 🔜 Next Steps





4. **Install remaining surface tools:**

   ```bash
   pacman -S surface-control surface-control-daemon
   ```

5. **System hardening, ricing, and documentation.**



-
