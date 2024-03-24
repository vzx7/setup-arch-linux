# setup-arch-linux
Step-by-step instructions for installing and configuring arch linux (i3wm)

## Get ISO and b2sums, verification
```bash
$ mkdir ARCH_INIT
$ cd ARCH_INIT
$ curl -O https://geo.mirror.pkgbuild.com/iso/latest/archlinux-[version]-x86_64.iso
$ curl -O https://geo.mirror.pkgbuild.com/iso/latest/b2sums.txt
# verificcation
$ $ b2sum -c b2sums.txt
# install to USB
$ lsblk
$ sudo dd bs=4M if=path/to/archlinux-version-x86_64.iso of=/dev/disk/by-id/usb-My_flash_drive conv=fsync oflag=direct status=progress
# restart system and start from USB
```

## Set WiFi connect

```bash
# see device name (wlan0)
ip addr show
iwctl
station wlan0 get-networks
exit
iwctl --passphrase "you_pass" station wlan0 connect network_name
```

## Disk partitioning

```bash
$ lsblk
fdisk /dev/disk_name
g
p
n # +1G
n # +1G
n # +100% FREE
w
```
## Disk formatting

```bash
mkfs.fat -F32 /dev/disk_name1
mkfs.ext4 /dev/disk_name2
# create LVM with crypt
cryptsetup luksFormat /dev/disk_name3
cryptsetup open --type luks /dev/disk_name3 lvm
pvcreate /dev/mapper/lvm
vgcreate volgroup0 /dev/mapper/lvm
lvcreate -L 40GB volgroup0 -n lv_root
lvcreate -L 100%FREE volgroup0 -n lv_home
lvdisplay
modprobe dm_mod
vgscan
vgchange -ay
mkfs.ext4 /dev/volgroup0/lv_root
mkfs.ext4 /dev/volgroup0/lv_home
```

## Mount

```bash
mount /dev/volgroup0/lv_root /mnt
mkdir /mnt/boot
mount /dev/disk_name2 /mnt/boot
mkdir /mnt/home
mount /dev/volgroup0/lv_root /mnt
mount /dev/volgroup0/lv_home /mnt/home
```

## Install system

```bash
pacstrap -i /mnt base
genfstab -U -p /mnt >> /etc/fstab
arch-root /mnt
passwd
useradd -mg users -G wheel user_name
passwd user_name
pacman -S base-devel dosfstools grub efibootmgr lvm2 mtools networkmanager os-prober sudo linux-lts linux-lts-headers linux-firmware mesa xorg-server xorg-xinit xorg-xrandr xorg-xrdb i3-wm i3status i3lock i3-nagbar dmenu xinit pcmanfm htop tilda xed nm-aplet setxkbmap ttf-dejavu ttf-liberation xf86-input-libinput ttf-croscore ttf-dejavu ttf-ubuntu-font-family ttf-inconsolata ttf-liberation ttf-font-awesome fontconfig freetype2-demos lxappearance gparted feh deepin-screen-recorder deepin-terminal viewnior ssh-keygen openssh libreoffice p7zip engrampa git code gvfs transmission pipewire pipewire-pulse nodejs npm docker docker-compose 
mkinitcpio -p linux-lts
nano /etc/locale-gen # ru and en
locale-gen
nano /etc/default/grub # GRUB_CDMLINE_LINUX_DEFAULT add before quite "cryptdevice=/dev/disk_name3:voigroup0"
mkdir /boot/EFI
mount /dev/disk_name1 /boot/EFI
grub-install --target=x86_64-efi --bootloader-id=grup_uefi --recheck
cp /usr/share/locale/en\@quot/LC_MASSAGES/grub.mo /boot/grub/locale/en.mo
grub-mkconfig -o /boot/grub/grub.cfg
systemctl enable NetworkMananger
exit 
umount -a
reboot
```

## Setting

```bash
systemctl enable pipewire-pulse.service
timedatectl set-timezone Europe/Moscow
$ groupadd docker
$ sudo usermod -aG docker ${USER}
```
Add in /etc/X11/xorg.conf.d/30-touchpad.conf
```ini
Section "InputClass"
    Identifier "touchpad"
    Driver "libinput"
    MatchIsTouchpad "on"
    Option "Tapping" "on"
    Option "TappingButtonMap" "lmr"
EndSection
```

Add in ~/.xinitrc
```bash
#!/bin/sh
userresources=$HOME/.Xresources
usermodmap=$HOME/.Xmodmap
sysresources=/etc/X11/xinit/.Xresources
sysmodmap=/etc/X11/xinit/.Xmodmap

# merge in defaults and keymaps

if [ -f $sysresources ]; then
    xrdb -merge $sysresources
fi

if [ -f $sysmodmap ]; then
    xmodmap $sysmodmap
fi

if [ -f "$userresources" ]; then
    xrdb -merge "$userresources"
fi

if [ -f "$usermodmap" ]; then
    xmodmap "$usermodmap"
fi

# start some nice programs

if [ -d /etc/X11/xinit/xinitrc.d ] ; then
 for f in /etc/X11/xinit/xinitrc.d/?*.sh ; do
  [ -x "$f" ] && . "$f"
 done
 unset f
fi

exec i3
```

Add in ~.Xresources
```bash
Xft.dpi: 96
Xft.antialias: true
Xft.hinting: true
Xft.rgba: rgb
Xft.autohint: false
Xft.hintstyle: hintslight
Xft.lcdfilter: lcddefault
```
Copy i3 configuration from https://github.com/vzx7/i3-config