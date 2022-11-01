---
layout: post
title: "OpenBSD on old netbook, N months in"
---
So I had a couple of itches to scratch recently:

1) So how do `pledge(2)` and `unveil(2)` work on OpenBSD? How do you write code like that?
2) Does my old netbook work?

Well, the battery was dead for like 8 years, but it should still power on. And it did. I went ahead and ordered a fresh new battery after confirming OpenBSD runs well.

**TODO** pic of netbook

I saw that OpenBSD still supports i386, and while that laptop is technically 64 bit, at 1GB of RAM minus some VRAM, I've always preferred the 32 bit builds to help save memory on pointers.

Unfortunately, I had picked a time when [7.1 had issues getting wifi to work](https://ftp.openbsd.org/pub/OpenBSD/patches/7.1/common/001_wifi.patch.sig). And it didn't ship with exFAT drivers. And there was a [blob](http://firmware.openbsd.org/firmware/) I needed. And the installer doesn't ship the sources. And it was impractical to get wired ethernet.

But once the wifi issues were sorted, OpenBSD just worked. I got X set up, I got LXQT set up (which I later reverted a more basic OpenBox + fbpanel setup, because Qt apps seem to hog the CPU tremendously) and away I went.

And this is when I discovered that OpenBSD is not unlike Linux in the mid 2000s. It mostly works, but there is some assembly required, and there are paper cuts here and there. It keeps me busy.

Aside the usual `afterboot(8)` stuff, picking my desktop background, getting global hotkeys to do what I want, remapping CAPS_LOCK to CONTROL, starting `apmd` on boot and so on, there were also things to be patched.

Here's stuff that I've been busy with:

- learning to rebuild the kernel (though I'm back on `GENERIC.MP` as of 7.2)
- learning how to build ports and getting a version of `curl(1)` that works with `sftp://`
- gaining the ability to build ports without being root
- getting a battery indicator in the panel
  + debugging fbpanel to figure out why its battery indicator plugin isn't there
  + writing a C program to read the battery state
- debugging TCSH and figuring out why it insists on building without unicode, and finally getting a good static tcsh in `/bin/tcsh`
- swapping out the aging hard drive for a [fresh new SSD](../../../2022/10/23/switching-spinning-rust-to-ssd-openbsd.html) which the BIOS doesn't acknowledge and getting the laptop botting from a really small usb drive that has the bootloader and kernel on it
- debugging why `obsdfreqd` doesn't like being told to track temperature and [patching it](https://tildegit.org/solene/obsdfreqd/commit/ee170551b0a4f4bf2f18f14f0746eb3452280773)
- setting up `mutt` for the first time since uni, because OpenBSD folks prefer old school text mail
- discovering that you're supposed to preserve the contents of `argv` in C programs, because the [`kvm_getargv(3)` man page kinda hints at it?](https://man.openbsd.org/kvm_getprocs.3); this, after going through `ps(1)`'s source code to understand why it wasn't printing the same command line I had started the program with
- discovering that rc daemon scripts invoke `ps -xf` to match the exact command line, and how rc fails to work properly if `ps` fails
