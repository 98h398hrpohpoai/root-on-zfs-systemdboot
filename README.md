# Dual-Boot ZFS-on-Root with Systemd-Boot
This example is completed with dual-booting two Debian-based distros both with root-on-zfs.  
Specifically, Debian Buster and Bullseye both on the same zpool, as well as an ext4-based installation of Ubuntu for recovery purposes.  
  
Zpool name = data  
Dataset hierarchy (bold datasets have no mountpoint and are for organization purposes only):  
- **data/dpool**  
- data/dpool/buster (OS)                     
- data/dpool/home                       
- data/dpool/home/root (mounted at /root)                  
- data/dpool/opt                        
- data/dpool/srv                        
- data/dpool/testing (OS)  
- **data/dpool/usr**                         
  - **data/dpool/usr/lib**                      
    - data/dpool/usr/lib/plexmediaserver     
- **data/dpool/var**                          
  - data/dpool/var/cache                   
  - **data/dpool/var/lib**                       
    - data/dpool/var/lib/plexmediaserver   
  
**dpool/buster** and **dpool/testing** are both root datasets with a mountpoint of "/".  
To avoid conflicts, both are set to **canmount=noauto** and only mounted via the entries in /efi/loader/entries.  
  
Known issues:
- First time switching between OSes on ZFS will causes initramfs to complain about the zpool last being loaded by a different OS (which is true) and will require you to "**zpool import -a -f && zpool export -a**" from initramfs and then reboot.  
System will boot the second time around.  
There's probably a simple fix for this, but it hasn't been a priority to figure it out.
- Conflicts will likely emerge with the use of "**zfs mount -a**" if both OSes are in the same pool, though can be avoid by changing the mount point prior to use e.g. "**zfs set mountpoint=/mnt data/dpool/testing**" and changing it back when done.  
Effectively the same as "**zpool import -R /mnt somepool**" if you maintain separate OS pools. 

# Boot partition (/boot/efi)
Boot partition is a separate device from the zpool, and formatted in vfat/fat32 with the boot flag enabled.  
xxx is the device name e.g. /dev/sda; y is the partition number of that device.  
In the case of a freshly formatted drive, /dev/xxx might be something like /dev/sda and y = 1 (whichever partition was just created) e.g. /dev/sda1.  
- "**parted /dev/xxx mkpart "Bootloader" fat32 4096s 1GB**" 
  - Creates a 1GB partition, although you can go smaller if you're space-constrained.  
  Three Linux kernels and a Windows bootloader + recovery only uses ~240M. 
- "**parted /dev/xxx set y BOOT ON**" 
  - Activates the "boot" flag the newly created partition
- "**mkfs.vfat -F 32 -s 1 /dev/xxxy**" 
  - Creates the fat32 filesystem on the newly created partition

"-s 1 (or -s 2)" is sometimes needed as the OS may complain about filesize.  
You can try without using "**mkfs.vfat -F 32 /dev/xxxy**", although there's no functional difference.  

"**blkid**" will provide the UUID for use in the fstab discussed below.

# efi
Example hierarchy of the systemd-boot/efi partition.

# fstab
Example fstab for mounting the systemd-boot partition to /boot/efi once the zfs-root is fully loaded.  
In this example the systemd-boot partition (/boot/efi) will only mount onto the zfs root once something tries to access it for the first time e.g. a kernel update.  
If you "**ls -la /boot/efi**" prior to actually trying to write to the device, you likely won't see anything.  
There's no need to mount an extra filesystem (especially a boot partition) if it won't be in use.
