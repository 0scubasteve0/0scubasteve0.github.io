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








