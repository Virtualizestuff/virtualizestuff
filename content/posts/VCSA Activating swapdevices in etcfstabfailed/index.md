---
title: "VCSA Activating swap-devices in /etc/fstab…failed"
description: ""
date: 2015-10-29
slug: vcsa-activating-swap-devices-in-etcfstabfailed
feature: "feature.png"
heroStyle: "background"
tags:
  - vmware
  - vcsa
  - activating-swap-devices-in-etcfstab
draft: false
---


In preparation for the VCIX-NV exam I wanted to setup NSX in the homelab and migrate from vSphere’s Standard Switches (vSS) to a single Distributed Switch (vDS). The migration was going smoothly with the following VMkernel adapters being migrated to the vDS without issue: Management, vMotion, iSCSI-A, iSCSI-B. Then the fun began, I migrated the last physical adapter from the iSCSI/NFS vSS to the vDS and forgot to migrate the NFS VMkernel (DOH!). Leaving the NFS VMkernel stranded on the vSS with no physical adapter to communicate with thus cutting off access to VMs on that datastore. The connection issue was quickly resolved by logging into the ESXI Hosts and reconnecting the physical adapter back to the iSCSI/NFS vSS. Then I jumped into the console for the VCSA and noticed it was frozen so I rebooted the VM. When it came back up I was greeted with the following error and troubleshooting began:

![VCSA Error 00](image-01.png)

I provided the root password and was presented with this error:

![Fig-02](image-02.png)

When trying to launch BASH using the shell and shell.set –enabled True commands:

![Fig-03 - Prior to this issue both SSH and BASH were enabled](image-03.png)

Using VMware’s [KB2069041](http://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=2069041) article was able to gain access to the shell:

![Fig-04](image-04.png)

With shell access I ran the `mount – n -o remount,rw /` command which returned:

![Fig-05](image-05.png)

The error message is recommending to use the `e2fsck` command:

![Fig-06](image-06.png)

<mark>A WARNING!!! is presented asking if you want to continue:</mark>

![Fig-07](image-07.png)

After pressing ‘*y*’ the command began fixing errors:

![Fig-08](image-08.png)

![Fig-09](image-09.png)

After completing I pressed Control + D to exit maintenance mode and rebooted the VCSA upon reboot was presented with a slightly different error message:

![Fig-10](image-10.png)

I booted back into the shell using [KB2069041](http://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=2069041) and ran the `mount – n -o remount,rw /` again which thankfully completed successfully this time. I pressed Control+D to exit and rebooted the VCSA again this time to be greeted with a familiar screen:

![Fig-11](image-11.png)

Moral of the story:

![](image-12.png)

Don’t forget to migrate those VMKernels!

I hope this post help those who might run into a similar issue where the VCSA file system is mounted in a read-only state.
