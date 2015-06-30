---
layout: post
title: Migrating a linux server using rsync
---

We are migrating from a XenServer 6.0 infrastructure to a Hyper-V 2012R2 based one.
We have various VMs (Linux and Windows) which we already migrated.
Only a few remains, but they are the ones with the biggest disks. One of those is a proxy/mail server, based on a customized Gentoo distro.
We usually can afford to turn off our servers on weekends, so we chose to do so, and export the various VHDs to a NAS and import them to the new infrastructure.
For Linux VMs, migrating from XenServer to Hyper-V means also fixing the device names (xvd* to sd\*) and the boot loader, and generating a new initrd.
This is a really slow and painful process, since it involves a long downtime and a double file copy, and the NAS is not so fast (it's a entry level QNAP NAS, ArmV6 based).
Moreover, the mail server has two disks, a 16GB and a 300GB disk, ~40% full, so it would take a very long time to export and import. Having a long downtime for the mail server isn't very nice, since incoming e-mails could be rejected.

So I decided to try a different method, and use rsync to create a clone of the mail server in the Hyper-V infrastructure.
I created a new VM with the Failover Cluster Manager in Hyper-V, called "Mail" with two disks with the same size of the ones in the original server.
I then started the VM from an OpenSuSe 12.3 Rescue DVD, and opened a Terminal.
I checked the partitions and the file systems on the old mail server, then created them on the new disks:
{% highlight bash %}
mail ~ # fdisk -l
{% endhighlight %}
<pre>
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
</pre>

  
I used yast to create the new partitions on /dev/sda and format them:
- sda1 is the swap partition
- sda2 is the root

The "data" disk doesn't have any partition, it's formatted as a whole using reiserfs. I couldn't massage yast to do that, so I used the commandline:

{% highlight bash %}
linux ~ # mkfs.reiserfs /dev/sdb
{% endhighlight %}

Then I mounted the freshly created filesystems inside /mnt

{% highlight bash %}
linux ~ # mkdir /mnt/root
linux ~ # mkdir /mnt/sdb
linux ~ # mount /dev/sda2 /mnt/root
linux ~ # mount /dev/sdb /mnt/sdb
{% endhighlight %}

The network interface was already up and working, so all I needed to do was doing the actual copy, using rsync from the source VM to the destination one.

{% highlight bash %}
linux ~ # rsync -aAHxvz root@mail:/ /mnt/root --exclude=dev --exclude=proc --exclude=sys --exclude=tmp

linux ~ # rsync -aAHxvz root@mail:/nucleus/ /mnt/sdb
{% endhighlight %}

The twist
-------------
After these first passes I tested the server in an isolated network. Everything seemed to work fine, except Postfix, which threw an error on start.
After a while I realized that the postfix queue folders had the wrong permissions.
This happened because the OpenSuse LiveCD I used to start the new VM maps users and groups to different uids/gids than the ones used in Gentoo.
In this case I should've added the "--numeric-ids" option to the rsync command, which avoids this problem. That's not a big issue, though: I still had to do a second rsync with all the services stopped, so I took that chance to fix the uid and gids:

{% highlight bash %}
linux ~ # rsync --numeric-ids -aAHxvz root@mail:/ /mnt/root --exclude=dev --exclude=proc --exclude=sys --exclude=tmp
linux ~ # rsync --numeric-ids -aAHxvz root@mail:/nucleus/ /mnt/sdb
{% endhighlight %}
  
  
After that, I installed and configured Grub, and the server was ready to... serve!
