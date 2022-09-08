<!-- TODO: Large logo -->

# <p align="center">CMU Kiosk Device — The Device Specification, Operating System Installation, Configuration, Scripts and dotfiles.</p>

## 💻 Device Specification

### Main Board

CMU Kiosk is powered by [ROCK Pi X](https://wiki.radxa.com/RockpiX). An x86 Single Board Computer (SBC) by [Radxa](https://wiki.radxa.com/Special:SpecialContact/).

Below is the specification table of ROCK Pi X Model B that is being developed.

| CPU | RAM | eMMC | GPIO | Power |
| --- | --- | ---- | ---- | ----- |
| Intel Atom® x5-Z8350<br>(4-Cores) | 4GB | 32GB | 40-Pin | USB Type-C PD 2.0<br>Qualcomm® Quick Charge™ 3.0/2.0 |

Refer to the [ROCK Pi X technical specifications page](https://wiki.radxa.com/RockpiX/hardware) for more details.

### Partition Layout

> ⚠ **This section needs a final decision:** It is unclear right now whether suspend is sufficient or hibernation is needed. For more information, [refer to this guide](https://help.ubuntu.com/community/SwapFaq#How_much_swap_do_I_need.3F) about handling swap sizes.

Below is the partition layout with the 32GB eMMC variant of the ROCK Pi X using `cfdisk /dev/mmcblkX`.

| Mount   | Size        | Partition          | Partition Type   |
| ------- | ----------- | ------------------ |----------------- |
| /boot   | 512M        | mmcblk<em>X</em>p1 | EFI System       |
| _SWAP_  | 2048M       | mmcblk<em>X</em>p2 | Linux Swap       |
| /       | _remaining_ | mmcblk<em>X</em>p3 | Linux filesystem |

> ℹ **Information:** mmcblk<em>X</em> might show a different number during live boot. It may be due to _race conditions_. It is **highly advised** to use `lsblk` to check what number it was assigned to.

Linux swap is _optional_ but **recommended** due to limited amount of available RAM. [According to this guide](https://help.ubuntu.com/community/SwapFaq#How_much_swap_do_I_need.3F), if _suspend_ is only needed, then the _square root_ of RAM is enough (e.g.: **√4GB** is **2GB**. Hence, swap size would be 2GB). However, if _hibernation_ is needed, add the size of _square root_ of RAM to the size of RAM (e.g.: **√4GB** is **2GB**. Hence, swap size would be 4GB + 2GB = **6GB**).

<!-- TODO: Small logo -->


## 📝 License

Copyright © 2022, [Vherniel Lebis](https://vherniellebis.tech). <br>This project is [MIT](https://github.com/Vherniel/cmu-kiosk-device/LICENSE) licensed.
