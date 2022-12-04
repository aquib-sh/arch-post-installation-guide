# arch-post-installation-guide
A personal basic guide of useful steps after installation of Arch Linux

## Enable dhcpcd service for DNS host name resolution and configure DNS nameserver
Install dhcpcd if you haven't already, for doing this you will need to chroot into the installed filesystem using arch bootable usb
once you load the bootable iso, mount your filesystem and then chroot into it.
**for example** 
if your Linux File System partition is at `/dev/nvme0n1p3` 
Swap at `/dev/nvme0n1p2` and
EFI at `/dev/nvme0n1p1`
then use 
```bash
mount /dev/nvme0n1p3 /mnt
mount --mkdir /dev/nvme0n1p1 /mnt/boot
swapon /dev/nvme0n1p2
arch-chroot /mnt
```
Now install dhcpcd and optionally also install a suitable text editor and man pages if you haven't already
```bash
pacman -Syu dhcpcd vim man-db man-pages texinfo
```
After  the installation is completed reboot into the installed system and login as root

### Edit `/etc/resolv.conf` to add DNS nameserver
open `/etc/resolv.conf` file using any text editor
```bash
vim /etc/resolv.conf
```
if you want to configure your router as the default method to resolve hostnames enter the below text in the file
```bash
nameserver 192.168.0.1
```
If you want to use another server for example: Google's DNS server to resolve hostnames then enter it's IP address in file.
```bash
nameserver 8.8.8.8
```

### Enable and start dhcpcd service
The system won't resolve the hostnames if you don't have dhcpcd or a proper network manager to use resolv.conf
Use the below commands to enable and then start the service.
```bash
systemctl enable dhcpcd.service
systemctl start dhcpcd.service
```

## Use reflector service to speed up pacman downloads
Install 
```bash
pacman -Syu reflector
```
Run to get the updated mirrorlist
```
reflector --latest 5 --protocol https --sort rate --save /etc/pacman.d/mirrorlist 
```
### Configure reflector to run in background
Edit `/etc/xdg/reflector/reflector.conf` as below.
```bash
# Set the output path where the mirrorlist will be saved (--save).  
--save /etc/pacman.d/mirrorlist  
  
# Select the transfer protocol (--protocol).  
--protocol https  
    
# Use only the  most recently synchronized mirrors (--latest).  
--latest 5  
  
# Sort the mirrors by Synchronization time (--sort)
--sort rate
```
Enable and start reflector service
```bash
systemctl enable reflector.service
systemctl start reflector.service
```