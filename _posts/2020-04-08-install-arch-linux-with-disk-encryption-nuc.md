---
id: 18
title: Install Arch Linux with Disk Encryption
date: 2020-04-08T10:22:11+00:00
author: fernando
description: This post might act as a guide for learning and it is going to help you to install Arch Linux with full disk encryption.
layout: post
permalink: /2020/04/08/install-arch-linux-with-disk-encryption/
image: assets/images/install_arch_disk_encryption_featured.jpg
comments: false
featured: true
hidden: true
categories: [ linux, security, encryption ]
tags:
  - linux
  - security
  - encryption
  - lvm
  - luks
  - arch
---
I'm a Linux fan, and the main reason is because of its open source nature: I have been using it for years and I gotta say a lot has changed since the early days... If you remember re-compiling the kernel in order to install an application, you know what I'm talking about... Fortunately that does not happen anymore, so do not freak out, not yet :). 

This article will act as a guide, which is goint to help you to install (and understand) Arch Linux with full disk encryption. We are also going to walk together through some of the concepts involved in the linux world, expecifically in the Arch Linux World that will provide you with a better picture of this ecosystem.

Disclaimer: give yourself time since it is a long text and you will need also patience, but I promise you will learn and have fun. Let's get started!

## Why Arch Linux?
I used SuSe, Red Hat, Devian, Ubuntu and Arch, in that order.
But First why Arch Linux? Mention Oriol
Give the reasons
There are many distros out there.... but let me enumerate the reasons why I choose Arch and made me feel in love with him
Do it yourself person. Installing arch linux is like building your own hause. 
Rolling release

## Why disk encryption?

## Before Start
Preparing the terrain
In this case I will do it on an Intel NUC I will use as a server but this guide could be applied to any other device.
Make sure that you check device specifics on the well documented Arch Linux Wiki while following this guide.  

## First steps

Once you've loaded the Arch Linux LiveCD and booted from it you'll find
yourself at a prompt. Graphical installations is not a thing Arch Linux
does, so this is where we'll start.

The prompt should say `root@archiso ~ #` to indicate you're at a root
prompt. I'll be shortening that to `$` in this guide, because `#` trips up
the syntax highlighting.

First things first, lets ensure we can actually read the console:

```bash
$ setfont latarcyrheb-sun32
```

Now, configure the network. I connected over WiFi, which you can set up
by launching `wifi-menu`.

The hostname of my system is `monoceros`, aka unicorn. You'll see this
come back in a few places, especially when setting up the volumes in
LVM. Swap it out for your own :). Or don't use the hostname at all but
some generic like `myvol` or `archlinux`.

Once that's done, we can start building up to the installation.

## Disk partitioning

My layout is as follows:

  * /dev/nvme0n1p1: /boot
  * /dev/nvmen0n1p2: LUKS
    * /dev/mapper/cryptlvm: lvm
      * /dev/mapper/monoceros-swap: swap
      * /dev/mapper/monoceros-root: / 
      * /dev/mapper/monoceros-home: /home

This results in a fully encrypted system, aside from the boot partition. This
is good enough for my threat model, which is mostly to ensure my data doesn't
land on the street if someone steals my laptop. It'll give most law
enforcement agencies a run for their money too, though if you want to protect
from nation state actors you'll probably need to come up with something
more.

I did this with parted:

```bash
$ parted /dev/nvme0n1

    (parted) mklabel gpt  # wipes out existing partitioning
    (parted) mkpart ESP fat32 1MiB 513MiB  # create the UEFI boot partition
    (parted) set 1 boot on  # mark the first partition as bootable
    (parted) mkpart primary  # turn the remaining space in one big partition
      File system type: ext2  # don't worry about this, we'll format it after anyway
      Start: 514MiB
      End: 100%
```

If you now check the layout:

```bash
    (parted) print
      Model: Unknown (unknown)
      Disk /dev/nvme0n1: 512GB
      Sector size (logical/physical): 512B/512B
      Partition Table: gpt
      Disk Flags: 
      
      Number  Start   End    Size   File system  Name  Flags
       1      1049kB  538MB  537MB  fat32              boot, esp
       2      539MB   512GB  512GB  ext2

    (parted) exit
```

### Setting up disk encryption

This will encrypt the second partition, which we'll then hand off to LVM to
manage the rest of our partitions. Doing it this way means everything is
protected by a single password. This is good enough for me, especially since
I don't share this machine with anyone else.

```bash
$ cryptsetup luksFormat /dev/nvme0n1p2
    WARNING!
    ========
    This will overwrite data on /dev/nvme0n1p2 irrevocably.
    
    Are you sure? (Type uppercase yes): YES
    Enter passphrase: 
    Verify passphrase:
```

Now we need to open the encrypted disk so LVM can do its thing:

```bash
$ cryptsetup open /dev/nvme0n1p2 cryptlvm

Enter passphrase for /dev/nvme0n1p2: 
```

### LVM

Time to setup LVM.

```bash
$ pvcreate /dev/mapper/cryptlvm  # create the physical volume
Physical volume "/dev/mapper/cryptlvm" successfully created.

$ vgcreate monoceros /dev/mapper/cryptlvm  # create the volume group
 Volume group "monoceros" successfully created

$ lvcreate -L 60G monoceros -n root  # create a 60GB root partition
 Logical volume "root" created.

$ lvcreate -L 18G monoceros -n swap  # create a RAM+2GB swap, must be bigger than RAM for hibernate
 Logical volume "swap" created.

$ lvcreate -l 100%FREE monoceros -n home  # assign the rest to home
 Logical volume "home" created.

```

You can check the layout by running `lvs`:

```bash
$ lvs
  LV   VG    Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  home monoceros -wi-a----- 398.43g                                                    
  root monoceros -wi-a-----  60.00g                                                    
  swap monoceros -wi-a-----  18.00g                                                    
```

### Format all the partitions

Now we're going to format all the partitions we've created so we can actually
use them.

First the root partition:
```bash
$ mkfs.ext4 /dev/mapper/monoceros-root
mke2fs 1.43.6 (29-Aug-2017)
Creating filesystem with 15728640 4k blocks and 3932160 inodes
Filesystem UUID: 055d8ed0-19c3-4c20-bcfe-b296939f7b9b
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
	4096000, 7962624, 11239424

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (65536 blocks): done
Writing superblocks and filesystem accounting information: done
```

Now our home partition:

```bash
$ mkfs.ext4 /dev/mapper/monoceros-home
mke2fs 1.43.6 (29-Aug-2017)
Creating filesystem with 104446976 4k blocks and 26116096 inodes
Filesystem UUID: 71738ad7-a620-496b-98a1-2b1e27b6a5e7
Superblock backups stored on blocks: 
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208, 
	4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968, 
	102400000

Allocating group tables: done                            
Writing inode tables: done                            
Creating journal (262144 blocks): done
Writing superblocks and filesystem accounting information: done
```

Swap:

```bash
$ mkswap /dev/mapper/monoceros-swap
Setting up swapspace version 1, size = 18 GiB (19327348736 bytes)
no label, UUID=8d7a92ae-0c61-4105-aaf0-71aa61082124
```

And boot. This must be a FAT32 formatted partition b/c UEFI:

```bash
$ mkfs.fat -F32 /dev/nvme0n1p1
mkfs.fat 4.1 (2017-01-24)
```

## Installing the base system

It's time to install the base system, which we can then chroot into and
further customise our installation.

### Mount all the partitions

Before we can install the OS we need to mount all the partitions and then
chroot into the moutpoint of the root partition. 

```bash
$ mount /dev/mapper/monoceros-root /mnt
$ mount /dev/mapper/monoceros-home /mnt/home
$ mount /dev/nvme0n1p1 /mnt/boot
$ swapon /dev/mapper/monoceros-swap
```

### Setting up the mirrorlist

Last step is to edit `/etc/pacman.d/mirrorlist` and put the mirrors closest
to you at the top. This'll help speed up the installation. You can also
[generate a mirrorlist](https://www.archlinux.org/mirrorlist/). I'd highly
recommend doing so and ensure you unckeck the `http` checkbox so you only
use mirrors you can fetch from over `https`.

### Installing `base`

Now that everything is set up we need to bootstrap the OS:

```bash
$ pacstrap -i /mnt base base-devel
```

It'll now prompt you to confirm your package selection and then start with the
installation of the base system.

## Configuring the new installation

Now that the base system is there, we can chroot into it to customise our
installation and finish it.

```bash
$ genfstab -U /mnt >> /mnt/etc/fstab
$ arch-chroot /mnt
```

Your prompt will now change to: `[root@archiso /]#`. 

The first thing I do is install `vim` so I can edit and customise my the
configuration:

```bash
$ pacman -Sy vim
```

### Locale

In order to setup your locale edit `/etc/locale.gen` and uncomment the locales
you want.

After that execute the following:

```bash
$ locale-gen
$ echo LANG=en_US.UTF-8 > /etc/locale.conf
$ export LANG=en_US.UTF-8
```

### Timezone

Set your timezone by running:

```bash
$ tzselect
```

Once you've selected your timezone we need to update a few more things. First
override the `/etc/localtime` file and symlink it to your timezone:

```bash
$ ln -sf /usr/share/zoneinfo/<continent>/<location> /etc/localtime
```

Sync the clock settings and set the hardware clock to UTC:

```bash
$ hwclock --systohc --utc
```

### vconsole

Set the keyboard layout and font to be used by default for the virtual
console. Create `/etc/vconsole.conf`:

```bash
FONT=latarcyrheb-sun32
KEYMAP=colemak
```

Careful with the keymap, you probably don't want to set that to `colemak` on
your system. If you don't set it you'll get the default of `us`.

### Hostname

Time to give your system a name by adding that to `/etc/hostname`.

Also, add a line for that same hostname to `/etc/hosts`:

```
127.0.1.1 monoceros.localdomain monoceros
```

And tell systemd about it:

```bash
$ hostnamectl set-hostname monoceros
```

### Graphics card power saving options

Create `/etc/modprobe.d/i915.conf` with the following content:

```
options i915 enable_guc_loading=-1 enable_guc_submission=-1
```

### mkinitcpio

`mkinitcpio` is what is used to generate the `initramfs` you'll soon boot form.
However, due to the hardware in this laptop and our disk partitioning we have
to update it a bit. This configuration will use a full systemd based boot
stack.

* set `MODULES` to: `(nvme i915 intel_agp)`
* set `HOOKS` to: `(base systemd autodetect keyboard sd-vconsole modconf block sd-encrypt sd-lvm2 filesystems fsck)` 

Now regenerate the initramfs:

```bash
$ mkinitcpio -p linux

==> Building image from preset: /etc/mkinitcpio.d/linux.preset: 'default'
  -> -k /boot/vmlinuz-linux -c /etc/mkinitcpio.conf -g /boot/initramfs-linux.img
==> Starting build: 4.13.9-1-ARCH
  -> Running build hook: [base]
  -> Running build hook: [systemd]
  -> Running build hook: [autodetect]
  -> Running build hook: [keyboard]
  -> Running build hook: [sd-vconsole]
  -> Running build hook: [modconf]
  -> Running build hook: [block]
  -> Running build hook: [sd-encrypt]
  -> Running build hook: [sd-lvm2]
  -> Running build hook: [filesystems]
  -> Running build hook: [fsck]
==> Generating module dependencies
==> Creating gzip-compressed initcpio image: /boot/initramfs-linux.img
==> Image generation successful
==> Building image from preset: /etc/mkinitcpio.d/linux.preset: 'fallback'
  -> -k /boot/vmlinuz-linux -c /etc/mkinitcpio.conf -g /boot/initramfs-linux-fallback.img -S autodetect
==> Starting build: 4.13.9-1-ARCH
  -> Running build hook: [base]
  -> Running build hook: [systemd]
  -> Running build hook: [keyboard]
  -> Running build hook: [sd-vconsole]
  -> Running build hook: [modconf]
  -> Running build hook: [block]
==> WARNING: Possibly missing firmware for module: wd719x
==> WARNING: Possibly missing firmware for module: aic94xx
  -> Running build hook: [sd-encrypt]
  -> Running build hook: [sd-lvm2]
  -> Running build hook: [filesystems]
  -> Running build hook: [fsck]
==> Generating module dependencies
==> Creating gzip-compressed initcpio image: /boot/initramfs-linux-fallback.img
==> Image generation successful
```

Don't worry about those two warnings, the XPS 13 doesn't have any hardware on
board that needs those drivers.

### Microcode

Sometimes bugs are discovered in processors for which microcode updates are
released. These are loaded together with the initramfs when your system boots
but we need to install the package for it first:

```bash
$ pacman -Sy intel-ucode
```

### Setting up the bootloader

First, we need to tell `bootctl` to install the necessary things onto `/boot`:

```bash
$ bootctl install --path=/boot
```

In the future you won't need to call `install`, but `update` instead.

Now, edit `/boot/loader/loader.conf` and make it look like this:

```
timeout 10
default arch
editor 1
```

One note, by setting `editor 1` it's possible for anyone to edit the kernel
boot parameters, add `init=/bin/bash` and become root on your system. However,
since the disk is still encrypted at this point they can't do much with it and
I find it rather convenient to be able to edit those options when something does
go wrong.

We now need to create the boot entry named `arch`. To that end, create the file
`/boot/loader/entries/arch.conf` with the following content:

```
title Arch Linux
linux /vmlinuz-linux
initrd /intel-ucode.img
initrd /initramfs-linux.img
options rd.luks.uuid=$UUID rd.luks.name=$UUID=cryptlvm root=/dev/mapper/monoceros-root rw resume=/dev/mapper/monoceros-swap ro intel_iommu=igfx_off
```

Replace `$UUID` with the value from `cryptsetup luksUUID /dev/nvme0n1p2`.

I usually also create another entry that allows me to boot with resume support
disabled, in case that's broken. To that end, create a file like
`/boot/loader/entries/arch-noresume.conf` with the same content as above, but
simply omit the `resume=/dev/mapper/monceros-swap ro` option.

### sudo

I prefer using `sudo` over changing to root. In order to do so we need to
install the `sudo` package and update the configuration:

```bash
$ pacman -Sy sudo
```

Now that the package is installed update the configuration and uncomment the
line that reads `%wheel ALL=(ALL) ALL`:

```bash
$ EDITOR=vim visudo
```

### Creating a user account

Create a user account for yourself and ensure you're added to the `wheel`
group:

```bash
$ useradd -m -G wheel,users -s /bin/bash daenney
$ passwd daenney
    New password: 
    Retype new password: 
    passwd: password updated successfully
```

### Installing GNOME

```bash
$ pacman -Sy gnome gnome-extra dhclient iw wpa_supplicant dialog network-manager-applet networkmanager xf86-input-libinput
```

I explicitly install `dhclient` because `dhcpd` isn't very good at dealing with
non-spec compliant DHCP implementations. Especially if you have a D-Link router
or might encounter one, install this package. It also avoids some issues I've
had on large networks like at the office, Eduroam etc.

Now, enable GDM and Network Manager:

```bash
$ systemctl enable gdm
$ systemctl enable NetworkManager
```


### Optional: Installing GNOME

I like to use GNOME as my DE so it's time to install that:

```bash
$ pacman -Sy gnome gnome-extra iw dialog network-manager-applet
```

Now, enable GDM:

```bash
$ systemctl enable gdm
```

## Boot into the new installation

We're actually done now. What a ride. First, exit the chroot by issuing `exit`.
Now, unmount our filesystems: `umount -R /mnt`.

And finally, `reboot`.

## TODO NEXT:
 - Install oh-my-zsh
 - pacaur
 - git and dev tools 

## PERSONAL 
 - docker