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

Thne it installed arch and correctly borught up the boat loader
-first I connect to wifi 
-iwctl
-station wlan0 scan
-station wlan0 get-networks
-station wlan0 connect YourSSID
-exit

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

 ---Then Finish the Base Setup:
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




-
