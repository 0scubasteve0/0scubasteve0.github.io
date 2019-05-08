---
layout: single
title:  "Resize a Centos 7 Virtual Machine in KVM/QEMU"
date:   2019-05-07 22:45:00
categories: [Linux Administration]
tags: linux kvm zfs virtualization vm ubuntu
comments: true
---

Being relatively new to the RHEL/Centos world, it's safe to say I'm learning a lot as I go. While Linux is mostly, well, *Linux* between distributions, each one has its own particular nuances.  

One of these nuances bit me last weekend on a new Centos 7 VM. I was spinnig up a borg backup server to back up my roughly 50GB Nextcloud instance, so I provisioned a 160GB qcow2 image to give it adequate wiggle room. After logging in for the first time post-install, I was dismayed to see only 100GB available for backup. It turns out that Centos 7 default paritioning included separate `/` and `/home` partitions, and allocated a whole 50GB for root. What good is 50GB going to do me when all I'm instaling is borg?  

One additional differentiator of RHEL/Centos from Ubuntu is the use of XFS as the default filesystem. XFS is a rock-solid filesystem, but one thing it doesn't do is shrink. This ruled out shrinking `/` and expanding `/home`, so I decided to eat my mistake and just expand the whole VM. Let's get started.  

## Extend the QCOW2 Image

Shut down your VM
``` bash
$ sudo poweroff
```

SSH into your host machine and run the `qemu-img` tool on your VM guest image. I added an additional 100GB in the example below. 
``` bash
$ sudo qemu-img resize /path/to/image.qcow2 +100GB
Image resized.
```

## Extend the Partition

Since resizing root LVM paritions can be difficult from a running system, we're going to boot into a live environment for the remaining steps. You can use any Linux live environment for this, but I'll be using GParted (download ISO here) to keep things simple.  

Open up virt-manager and load the GParted ISO into the VM's optical drive, then click Apply.

![resize01](/assets/images/screenshots/resize01.png){:class="img-responsive"}
\\

Select Boot Options and enable the optical drive before moving it to first in the boot order, then click Apply.

![resize02](/assets/images/screenshots/resize02.png){:class="img-responsive"}
\\

Start your VM and follow the prompts to boot into the GParted live environment.

![resize02](/assets/images/screenshots/resize02.png){:class="img-responsive"}
\\

Open GParted, select the LVM parition and click on Resize/Move.

![resize02](/assets/images/screenshots/resize02.png){:class="img-responsive"}
\\

Drag the slider to fill in the remaining space and click OK.

![resize02](/assets/images/screenshots/resize02.png){:class="img-responsive"}
\\

Click Apply.

![resize02](/assets/images/screenshots/resize02.png){:class="img-responsive"}
\\

## Extend the Logical Volume

Close/minimize GParted and open the Terminal in the live environment.

![resize02](/assets/images/screenshots/resize02.png){:class="img-responsive"}
\\

We have to help the live environment disover the logical volumes.

``` bash
$ sudo pvscan
PV /dev/vda2   VG centos          lvm2 [<259.00 GiB / 100.00 GiB free]
Total: 1 [<259.00 GiB] / in use: 1 [<259.00 GiB] / in no VG: 0 [0   ]
```

``` bash
$ sudo vgscan
Reading volume groups from cache.
Found volume group "centos" using metadata type lvm2
```

``` bash
$ sudo vgdisplay
  --- Volume group ---
  VG Name               centos
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  5
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                3
  Open LV               0
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               <259.00 GiB
  PE Size               4.00 MiB
  Total PE              66303
  Alloc PE / Size       40702 / 158.99 GiB
  Free  PE / Size       25601 / 100.00 GiB
  VG UUID               r8vEjh-ZHBf-WWfi-lBSb-xfSd-HiTy-02TPMe
```
\\

You can see from the above information that our Volume group is resizable and has 100GB available for expansion. Take note of the number after "Free PE / Size".  
\\
Let's confirm our available volumes.

``` bash
$ sudo lvs
  LV   VG     Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  home centos -wi-a----- 106.99g                                                    
  root centos -wi-a-----  50.00g                                                    
  swap centos -wi-a-----   2.00g 
```

Now we'll use the `lvextend` tool tool to expand our '/home' volume. The number noted earlier from the 'vgdisplay' command will be used to signify the size increase.

```bash
$ sudo lvextend -l +25601 /dev/centos/home
  Size of logical volume centos/home changed from 106.99 GiB (27390 extents) to <207.00 GiB (52991 extents).
  Logical volume centos/home successfully resized.
```

## Expand the Filesystem





