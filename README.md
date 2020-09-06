# ZFS-on-Root with Systemd-Boot
This example is completed with debian based distros.  
Specifically, Debian Buster and Bullseye both on the same zpool, as well as an ext4-based installed of Ubuntu for recovery purposes. 
Zpool name = data
Data set hierarchy:  
data/dpool (no mountpoint)    
data/dpool/buster                     
data/dpool/home                       
data/dpool/home/root (mounted at /root)                  
data/dpool/opt                        
data/dpool/srv                        
data/dpool/testing (debian bullseye/testing)  
data/dpool/usr (no mountpoint)                        
data/dpool/usr/lib (no mountpoint)                     
data/dpool/usr/lib/plexmediaserver     
data/dpool/var (no mountpoint)                         
data/dpool/var/cache                   
data/dpool/var/lib (no mountpoint)                      
data/dpool/var/lib/plexmediaserver   
  

# fstab
Example fstab for mounting the systemd partition; current settings won't mount the /boot/efi partition once the system switches over to zfs unless something tries to access it.  
There's no need to mount an extra filesystem (especially a boot partition) if it won't be in use.
