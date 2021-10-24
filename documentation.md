# Arch Install Documentation
### Justin McNiel

Using VMware Workstation Pro

1. Created VM
    - 20GB Hard Drive
    - 2GB Ram
    - 4 Cores (1 Logical Procesor each, total 4 Logical Processors)
    - CD/DVD drive
    - NAT Networking
    - Sound Card/Display - auto-set to "Auto detect"
    - USB Controller/Printer - auto-set to "Present"
    - LSI Logic for the disks
        - SCSI type disk
    - Custom VM
2. Mount Arch installer to CD/DVD drive
    - Double Click CD/DVD (next to the VM)
    - Select "Use ISO image file"
    - Select the Arch Installation ISO
3. Close VMware
4. Edit Configuration to allow for UEFI boot mode/make UEFI the default
    - Navigate to the folder where the VM is being stored
    - Select the VM configuration file (the \*.vmx file)
    - Open it with text editor of your choice (I used notepad)
    - Insert `firmware="efi"` as the second line in the file
    - Save
5. Re-Open VMware and start the VM
##VMware failed to Open the VM
- I edited the wrong file
    - Erase the VM and start from scratch
- This time it worked
# Begin Formatting the drives
6. Boot to the Arch installer
7. Set the console keyboard layout
    - Skipped because the default is EN-US
8. Verify the boot mode
    - `ls /sys/firmware/efi/efivars`
    - if it shows stuff, then it worked right, otherwise, something went wrong
9. Connect to the internet
    - `ip link`
    - `ping archlinux.org`
        - Make sure that it did connect
        - On Arch, you need to hit ctrl+c to make it stop ping-ing, otherwise, it apparently goes on endlessly (mine sent 57 pings before I canceled it).
    - It connected correctly
10. Update the system clock
    - `timedatectl set-ntp true`
    - `date`
        - Check that the date and time are correct
        ## The date was correct, and the time was off by 5 hours, this is consistent with how linux/windows dual boots work with grub... so I ignored it for now
11. Partition the disks
    - `lsblk`
        - Show the block devices to identify the disk we will be installing to
        - Disk we selected was labled as `sda`
    - Using `parted` for modifying the partition tables
        - First command was `parted /dev/sda`
    - Started by using `help` to see the commands
    - `print`
        - To show there was no existing partition table
    - `mktable msdos`
        - Create a new partition table of type MBR
    - `mkpart`
        - "Partition type? primary/extended?" `primary`
        - "File system type? [ext2]?" `` (just hit enter, efi/EFI was not recognized)
        - "Start?"  `1MiB`
        - "End?"  `501MiB` (technically this is bigger than 500MB, but that just gives me a little extra wiggle room)
    - `print` to make sure that the file system was created right
    - `mkpart`
        - "Partition type? primary/extended?" `primary`
        - "File system type? [ext2]?" `` (just hit enter, efi/EFI was not recognized)
        - "Start?"  `502MiB`
        - "End?"  `100%`
    - `print` to make sure that the file system was created right
    - `set 1 esp on` to set the esp flag for the EFI partition to on (part of the EFI specification)
    - `print` to make sure this was done correctly
    - `quit` to exit the parted interactive console
    - `lsblk` to verify that the new partitions were created properly
    - `mkfs.fat -F32 /dev/sda1` to make /dev/sda1 an EFI file system by formatting it to Fat32 (since the esp flag is already on)
    - `mkfs.ext4 /dev/sda2` to format the system drive to ext4 (for installing arch onto)
    - `parted /dev/sda print` to check that the partitions are formatted/flagged correctly
12. Mount the file systems
    - `mount /dev/sda2 /mnt`
    - `mount /dev/sda1 /mnt/efi`
        - Mount point did not exist, fixed with `mkdir /mnt/efi` then rerunning the mount command
# Begin Installation of Arch
13. Install the "essentials"
    - `pacstrap /mnt base linux linux-firmware`
    - Technically speaking you can ommit the firmware, since it's a VM, but I want to install as if it was on a real machine
  1. Install filesystem management things
    - `pacstrap /mnt btrfs-progs dosfstools exfatprogs f2fs-tools e2fsprogs jfsutils nilfs-utils ntfs-3g reiserfsprogs udftools xfsprogs` 
    - Recieved warning about possibly missing firmware for the following modules: `aic94xx` `wd719x` `xhci_pci`
      - Ran `pacstrap /mnt aic94xx wd719x xhci_pci` in an attempt to fix it
      - Targets were not found, will install with `pacman` after using arch-chroot
  2. Install text editors
    - `pacstrap /mnt nano gedit`
    - Also installed all the dependencies
  3. Install networking utilities/software
    - `pacstrap /mnt iputils systemd-resolved systemd-networkd net-tools iproute2 systemd dhcpcd dhclient`
    - Got errors relating to `systemd-resolved` and `systemd-networkd`, rerunning the previous command with those removed
    - iputils, iproute, and systemd were reinstalled, so removing those too (since they were already installed they were not needed in the first place)
    - `pacstrap /mnt net-tools dhcpcd dhclient`
  4. Install `parted` and documentation tools
    - `pacstrap /mnt gparted parted man-db man-pages`
  5. Install various other desired packages
    - `pacstrap /mnt memtest86+ openssh sudo syslinux wpa_supplicant zsh`
    - memtest86+ - I use similar tools (and recognize this one) on my hardware, so it would be usefull to have
    - sudo, syslinux, wpa_supplicant, zsh - seemed important
## Configuring the system
14. `genfstab -U /mnt >> /mnt/etc/fstab`
    - Honestly no idea what this did (because i do not know what UUID is)
    - Checked the output with `cat /mnt/etc/fstab`
15. Chroot into the new system
    - `arch-chroot /mnt`
    - Install the packages that failed to pacstrap earlier - `pacman -S aic94xx wd719x xhci_pci`
    - Targets were not found, moving on
16. Set time zone
    - `timedatectl list-timezones` to list the timezones
        - using America/Chicago
    - `ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime`
        - create a symbolic link that links local time to `America/Chicago`
    - `hwclock --systohc`
        - generates `/etc/adjtime`
17. Set localization
    - `nano /etc/locale.gen`
        - uncommented `en_US.UTF-8 UTF-8` and `en_US ISO-8859-1` to allow them to be used
    - `locale-gen` to generate the locales
    - Check the locale configuration
        - `cat /etc/locale.conf
        - did not exist
    - Create the locale configuration and set the language
        - `echo "LANG=en_US.UTF-8" > /etc/locale.conf`
        - `cat /etc/locale.conf` to check that I did it right
    - No need to make the console keyboard layout persistent, since it was not modified
18. Set the hostname configurations
    - `cat /etc/hostname` to check the hostnames file (did not exist)
    - `echo LocalHost > /etc/hostname`
        - Not actually sure why this is needed, but I did it anyways, I chose my hostname to be `LocalHost`
    - Modify hosts to include corresponding entried
        - `nano /etc/hosts`
        - `127.0.0.1       localhost`
        - `::1             localhost`
        - `127.0.1.1       LocalHost`
19. Configure the network
    - Check that it is currently working with `ip addr show` (it was)
    - `pacman -S networkmanager`
        - Install `networkmanager`
    - Enable networkmanager: `systemctl enable NetworkManager.service`
        - Credit to some folks on [this arch forum](https://bbs.archlinux.org/viewtopic.php?id=213914) for providing the commands I never would have found on my own
    - Start networkmanager: `systemctl start NetworkManager.service`
20. Initramfs
    - Should have implicitly been done when I ran  pacstrap on the kernel (linux)
21. Set the root password
    - `passwd`
22. Install the bootloader
    - `grub-install --target=amd64 /dev/sda`
        - `grub-install` was not found
    - `pacman -S grub`
        - Now I can re-run: `grub-install --target=amd64 /dev/sda`
    - amd64 was not a valid target, valid targets were `i386-efi`, `i386-pc` and `x86_64-efi`
    - `grub-install --target=i386-pc /dev/sda`: settled on `i386-pc` after a bunch of forum reading
  - Enable Micro-Code for AMD CPUs
    - `pacman -S amd-ucode`
    - `grub-mkconfig -o /boot/grub/grub.cfg`
    - `update-grub` (just for good measure)
        - Command not found but [this forum](https://unix.stackexchange.com/questions/111889/how-do-i-update-grub-in-arch-linux) says that `update-grub` is just a script which would run the previous command
23. Additional installs
    - `pacman -S sudo`
24. Exit the chroot, and check that no partitions are "busy"
    - `exit`
    - `umount -R /mnt`
25. Reboot
    - `reboot`
    - It booted to the CD/DVD drive
        - Shut down the VM and "removed" the install iso
    - Failed to find the bootloader
        - Found out it's set to go to the CDROM second, so I don't need to remove it next time
        - Boot to CDROM and try re-installing grub
26. Second attempt to install the boot loader
    - `mount /dev/sda2 /mnt`
    - `mount /dev/sda1 /mnt/efi`
    - `arch-chroot /mnt`
    - `grub-install /dev/sda`
        - Could not find EFI directory
        - Keeping in mind that I am in the chroot
        - `mkdir /EFI`
        - `mount /dev/sda1 /EFI`
        - Re-running the grub install command
        - Same issue
        - Following instructions from [this forum](https://bbs.archlinux.org/viewtopic.php?id=252051) - mount it to `/mnt/boot/efi` (from inside chroot, it will be `/boot/efi`)
        - Rerun the grub install command
        - This time it said that `efibootmgr` was not found
        - `pacman -S efibootmgr` (on the advice from [thisforum](https://bbs.archlinux.org/viewtopic.php?id=169025))
        - Rerun the grub install command, this time it worked
    - Reboot... IT WORKED!!!!!
# Customizing Arch
27. Installing a GUI
    - Choice was Cinnamon
        - It's what I'm used to from Linux Mint
        - It's an AUR package, so it also counts as installing an AUR package
    - Make sure the system is up to date
        - `pacman -Syu`
    - Install `yay AUR Helper`
        - `pacman -S base-devel git`: installs some installation prerequisites
      - `cd /opt`
      - `sudo git clone hhtps://aur.archlinux.org/yay.git`
      - `sudo chown -R root:root ./yay`
      - `cd yay`
      - `makepkg -si`
        - Not allowed as root, creating my other user now
28. Adding other users
    - `adduser [username]`
      - The correct command was `useradd`
      - To add them to superusers: `usermod -aG wheel [username]` (did not work)
      - Set the password: `passwd [username]` (done without chaning out of root)
      - `pacman -S vi` followed by `visudo` then uncommenting the line `%wheel ALL=(ALL) ALL` to enable sudo access for the wheel group
      - Installing sudo: `pacman -S sudo`
      - Set password to change after login (for codi and sal): `passwd --expire [username]`
    - To show sudoers: `cat /etc/group | grep wheel`
    - Set ownership of users home directory: `chown -R [username]: /home/[username]
29. Back to installing yay
    - Needed write permissions on the directory: `chmod 764 ./PKGBUILD` then `chown -R [username]:wheel ./`
    - `su [username]`
    - `makepkg -si`
      - Failed to get permission for `/home/[username]` with mkdir, doing it manually with `mkdir -p /home/[username]/.cache/go-build`
      - One of the scripts in this command forgot the `-p` after `mkdir` so I have to make the directories manually
30. Continuing installation of cinnamon
    - Continuing as `root` in `/` directory
    - `pacman -S cinnamon gnome-terminal`
    - `pacman -S xorg lightdm lightdm-gtk-greeter`
