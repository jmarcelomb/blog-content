---
author: ["Marcelo Borges"]
title: "How to remove flash protection from the nRF52 using Raspberry Pi Zero + OpenOCD"
date: "2024-08-10"
description: "A detailed guide on how to remove flash protection from the nRF52 microcontroller using a Raspberry Pi Zero and OpenOCD."
summary: "Learn how to unlock and flash your nRF52 microcontroller using a Raspberry Pi Zero and OpenOCD. This guide covers the installation, configuration, and necessary commands to remove flash protection."
tags: ["DIY", "Mechanical Keyboards", "nRF52", "OpenOCD", "Raspberry Pi"]
categories: ["Technology", "DIY Projects", "Tutorials"]
series: ["DIY Mechanical Keyboards"]
ShowToc: true
TocOpen: true
cover:
  image: images/connections.png
  alt: nRF52 connected to Raspberry Pi Zero + OpenOCD
  caption: Connections between Raspberry Pi Zero and OpenOCD.
  relative: true
---

# My Journey into the World of DIY Mechanical Keyboards

I recently fell into the rabbit hole of DIY mechanical keyboards. It all started when I began experiencing pain in my hands, prompting me to search for ergonomic keyboards. That’s when I discovered the ZSA Moonlander Mark 1. However, its high price led me to explore cheaper alternatives, and I stumbled upon the fascinating world of DIY keyboards. After considering options like the Lily58 Pro, Corne, and Ferris, I ultimately chose the 34-key Sweep keyboard.

![Sweep keyboard built](images/keyboard.png#center)

Most build examples I found online used the nice!nano v2. In an effort to save money, I opted for a compatible clone from AliExpress, the Super Mini NRF52840. Unfortunately, it didn’t come with the nice!nano bootloader and had Access Port Protection (APP) enabled, resulting in the error message: `Error: Could not find MEM-AP to control the core` when attempting to write into memory.

# TL;DR

To solve this issue, use the following commands:

```shell
openocd -f interface/raspberrypi-native.cfg -f target/nrf52.cfg -c "gdb_flash_program enable" -c "gdb_breakpoint_override hard" -c "init" -c "reset halt" -c "nrf52_recover"
```

or 

```shell
openocd -f interface/raspberrypi-native.cfg -f target/nrf52.cfg -c "gdb_flash_program enable"
```

And in another shell:

```shell
telnet localhost 4444
nrf52.dap apreg 0x04 0x01
```

# Configuration for interface/raspberrypi-native.cfg

With the follwing `interface/raspberrypi-native.cfg`:
```shell
# SPDX-License-Identifier: GPL-2.0-or-later

adapter driver bcm2835gpio
bcm2835gpio peripheral_base 0x20000000
bcm2835gpio speed_coeffs 113714 28

transport select swd
# Each of the SWD lines need a gpio number set: swclk swdio
# Header pin numbers: 23 22
adapter gpio swclk -chip 0 11
adapter gpio swdio -chip 0 25

# If you define trst or srst, use appropriate reset_config
# Header pin numbers: TRST - 26, SRST - 18

# adapter gpio trst -chip 0 7
#  reset_config trst_only

adapter gpio srst -chip 0 24
reset_config srst_only srst_push_pull

# or if you have both connected,
# reset_config trst_and_srst srst_push_pull
```

# Raspberry Pi OpenOCD installation

I installed Raspberry Pi OS using [Raspberry Pi Imager](https://www.raspberrypi.com/software/), configured it, and connected via SSH. To install OpenOCD on it, I executed the following commands:

```shell
cd ~
sudo apt-get update
sudo apt-get install git autoconf libtool make pkg-config libusb-1.0-0 libusb-1.0-0-dev
git clone http://openocd.zylin.com/openocd
cd openocd
./bootstrap
./configure --enable-sysfsgpio --enable-bcm2835gpio
make
sudo make install
```

# Connections

I soldered the back SWCLK and SWDIO to GPIO 11 and 25, respectively, connected a GND, and the NRF board reset pin to GPIO 24, as shown in the figure below.

![NRF and Raspberry](images/connections.png#center)

# Understanding Access Port Protection

The CTRL-AP (Control Access Port) allows control of the device even if other access ports are disabled by access port protection. If APP is enabled in the APPROTECT register (0x10001208) of the UICR, debugger access to CPU registers and memory is **blocked**.

To check the APPROTECTSTATUS register (0x00C) of the CTRL-AP, execute the following in OpenOCD:

```shell
nrf52.dap apreg 1 0x0c
```

- If the least significant bit is ‘0’, access port protection is enabled.
- If the least significant bit is ‘1’, access port protection is not enabled. Proceed to halt the CPU.

To unlock a device with APP enabled, issue an ERASEALL command through the CTRL-AP, followed by a reset. This erases the entire code flash, UICR area, and RAM. Perform this with:

```shell
openocd -f interface/raspberrypi-native.cfg -f target/nrf52.cfg -c "gdb_flash_program enable" -c "gdb_breakpoint_override hard" -c "init" -c "reset halt" -c "nrf52_recover"
```

or 

```shell
openocd -f interface/raspberrypi-native.cfg -f target/nrf52.cfg -c "gdb_flash_program enable"
```

And in another shell:

```shell
telnet localhost 4444
nrf52.dap apreg 0x04 0x01
```

Re-reading the APPROTECTSTATUS register (0x00C) should now return 1. After unlocking, I flashed the bootloader and began training my muscle memory to use this new layout. I’m already using the new keyboard as I write this post! 😂

The command that I used to flash the bootloader, was:

```shell
$ openocd -f interface/raspberrypi-native.cfg -f target/nrf52.cfg -c "gdb_flash_program enable" -c "gdb_breakpoint_override hard" -c "init" -c "reset halt" -c "flash write_image erase ./nice_nano_bootloader-0.6.0_s140_6.1.1.hex"
```

# References

- [Raspberry Pi and OpenOCD – Lean2](https://iosoft.blog/2019/01/28/raspberry-pi-openocd/)
- [Nordic Semiconductor Infocenter](https://infocenter.nordicsemi.com/index.jsp?topic=/nwp_027/WP/nwp_027/nWP_027_protected.html)
- [Removing flash protection from the nRF52 using JLink + OpenOCD](https://blog.dbrgn.ch/2020/5/16/nrf52-unprotect-flash-jlink-openocd/)
- [Troubleshooting - nice!nano](https://nicekeyboards.com/docs/nice-nano/troubleshooting)

I hope this helps someone save time figuring it all out.
Feel free to ask if you need further assistance!
See you in the next post!

**jmmb**