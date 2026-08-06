---
title: Learning Linux Drivers One Tiny Kernel Module at a Time
date: 2026-08-06 11:00:00
category: projects
tags:
  - Linux
  - kernel
  - C
  - drivers
---

Linux driver development has an awkward learning curve. A minimal mistake is no longer a friendly exception; it can be a frozen machine and a forced reboot. I made [linux_driver_introduction](https://github.com/sanleches/linux_driver_introduction) to learn the pieces in small enough steps that I could tell which idea broke.

This is not one production driver. It is a sequence of tiny C modules, each focused on one kernel concept: module lifecycle, metadata, log levels, character-device registration, file callbacks, and inspecting USB devices from user space.

## Start with hello, but in kernel space

The first module only registers init and exit functions. `insmod` prints hello to the kernel log and `rmmod` prints goodbye. The second version adds `static`, `__init`, `__exit`, and module metadata so tools such as `modinfo` can describe it.

That sounds trivial, but it establishes the basic loop:

```bash
make
sudo insmod hello.ko
sudo dmesg -w
sudo rmmod hello
```

Kbuild does the real compilation by entering the build tree for the currently running kernel and treating the lesson directory as an external module.

## A character device that returns absolutely nothing

The next interesting lesson calls `register_chrdev()` with major number `0`, asking the kernel to allocate a major dynamically. After loading the module, I read that number from `dmesg` and create a node manually:

```bash
sudo mknod /tmp/hello0 c <major> 0
sudo cat /tmp/hello0
```

The driver's `.read` callback logs that it was called and returns `0`. In the Unix read contract, zero means EOF. It does not copy a message to user space and it does not pretend to be a useful device. The whole point is to watch a normal `cat` travel through the VFS into my `struct file_operations` callback.

Another lesson registers `.open` and `.release`. A small user-space helper opens the node as read-only, read/write with `O_SYNC`, and write-only with `O_NONBLOCK`. The module prints the inode major/minor plus `f_pos`, `f_mode`, and `f_flags`, making the relationship between user-space `open()` flags and kernel-side file state visible.

## Logging is part of the interface

One module exists only to emit `KERN_INFO`, `KERN_WARNING`, `KERN_ALERT`, `KERN_DEBUG`, and the equivalent `pr_*` messages. Kernel debug output can disappear depending on the active console and logging configuration. Learning that with a throwaway module is better than wondering why a real driver's diagnostic message never appeared.

## Looking up my USB experiment

The last lesson includes a separate libudev program that searches the USB subsystem for vendor `6666`, product `0001`, and device revision `0100`. It prints the matching sysfs path, model, device node, major/minor numbers, and attached driver when available.

That helper is not a USB kernel driver. It is a normal user-space inspector reading information Linux already exposes through udev and sysfs. There is no `probe()` callback or HID report handling yet. Keeping that boundary explicit is part of the lesson.

## The safety section nobody should skip

These modules are intentionally incomplete. Device nodes are manual, there is no real data transfer, and the examples do not include the synchronization, ownership, permission policy, automatic device creation, or unload protections expected from production code.

I run this kind of code on a development machine or VM, unload one lesson before loading another with the same module name, and assume the kernel will punish unjustified confidence.

The project is useful because every step is small enough to reason about. Load a module, watch one kernel mechanism happen, remove it, and then add the next piece.

Code: [github.com/sanleches/linux_driver_introduction](https://github.com/sanleches/linux_driver_introduction)
