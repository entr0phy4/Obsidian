Is a computer program that is responsible for booting a computer and booting an operating system. If it also provides an interactive menu with multiple boot choices then it's often called a boot manager.
When a computer is turned off, its software including operating systems, application code, and data remains stored on [[Non-volatile memory]].
When the computer is powered on, it typically does not have and operating system or its loader in [[Random Access Memory (RAM)]]. The computer first executes a relatively small program stored in the [[ROM]], which is read-only memory ([[ROM]], [[EEPROM]], [[NOR flash]]) along with some needed data, to initialize hardware devices such as [[Central Processing Unit (CPU)]], [[Motherboard]], [[Memory]], storage and other I/O devices.

### First-stage boot loader
Examples of first-stage bootloaders include [[BIOS]], [[UEFI]], [[coreboot]], [[Libreboot]], and [[Das U-boot]]. It initialize hardware devices such as [[Central Processing Unit (CPU)]]. [[Motherboard]], [[Memory]], etc. 
### Second-stage boot loader
Second-stage such as [[Grand Unified Bootloader (GRUB)]] `expand this...`
Second-stage implementations can include interactive user interfaces, allowing boot option selection and parameter modification. They handle kernel loading, including processing of [[initrd]]/[[initramfs]] images, and can pass boot parameters to the [[Kernel]]. Many implement modular designs supporting loadable modules for additional functionality.