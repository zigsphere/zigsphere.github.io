---
layout: post
title: How to make Windows USB flash install media from Linux
tags: [Tech, How-to]
author: zigsphere
excerpt_separator: <!--more-->
---

I've already shared how to do this on my forum, but thought it would be beneficial to share it here as well, since many not view the forum.

After trying many ways of creating a bootable flash drive for installing Windows shown online, I successfully found a method that actually works.

###### **Requirements:**

* Gparted sudo apt-get install gparted
* USB flash drive

--------
###### **Instructions:**

1. Insert the USB flash drive, then open gparted.
2. In the upper right corner of gparted, click on the dropdown menu and select the USB drive. Note the filesystem. In this example, lets assume it is /dev/sdf
In the main window in gparted, delete all of the partitions and format the drive as NTFS.
3. After formatting, right-click the drive in the gparted main menu then select "Manage Flags". Select the "boot" checkbox, then close.
4. Open terminal and run the following command: 
`sudo dd if=/usr/lib/syslinux/mbr/mbr.bin of=/dev/sdf`
5. Assuming you are in the same directory as your Windows ISO image, run the following commands to mount the ISO and USB drive:
`sudo mkdir /mnt/iso; sudo mount -o loop windows.iso /mnt/iso` <br />
`sudo mkdir /mnt/usb; sudo mount /dev/sdf1 /mnt/usb`
6. Copy over the files:
`sudo cp -r /mnt/iso/* /mnt/usb/`
7. Run sync:
`sync`
8. You are done!