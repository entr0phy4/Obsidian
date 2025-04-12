## In GNU/Linux
### Using basic command line utilities
This method is recommended due to its simplicity and universal availability, since these tools are part of coreutils (pulled in by the base meta packages)
Find out the name of your USB drive with `ls -l /dev/disk/by-id/usb-*` and check with `lsblk` to make sure that is not mounted.
### dd

```bash 

dd bs=4M if=path/to/archlinux-version-x86_64.iso /dev/disk/by-id/usb-flash_usb_name conv=fsync oflag=direct status=progress

```

Executing `sync` with root privileges after the respective command ensures buffers are fully written to the device before you remove it.
> To restore the USB drive as an empty, usable storage device after using the Arch ISO image, the ISO 9660 files system signature needs to be removed by running
> `wipe --all /dev/disk/by-id/usb-flash_drive_name` as root, before repartitioning and reformatting the USB drive. 