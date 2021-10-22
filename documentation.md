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
# Begin Installation of Arch
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











[link](url)
