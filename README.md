# Arch Linux Encrypted LVM UEFI

`beware typos!`

will clean this up later.

- https://gist.github.com/loneicewolf/0bb0c303c331aa462754a0beebe0329e
- https://gist.githubusercontent.com/loneicewolf/0bb0c303c331aa462754a0beebe0329e/raw/fb20081f1ffabeb10a5bf9c5765517097b154823/Guide.MD

*Made this in my free time on a break as 'addons' to both my ARCH-PCI-PASSTROUGH guide, as well as to my OSCP Certification (and, to be part in  any upcomming work/projects I will do in this kind of field)*
- _William M_


```
hdparm -I /dev/sda
hdparm -I /dev/sdb
hdparm -I /dev/(some other disk(s))


# verify BOOT mode
# if these dirS exists, then you are in UEFI(and, therefore - you are in the correct place!)
ls /sys/firmware/efi/efivars

lsblk --list --fs
fdisk /dev/YOUR_TARGET_DISK
# make 2 partitions
# 1 1500MB ( ~ 1.50GB ) of type EFI
# 2 remaining space of type (ext4)

cryptsetup luksFormat /dev/sda2
L="whatever"
cryptsetup open /dev/sda2 $L

pvcreate /dev/mapper/$L
vgcreate main /dev/mapper/$L

lvcreate -L 8GB       -n  swap      main
lvcreate -L 20GB      -n  root      main
lvcreate -L 50GB      -n  research  main
lvcreate -l 100%FREE  -n  home      main

mkfs.vfat -F32 -n BOOT /dev/sda1
mkfs.ext4 -L root /dev/mapper/main-root
mkfs.ext4 -L home /dev/mapper/main-home
mkswap    -L swap /dev/mapper/main-swap


## MountPoint Assigning
A="boot"
B="home"
C="research"
mkdir -p /mnt/{$A,$B,$C}

mount /dev/mapper/main-root     /mnt
mount /dev/sda1                 /mnt/boot
mount /dev/mapper/main-home     /mnt/home
mount /dev/mapper/main-research /mnt/research


#NOTE# ATTENTION REQUIRED!
## I wanted my installation to be airgrapped.
## You(probably) don't want your installation to be that, if that is the case, add networkmanager to the below pacstrap command.
## it should be obvious here I use intel.
pacstrap /mnt base base-devel syslinux nano linux linux-firmware mkinitcpio lvm2 intel-ucode bash sudo
syslinux-install_update -i -a -m -c /mnt
nano /mnt/boot/syslinux/syslinux.cfg
APPEND cryptdevice=/dev/sda2:main root=/dev/mapper/main-root rw lang=en locale=en_US.UTF-8

swapon -L swap
genfstab -Up /mnt >> /mnt/etc/fstab
arch-chroot /mnt
## IF YOU did not want a airgrapped system
## DO this:  `systemctl enable NetworkManager`

# Lang , Geo and Timezone

## I live in Stockholm
## (if you do that leave this as is)
## (if not, please tweak it where you live)
ln -sf /usr/share/zoneinfo/Europe/Stockholm /etc/localtime

# same
echo LANG=en_US.UTF-8 >> /etc/locale.conf
nano /etc/locale.gen
locale-gen

nano /etc/mkinitcpio.conf
## ! MODULES=(ext4)
## ! HOOKS=(base udev autodetect modconf block keyboard encrypt lvm2 filesystems fsck)

#..#

mkinitcpio -p linux

# I do not want any user who can get root just by typing sudo,..
# so I did not bother giving them any privs like that.
`useradd -m -g users -s /bin/bash	<YOUR USERNAME>`
`passwd <YOUR USERNAME>`

## Hostnames, Domain and Networking setup:
echo "HOSTNAME" >> /etc/hostname
echo "127.0.1.1 localhost.localdomain HOSTNAME" >> /etc/hosts



bootctl --path=/boot install
## opt: cat /boot/loader/loader.conf
nano /boot/loader/entries/arch.conf

## PAY ATTENTION HERE ##
# Read the manual here - do NOT COPY PASTE
## 
##------##
  title	Arch Linux
  linux	/vmlinuz-linux
  initrd	/intel-ucode.img
  initrd	/initramfs-linux.img
  options	cryptdevice=UUID="$(blkid|grep sda2|cut -d\" -f2)":$L root=/dev/mapper/main-root rw
##------##

# optional but, 'quiet' can be appended after 'main-root' (before 'rw')



exit and reboot!

# ===================== #
# optional steps ahead. #

# if you want xfce4, you:
pacman -S xfce4

# if, you want LXDE:
pacman -S lxde

# ======= #
# but wait!
# if I want no GUI? it's lagging the OS!
## oh like me? Then really skip that part.

# ============================ #
## For those who want XFCE4 (e.g)
## load of ways to do this again
## for nvidi users it (MIGHT) be something
## on those lines.....
# pacman -S nvidia nvidia-utils
# pacman -S nvidia networkmanager
# pacman -S xorg xorg-xinit xorg-server
# pacman -S xfce4 lightdm lightdm-gtk-greeter
# echo "exec startxfce4" > ~/.xinitrc
# systemctl enable lightdm
```
