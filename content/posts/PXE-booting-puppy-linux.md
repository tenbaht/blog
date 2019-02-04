+++
title = "PXE Booting Puppy Linux"
date = 2018-10-19T19:18:05+02:00
draft = false
tags = ["PXE"]
categories = ["network"]
+++

PXE booting Puppy Linux is actually quite easy if you already have a PXE
server up and running. The idea is simple: Add the `*.sfs` file containing
the file system into the initrd and use this for booting.

This post assumes your tftpserver is already set up
and serves files from `/opt/tftpboot`.

The description at [PXE-Netbooting using Puppy
Linux]https://docs.google.com/document/d/1bkMJ-2wjAAC8HVZZTZBFxBELbojoGctLMg27KazqvRw/edit?hl=en&pli=1#
tells you in detail what to do to get it working using a Puppy Linux Server
Host. But it does not list the (few and simple) steps needed to get it
running on an existing PXE server.

Mount the iso image (do all these steps as root):

	mount puppy.iso /mnt

Copy the kernel into your tftp directory:

	mkdir /opt/tftpboot/puppy
	cp -av /mnt/vmlinuz /opt/tftpboot/puppy

Unpack the initrd (as root):

	mkdir /tmp/x
	cd /tmp/x
	zcat /mnt/initrd.gz | cpio -i

Add the `*.sfs` file from the iso and repack the initrd:

	cp -av /mnt/*.sfs .
	find . | cpio -o -H newc | gzip -9 > /opt/tftpboot/puppy/initrd.gz

Add this to your /opt/tftpboot/pxeboot.cfg/default:

	# Puppy Linux
	LABEL puppy
	MENU LABEL PuppyLinux
	KERNEL /puppy/vmlinuz
	APPEND initrd=/puppy/initrd.gz

Done!


## Links

- [netboot script PET](http://www.murga-linux.com/puppy/viewtopic.php?mode=attach&id=37112)
  that contains the original mknetboot.sh script.
- [PXE-Netbooting using Puppy Linux]https://docs.google.com/document/d/1bkMJ-2wjAAC8HVZZTZBFxBELbojoGctLMg27KazqvRw/edit?hl=en&pli=1#
