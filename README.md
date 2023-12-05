# Introduction
Welcome to the Intel TDX on Linux Github - your source for Intel Linux TDX enabling information. 
If you are navigating through the world of Linux TDX patches, trying to keep your system up-to-date with the latest improvements, here is the resource for you.
The aim is to provide a tested recipe of TDX patches with an upstream verifiable path, that works on top of recent major kernel release.

**What We Offer:**
* Linux kernel patches required on a recent kernel to build base TDX host and guest 
* Related documentation
  * [Instruction to set up TDX host and guest](https://github.com/intel/tdx-linux/wiki/Instruction-to-set-up-TDX-host-and-guest)
  * [List of Linux kernel patches for base TDX](https://github.com/intel/tdx-linux/wiki/List-of-Linux-kernel-patches-for-base-TDX)

# How to use this repo
This repo contains patches that are necessary to build a base TDX host kernel environment, then boot a trusted virtual machine (TD VM) using Intel's TDX technology.
Untar the tarball in folder "kernel-patch" and then apply all the patches to the recent vanilla kernel (See NEWS for kernel version and other components' version).

Refer to the wiki page [Instruction to set up TDX host and guest](https://github.com/intel/tdx-linux/wiki/Instruction-to-set-up-TDX-host-and-guest) for how to build, configure the host OS and boot a TD VM. 
