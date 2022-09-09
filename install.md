# 🚀 Installation Guide

_Created on: September 10, 2022_

This is an installation process of [Arch Linux](https://archlinux.org/) from the ground up focusing on ROCK Pi X device while implementing the conceptual model of a kiosk. It is suggested to refer with the [official installation guide](https://wiki.archlinux.org/title/Installation_guide) for more up-to-date steps while also refering this guide for kiosk specific procedures.

> 🔲 **TODO:** Diagram of conceptual model of kiosk.

## 📙 Prerequisites

> ⚠ **Warning:** The following operations will **_WIPE OUT ALL OF THE DRIVE’S DATA_**. Make sure to backup all **_IMPORTANT FILES_** to other safe drive before proceeding.

- USB flash drive with a capacity of at least **2GB**
- Download the [Arch Linux iso](https://archlinux.org/download/)
- For Windows, use [Rufus](https://rufus.ie/en/) or any similar tools
- For Linux and Mac, `dd` command is enough. [Refer to this guide](https://wiki.archlinux.org/title/USB_flash_installation_medium#:~:text=using%20dd:) for more information
- Format the flash drive in **UEFI** mode and **GPT** partition scheme
- Power on the ROCK Pi X with ethernet cable connected
- Access the BIOS of ROCK Pi X by pressing <kbd>F12</kbd> on boot
- Temporarily disable secure boot and fast boot
- Plug-in the bootable flash drive, save and exit

> ℹ **Information:** Supposedly, it will automatically boot the installation medium, if incase it went straight to UEFI shell, enter the command `reset`. It will reboot the device and that will be the chance to access the BIOS again. Change the **boot priority order** with the _name_ of the bootable drive or alternatively, select it in the **boot override** section.

> ℹ **Information:** During the boot up of the installation medium, it will ask to choose which mode of Arch Linux to boot. Just select the default by pressing <kbd>Enter ↵</kbd> and after some time, it will logged-in to the **virtual console.**

## 📋 Procedures

> ⚠ **The AP6255 WiFi & BT Firmware seems to be buggy:** Based from the several articles found on the internet, the wireless module looks like buggy. This installation guide will not cover fixing the wireless bugs yet but it is planned to fix it some time.

> ℹ **Information:** This assumes that the device network will be connected via **wired connection** as the following steps focuses on **ethernet configuration setup.** [Refer to this guide](https://wiki.archlinux.org/title/Network_configuration/Wireless) for more details about setting up with wireless connection.

### Step 1

##### Enlarge console font size
```console
# setfont ter-122n
```

#### Test internet if it's already working
```console
# ping google.com
```

#### Ensure system clock is accurate
```console
# timedatectl set-ntp true
# timedatectl set-timezone Asia/Manila
# timedatectl status
```

### Step 2

#### Manage disks & partitions with ext4
```console
# lsblk
# cfdisk /dev/mmcblk1
```

Follow this partition layout for 4GB RAM and 32GB eMMC variant of ROCK Pi X. For other variants with different configurations, the `/boot` and `/` can stay the same but the `SWAP` can vary depending on the availability of RAM and storage.

| Mount   | Size        | Partition          | Partition Type   |
| ------- | ----------- | ------------------ |----------------- |
| /boot   | 512M        | mmcblk<em>X</em>p1 | EFI System       |
| _SWAP_  | 2048M       | mmcblk<em>X</em>p2 | Linux Swap       |
| /       | _remaining_ | mmcblk<em>X</em>p3 | Linux filesystem |

#### Format partitions
```console
# mkfs.fat -n EFI /dev/mmcblk1p1
# mkswap /dev/mmcblk1p2
# swapon /dev/mmcblk1p2
# mkfs.ext4 -L Linux /dev/mmcblk1p3
```

#### Mount partitions
```console
# mount /dev/mmcblk1p3 /mnt
# mkdir /mnt/boot
# mount /dev/mmcblk1p1 /mnt/boot
```

### Step 3

#### Set latest & nearest synchronized mirrors
```console
# reflector --country 'Singapore,Taiwan,Hong Kong,Japan,' --sort rate --save /etc/pacman.d/mirrirlist.bak
```

#### Combine the latest to old mirrorlist
```console
# cd /etc/pacman.d
# cat mirrorlist >> mirrirlist.bak
# rm mirrorlist
# mv mirrirlist.bak mirrorlist
# cd ~
```

### Step 4

#### Install Arch Linux base system to /mnt
```console
# pacman -Sy
# pacstrap -i /mnt base base-devel linux linux-firmware ntfs-3g vim git dhcpcd intel-ucode booster
```

#### Generate fstab & chroot to /mnt
```console
# genfstab -U -p /mnt >> /mnt/etc/fstab
# arch-chroot /mnt
```

#### Set timezone
```console
# ls /usr/share/zoneinfo/
# ln -sf /usr/share/zoneinfo/Asia/Manila > /etc/localtime
# hwclock --systohc --utc
```

#### Configure language
> ℹ **Information:** To do basic editing and exiting using vim, press <kbd>i</kbd> to go into insert mode. This will allow text editing. When done, to save and exit, press <kbd>Esc</kbd> and then followed by <kbd>:wq</kbd>.
```console
# vim /etc/locale.gen
```
> _uncomment:_ **en_US.UTF-8 UTF8**
```console
# locale-gen
# echo LANG=en_US.UTF-8 > /etc/locale.conf
# export LANG=en_US.UTF-8
```

#### Add multilib repository
```console
# vim /etc/pacman.conf
```
> _uncomment:_ **[multilib]** and **include = /etc/pacman.d/mirrorlist**

#### Sync and refresh the package database
```console
# pacman -Sy
```

### Step 5

#### Set hostname
```console
# echo kiosk > /etc/hostname
# vim /etc/hosts
```
> Use this [hosts file](./archrootfs/etc/hosts) for reference.

#### Configure dhcpcd for quiet and fast start up initialization
```console
# vim /etc/dhcpcd.conf
```
> Add [this lines](./archrootfs/etc/dhcpcd.conf) to the end of the file.

### Step 6

#### Set root password and create admin user
```console
# passwd
# useradd -m admin -G wheel -s /bin/bash -c "Admin"
# passwd admin
```

#### Allow users in wheel group to do sudo tasks
```console
# EDITOR=vim visudo
```
> _uncomment:_ **%wheel ALL=(ALL:ALL) ALL**

#### Create XDG compliant user directories
```console
# pacman -S xdg-user-dirs
# su admin
$ cd ~
$ xdg-user-dirs-update
$ exit
```

### Step 7

#### Configure systemd bootloader with microcode patches
```console
# bootctl install
# blkid | grep mmcblk1p3 > /boot/loader/entries/arch.conf
# vim /boot/loader/entries/arch.conf
```
> Use this [boot entry configuration file](./archrootfs/boot/loader/entries/arch.conf) for reference.

#### Unmount partitions & reboot
```console
# exit
# umount -R /mnt
# reboot
```

## ⏩ Post-Installation

### Step 1

#### Enable some necessary services
```console
$ sudo timedatectl set-ntp true
$ sudo timedatectl set-timezone Asia/Manila
$ sudo timedatectl status
$ sudo systemctl enable --now dhcpcd.service fstrim.timer
```

### Step 2

#### Install Intel Drivers (for codename: Cherry Trail)
```console
$ sudo pacman -S mesa lib32-mesa vulkan-intel lib32-vulkan-intel libva-intel-driver libva-utils xorg-xwayland
```

### Step 3

#### Install Cage Kiosk Compositor
```console
$ sudo pacman -S cage
$ sudo systemctl enable --now seatd.service
```

### Step 4

#### Add guest user that is only allowed to cage
```console
$ sudo useradd -m guest -G seat -s /bin/bash -c "Guest"
$ sudo passwd guest
```

