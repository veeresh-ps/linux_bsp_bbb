# Linux Boot Process — Part 3: What Exactly Does SPL Do?**

In the previous post, we saw how the **AM335x Boot ROM** decides where to look for the first boot image.

But what happens after Boot ROM loads it?

This is where **SPL — Secondary Program Loader** takes over.

On the BeagleBone Black, the first-stage boot image is commonly the **MLO/SPL** image.

So the boot chain is:

```text id="x4nq2f"
Power ON
   ↓
AM335x Boot ROM
   ↓
MLO / SPL
   ↓
U-Boot
   ↓
Linux Kernel
   ↓
Userspace
```

But why do we need SPL?

Because Boot ROM cannot initialize the entire system and run a full-featured bootloader.

SPL acts as the **bridge between the minimal Boot ROM environment and full U-Boot**.

---

## What does SPL actually do?

A simplified view is:

```text id="v7p2xk"
              Boot ROM
                  │
                  ▼
              MLO / SPL
                  │
        ┌─────────┼─────────┐
        ▼         ▼         ▼
      Clocks     DDR      Pinmux /
      / PLL      init     peripherals
        │         │         │
        └─────────┼─────────┘
                  ▼
            Load U-Boot
                  │
                  ▼
               U-Boot
```

The exact initialization depends on the board and SPL configuration.

But **DDR initialization is particularly important**.

Once the required hardware is initialized, SPL can load the much larger U-Boot image into memory and transfer execution to it.

---

## Now let's break it

This is where understanding SPL becomes useful for BSP debugging.

### Failure #1 — MLO/SPL is missing

Remove or rename `MLO` from the boot media.

Then power on the BBB.

```text id="3o8q5e"
Boot ROM
   │
   ├── Select boot device
   │
   ├── Look for first-stage image
   │
   └── MLO unavailable
           │
           X
```

You won't get normal SPL or U-Boot output because **SPL never started**.

This tells us:

**The failure is before SPL.**

---

### Failure #2 — SPL starts, but U-Boot doesn't

Now restore a valid MLO/SPL.

Suppose the UART output looks like:

```text id="o3q1t8"
U-Boot SPL ...
Trying to boot from MMC...
```

and then stops.

But you never see the full U-Boot banner.

```text id="s0n5ik"
U-Boot SPL
     │
     │
     X
     │
     ▼
  U-Boot
```

This is a very different failure.

We have evidence that:

**Boot ROM → SPL succeeded.**

But:

**SPL → U-Boot did not complete.**

Now our investigation should focus on SPL.

Possible areas include:

* DDR initialization
* Clock configuration
* Pinmux
* Boot-device access
* Loading the U-Boot image
* Image location/format
* Memory corruption
* SPL configuration

---

### Failure #3 — DDR initialization failure

This is where things get more interesting.

SPL needs working memory to continue with larger software.

If DDR initialization/configuration is incorrect, the board can stop very early.

You may see SPL output followed by a hang, reset, or no further progress.

Conceptually:

```text id="ljy2br"
Boot ROM
   ↓
SPL
   ↓
Clock initialization
   ↓
DDR initialization
   X
   │
   └── U-Boot never starts
```

The important lesson isn't the exact error message.

It is the **failure boundary**.

If SPL is running but full U-Boot isn't, don't jump immediately to Linux debugging.

---

### Failure #4 — U-Boot image cannot be loaded

Another possibility is that SPL starts correctly but cannot obtain the next-stage image.

```text id="5d2v5j"
Boot ROM
   ↓
SPL
   ↓
Initialize hardware
   ↓
Access boot device
   ↓
Find/load U-Boot
   X
   │
   └── U-Boot never executes
```

Again, the debugging boundary is clear:

**Boot ROM is no longer your first suspect.**

**Linux is definitely not your first suspect.**

Start with SPL and the boot medium.

---

## This gives us a useful debugging model

When a BBB doesn't boot, don't ask:

**"Why isn't Linux booting?"**

Ask:

**"What is the last stage I can prove executed?"**

For example:

```text id="2y9k8d"
                 Boot ROM
                    │
                    ▼
                   SPL
                    │
             ┌──────┴──────┐
             │             │
          Success        Failure
             │             │
             ▼             X
           U-Boot       STOP HERE
             │
             ▼
           Kernel
             │
             ▼
          Userspace
```

Your serial console becomes a **checkpoint system**.

```text id="f5t6u4"
SPL banner       → SPL executed
U-Boot banner    → U-Boot executed
Kernel messages  → Linux executed
systemd messages → Userspace started
login prompt     → Userspace is operational
```

Each checkpoint narrows the problem.

That is one of the most important mental models in **Linux BSP debugging**.

> **Don't debug the entire boot chain at once. Find the last stage that definitely executed, then debug forward from there.**

Next:

**Linux Boot Process — Part 4: How Does U-Boot Load the Linux Kernel?**

We'll follow the actual BBB U-Boot environment and see how U-Boot finds the kernel, Device Tree and boot arguments before handing control to Linux.

#EmbeddedLinux #LinuxBSP #BeagleBoneBlack #BoardBringup #SPL #Uboot #AM335x #EmbeddedSystems #Bootloader
