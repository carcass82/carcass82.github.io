---
layout: post
title: "Random Access Failure"
tags: [rants, ram, linux, windows]
---

Everything started with random reboots.

I've been using my [laptop](https://www.notebookcheck.net/Acer-Aspire-5930G.10263.0.html) for more than ten years so i was quite expecting that the hardware would start failing at some point.

After the usual troubleshooting i ended up running _memtest_ to check ram health.

![memtest result](/img/memtest86.jpg)

Well, the ram looks damaged. If _that really is the problem_ - it could be the memory controller or a memtest false positive, the good thing is that only a specific address seems damaged. If only I could mark the damaged region as _don't use me_ the laptop could still be usable. Without reboots or risking to damage data while being written to disk. Here's what I found on the internet for linux and windows.

### linux

The Linux kernel has a specific option to mark a memory region as reserved, and the Linux Kernel parameters [documentation](https://www.kernel.org/doc/Documentation/admin-guide/kernel-parameters.txt) helps with a perfect example:

    memmap=nn[KMG]$ss[KMG]
        [KNL,ACPI] Mark specific memory as reserved.
        Region of memory to be reserved is from ss to ss+nn.
        Example: Exclude memory from 0x18690000-0x1869ffff
                 memmap=64K$0x18690000
                 or
                 memmap=0x10000$0x18690000

So I modified kernel parameters and added `memmap=64K\$0x39200000`.

After rebooting a `dmesg | grep 39200000` shows that the kernel is doing what I wanted:

    [    0.000000] user: [mem 0x0000000039200000-0x000000003920ffff] reserved


_Note that if you're using GRUB2 and your distribution has some flavour of `grub-mkconfig` which generates `grub.cfg` from a configuration file you have to add the option there as `64K\\\$0x3920a87c` (both the backslash and the dollar symbol must be escaped)._

### windows

Finding a way to exclude bad ram on windows was a bit more challenging. But it can be done nonetheless. The tool we need is `bcdedit`. The following commands must be issued in a _(administrator)_ command prompt:

    C:\WINDOWS\system32>bcdedit /set badmemoryaccess no

    C:\WINDOWS\system32>bcdedit /set {badmemory} badmemorylist 0x3920

To make sure the commands were executed correctly, we can use:

    C:\WINDOWS\system32>bcdedit /enum {badmemory}
    
    RAM Defects
    -----------
    identifier              {badmemory}
    badmemorylist           0x3920


The laptop stopped rebooting randomly, so the whole thing seems to be working.
