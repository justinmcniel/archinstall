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
        - mount point did not exist, fixed with `mkdir /mnt/efi` then rerunning the mount command
# Begin Installation of Arch
13. Install the "essentials"
    - `pacstrap /mnt base linux linux-firmware`
    - technically speaking you can ommit the firmware, since it's a VM, but I want to install as if it was on a real machine
  1. Install filesystem management things
    - `pacstrap /mnt btrfs-progs dosfstools exfatprogs f2fs-tools e2fsprogs jfsutils nilfs-utils ntfs-3g reiserfsprogs udftools xfsprogs` 
    - recieved warning about possibly missing firmware for the following modules: `aic94xx` `wd719x` `xhci_pci`
      - ran `pacstrap /mnt aic94xx wd719x xhci_pci` in an attempt to fix it
      - targets were not found, will install with `pacman` after using arch-chroot
  2. Install text editors
    - `pacstrap /mnt nano gedit`
    - also installed all the dependencies
  3. Install networking utilities/software
    - `pacstrap /mnt iputils systemd-resolved systemd-networkd net-tools iproute2 systemd dhcpcd dhclient`
    - got errors relating to `systemd-resolved` and `systemd-networkd`, rerunning the previous command with those removed
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
    - honestly no idea what this did (because i do not know what UUID is)
    - checked the output with `cat /mnt/etc/fstab`
15. chroot into the new system
    - `arch-chroot /mnt`
    - install the packages that failed to pacstrap earlier - `pacman -S aic94xx wd719x xhci_pci`
    - targets were not found, moving on
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
    - check the locale configuration
        - `cat /etc/locale.conf
        - did not exist
    - create the locale configuration and set the language
        - `echo "Lang=en_US.UTF-8" > /etc/locale.conf`
        - `cat /etc/locale.conf` to check that I did it right







[link](url)
