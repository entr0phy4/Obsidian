 https://archlinux.org/
 *A simple, lightweight distribution* 
 
### How install Arch Linux from scratch
Installing Arch Linux from scratch gives you complete control over what is installed, allowing you to tailor the system to your needs. The installation process also helps you understand how Linux works, from partitioning your disk to configuring the [[Bootloader]] ([[Grand Unified Bootloader (GRUB)]]).

## Step 1: Pre installation
### 1.1 Acquire an installation image
Download a ISO image or a netboot image
### 1.2 Verify signature
It is recommended to verify the image signature before use, especially when downloading from and HTTP mirror, where downloads are generally prone to be intercepted to serve malicious images
### 1.3 Prepare and installation medium
The ISO can be supplied to the target machine via [[USB flash drive]]
### 1.4 Boot the live environment
> Arch Linux installation images do not support Secure Boot. You will need to disable Secure boot to boot the installation medium.

1. Point the current boot device to the one which has the Arch Linux installation medium. Typically it is archived by pressing a key during the [[POST]] phase, as indicated on the splash screen. Refer to your motherboard's manual for details.
2. When the installation medium's boot loader menu appears,
	- If you used the ISO, select _Arch Linux install medium_ and press `Enter` to enter the installation environment.
	- If you used the Netboot image ... `*expand this*` 
3. You will be logged in on the first [[Terminal]] as the root user, and presented with a Zsh shell prompt.
### 1.5 Set the console keyboard layout and font
The default console keymap is US. Available layouts can be listed with: `localectl list-keymaps`
Console fonts are located in `/usr/share/kdb/consolefonts/` and can likewise be set with `setfont` omitting the path and file extension.
`setfont ter-132b`
### 1.6 Verify the boot mode
To verify the boot mode, check the UEFI [[Bitness]].
`cat /sys/firmware/efi/fw_platform_size`
if returns `No such file or directory`, the system may be booted in `BIOS` (or CMS) mode.