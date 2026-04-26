+++
title = 'Linux is not an operating system'
date = 2026-03-29
draft = true
tags = ['linux']
summary = "...but it doesn't matter."
+++

## The GNU/Linux meme

Explain the [GNU/Linux meme](https://gnulinux.meme/), the arguments for this view, and why it's also not accurate.
In fact, it is far *less* accurate than just using "Linux" for distributions such as [Alpine](https://www.alpinelinux.org/), which uses BusyBox and musl in place of GNU coreutils and glibc.
In order to see why, we must dive into the software (and firmware) stack that actually makes a computer run.

## What is Linux, then, if not an OS?

It's a kernel.

### Then what else is there?

Essential components of a Linux system:
- hardware: CPU, RAM, motherboard, GPU, storage
- firmware: BIOS, UEFI
- bootloader: GRUB
- kernel: Linux
- init system: systemd, openrc
- c standard library: glibc, musl
- core utilities: GNU coreutils, BusyBox
- shells: Bash, zsh
- display servers: X11, Wayland (actually a protocol that other software can implement via a compositor)
- window managers: i3wm, dwm, sway (Wayland wms are also compositors)
- desktop environments: KDE Plasma, GNOME
- distributions: Gentoo, Arch, Debian

## Kernel space versus userspace

### The grey area

Includes things most people would consider essential for interfacing with a computer, including
- shell
- coreutils

## Recent events that highlight the importance of an init system

Relevant links concerning the systemd debacle:
- [General article](https://itsfoss.com/news/systemd-fork-strips-out-age-verification/)
- [Slightly more strongly-worded article](https://www.sambent.com/the-engineer-who-tried-to-put-age-verification-into-linux-5/)
- [The commit itself](https://github.com/systemd/systemd/pull/40954)
- [Q&A with the author of the commit](https://itsfoss.com/dylan-taylor-systemd-controversy/)
- [Lennart Poettering's new company, Amutable](https://amutable.com/blog/introducing-amutable)

systemd has been hated for a long time, maybe for good reason:
- [Suckless](https://suckless.org/sucks/systemd/)
- [Arguments against systemd](https://without-systemd.org/wiki/index_php/Arguments_against_systemd/)
- [Yes, there is an entire website](https://nosystemd.org/)

### Process identifier 1

If you open `btop` or an equivalent system monitor and sort processes by PID, you will find `systemd` or an equivalent init system occupying the PID 1 slot.
What does this mean, and why is it important?

Linux spawns new processes using the `fork()` system call (syscall).
Therefore, PID 1, and by extension the init process, is the parent of every other process running on the system.
