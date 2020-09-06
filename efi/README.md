Root of the boot partition for systemd-boot (/boot/efi).  
Typically formated in vfat/fat32 and mounted to /boot  

Mounted to /boot because when generating new kernels, debian-based distros cannot effectively symlink on fat32 partitions, and run into issues when the systemd-boot partition is mounted directly as /boot instead of /boot/efi  
  
/boot should be part of the root file system; with zfs-on-root this can be either a separate bpool, or simply part of the root filesystem on rpool/ROOT/default  
Personally I don't see a point for a separate bpool when going this route as it adds another layer of complexity when it comes to pool mounting
