# zfs
Dual-boot zfs-on-root config for Debian w/ systemd-boot
  

# fstab
Example fstab for mounting the systemd partition; current settings won't mount the /boot/efi partition once the system switches over to zfs unless something tries to access it.
There's no reason to mount an extra filesystem (especially a boot partition) if it won't be in use.
