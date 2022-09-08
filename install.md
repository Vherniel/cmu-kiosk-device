# 🚀 Installation Guide

_Created on: September 10, 2022_

This is an installation process of [Arch Linux](https://archlinux.org/) from the ground up focusing on ROCK Pi X device while implementing the conceptual model of an kiosk. It is suggested to refer with the [official installation guide](https://wiki.archlinux.org/title/Installation_guide) for more up-to-date steps while also refering this guide for kiosk specific procedures.

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
    # setfont ter-122n

#### Test internet if it's already working
    # ping google.com

#### Ensure system clock is accurate
    # timedatectl set-ntp true
    # timedatectl set-timezone Asia/Manila
    # timedatectl status

### Step 2

#### Manage disks & partitions with ext4
    # lsblk
    # cfdisk /dev/mmcblk1

Follow this partition layout for 4GB RAM and 32GB eMMC variant of ROCK Pi X. For other variants with different configurations, the `/boot` and `/` can stay the same but the `SWAP` can vary depending on the availability of RAM and storage.

| Mount   | Size        | Partition          | Partition Type   |
| ------- | ----------- | ------------------ |----------------- |
| /boot   | 512M        | mmcblk<em>X</em>p1 | EFI System       |
| _SWAP_  | 2048M       | mmcblk<em>X</em>p2 | Linux Swap       |
| /       | _remaining_ | mmcblk<em>X</em>p3 | Linux filesystem |

#### Format partitions
    # mkfs.fat -n EFI /dev/mmcblk1p1
    # mkswap /dev/mmcblk1p2
    # swapon /dev/mmcblk1p2
    # mkfs.ext4 -L Linux /dev/mmcblk1p3

#### Mount partitions
    # mount /dev/mmcblk1p3 /mnt
    # mkdir /mnt/boot
    # mount /dev/mmcblk1p1 /mnt/boot