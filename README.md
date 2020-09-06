# ZFS-on-Root with Systemd-Boot
This example is completed with Debian-based distros.  
Specifically, Debian Buster and Bullseye both on the same zpool, as well as an ext4-based installed of Ubuntu for recovery purposes.  
  
Zpool name = data  
Data set hierarchy (bold datasets have no mountpoint and are for organization purposes only):  
- **data/dpool**  
- data/dpool/buster                     
- data/dpool/home                       
- data/dpool/home/root (mounted at /root)                  
- data/dpool/opt                        
- data/dpool/srv                        
- data/dpool/testing (debian bullseye/testing)  
- **data/dpool/usr**                         
  - **data/dpool/usr/lib**                      
    - data/dpool/usr/lib/plexmediaserver     
- **data/dpool/var**                          
  - data/dpool/var/cache                   
  - **data/dpool/var/lib**                       
    - data/dpool/var/lib/plexmediaserver   
  
dpool/buster and dpool/testing are both root datasets with a mountpoint of "/".  
To avoid conflicts, both are set to canmount=noauto and only mounted via the entries in /efi/loader/entries.  
Conflicts will likely emerge with the use of "zfs mount -a" for this pool, though can be avoid by changing the mount point prior to use e.g. "zfs set mountpoint=/mnt data/dpool/testing".  
Effectively the same as "zpool import -R /mnt somepool" if you maintain separate OS pools.

# fstab
Example fstab for mounting the systemd partition; current settings won't mount the /boot/efi partition once the system switches over to zfs unless something tries to access it.  
There's no need to mount an extra filesystem (especially a boot partition) if it won't be in use.
