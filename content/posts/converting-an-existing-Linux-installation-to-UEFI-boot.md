+++
title = "Converting an Existing Linux Installation to UEFI Boot"
date = 2019-10-09T13:54:54+02:00
draft = false
tags = ["Linux"]
categories = ["system administration", "software"]
+++

![Converting an Existing Linux Installation to UEFI Boot](/images/efiboot.png)

Due to a hardware failure I had to move my Linux installation to a new
Laptop that was configured for UEFI boot instead of the conventional BIOS
boot.

Transfering the partition data from the old to the new laptop is
straight forward. The problem is: How to add the existing installation to
the UEFI boot menu?

As it turns out, this task is way easier than expected.

<!--more-->


### Boot into UEFI mode

Boot the new laptop in **UEFI mode** using a current installation or rescue
media. I used the Mint19 installation media.

Check, if it's really in UEFI mode. This should now and list the
pre-existing boot loader (most likely: Windows)

	efibootmgr -v


### chroot into the new system

Mount the root-partition-to-be to /mnt and bind the dynamic system
directories:

	mount /dev/mapper/root /mnt
	cd /mnt
	cp /etc/resolv.conf etc		# only needed if using NetworkManager
	mount --bind /dev/ dev
	mount --bind /dev/pts/ dev/pts
	mount --bind /proc/ proc
	mount --bind /sys/ sys

Since I normally use the NetworkManager to dynamically generate the
resolv.conf I had to copy it by hand this time.

Now chroot into the installed system, leaving the live system behind:

	chroot /mnt


### Install grub on the EFI partition

Make the EFI partion accessable from the freshly transfered installation:

	# mount point
	mkdir /boot/efi
	# add entry for the EFI partition to /etc/fstab

	# mount all filesystems
	mount -a

Now everything should work as usual and it is time to install grub into the
EFI partition.

	apt install grub-efi
	grub-install

Done! The UEFI boot menu contains two entries now, with grub beeing the
first one in the boot order list:

	$ efibootmgr -v
	BootCurrent: 0003
	Timeout: 1 seconds
	BootOrder: 0003,0000
	Boot0000* Windows Boot Manager	HD(2,GPT,1774d14a-7d6c-49d3-a41c-94d911817e60,0x109000,0x31800)/File(\EFI\MICROSOFT\BOOT\BOOTMGFW.EFI)WINDOWS.........x...B.C.D.O.B.J.E.C.T.=.{.9.d.e.a.8.6.2.c.-.5.c.d.d.-.4.e.7.0.-.a.c.c.1.-.f.3.2.b.3.4.4.d.4.7.9.5.}....................
	Boot0003* ubuntu	HD(2,GPT,1774d14a-7d6c-49d3-a41c-94d911817e60,0x109000,0x31800)/File(\EFI\ubuntu\grubx64.efi)
