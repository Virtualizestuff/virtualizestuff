---
title: "Homelab Rebuild"
description: "Rebuilding a homelab with vSphere, detailing equipment setup, custom ESXi image creation, and network configuration in a multi-part series"
date: 2016-11-03
slug: homelab-rebuild
feature: "feature.png"
heroStyle: "background"
tags:
  - homelab
  - homelab-rebuild
draft: false
---


Seeing how I have some free time on my hands I decided why not rebuild my homelab. My homelab hasn’t been powered-on in over a year now, as my previous job allowed me to work with *vSphere*, *Horizon Suite*, *vRealize Suite*, & *NSX*. So needless to say there was allot of dust build-up which was quickly remedied by a AirBoss Compressor and the trusty Swiffer an hour later everything was dust free.  😀

This will be a short series of me rebuilding my homelab with vSphere/vCenter:

1. Environment Overview
    
2. [Create a Custom ESXi Image](https://virtualizestuff.com/how-to-create-a-custom-esxi-image)
    
3. [Installing a Custom ESXi 6.5 Image](https://virtualizestuff.com/installing-a-custom-esxi-65-image)
    
4. [Configure a vSphere 6.5 ESXi Host (Part1)](https://virtualizestuff.com/how-to-configure-a-vsphere-65-host-part1)
    
5. [Configure a vSphere 6.5 ESXi Host (Part2)](https://virtualizestuff.com/how-to-configure-a-vsphere-65-host-part2)
    
6. Setting up Infrastructure Components (AD / DNS / DHCP)
    
7. Deploying a vCenter 6.5 Appliance
    

## Environment Overview

A couple years ago I put together a parts list of all equipment used in my lab. The environment hasn’t changed much besides adding more SSD/HDD for vSAN last year. I went with dedicated hardware as I am a tech nerd at heart, who loves any excuse to build something. The environment consists of 3 hosts and all 3 hosts will participate in a vSAN cluster running over the 20GB infiniband network. The Synology 1813+ is used for iSCSI and NFS datastores. As you can imagine my biggest constraint is memory (96GB), so next year I hope to add 3 additional hosts (128GB of memory each) and a 10GB network. Which may require upgrading electric panels to a higher amperage to support it. That’s gonna to wrap up this post stay tuned for the next post where I’ll go over how to Create a Custom ESXi Image.

The diagram below provides an overview of the environment:

![Diagram of a home lab and network setup, featuring VMware with virtual machines, connected to a Netgear layer 3 switch and Cisco switches via trunked VLANs. Storage includes WD Blue drives and Sandisk SSDs. The home network connects PCs, a QNAP NAS for VM backup, a Synology NAS, and other storage devices.](image-01.png)

![Diagram listing VLANs and corresponding IP ranges. It includes: VLAN 30 for Servers, VLAN 50 for Management, VLAN 55 for Workstations, VLAN 60 for iSCSI & NFS, VLAN 70 for vMotion, VLAN 71 for Fault Tolerance, and VLAN 100 for DMZ. All IPs are in the format 192.168.x.0/24, with 'x' representing the VLAN number.](image-02.png)
