---
layout: default
title: Kernel Modules
parent: Linux Analysis
nav_order: 2
---

## Kernel Modules
The term kernel modules is used to represent the technically correct term, loadable kernel modules (LKMs).
LKMs provide a way to add code to the Linux kernel while it is running, as opposed to adding code to the kernel and recompiling it.
Granted LKMs provide flexibility to customize a system's kernel on the fly, they also provide a nice opportunity to install kernel-level rootkits.

__Loaded Kernel Modules__
```bash
lsmod
# The lsmod utility is used to show the statuses of currently loaded modules in the Linux Kernel.
# The Module column displays the module name.
# The Size column dispays the amount of memory being used by the module in bytes.
# The Used by column displays the number of instances of the module in use. It also shows any system information, such as what might be using the module.
# Source: https://www.computerhope.com/unix/lsmod.htm
```
```bash
less /proc/modules
# The /proc/modules file lists all modules currently loaded into the kernel.
```
__Module Information__
```bash
modinfo <MODULE_NAME>
# The modinfo utility is used to show information about a specified module.
```
  
#### Demonstration:
We'll take a look at the Diamorphine rootkit, a well-known open source Linux rootkit used for research purposes, to demonstrate their dangers.
First, download the Diamorphine project from GitHub (Copyright (c) 2014, Victor N. Ramos Mello
All rights reserved.):
```bash
git clone https://github.com/m0nad/Diamorphine
```
This will create a directory titled 'Diamorphine' in the directory you cloned the project from. Change you working directory to Diamorphine:
```bash
cd Diamorphine
```
Next, we need to compile the module using the provided Makefile:
```bash
make
```
If you take another look at the directory, you'll see a few more files added. Particularly, we care about the one with the .ko extension, as it is the actual kernel object, or LKM.
We can then load this module into the kernel:
```bash
insmod diamorphine.ko
```
What can you see? Or rather, what can you not see? Is it actually loaded? Try issuing the following command and then check what user you are:
```bash
kill -64 0
```
By default, the diamorphine module is hidden. To make it visible, issue the following command:
```bash
kill -63 0
```
Now check what modules are loaded and you should see diamorphine. Also, take a look at the module info for the diamorphine.ko file and compare it to a legitimately loaded module.
When you're done, you can remove diamorphine with the command:
```bash
rmmod diamorphine
```
