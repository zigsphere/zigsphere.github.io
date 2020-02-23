---
layout: post
title: Freeing Up Space in the Linux Boot Partition
---

Is your /boot partition full or almost full and don't know how to clean it up? These instructions will explain how to free up that space. The /boot partition is a very small partition, which is usually ~240MB with a default Ubuntu install. The size may vary with other flavors of Linux. Ubuntu frequently releases updated kernel images, which are automatically installed. Old kernel images are left on the system just in case a newly installed image were to fail. Unfortunately, older images are kept indefinitely if they are not cleaned up. This leads to the /boot partition to slowly fill up.

----

##### What you will need:

* A computer with Linux (preferably debian-based) but should be similar for all flavors
* An Internet Connection

---

* Use the following command to remove packages that are not longer needed by the system

 $`sudo apt-get autoremove --purge`

* Type the following to print out the current version of your kernel

 $`uname -r`

* Use the following command to print out all the kernels you have installed that aren't your newest kernel

 $```dpkg -l linux-{image,headers}-"[0-9]*" | awk '/^ii/{ print $2}' | grep -v -e `uname -r | cut -f1,2 -d"-"` | grep -e '[0-9]'```

Make sure that the output doesn't contain your current kernel version. We will be deleting all of the versions that are displayed from the command above.

* Pipe the purge command to the last command to delete the old kernel versions

 $```dpkg -l linux-{image,headers}-"[0-9]*" | awk '/^ii/{ print $2}' | grep -v -e `uname -r | cut -f1,2 -d"-"` | grep -e '[0-9]' | xargs sudo apt-get -y purge```