---
layout: post
title: How-To &#58; Encrypted Btrfs install on Arch Linux
category: guide
tags: how-to, btrfs, luks, archlinux
---
Setting up a Btrfs install while juggling encyption, UEFI, and swap can be tricky so I've decided to put down my method here. Btrfs is very flexible and can do a lot of things from replacing the MBR/GPT schemes to RAID and then some. Unfortunately it does have some limitations. It doesn't have built-in encyption, doesn't support swap files and -not its fault- can't use UEFI to boot. Encryption can be solved using dm-crypt and the other issues by having their own partitions.

## Partitioning
To begin with decide how exactly the install will be set up. If UEFI boot isn't needed then there is no need for a separate EFI System Partition.

**Note:** Skip this and the *Formatting* step if neither UEFI or swap is required.

For this example, create 3 partitions:

1. A small EFI System Partition for UEFI boot. *~ 512M*  
2. Swap partition for hibernation. *~ Size of RAM*  
3. A final partition that will be the encrypted Btrfs Arch Linux install. *Rest of the drive.*  

When done it should look something like this:

    sda
     ├─ sda1     EFI     512 MB
     ├─ sda2     swap    2 GB
     └─ sda3             114 GB

Set the EFI partition as bootable using whatever method you used to create the partitions.

## Formatting
Format the EFI and swap partitions correctly.

    mkfs.fat -F32 /dev/sda1
    mkswap /dev/sda2

And activate swap:

    swapon /dev/sda2

## Encyption
The first thing that needs to be done is to create a LUKS encrypted container in the final partition using dm-crypt.

    cryptsetup -vy luksFormat /dev/sda3

The -y option is required to verify the password. Make sure it's a secure password and don't forget it. It will be required at the start of every boot..

Next unlock the encrypted container and enter the password:

    cryptsetup open /dev/sda3 btrfsroot

Replace `btrfsroot` with any preferred name. This will map the LUKS container to `/dev/mapper/btrfsroot`.

Create the btrfs filesystem:

    mkfs.btrfs -L "arch" /dev/mapper/btrfsroot

The `arch` label will be used to mount the various Btrfs subvolumes in fstab. The layout should now look something like this:

    sda
     ├─ sda1             EFI             512M
     ├─ sda2             swap            2G
     └─ sda3             crypto_LUKS     114G
          └─ btrfsroot   Btrfs           114G

**Note:** Encrypting the swap partition is outside the scope of this article but is possible if needed. See [dmcrypt/Swap encryption](https://wiki.archlinux.org/index.php/Dm-crypt/Swap_encryption).

## Setting up Btrfs and installing Arch Linux
Properly create and mount the subvolumes for the actual Arch Linux install.

First mount the container:

    mount /dev/mapper/btrfsroot /mnt
    cd /mnt

Now create the subvolumes. In Btrfs these essentially replace (or complement) the traditional partitions scheme. The ultimate scheme is preferencial. This is just one example. There are an infinite number of ways to create and mount subvolumes.

    btrfs subvolume create __arch
    btrfs subvolume create __arch/root
    btrfs subvolume create __arch/home
    btrfs subvolume create __snapshots

There is one subvolume for the root(/) directory and one for the home(/home) directory. The `__snapshots` subvolume is where the snapshots of the subvolumes will be stored. Snapshots can be created of the `root` or `home` subvolumes (or both by snapshotting `__arch`) prior to major upgrades as a form of backup. They can also be automated via simple scripts.

View the subvolumes:

    btrfs subvolume list .

Unmount the LUKS container:

    cd
    umount -R /mnt

Decide where to mount the subvolumes and create the appropriate directories.

    mount -o subvol=__arch/root /dev/mapper/btrfsroot /mnt
    mkdir /mnt/{home,.snapshots}
    mount -o subvol=__arch/home /dev/mapper/btrfsroot /mnt/home
    mount -o subvol=__snapshots /dev/mapper/btrfsroot /mnt/.snapshots

This mounts the corresponding subvolumes of the btrfsroot Btrfs filesystem to their appropriate locations.

Mount the EFI System Partition:

    mkdir /mnt/boot
    mount /dev/sda1 /mnt/boot

Select the fastest mirrors and install Arch Linux

    pacstrap -i /mnt base base-devel btrfs-progs

The btrfs-progs package is required in order to manipulate the btrfs install.

## Post Installation
Generate the fstab file:

    genfstab -U /mnt > /mnt/etc/fstab

genftab doesn't handle btrfs subvolumes very gracefully so edit the file. The subvolume mounting should look something like this:

    LABEL=arch  /           btrfs   rw,relatime,space_cache,subvol=__arch/root  0   0
    LABEL=arch  /home       btrfs   rw,relatime,space_cache,subvol=__arch/home  0   0
    LABEL=arch  /.snapshots btrfs   rw,relatime,space_cache,subvol=__snapshots  0   0

Remove any reference to subvolid in the fstab file. It will conflict with snapshot recovery because snapshots have a seperate subvolume ID. This makes it tricky to easily recover subvolumes using older snapshots because the kernel will still be looking for the subvolume ID of the original subvolume. Subvolid can still be used but requires additional steps to be taken during the snapshotting and recovery process that can be avoided by relying specificallly on the subvolume name.

Chroot into the install:

    arch-chroot /mnt /bin/bash

Edit the /etc/mkinitcpio.conf file to include the encrypt HOOK before the filesystems hook:

    HOOKS="... encrypt ... filesystems ..."

Regenerate the initramfs image with the `encrypt` hook:

    mkinitcpio -p linux

Install the systemd-boot bootloader:

    bootctl install

This will by default install the bootloader in /boot.

Create a file in `/boot/loader/entires` called `arch.conf`. Determine the unique UUID of the encrypted partition (/dev/sda3) and of the btrfs LUKS container (/dev/mapper/btrfsroot)..

    title   Arch Linux
    linux   /vmlinuz-linux
    initrd  /initramfs-linux.img
    options cryptdevice=UUID=</dev/sda3 UUID>:btrfsroot root=UUID=</dev/mapper/btrfsroot UUID> rootflags=subvol=__arch/root rw quiet

Do not include the chevrons (<, >) in the file. Configure the rest of the system and reboot. Enter the password on restart that was chosen when creating the encrypted LUKS container.

## Snapshots
Snapshots are very straightforward in Btrfs. They act as another subvolume linked to the original. They take up almost no space unless changes are made on the original subvolume in which case they retain the files and hierarchy that existed at the time of the creation of the snapshot.

To create a snapshot the subvolume must be mounted.

    btrfs subvolume snapshot /home /.snapshots/home-$(date "+%F")

This will create a complete copy of the `__arch/home` subvolume in `__snapshots`.

Because snapshots act as subvolumes they can be viewed, deleted and mounted them the same way.

    btrfs subvolume list /
    btrfs subvolume delete /.snapshots/home-2016-01-25
    mount -o subvol=__snapshots/home-2016-01-25 /dev/mapper/btrfsroot /mnt/home-snapshots

Snapshots and Subvolumes behave in many ways similar to directories and so they can be renamed using the `mv` command.

    mv /.snapshots/home-2016-01-25 /.snapshots/home/2016-01-25

In order to rollback to a snapshot, first mount the toplevel Btrfs subvolume and simply replace the desired subvolume with the preferred snapshot.

    mkdir /mnt/btrfs-toplevel
    mount /dev/mapper/btrfsroot /mnt/btrfs-toplevel
    cd /mnt/btrfs-toplevel
    mv __arch/home __arch/home-old
    mv __snapshots/home-2016-01-25 __arch/home

Then reboot and the kernel will mount the new `__arch/home` subvolume rather than the `__arch/home-old` because fstab determines the appropriate subvolume by name not ID in which case it would attemp to mount `__arch/home-old` again.
