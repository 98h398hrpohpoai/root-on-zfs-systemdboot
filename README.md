# ZFS-on-Root with Systemd-Boot
This example is completed with debian based distros.  
Specifically, Debian Buster and Bullseye both on the same zpool, as well as an ext4-based installed of Ubuntu for recovery purposes.  

# fstab
Example fstab for mounting the systemd partition; current settings won't mount the /boot/efi partition once the system switches over to zfs unless something tries to access it.  
There's no need to mount an extra filesystem (especially a boot partition) if it won't be in use.
