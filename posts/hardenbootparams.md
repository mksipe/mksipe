<p style="text-align: center; font-size: 40px;">Hardening Boot Parameters</p>

---

##### Metadata

###### Last revised - 11/8/21

###### Author       - Mason Sipe

---

## Preamble

This article shows the different kinds of settings that can be applied to the bootloader's settings when booting a Linux system. These are all security suggestions that allow for increased security. However, be careful and thorough research some of these settings as some may lead to your system not booting. I would HIGHLY suggest trying to apply settings temporarily through grub's editor upon system startup. You can do this by pressing `e` while the OS selection screen is shown. 


## Background:

### GRUB

The Grand Unified Bootloader is what loads the Linux system into memory. It's impossible to start an operating system without it. 

According to TutorialSpot, Grub supports the following:

```
GRUB is the default bootloader for many of the Linux distributions. GRUB is because it is better than many of the previous versions of the bootloaders. Some of its features are:

* GRUB supports LBA (Logical Block Addressing Mode), which puts the addressing conversion used to find files into the firmware of the hard drive
* GRUB provides maximum flexibility in loading the operating systems with required options using a command[-]based, pre-operating system environment.
* The booting options such as kernel parameters can be modified using the GRUB command line.
* There is no need to specify the physical location of the Linux kernel for GRUB. It only required the hard disk number, the partition number[,] and file name of the kernel.
* GRUB can boot almost any operating system using the direct and chain loading boot methods.
```

Overall, it is a critical part of the fundamental process of booting Linux, which makes it a valuable target for perpetrators. 


### Boot Parameters

Boot parameters are the settings that are passed through grub to allow for your Linux system to boot. After installing a Debian system, there is a default of 5 seconds of wait time until you automatically boot into the system. This is an example of a default boot parameter. This example specifically can be mitigated quite simply. However, several other options are available to prevent more malicious events from happening rather than a boot hijack.

## Table of Contents


- [Boot Parameters](#boot-parameters)
    - [Slab Nomerge](#slab-nomerge)
    - [Slub Debug](#slub-debug)
    - [Zeroing Memory](#zeroing-memory)
    - [Shuffle Page Allocations](#shuffle-page-allocations)
    - [Mitigate Known Issues](#mitigate-known-issues)
    - [Disable Vsyscalls](#disable-vsyscalls)
    - [Protect FS Debugs](#protect-fs-debugs)
    - [Set OOPS Action](#set-oops-action)
    - [Only Allow Signed Kernel Modules](#only-allow-signed-kernel-modules)
    - [Kernel Lockdown LSM](#kernel-lockdown-lsm)
    - [Protect ECC Memory](#protect-ecc-memory)
    - [Prevent Information Leaks During Boot](#prevent-information-leaks-during-boot )
- [CPU Vulnerabilities](#cpu-vulnerabilities)
- [Results](#results)



---


## Boot Parameters

Configuration = `/etc/defaults/grub` or `/etc/syslinux/syslinux.cfg`

If you are using grub, add the parameters after `GRUB_CMDLINE_LINUX_DEFAULT=`

### Slab Nomerge

It is wise to disable slab merging. Slab merging allows for overwriting objects in merged caches. This is a wide vector to heap exploitation.

`slab_nomerge`

### Slub Debug

This setting checks for sanity checks and rezoning. From this, it will have several reviews to ensure no corruption or specific dangerous slab operations are passed. It also has red zones, which adds space to the slab. When a slab is written past its natural size and into the red zone, it can help detect overflows. 

`slub_debug=FZ`

### Zeroing Memory

This enabled zeroing of memory during allocation and free time. This essentially erases and overwritten memory after it is used. 

`init_on_alloc=1 init_on_free=1`

### Shuffle Page Allocations

By randomizing the page allocator, free lists make allocating data less predictable. 

It also improves performance.

`page_alloc.shuffle=1`


### Mitigate Known Issues

Issues with specific hardware can be fixed with Kernel Page Table Isolation. This helps prevent MELTDOWN, and some KASLR bypasses.

`pti=on`

### Disable Vsyscalls

vsyscalls have been discontinued and are determined to be obsolete. They have been replaced with vDSO. vsyscalls are also suspect of ROP attacks. 

`vsyscall=none`


### Protect FS Debugs

The kernel can dump sensitive information about your filesystem. 

`debugfs=off`

### Set OOPS Action

Some kernel exploits can cause an 'oops,' which can cause the kernel to panic, which prevents the exploit. This may be a determinate setting as bad drivers can cause oops to happen and push your system to crash.

`oops=panic`

### Only Allow Signed Kernel Modules

This will only allow kernel modules that have been signed by an authentic author, thus preventing malicious kernel modules from being loaded.

`module.sig_enforce=1`

### Kernel Lockdown LSM

This prevents various ways that userspace code can be exploited to gain kernel privilege and gather information. 

`lockdown=confidentiality`

This is the most strict setting. 


### Protect ECC Memory

Causes Kernel to panic under uncorrectable errors has been detected in ECC memory which could be exploited. 

`mce=0`

Take note that this only applies to systems that have ECC memory.

### Prevent Information Leaks During Boot 

`quiet loglevel=0`

## CPU Vulnerabilities

You will need to research this option for yourself, as this depends specifically on your CPU. 

These options can be applied to your CPU to help harden known problems.

```
spectre_v2=on 
spec_store_bypass_disable=on 
tsx=off 
tsx_async_abort=fullnosmt 
mds=full,nosmt 
l1tf=full,force 
nosmt=force 
kvm.nx_huge_pages=force
```

## Results

Most of these settings from [madaidians GitHub page](https://madaidans-insecurities.github.io/guides/linux-hardening.html#boot-kerne) states that this is what your configuration line should look like with your specific locations (as you hopefully have adapted to)

`slab_nomerge slub_debug=FZ init_on_alloc=1 init_on_free=1 page_alloc.shuffle=1 pti=on vsyscall=none debugfs=off oops=panic module.sig_enforce=1 lockdown=confidentiality mce=0 quiet loglevel=0`

---

###### Sources:

- ###### https://madaidans-insecurities.github.io/guides/linux-hardening.html#boot-kernel
- ###### https://www.tutorialspoint.com/what-is-grub-in-linux

---

###### [Home](https://mksipe.github.io/mksipe/) | [Privacy Policy](https://mksipe.github.io/mksipe/Privacy)