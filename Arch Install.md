# Arch Linux Install Guide (UEFI)

In this guide we´ll see how to install Arch Linux

## Pre-installation

[Download](https://www.archlinux.org/download/) ISO

### Verify signature (optional)

[Download](https://www.archlinux.org/download/) .sig file ("PGP signature" under Checksums)

On a system with GnuPG installed, do this by downloading the PGP signature (under Checksums in the Download page) to the ISO directory, and verifying it with:

```zsh
gpg --keyserver-options auto-key-retrieve --verify archlinux-version-x86_64.iso.sig
```

Alternatively, from an existing Arch Linux installation run:

```zsh
pacman-key -v archlinux-version-x86_64.iso.sig
```

## Create a Installation medium

In my case it will be an USB Flash drive

## Boot live environment

Boot the computer from the Installatiom medium and select the option `Arch Linux install medium` and press `Enter`

## Set Keyboard layout

```zsh
loadkeys es
```

## Verify the boot mode

To verify the boot mode, list the efivars directory:

```zsh
ls /sys/firmware/efi/efivars
```

If the command shows the directory without error, then the system is booted in UEFI mode. If the directory does not exist, the system may be booted in BIOS (or CSM) mode.

## Connect to the internet

Make sure you have internet

```zsh
ping archlinux.org
```

## System Clock

Active network time synchronization:

```zsh
timedatectl set-ntp true
```

Check Status:

```zsh
timedatectl list-timezones
```

If the timezone is not correct, change it for one of the list:

```zsh
timedatectl list-timezones
```

Set timezone

```zsh
timedatectl set-timezone Europe/Madrid
```

## Format Drives

```zsh
cfdisk
```

Select `gpt` option

-   Partition1 (/dev/sda1): Create new partition of 550M
-   Partition2 (/dev/sda2):Create new partition of all space left

If your device has low ram (4 GB or less) is recommended to made a Swap partition

-   Partition1 (/dev/sda1): Create new partition of 550M
-   Partition2 (/dev/sda2):Create new partition of 2GB
-   Partition3 (/dev/sda3):Create new partition of all space left

Then asign the correspondent type for each partition:

-   Partition1: EFI System
-   Partition2: Linux Swap
-   Partition3: Linux Filesystem

Select `write`, type `yes` and press `Enter`. Then quit cfdisk

To check that everything is fine:

```zsh
fdisk -l
```

## Format the partitions

EFI:

```zsh
mkfs.fat -F32 /dev/sda1
```

SWAP:

```zsh
mkswap /dev/sda2
```

```zsh
swapon /dev/sda2
```

LINUX:

```zsh
mkfs.ext4 /dev/sda3
```

## Mount the file systems

```zsh
mount /dev/sda3 /mnt
```

## Install Arch Packages

For a fastest download select the fastest server:

```zsh
reflector --latest 200 --protocol http --protocol https --sort rate --save /etc/pacman.d/mirrorlist
```

Then install the arch base packages, linux hardware support and linux kernel  
(Note: You can substitute linux-lts for a kernel package of your choice)

```zsh
pacstrap /mnt base base-devel linux-lts linux-firmware
```

## Configure the system

### Fstab

Generate fstab file:

```zsh
genfstab -U /mnt >> /mnt/etc/fstab
```

### Chroot

Change root into the new system:

```zsh
arch-chroot /mnt
```

### Time zone

Set the time zone:

```zsh
ln -sf /usr/share/zoneinfo/Europe/Madrid /etc/localtime
```

Generate `/etc/adjtime`:

```zsh
hwclock --systohc
```

### Localization

Edit `/etc/locale.gen`

using

```zsh
pacman -S nano
```

and

```zsh
nano /etc/locale.gen
```

and uncomment `en_US.UTF-8 UTF-8` and other needed locales. Generate the locales by running:

```zsh
locale-gen
```

Create the locale.conf file, and set the LANG variable accordingly:

```zsh
echo "LANG=en_US.UTF-8" >> /etc/locale.conf
```

If you set the keyboard layout, make the changes persistent in vconsole.conf:

```zsh
echo "KEYMAP=es" >> /etc/vconsole.conf
```

### Network configuration

Create the hostname file:

```zsh
echo "myhostname" >> /etc/hostname
```

Add matching entries to hosts:

```zsh
nano /etc/hosts
```

```
127.0.0.1	localhost
::1		    localhost
127.0.1.1	myhostname.localdomain	myhostname
```

### Root password

Set the root password

```zsh
passwd
```

### Create your User

```zsh
useradd -m username
```

Set Password for the new user:

```zsh
passwd username
```

Install sudo

```zsh
pacman -S sudo
```

Add the user to sudo group

```zsh
usermod -aG wheel username
```

Enable wheel group in visudo:

```zsh
EDITOR=NANO visudo
```

Uncomment the `%wheel ALL=(ALL) ALL` line and save

## Setting up Boot

Install grub and EFI support

```zsh
pacman -S grub efibootmgr dosfstools os-prober mtools
```

Mount EFI:

```zsh
mkdir /boot/EFI
```

```zsh
mount /dev/sda1 /boot/EFI
```

```zsh
grub-install --target=x86_64-efi --bootloader-id=grub_uefi --recheck
```

```zsh
grub-mkconfig -o /boot/grub/grub.cfg
```

## Last installs

```zsh
pacman -S networkmanager man-db
```

```zsh
systemctl enable NetworkManager
```

## Reboot

```zsh
exit
```

```zsh
umount -R /mnt
```

```zsh
reboot now
```

or

```zsh
shutdown now
```
