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
/dev/xxx is the device name e.g. /dev/sda; y is the partition number of that device.  
In the case of a freshly formatted drive, /dev/xxx might be something like /dev/sda and y = 1 (whichever partition was just created) e.g. /dev/sda1.  
- "**parted /dev/xxx mkpart "Bootloader" fat32 4096s 1GB**" 
  - Creates a 1GB partition, although you can go smaller if you're space-constrained.  
  Three Linux kernels and a Windows bootloader + recovery only uses ~240M. 
- "**parted /dev/xxx set y BOOT ON**" 
  - Activates the "boot" flag the newly created partition
- "**mkfs.vfat -F 32 -s 1 /dev/xxxy**" 
  - Creates the fat32 filesystem on the newly created partition
  - "-s 1" (or "-s 2") is sometimes needed as the OS may complain about the filesystem size.  

"**blkid**" will provide the UUID for use in the fstab discussed below.

Once formatting is complete, the bootloader can be installed via "**bootctl install --path=/boot/efi**" in a debian-based distro.  
Run **efibootmgr** to verify installation.
  
To make your zpool bootable you have two options.  
Both require you to mount the new boot partition to /boot/efi on the filesystem you'd like to boot (which may already be done if bootctl was installed via chroot).  
1. Mount the root fs and move the kernel files to the boot partition manually, making sure to match the layout in the efi directory referenced here.
2. Chroot into the zfs filesystem, install the kernel-update script below, and run "**update-initramfs -u -k all**".  (this validates the script as well)
If it goes well, it should take care of the boot for you.  
  
Note: With method #2 you should see (xxxxxx being the kernel version):  
- update-initramfs: Generating /boot/initrd.img-xxxxxx
- systemd-boot xxxxxx

If there's no output then there's an issue generating the kernel e.g. missing headers, /boot/efi isn't mounted, /boot isn't a writable fs, directory errors in the script, etc.  
If you've ruled out any errors in the update script, I've sometimes found purging and reinstalling the kernel resolves the issue.

# efi folder
Example hierarchy of the systemd-boot/efi partition.  
Includes loader entries for zfs-on-root as well as standard (ext4/UUID) partitions.  
Note the full path may look like /boot/efi/EFI/Debian-Buster/kernel-data or /boot/efi/loader/entries/Debian-Buster.conf

# fstab
Example fstab for mounting the systemd-boot partition to /boot/efi once the zfs-root is fully loaded.  
In this example the systemd-boot partition (/boot/efi) will only mount onto the zfs root once something tries to access it for the first time e.g. a kernel update.  
If you "**ls -la /boot/efi**" prior to actually trying to write to the device, you likely won't see anything.  
There's no need to mount an extra filesystem (especially a boot partition) if it won't be in use.

# Kernel-update scripts
Originally published by Josh Stoik at [Blobfolio](https://blobfolio.com/2018/06/replace-grub2-with-systemd-boot-on-ubuntu-18-04/) and adapted for zfs.   
This is necessary if you want to automatically update the kernel on the boot partition (otherwise the booting kernel will never change despite upgrading it on the zfs filesystem).  
Per Blobfolio's instructions, the "zz-" script can be copied into "**/etc/kernel/postinst.d/**".  
Only copy **one script** *--whichever is appropriate to the OS--* e.g. zz-update-systemd-boot-zfs for a zfs root, or zz-update-systemd-boot-uuid for a non-zfs root i.e. ext4.  
Copying both may result in the system not booting.  
Once the script is in place, follow the instructions below:  
```
# Set the right owner.
chown root: /etc/kernel/postinst.d/zz-update-systemd-boot
# Make it executable.
chmod 0755 /etc/kernel/postinst.d/zz-update-systemd-boot

# One for the kernel's postrm:
cd /etc/kernel/postrm.d/ && ln -s ../postinst.d/zz-update-systemd-boot zz-update-systemd-boot

# Note: Ubuntu does not usually create the necessary hook folder
# for initramfs, so the next line will take care of that if
# needed. (And yes, it *is* supposed to be "initramfs" and not
# "initramfs-tools"!)
[ -d "/etc/initramfs/post-update.d" ] || mkdir -p /etc/initramfs/post-update.d

# And now we can add the symlink:
cd /etc/initramfs/post-update.d/ && ln -s ../../kernel/postinst.d/zz-update-systemd-boot zz-update-systemd-boot
```  
Note: If grub was previously installed then remove grub as well.
```
# Purge the packages.
apt-get purge grub*

# Purge any obsolete dependencies.
apt-get autoremove --purge
```
