---
layout: post
title: "Linux block devices"
description: "yay storage yay"
category: [Linux, AWS]
---
{% include JB/setup %}

One day at work I was presented with a problem. We use [Amazon Web Services' EC2](http://aws.amazon.com/ec2/) 
for a lot of things. An instance was storing lots of data in a specific directory, and it ran out of space. 

* The machine at hand: an Ubuntu 14.04 instance, the Trusty Tahr. I made a blank micro instance for demo purposes. 
* The solution: Elastic Block Store, or EBS.

### What is an EBS volume?

From the Amazon documentation, 

>An Amazon EBS volume is a durable, block-level storage device that you can attach to a single EC2 instance.

Basically it's a bit of storage that can have a filesystem and be treated by the operating system mostly like it would a physical drive you add or remove from your computer. However, it's connected the the server over the network, and can therefore be umounted and saved or shifted from one instance to the next. They're a really core AWS product - most EC2 instances these days are "EBS-backed", meaning an EBS volume taken from a standard snapshot is mounted at the root directory. You can turn off the instance and keep all its data intact if you want. 

The other kind of instance is "[instance store](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/RootDeviceStorage.html)", which doesn't persist and mostly seems to be around for legacy reasons.

### Mounting

What does all of this mounting mean? Presumably no steeds are involved. 

Well, a computer's files can be spread across a bunch of different storage devices. [Mount](http://en.wikipedia.org/wiki/Mount_%28Unix%29) is the Unix command that tells your operating system that some storage with a filesystem is available and what its "mount point" is, i.e. the part of the directory tree for which it should be used. If you were dealing with a physical computer sitting on your lap or desk, you might use `mount` to indicate a new SD drive or DVD. As you might expect, `umount` is its opposite, and disengages the storage. 

Some really simple sample usage:

    $mount  # Just tells you what's going on currently. 
    /dev/xvda1 on / type ext4 (rw,discard)
    proc on /proc type proc (rw,noexec,nosuid,nodev)
    sysfs on /sys type sysfs (rw,noexec,nosuid,nodev)
    none on /sys/fs/cgroup type tmpfs (rw)
    none on /sys/fs/fuse/connections type fusectl (rw)
    none on /sys/kernel/debug type debugfs (rw)
    none on /sys/kernel/security type securityfs (rw)
    udev on /dev type devtmpfs (rw,mode=0755)
    devpts on /dev/pts type devpts (rw,noexec,nosuid,gid=5,mode=0620)
    tmpfs on /run type tmpfs (rw,noexec,nosuid,size=10%,mode=0755)
    none on /run/lock type tmpfs (rw,noexec,nosuid,nodev,size=5242880)
    none on /run/shm type tmpfs (rw,nosuid,nodev)
    none on /run/user type tmpfs (rw,noexec,nosuid,nodev,size=104857600,mode=0755)
    none on /sys/fs/pstore type pstore (rw)
    systemd on /sys/fs/cgroup/systemd type cgroup (rw,noexec,nosuid,nodev,none,name=systemd)

    $mount /dev/xvdh ~/data # Would put the xvdf drive in my ~/data direcotry
    $umount /dev/xvdh  # Unmount! Same things as `umount ~/data` really

Meanwhile, need to know how much space and you have and where? `df` is [your gal](http://tldp.org/LDP/abs/html/devref1.html). This is what it looks like for my test instance

    /dev/xvda1       8125880 788204   6901864  11% /
    none                   4      0         4   0% /sys/fs/cgroup
    udev              290380     12    290368   1% /dev
    tmpfs              60272    188     60084   1% /run
    none                5120      0      5120   0% /run/lock
    none              301356      0    301356   0% /run/shm
    none              102400      0    102400   0% /run/user

### So... where's my EBS volume?

First of all, talking specifically about AWS, you have to do some configuration dances before you get an actual extra EBS volume anywhere near your instance. As you can see from the output above, there is already a device called `/dev/xvda1` hanging out at the root directory, which is the root device. 

When you're creating an instance, there's a handy button that says "Add new volume" and then you have to choose a name for your volume, use a specific snapshot, and various other specifications. The names are things like "/dev/sdb1". 

What does this name mean? The helpful popover says:

>Depending on the block device driver of the selected AMI's kernel, the device may be attached with a different name than what you specify. 

Wat?

### A little history of technology
EBS volumes are "block devices", which means they read data in and out in chunks, do some buffering, allow random access, and in general have nice high level features. Their counterpoint is the character device, which is less abstracted from its hardware implementation. They are sound cards, modems, etc. Their interfaces exist as "special" or "device" files which live in the [`/dev` directory](http://www.tldp.org/LDP/sag/html/dev-fs.html).

The options for block device names in the AWS console pretty much all start with `sd[a-z]{2}`. This stands for "SCSI disk", which is pronounced like "scuzzy disk". SCSI is a protocol for interacting with storage developed long ago. It was the newfangled way to connect storage to your computer compared to IDE cables, which are those ribbons of wire I at least remember my dad pulling out of the computer to detach a floppy drive in the '90s. 

More nomenclature: `hd` would indicate a hard disk, and `xvd` is Xen virtual disk. Everything in Amazon is virtualized, and the tricky thing is that only some kinds of instances understand that and adjust operations to account for it. Those that do, like Ubuntu 14.04, convert the `sd` name you chose to `xvd`. Anyway, you can suss out what exactly your instance's OS will think the device is name using [this reference](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/block-device-mapping-concepts.html). 

#### What does this all mean PRACTICALLY? Like how do I actually get to the volume?
Try `lsblk`.
    
    $ lsblk
    NAME  MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
    xvdh  202:112  0   8G  0 disk
    xvda1 202:1    0   8G  0 disk /

Looks like my EBS volume got changed from `sdh` to `xvdh`. Yay! It's virtualized. 

Anyway, the next step is to mount it, right? I have to prefix `/dev`, since that's the directory the special file is in.

    $ sudo mount /dev/xvdh /data
    mount: block device /dev/xvdh is write-protected, mounting read-only
    mount: you must specify the filesystem type

Ok so actually there's another step I haven't explained yet. 

#### Filesystems
I originally thought that filesystems were part of the operating system, since with OSX or Windows (which is what I've used historically), you'd have to go out of your way to use a non-standard one. With Linux, choice is abundant. [This article](http://www.howtogeek.com/howto/33552/htg-explains-which-linux-file-system-should-you-choose/) has a lot of good things to say clarifying the many Linux filsystem options. For devices that are not crazily large, the standard on Linux these days is ext4 - it's journaling (meaning it checks in both before and after writes which prevents corruption issues), it's backwards compatible (so you can mount its predecessor ext filesystems as ext4), and it's fast.

Important note: EBS volumes _do not come with a filesystem already set up_. That line about telling you to "specify" in the error above is a red herring. First, you have to make a filesystem with `mkfs`.

    $ sudo mkfs -t ext4 /dev/xvdh
    mke2fs 1.42.9 (4-Feb-2014)
    Filesystem label=
    OS type: Linux
    Block size=4096 (log=2)
    Fragment size=4096 (log=2)
    Stride=0 blocks, Stripe width=0 blocks
    524288 inodes, 2097152 blocks
    104857 blocks (5.00%) reserved for the super user
    First data block=0
    Maximum filesystem blocks=2147483648
    64 block groups
    32768 blocks per group, 32768 fragments per group
    8192 inodes per group
    Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632

    Allocating group tables: done
    Writing inode tables: done
    Creating journal (32768 blocks): done
    Writing superblocks and filesystem accounting information: done

YAYYYYYYY now we can actually `sudo mount /dev/xvdh /data` and dump all our cat pictures on there. 

                    /^--^\     /^--^\     /^--^\
                    \____/     \____/     \____/
                   /      \   /      \   /      \
                  |        | |        | |        |
                   \__  __/   \__  __/   \__  __/
    |^|^|^|^|^|^|^|^|^\ \^|^|^|^/ /^|^|^|^|^\ \^|^|^|^|^|^|^|^|
    | | | | | | | | | |\ \| | |/ /| | | | | | \ \ | | | | | | |
    ##################/ /######\ \###########/ /###############
    | | | | | | | | | \/| | | | \/| | | | | |\/ | | | | | | | |
    |_|_|_|_|_|_|_|_|_|_|_|_|_|_|_|_|_|_|_|_|_|_|_|_|_|_|_|_|_|


### Some more resources
http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-using-volumes.html

Special thanks to Andy Brody and Steve Woodrow for pointers and company.
