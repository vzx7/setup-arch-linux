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
```
