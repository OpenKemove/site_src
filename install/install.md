---
title: How to Install
layout: page
nav_order: 2
---

# How to install

**Please follow the steps carefully. Make sure you keep the backup of the original firmware in a safe place in case you need to perform a recovery.**

**Current QMK only works for PCB revision 3.0, if you have 1.2 please wait as
we are working on a port for you. Please join the discord for more info on this**

# Understanding how it works

Before we begin, I would like to explain how this works in general. So in the
off chace that things go wrong, you at least know what to google or how to ask
for help.

Please review the [How it all works]({% link install/theory.md %}) page.

Of course the discord server is always a great place to ask for help or just
have any kind of general question. People are always helpful there.
[Open Kemove Discord Server](https://discord.gg/TFeG4cb3yk)

# Step 0: Gather Files

<details>
    <summary>Windows</summary>
    We will assume you have Windows 10 and have WSL or WSL2 installed. If not
    please install that. We will have tags that explain what you need to do
    differently for certain sections of the instruction.
</details>

## ARM GCC Compiler / Toolchain

This is often a package in your package manager. On Ubuntu this can be done using
```bash
sudo apt install gcc-arm-none-eabi binutils-arm-none-eabi make
```

## Git
```bash
sudo apt install git
```

## DFU-Tools

For linux users, we strongly recommend the `dfu-util` tool to flash the firmware. If you run into permission problems you can always create udev rules, or simply run the tool using sudo.

```bash
sudo apt install dfu-util
```

**NOTE: Do not run this under WSL it will not work. WSL has no USB support as of 2020**

# Step 1: Get and Build bootloader

0. Clone the bootloader repository.

```bash
git clone https://github.com/OpenKemove/kemove_lpc_dfu.git --recursive --depth=1
```
`recursive` here will tell git to clone all submodules.

`depth=1` will tell git to not clone history. You can omit this if you planning on developing the firmware.

0. Build the bootloader firmware
```bash
cd kemove_lpc_dfu
make
```

** OR You can download a pre-built binary **
[Automatic Build System](https://ci.codetector.org/job/OpenKemove/job/kemove_lpc_dfu/job/master/)
Download the `lpc_boot.bin`

Yep, that's it. simple. Hopefully it has successfully built with no errors.
You should see a few files under build. Most imporantly you see the `lpc_boot.bin`

# Step 2: Build QMK

0. Clone our QMK Fork
```bash
git clone https://github.com/OpenKemove/qmk_firmware.git --recursive --depth=1
```

0. Build QMK
```bash
cd qmk_firmware
make kemove/snowfox
```
Once this is done, you should see a `kemove_snowfox_default.bin` in the directory.

# Step 3: Backup!

** WARNING **
> This step might require Windows / macOS. It seem to often have problem on Linux. Feel free to try, (and reference the macOS steps). If you have problems,
try use a Windows VM or a Windows machine.

Let's start by taking a backup of the existing firmware. To do this, you will need to remove the PCB from the keyboard body and all the switches. Once you are done you should be left with a bare PCB. On the front side (Side that
switches would be installed onto), you should be able to find some exposed
metal contacts (These are called test points, used for testing). What you want
to do is find a resistor (100-1K Ohm will work). (If you can't find a resistor a wire or paper clip could work too but not ideal.)

Since you should alreay have the battey detached, the keyboard should have no
power unless we plug in the cable.

You want to find the test point that says `ISP` next to it. Then you will need to find another test point that says `GNDx` (x here could be any number). Any ground pad could work. You will connect the `ISP` point and `GND` with your resistor/wire, then plugin the keyboard. You can remove the wire after you plug in. You should see a new "flash drive" show up on your computer. Open it,
you can see the `firmware.bin` file. Copy it to a safe place, it should be exactally `64 KB` in size. (This might vary depending on how Windows / macOS /
Linux round file size differently.) This is your backup of the old firmware.
Once this is done, you can `DELETE` this file from the "flash drive". Then
you can copy the bootloader (`lpc_boot.bin`) into the "flash drive". You do
not need to rename it.

Once you are done with that, unplug the keyboard, and replug while holding
the `ESC` key. This is the key that will trigger the DFU mode in the future.
You should see a new device pop up in lsusb / device manager. It should be
`OpenKemove DFU`

** NOTE: macOS **
For macOS users, please **DO NOT** open the flash drive in Finder. Instead
please complete the operation using command line interface. You can do this
with the following steps:

0. Wait for the drive icon to show up in desktop.
0. Open a termial
0. `cd ` (space after `cd`) then drag the drive icon into your terminal. It should fill in the path. Then press enter.
0.  ```bash
    rm -rf firmware.bin
    sync
    ```
0. `cp PATH_TO_LPC_BOOT.BIN .` (You can do the drag thing as well for `lpc_boot.bin`)
0. `sync`
0. `diskutil list`, then find the diskX where X is a number which contains the flash drive.
0. `diskutil unmountDisk diskX` this is the diskX you found in the previous step.




# Step 4: Flash QMK!
Now we can use dfu-util to flash the device. Run `dfu-util` in the
`qmk_firmware` directory. Make sure it can detect the device. You can verify this by running. Check for the pair *feed:6969*.

**Note: Windows Users** Please follow [This Guide](https://www.hanselman.com/blog/how-to-fix-dfuutil-stm-winusb-zadig-bootloaders-and-other-firmware-flashing-issues-on-windows) on how to install the generic libusb driver


```bash
dfu-util -l
```
you should see something like this:
```
dfu-util 0.9

Copyright 2005-2009 Weston Schmidt, Harald Welte and OpenMoko Inc.
Copyright 2010-2016 Tormod Volden and Stefan Schmidt
This program is Free Software and has ABSOLUTELY NO WARRANTY
Please report bugs to http://sourceforge.net/p/dfu-util/tickets/

Found DFU: [feed:6969] ver=0200, devnum=4, cfg=1, intf=0, path="1-12", alt=0, name="UNKNOWN", serial="604"
```
Then you can flash the firmware by using
```bash
dfu-util -d feed:6969 -D kemove_snowfox_default.bin
```
If you run into any error, please try connecting the keyboard directly to
the computer without any HUB etc. Or just try a few times. Remeber to unplug / replug (while holding esc) between tries.

Once it is done, you board should be running QMK. (you might need to unplug / replug) for it to boot QMK. The auto boot works most of the time but occasionally seems buggy.
