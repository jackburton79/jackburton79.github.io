---
layout: post
title: Migrating a linux server using rsync
---


We are migrating from a XenServer 6.0 infrastructure to a Hyper-V 2012R2 based one. We have various VMs (Linux and Windows) which we already migrated.
Only a few remains, but they are the ones with the biggest disks. One of those is a proxy/mail server, based on a customized Gentoo distro.
We usually can afford to turn off our servers on weekends, so we chose to do so, and export the various VMs's VHDs to a NAS and import them to the new infrastructure.
For Linux VMs, migrating from XenServer to Hyper-V means also fixing the device names (xvd* to sd*) and the boot loader.
This is a really slow and painful process, since it involves a long downtime and a double file copy, and the NAS is not so fast (it's a entry level QNAP NAS, ArmV6 based).
Moreover, the mail server has two disks, a 16GB and a 300GB disk, 40% full, so it would take a very long time to export and import. Having a long downtime for the mail server isn't very nice, since incoming e-mails could be rejected.

So I decided to try a different method, and using rsync to create a clone of the mail server in the Hyper-V infrastructure.
I created a new VM with the Failover Cluster Manager in Hyper-V, called "Mail" with two disks with the same size of the ones in the original server.
I then started the VM with an OpenSuSe 12.3 Rescue DVD, and opened a Terminal.
I checked the partitions and the file systems on the old mail server, then created them on the new disks:
 
`mail ~ # fdisk -l`

Disk /dev/xvda: 16 GiB, 17179869184 bytes, 33554432 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xf76f33b8

Device     Boot     Start       End   Blocks  Id System
/dev/xvda1           2048   8390655  4194304  82 Linux swap / Solaris
/dev/xvda2 *      8390656  33554431 12581888  83 Linux


Disk /dev/xvdb: 300 GiB, 322122547200 bytes, 629145600 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


`mkfs.reiserfs /dev/sda2`




