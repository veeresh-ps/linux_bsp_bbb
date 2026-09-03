# Linux Boot Process — Part 1: What Executes First on a BeagleBone Black?

You power on a BeagleBone Black.

What executes first?

**Linux? No.**

**U-Boot? No.**

The first code that gets control is the **Boot ROM inside the TI AM335x SoC**.

This is where the Linux boot process actually begins.

A simplified BBB boot flow is:

**Power-On / Reset**
↓
**AM335x Boot ROM**
↓
**SPL / MLO**
↓
**U-Boot**
↓
**Linux Kernel**
↓
**Root Filesystem**
↓
**Userspace**

The Boot ROM is not something you installed on the board.

It is code already present inside the SoC.

Its job is to perform the very early boot operations required to find and load the next stage.

On the BeagleBone Black, the boot configuration determines which boot source the AM335x should use.

The board has:

* On-board eMMC
* microSD
* USB boot capability
* Serial boot capability

By default, the board boots from eMMC. The boot button can be used to select the microSD boot path.

At this point, we are still **far away from Linux**.

No Linux kernel.

No Device Tree.

No root filesystem.

No userspace.

The processor is still executing the very early boot code required to get the system to the next stage.

This gives us an important BSP debugging principle:

**When a board doesn't boot, don't immediately debug Linux.**

First ask:

**"How far did the processor actually get?"**

If execution never reaches SPL, debugging U-Boot is premature.

If U-Boot never starts, debugging the Linux kernel is premature.

If Linux starts but userspace doesn't, the problem is somewhere further down the boot chain.

This is the mental model I want to build throughout this series.

In the next post, we'll go one level deeper:

**How does the AM335x Boot ROM actually find the next image on the BeagleBone Black?**

We'll look at boot configuration, eMMC/microSD, **MLO/SPL**, and what is actually sitting on the boot media.
