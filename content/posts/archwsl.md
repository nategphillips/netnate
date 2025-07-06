+++
title = 'archwsl'
date = 2025-07-06
draft = false
tags = ['computing']
summary = 'Installing Arch Linux on WSL.'
+++

This webpage was written using Arch on WSL set up using the steps written here! There is [official documentation](https://wiki.archlinux.org/title/Install_Arch_Linux_on_WSL) for installing Arch Linux on WSL, but frankly it's not the best in terms of clarity. Before the [official](https://gitlab.archlinux.org/archlinux/archlinux-wsl) Arch Linux WSL project was released in April 2025, [yuk7](https://github.com/yuk7) maintained a fantastic community version called [ArchWSL](https://github.com/yuk7/ArchWSL). The [docs](https://wsldl-pg.github.io/ArchW-docs/How-to-Setup/) for that project are helpful, but the order of things has changed a bit with the official packaged release.

## Enabling virtualization

First, virtualization must be turned on in the UEFI or BIOS of your Windows machine, and the Virtual Machine Platform feature must be enabled. See Microsoft's [docs](https://support.microsoft.com/en-us/windows/enable-virtualization-on-windows-c5578302-6e43-4b4b-a449-8ced115f58e1) for details.

## Installing WSL

In Windows 10 version 2004 and higher (Build 19041 and higher) or Windows 11, the Windows Subsystem for Linux Windows feature does not need to be enabled manually and WSL commands should work out of the box. First, ensure WSL is up-to-date:

```powershell
wsl --update
```

You can then view the list of available distributions by typing:

```powershell
wsl -l -o
```

Now, run

```powershell
wsl --install -d archlinux
```

to install Arch Linux instead of the default distribution, which is Ubuntu. More information can be found [here](https://learn.microsoft.com/en-us/windows/wsl/install).

## Setup

### Setting the root password

Once Arch has been downloaded and installed, set a root password before exiting using `passwd`.

### Initializing the pacman keyring

Since the pacman keyring requires elevated permissions to be created, we must use root to initialize it. (Otherwise, we would need sudo, which needs pacman to be installed, which needs sudo, etc.)

```bash
pacman-key --init
pacman-key --populate
pacman -Sy archlinux-keyring
pacman -Syu
```

### Installing sudo

While still logged in as root, install the sudo package:

```bash
pacman -S sudo
```

### Setting up the default user

Create the sudoers file:

```bash
mkdir /etc/sudoers.d
touch /etc/sudoers.d/wheel
```

Then, setup the wheel:

```bash
echo "%wheel ALL=(ALL) ALL" > /etc/sudoers.d/wheel
```

Add the new user:

```bash
useradd -m -G wheel -s /bin/bash {username}
```

Set a password for the new user:

```bash
passwd {username}
```

Exit the distro and set the default user when starting WSL:

```bash
exit
wsl --manage archlinux --set-default-user {username}
```

## Quality of life

### Setting the locale

To set the locale to US English (or whatever locale you like, see [the wiki page](https://wiki.archlinux.org/title/Locale) for more), run:

```bash
sudo localectl set-locale LANG=en_US.UTF-8
```

### Updating pacman mirrors

First, back up the current mirrorlist in case of bad overwrites:

```bash
sudo cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup
```

Install `reflector`:

```bash
sudo pacman -S reflector
```

Use reflector to save a list of the 20 most recently syncronized mirrors (located in the US and using the https protocol) into the pacman mirrorlist, sorted by download rate:

```bash
sudo reflector --verbose --latest 20 --country US --protocol https --sort rate --save /etc/pacman.d/mirrorlist
```

### Enabling extra pacman features

Turn on coloring and parallel downloads in pacman (parallel downloads should be enabled by default) by opening `/etc/pacman.conf` and uncommenting the two lines:

```bash
# Misc options
Color
...
ParallelDownloads = 5
```

## Personalizing the shell

As an example, my `.bashrc` file looks like the following:

```bash
#
# ~/.bashrc
#

# If not running interactively, don't do anything
[[ $- != *i* ]] && return

# Useful coloring
alias ls='ls --color=auto'
alias grep='grep --color=auto'

# Shortcuts
alias la='ls -la'
alias ll='ls -l'
alias cls='clear'
alias ff='fastfetch'
alias py='python'

# Safety nets
alias rm='rm -i'
alias mv='mv -i'
alias cp='cp -i'
alias ln='ln -i'
alias chown='chown --preserve-root'
alias chmod='chmod --preserve-root'
alias chgrp='chgrp --preserve-root'

PS1='[\u@\h \W]\$ '

eval "$(starship init bash)"
```

Apply the changes:

```bash
source .bashrc
```

I like the Starship Nerd Fonts preset, which can be found [here](https://starship.rs/presets/nerd-font).

## Setting up git

Follow the steps [here](https://git-scm.com/book/en/v2/Getting-Started-First-Time-Git-Setup). In short, run:

```bash
git config --global user.name "John Doe"
git config --global user.email johndoe@example.com
git config --global core.editor nvim
git config --global init.defaultBranch main
```

If you use GitHub and would like to keep your personal email private (i.e., not attach it to commits), go [here](https://github.com/settings/emails), set your email to private, and GitHub will give you an email specifically for authoring commits, merges, etc.

## Using WSL with VSCodium

If you'd like to use WSL with VSCodium, you must have `which` installed, and either `wget` or `curl` installed so that the VSCode server can run its bash setup script without encountering any errors. Once you've installed those, run `codium .` on any folder, then click the WSL indicator in the bottom left corner and select Connect to WSL. The server will install itself and that's it. If you have trouble, consult the [docs](https://code.visualstudio.com/docs/remote/wsl).
