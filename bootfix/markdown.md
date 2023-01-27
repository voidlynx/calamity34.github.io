# oh shit, i broke my boot loader!

so, you broke your boot loader by installing windows? or was it a grub misconfiguration? maybe something else entirely? here's a little guide on how to fix your goddamn bootloader issues!

this guide will focus on a BIOS/MBR setup like i have, because that's what tends to have most issues. in case of UEFI - [read here](#post-scriptum)

## getting ready to fix shit

1. get yourself an archlinux live usb, no matter what kinda distro you've got, the arch rescue usb will be your best friend.
2. boot into that mf, figure out your root partition using `lsblk`, and mount it using `mount /dev/sdXY /mnt`, where X is your disk and Y is your partition. it may differ if your drive isn't connected via SATA (like if you're running a M.2 SSD).
3. chroot into it. your archlinux usb should've already come with a nice script to do that. do `arch-chroot /mnt` and it should pop you right into the root partition of your installation.

## time to fix shit
first, if you did notice a small (about 32M in size, NTFS-formatted) partition - it's where your windows bootloader lives now. congratulations, do not fuck it up. if you don't run an EFI system - your first 512 bytes of the disk have been overwritten by windows. you're going to have to reinstall grub on them.

would suck if something went wrong, so backup them using dd (X is your disk which you boot off, the `of` path can be anywhere you want, as long as you can properly find it. i prefer saving in `/var`). after that, reinstall grub on your disk's master boot record. 
```sh
dd if=/dev/sdX of=/var/MBR_BACKUP.img bs=512 count=1
grub-install --target=i386-pc /dev/sdX
```

if you still wanna boot into windows, you're going to need to install `os-prober` using your installation's package manager, then edit `/etc/default/grub` using an editor of your choice, where you're gonna **uncomment** the following line:
```
GRUB_DISABLE_OS_PROBER=false
```
after that, regenerate your config.
```sh
grub-mkconfig -o /boot/grub/grub.cfg
```
output should say that it found windows on the small partition. make changes to the config if you require any (99% of users wouldn't need to). congratulations, you did it, you should've fixed your boot now!
reboot into the system, grub should now open itself with your linux installation as well as windows.

## post scriptum
if you actually have a UEFI system (which most people would do nowadays) - it shouldn't be too hard to fix any issues, since UEFI doesn't require overwriting the boot record, only to have EFI boot partitions. maybe check that the boot order is correct, the [arch wiki](https://wiki.archlinux.org/) is your best friend (check the [General Troubleshooting article](https://wiki.archlinux.org/title/General_troubleshooting), maybe?).

if you no longer want to use windows - wipe the windows partition/disk using a method of your choice, in case of a separate physical windows drive, i prefer zeroing out with DD for cleaniness, but may take a bit too long, try just removing the windows partitions. and do not forget to also delete the windows's 32M boot partition. after that, regenerate the grub config.

a thing you could try (but i haven't tested) if you have a separate drive for windows is flashing the backup of the MBR you did onto the separate drive. windows actually decided to somehow overwrite the MBR on my linux drive, no idea how, but didn't even touch the windows's own drive. i'm not saying that is something you should do, but that could be an option.