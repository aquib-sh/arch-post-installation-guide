# Arch Linux : Post Installation Guide
A personal guide of useful steps after installation of Arch Linux

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

## Mounting a new storage device
### Temporarily mounting a device
```bash
mkdir /mnt/<friendly-name>
mount /dev/<paritition-name>
```
**example**:
```bash
mkdir /mnt/games
mount /dev/sdb1
```
### Permanently mounting a device
Actually there is no permanent mounting of a device. Each time on boot the parititons are mounted as per their definition in `/etc/fstab`
To add a device to mount at a specific mountpoint each time on boot we have to add it's entry in `/etc/fstab` using it's UUID.

**Create a mountpoint if it does not exists**
```bash
mkdir /mnt/games
```
**Finding UUID of a partition**
To get the details on available partitions
```bash
lsblk
```
To get the UUID of a specific partition
```bash
blkid /dev/sdb1
```

Now note of value of UUID in the output of previous command along with it's file system type and add entry in `/etc/fstab` in the sequence of 
`<uuid> <mount-point> <filesystem-type> defaults`
```bash
UUID=BCCE0ECDCE0E803E /mnt/games ntfs defaults
```
Save the file and execute the below commands to reload the filesystem to mount the defined devices.
```bash
systemctl daemon-reload
systemctl restart local-fs.target
```

## Install recommended packages
### Fonts
```bash
pacman -Syu ttf-firacode-nerd ttf-hanazono ttf-inconsolata ttf-indic-otf ttf-jetbrains-mono ttf-liberation ttf-opensans ttf-roboto-mono adobe-source-code-pro-fonts adobe-source-han-sans-jp-fonts cantarell-fonts gnu-free-fonts gsfonts libfontenc libxfont2 noto-fonts-emoji terminus-font wqy-bitmapfont
```
###  Multimedia tools
```bash
pacman -Syu gimp krita obs-studio audacity openshot vlc
```

### Tools
```bash
pacman -Syu xclip neofetch htop lm_sensors
```

## GNOME Configuration
### Force GNOME to Alt + Tab only in current workspace
Run the below command as **non-root**
```bash
gsettings set org.gnome.shell.app-switcher current-workspace-only true
```

## Further recommended configurations
### Setup dnsmasq
Setup `dnsmasq` to reduce the time spent with resolving DNS 

**Install**
```bash
pacman -Syu dnsmasq
```

**Enable/Start the service**
```bash
systemctl enable dnsmasq && systemctl start dnsmasq
```

**Edit `/etc/dnsmasq.conf`**
Uncomment and the below lines in the file or simply append them to it
```
listen-address=::1,127.0.0.1
cache-size=1000
no-resolv
server=8.8.8.8
server=8.8.4.4
```

**Edit `/etc/resolv.conf`**
```
nameserver ::1
nameserver 127.0.0.1
options trust-ad
```

if you are using NetworkManager (which most likely you are) then **make `/etc/resolv.conf` immutable** to avoid the file getting overwritten after reboot
```bash
chattr +i /etc/resolv.conf
```

**Restart dnsmasq**
```
systemctl restart dnsmasq
```

**Stop dhcpcd**, we don't need it anymore
```
systemctl disable dhcpcd && systemctl stop dhcpcd
```
[Test dnsmasq](https://wiki.archlinux.org/title/dnsmasq#Test)

References: 
- [dnsmasq](https://wiki.archlinux.org/title/dnsmasq)

### Add .vimrc
```
filetype plugin indent on
syntax on

" enable copy pasting from mouse using shift
set mouse=a

" show existing tab with 4 spaces width
set tabstop=4

" when indenting with '>', use 4 spaces width
set shiftwidth=4

" On pressing tab, insert 4 spaces
set expandtab

" show line numbers
set number

" Highlight search results
set hlsearch

" Show matching brackets when text indicator is over them
set showmatch

" Turn backup off, since most stuff is in SVN, git etc. anyway...
set nobackup
set nowb
set noswapfile

set ai "Auto indent
set si "Smart indent
```





  

