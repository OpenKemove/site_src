---
title: How it all works
layout: page
nav_order: 1
---

> Currently we only have information about the Snowfox, but this should
generally apply to any keyboard. We will update this information as we bring
the port to more kemove boards.

## Kemove Snofox Hardware

If you take apart the board, and on the back of the PCB we can see a few
Integrated Circuits (ICs). We will go over them one by one.

With the battery connector facing up, we can see on the bottom there are the main controller on the right. It is an NXP LPC11U35/401. It is an ARM
Cortex-M0 micro-controller running at 48Mhz. This is the "brain"
of the keyboard. It handles matrix scanning (this is how the keyboard is able
to tell what switch is currently being pressed), USB Communication, and talks
to the LED controllers to set LED brightness and color.

Next to the left, on the green board that is soldered down, it is the Bluetooth
module. At its core it is a BCM20730 (CYW20730). It is also an ARM controller,
but with bluetooth capability. However the programming interface of this chip
is not the standard ARM SWD interface, and in fact a bluetooth firmware could
be quite complicated, thus we will not be touching it at this moment.

## Software: Main Controller (MCU)

We are mostly interested in replacing the firmware on the main controller.
Usually firmware on these ARM based controller can be updated using a debugger
that supports the ARM SWD (Serial Wire Debug) interface. E.g. J-Link, ST-Link.
However this is quite annoying if the user has to update the firmware like in
the case of the keyboard. So often these devices actually run two firmware on
it, one is the main firmware that implements the function of the device, this
our case the keyboard; and another firmware act as an update tool for the other
firmware, we often call this bootloader.

This bootloader can range from simple to complicated, and can be very generic
or very specific to what kind of firmware it can accept. The one came with the
kemove board doesn't seem to have a way for us to "force" it into the update
mode. This is rather undesireable because we would like to have the ability to
recover even if the main firmware is completely dead or misbehaving. For
that reason, we will be replacing both the firmware and the bootloader on the keyboard.

## Replacing the bootloader
You might ask, how would one replace the bootloader without a programmer?
Fortunately for us, the LPC11U35 has a built in update mode where it will
present itself as a USB-MSD device (USB Mass Storage). Put it more plainly,
this will make the controller show up as a "Flash Drive" on you computer.
And you just need to copy the firmware in and out of it to flash it!

So we will flash the bootloader this way.
