+++
title = 'archinstall'
date = 2024-05-31
draft = true 
tags = ['computing']
summary = 'Like archinstall, but not.'
+++

{{< notice note >}}
I wrote this guide to supplement the [official installation guide](https://wiki.archlinux.org/title/Installation_guide) while installing Arch.
You should probably just read the wiki.
{{< /notice >}}

I also added some steps in the pre-installation section that are aimed at Windows users who might be unfamiliar with the processes of torrenting, verifying digital signatures, and preparing ISO files.
For those who would like to test Arch out on a VM first, I've included a general setup guide for VirtualBox.
At the end, I've outlined a few post-installation steps for getting the system into a usable state with a working compositor, window manager, and a few other essentials.

Although this guide goes over all the installation steps in detail, I'd actually recommend using the [archinstall](https://wiki.archlinux.org/title/Archinstall) installation script in most cases.
If you decide to use archinstall, please read the wiki first, as there are a few gotchas that you must keep in mind.

## 1 Pre-installation

### 1.1 Acquire an installation image

A torrent client is recommended for downloading the ISO file---the [qBittorrent](https://www.qbittorrent.org/download) client is a good choice.
Download the latest stable version, but do not open the executable yet.
On the qBittorrent downloads page, open the `checksums and library versions` tab and copy the SHA-256 checksum for the 64-bit installer.
Open a PowerShell instance, navigate to the folder containing the executable, and verify the checksum by typing:

```ps1
(Get-FileHash ".\qbittorrent_version_x64_setup.exe").Hash -eq "expected-hash"
```

If the prompt returns `True`, go ahead and install qBittorrent.

Next, visit the Arch Linux [download](https://archlinux.org/download/) page and click the magnet link under the BitTorrent section to download the ISO.
This will trigger a prompt to open the selected file using qBittorrent, after which the file will begin the torrent process.
Once the download is complete, the client will begin "seeding" the file for other users.
This process can be terminated manually by removing the file from the transfer list.

### 1.2 Verify signature

Before using the Arch Linux ISO, the signature attached to the file must be verified using [GNU Privacy Guard](https://wiki.archlinux.org/title/GnuPG) (GPG).
In a PowerShell instance, install `gpg` via `winget`:

```ps1
winget install GnuPG.GnuPG
```

Visit the [checksums](https://archlinux.org/download/#checksums) section of the Arch Linux download page and download the ISO PGP signature file.
Navigate to the directory containing the ISO file and validate the signature using `gpg`:

```ps1
gpg --keyserver-options auto-key-retrieve --verify archlinux-version-x86_64.iso.sig archlinux-version-x86_64.iso
```

The expected output is:

```ps1
Good signature from "First Last <developername@archlinux.org>" [unknown]
```

The User ID might not be certified, and will generate the following output:

```ps1
WARNING: The key's User ID is not certified with a trusted signature!
```

To be extra cautious, visit the [developers](https://archlinux.org/people/developers/) page and match the primary key fingerprint output by `gpg` to the PGP key listed under the appropriate developer.

### 1.3 Prepare an installation medium

For a hardware installation, dowload [Rufus](https://rufus.ie/en/) to create a bootable USB flash drive.
After downloading, right-click the executable, go to `Properties`, and navigate to the `Digital Signatures` tab.
The signed name should state "Akeo Consulting."

Plug in a USB flash drive and launch Rufus.
Select the ISO downloaded earlier and change the partition scheme to `GPT`.
The target system should automatically switch to `UEFI (non CSM)`. Name the volume and leave the file system as `Large FAT32` with a cluster size of `32 kB`.
Click `START` to write the ISO to the removable drive.
If a pop-up is encountered, continue with writing the disk in ISO image mode.
Rufus can be closed when the status bar reads `READY`.
Eject the drive and physically remove it.

### 1.4 Boot the live environment

{{< notice note >}}
Arch Linux installation images do not support Secure Boot.
You will need to disable Secure Boot to boot the installation medium.
If desired, Secure Boot can be set up after completing the installation.
{{< /notice >}}

Boot to the BIOS settings on the target machine.
Find Secure Boot and disable it, as this feature will prevent booting from an external device if this step is not performed.
While in the BIOS, hyper-threading and CPU virtualization should also be enabled.
Save the changed settings and exit the BIOS.

Plug the removable drive into the target machine and power it on.
Enter the startup interrupt menu and proceed to the boot selection menu.
Select the removable drive as the boot device. After arriving at the GNU GRUB menu, select `Arch Linux install medium (x86_64, UEFI)`.
You're loaded in once the [Zsh](https://wiki.archlinux.org/title/Zsh) tty prompt appears:

```zsh
root@archiso ~ #
```

### 1.5 Set the console keyboard layout and font

For users based in the United States, these settings can remain unchanged as the default [console keymap](https://wiki.archlinux.org/title/Linux_console/Keyboard_configuration) is US.
The available layouts can be listed with:

```zsh
localectl list-keymaps
```

To set the keyboard layout, pass its name to [`loadkeys(1)`](https://man.archlinux.org/man/loadkeys.1).
For example, to set a German keyboard layout:

```zsh
loadkeys de-latin1
```

[Console fonts](https://wiki.archlinux.org/title/Console_fonts) are located in `/usr/share/kbd/consolefonts/` and can likewise be set with [`setfont(8)`](https://man.archlinux.org/man/setfont.8) omitting the path and file extension.
For example, to use one of the largest fonts suitable for [HiDPI](https://wiki.archlinux.org/title/HiDPI#Linux_console_(tty)) screens, run:

```zsh
setfont ter-132b
```

### 1.6 Verify the boot mode

Verify the boot mode by checking the UEFI bitness:

```zsh
cat /sys/firmware/efi/fw_platform_size
```

If the command returns `64`, then the system is booted in UEFI mode and has a 64-bit x64 UEFI.
If the command returns 32, then system is booted in UEFI mode and has a 32-bit IA32 UEFI; while this is supported, it will limit the boot loader choice to systemd-boot.
If the file does not exist, the system may be booted in [BIOS](https://en.wikipedia.org/wiki/BIOS) (or [CSM](https://en.wikipedia.org/wiki/Compatibility_Support_Module)) mode.
If the system did not boot in the mode you desired (UEFI vs BIOS), refer to your motherboard's manual.

### 1.7 Connect to the internet

Ensure that your [network interface](https://wiki.archlinux.org/title/Network_interface) is listed and enabled by typing:

```zsh
iplink
```

Some number of network devices should appear, the amount and type dependent upon the current machine.
Devices named `lo` and labeled `link/loopback` can be ignored.
Once the network has been set up using one of the options below, test that the network has been properly connected to by running:

```zsh
ping archlinux.org
```

{{< notice note >}}
In the installation image, [`systemd-networkd`](https://wiki.archlinux.org/title/Systemd-networkd), [`systemd-resolved`](https://wiki.archlinux.org/title/Systemd-resolved), [`iwd`](https://wiki.archlinux.org/title/Iwd) and [`ModemManager`](https://wiki.archlinux.org/title/ModemManager) are preconfigured and enabled by default.
That will not be the case for the installed system.
{{< /notice >}}

#### Ethernet

Ethernet connectivity works out of the box.
Simply plug in the ethernet cable.

#### Wireless

For wireless connectivity, enter the [`iwctl`](https://wiki.archlinux.org/title/Iwctl) utility:

```zsh
iwctl
```

Get a list of the wireless devices:

```zsh
device list
```

Here, something like `wlan0` should appear.
Ensure the device is powered on by reading the appropriate column.
If the device is powered off, type:

```zsh
device device_name set-property Powered on
```

Still in the `iwd` prompt, type adapter list and ensure that the adapter is also powered on.
If not, type:

```zsh
adapter adapter_name set-property Powered on
```

Begin the scan for networks within range by typing:

```zsh
station device_name scan
```

You can then list all available networks:

```zsh
station device_name get-networks
```

Connect to the appropriate network with the command

```zsh
station device_name connect SSID
```

and enter the password for the network.
Finally, exit the `iwd` prompt by typing `exit`.

### 1.8 Update the system clock

In the live environment, [`systemd-timesyncd`](https://wiki.archlinux.org/title/Systemd-timesyncd) is enabled by default and time will be synced automatically once a connection to the internet is established.
Use [`timedatectl(1)`](https://man.archlinux.org/man/timedatectl.1) to ensure the system clock is accurate:

```zsh
timedatectl
```

For now, the time zone should be UTC, the system clock should be syncronized, and network time protocol (NTP) should be active.
If NTP reads inactive, type:

```zsh
timedatectl set-ntp true
```

### 1.9 Partition the disks

Before getting into the details of disk partitioning, some choices must be made about the overall file structure of the system.

The official wiki suggests using a single root partition for simplicity.

#### Single root partition

| Mount point | Partition             | GPT partition type  | GUID                                 | Suggested size                               |
| ----------- | --------------------- | ------------------- | ------------------------------------ | -------------------------------------------- |
| `/mnt/boot` | `/dev/efi_partition`  | EFI system          | C12A7328-F81F-11D2-BA4B-00A0C93EC93B | 1 GiB                                        |
| `[SWAP]`    | `/dev/swap_partition` | Linux swap          | 0657FD6D-A4AB-43C4-84E5-0933C84B4F4F | At least 4 GiB                               |
| `/mnt`      | `/dev/root_partition` | Linux root (x86-64) | 4F68BCE3-E8CD-4DB1-96E7-FBCAF984B709 | Remainder of the device. At least 23–32 GiB. |

#### Discrete partitions

| Mount point | Partition             | GPT partition type  | GUID                                 | Suggested size          |
| ----------- | --------------------- | ------------------- | ------------------------------------ | ----------------------- |
| `/mnt/boot` | `/dev/efi_partition`  | EFI system          | C12A7328-F81F-11D2-BA4B-00A0C93EC93B | 1 GiB                   |
| `[SWAP]`    | `/dev/swap_partition` | Linux swap          | 0657FD6D-A4AB-43C4-84E5-0933C84B4F4F | At least 4 GiB          |
| `/mnt`      | `/dev/root_partition` | Linux root (x86-64) | 4F68BCE3-E8CD-4DB1-96E7-FBCAF984B709 | At least 23–32 GiB      |
| `/mnt/home` | `/dev/home_partition` | Linux home          | 933AC7E1-2EB4-4F13-B844-0E14E2AEF915 | Remainder of the device |

When recognized by the live system, disks are assigned to a [block device](https://wiki.archlinux.org/title/Block_device) such as `/dev/sda`, `/dev/nvme0n1` or `/dev/mmcblk0`.
To identify these devices, use [`lsblk`](https://wiki.archlinux.org/title/Lsblk) or [`fdisk`](https://wiki.archlinux.org/title/Fdisk):

```zsh
fdisk -l
```

Remove old partitions.

{{< notice tip >}}
Check that your NVMe drives and Advanced Format hard disk drives are using the [optimal logical sector size](https://wiki.archlinux.org/title/Advanced_Format) before partitioning.
{{< /notice >}}

The following [partitions](https://wiki.archlinux.org/title/Partition) are required for a chosen device:

- One partition for the [root directory](https://en.wikipedia.org/wiki/Root_directory) `/`.
- For booting in [UEFI](https://wiki.archlinux.org/title/UEFI) mode: an [EFI system partition](https://wiki.archlinux.org/title/EFI_system_partition).

If you want to create any stacked block devices for [LVM](https://wiki.archlinux.org/title/Install_Arch_Linux_on_LVM), [system encryption](https://wiki.archlinux.org/title/Dm-crypt) or [RAID](https://wiki.archlinux.org/title/RAID), do it now.

Use a [partitioning tool](https://wiki.archlinux.org/title/Partitioning#Partitioning_tools) like `fdisk` to modify partition tables.
For example:

Create a new GUID Partition Table (GPT).

```zsh
fdisk /dev/selected_disk
```

### 1.10 Format the partitions

[Format](https://wiki.archlinux.org/title/EFI_system_partition#Format_the_partition) the EFI system partition to FAT32 using [`mkfs.fat(8)`](https://man.archlinux.org/man/mkfs.fat.8).

{{< notice warning >}}
Only format the EFI system partition if you created it during the partitioning step.
If there already was an EFI system partition on disk beforehand, reformatting it can destroy the boot loaders of other installed operating systems.
{{< /notice >}}

```zsh
mkfs.fat -F 32 /dev/efi_partition
```

If you created a partition for [swap](https://wiki.archlinux.org/title/Swap), initialize it with [`mkswap(8)`](https://man.archlinux.org/man/mkswap.8):

```zsh
mkswap /dev/swap_partition
```

Create an [Ext4](https://wiki.archlinux.org/title/Ext4) file system on the root partition:

```zsh
mkfs.ext4 /dev/root_partition
```

Same with home:

```zsh
mkfs.ext4 /dev/home_partition
```

### 1.11 Mount the file systems

[Mount](https://wiki.archlinux.org/title/Mount) the boot partition:

{{< notice tip >}}
Run [`mount(8)`](https://man.archlinux.org/man/mount.8) with the `--mkdir` option to create the specified mount point.
Alternatively, create it using [`mkdir(1)`](https://man.archlinux.org/man/mkdir.1) beforehand.
{{< /notice >}}

```zsh
mount --mkdir /dev/efi_partition /mnt/boot
```

Enable the swap partition with [`swapon(8)`](https://man.archlinux.org/man/swapon.8):

```zsh
swapon /dev/swap_partition
```

Root:

```zsh
mount /dev/root_partition /mnt
```

Home:

```zsh
mount --mkdir /dev/home_partition /mnt/home
```

## 2 Installation

### 2.1 Select the mirrors

Before installing any packages with `pacman`, it is beneficial to select a list of [mirror servers](https://wiki.archlinux.org/title/Mirrors) optimized for geographic locality and download speed.
On the live system, after connecting to the internet, [reflector](https://wiki.archlinux.org/title/Reflector) updates the mirror list by choosing 20 most recently synchronized HTTPS mirrors and sorting them by download rate.
However, I'd like to show the process manually.

First, back up the current mirrorlist in case of bad overwrites:

```zsh
cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup
```

Use the built-in `reflector` script to save a list of the 20 most recently syncronized mirrors (located in the US and using the https protocol) into the `pacman` mirrorlist, sorted by download rate:

```zsh
reflector --verbose --latest 20 --country US --protocol https --sort rate --save /etc/pacman.d/mirrorlist
```

To enable parallel downloads, open the `pacman` configuration file for editing:

```zsh
vim /etc/pacman.conf
```

The following lines will be listed under the `[options]` section.
Uncomment them to enable colors (for increased clarity when downloading and installing packages) and parallel downloads:

```zsh
[options]
#Colors
...
#ParallelDownloads = 5
```

### 2.2 Install essential packages

{{< notice note >}}
No software or configuration (except for `/etc/pacman.d/mirrorlist`) gets carried over from the live environment to the installed system.
{{< /notice >}}

Use the [`pacstrap(8)`](https://man.archlinux.org/man/pacstrap.8) script to initialize an empty `pacman` keyring into the `/mnt` target, followed by installing the [`base`](https://archlinux.org/packages/?name=base) package, Linux [kernel](https://wiki.archlinux.org/title/Kernel), and firmware for common hardware:

```zsh
pacstrap -K /mnt base linux linux-firmware
```

{{< notice tip >}}

- You can substitute [`linux`](https://archlinux.org/packages/?name=linux) with a kernel package of your choice, or you could omit it entirely when installing in a [container](https://en.wikipedia.org/wiki/Container_(virtualization))
- You could omit the installation of the firmware package when installing in a virtual machine or container
{{< /notice >}}

The `base` package does not include all tools from the live installation, so installing more packages may be necessary for a fully functional base system.

## 3 Configure the system

### 3.1 Fstab

Generate an [`fstab`](https://wiki.archlinux.org/title/Fstab) file (use `-U` or `-L` to define by [UUID](https://wiki.archlinux.org/title/UUID) or labels, respectively):

```zsh
genfstab -U /mnt >> /mnt/etc/fstab
```

Check the resulting file:

```zsh
vim /mnt/etc/fstab
```

It should read something like:

| Mount point | Partition | Type |
| ----------- | --------- | ---- |
| `/boot/efi` | sda1      | vfat |
| `[SWAP]`    | sda2      | swap |
| `/`         | sda3      | ext4 |
| `/home`     | sda4      | ext4 |

### 3.2 Chroot

[Change root](https://wiki.archlinux.org/title/Change_root) into the new system:

```zsh
arch-chroot /mnt
```

### 3.3 Time

Set the [time zone](https://wiki.archlinux.org/title/Time_zone):

```bash
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
```

Run [`hwclock(8)`](https://man.archlinux.org/man/hwclock.8) to generate `/etc/adjtime`:

```bash
hwclock --systohc
```

This command assumes the hardware clock is set to UTC.
To prevent clock drift and ensure accurate time, set up [time synchronization](https://wiki.archlinux.org/title/Time_synchronization) using a [Network Time Protocol](https://en.wikipedia.org/wiki/Network_Time_Protocol) (NTP) client such as [systemd-timesyncd](https://wiki.archlinux.org/title/Systemd-timesyncd).

### 3.4 Localization

Edit `/etc/locale.gen` and uncomment `en_US.UTF-8 UTF-8` and other needed UTF-8 [locales](https://wiki.archlinux.org/title/Locale).
Generate the locales by running:

```bash
locale-gen
```

Create the [`locale.conf(5)`](https://man.archlinux.org/man/locale.conf.5) file, and [set the `LANG` variable](https://wiki.archlinux.org/title/Locale#Setting_the_system_locale) accordingly:

```bash
/etc/locale.conf
----------------
LANG=en_US.UTF-8
```

If you [set the console keyboard layout](https://wiki.archlinux.org/title/Installation_guide#Set_the_console_keyboard_layout_and_font), make the changes persistent in [`vconsole.conf(5)`](https://man.archlinux.org/man/vconsole.conf.5):

```bash
/etc/vconsole.conf
------------------
KEYMAP=de-latin1
```

### 3.5 Network configuration

Create the [hostname](https://wiki.archlinux.org/title/Hostname) file:

```bash
/etc/hostname
-------------
yourhostname
```

Complete the [network configuration](https://wiki.archlinux.org/title/Network_configuration) for the newly installed environment.
That may include installing suitable [network management](https://wiki.archlinux.org/title/Network_management) software, configuring it if necessary and enabling its `systemd` unit so that it starts at boot.

### 3.6 Initramfs

### 3.7 Root password

Set the root [password](https://wiki.archlinux.org/title/Password):

```bash
passwd
```

### 3.8 Boot loader

Install the GRUB [boot loader](https://wiki.archlinux.org/title/Boot_loader) and efi boot manager:

```bash
pacman -S grub efibootmgr
```

Execute the following command to install the GRUB EFI application `grubx64.efi` to `/boot/efi/EFI/GRUB/` and install its modules to `/boot/grub/x86_64-efi/`:

```bash
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB
```

Use the `grub-mkconfig` tool to generate `/boot/grub/grub.cfg`:

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

To acquire updated [microcode](https://wiki.archlinux.org/title/Microcode), depending on the processor, install one of the following packages:

```bash
pacman -S ...
```

- `amd-ucode` for AMD processors,
- `intel-ucode` for Intel processors.

GRUB will automatically detect the microcode update and configure itself appropriately.
After installing the microcode package, regenerate the GRUB configuration to activate loading the microcode update by running:

```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

## 4 Reboot

Exit the chroot environment by typing `exit`.

Manually unmount all the partitions with `umount -R /mnt`: this allows noticing any "busy" partitions, and finding the cause with [`fuser(1)`](https://man.archlinux.org/man/fuser.1).

Finally, restart the machine by typing `reboot`: any partitions still mounted will be automatically unmounted by systemd.
Remember to remove the installation medium and then login into the new system with the root account.
