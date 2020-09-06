Root of the boot partition for systemd-boot (/boot/efi).  
Typically formated in vfat/fat32 and mounted to /boot  

Mounted to /boot because when generating new kernels, debian-based distros cannot effectively symlink on fat32 partitions, and run into issues when the systemd-boot partition is mounted directly as /boot instead of /boot/efi
