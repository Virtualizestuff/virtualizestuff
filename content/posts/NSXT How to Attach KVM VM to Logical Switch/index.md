---
title: "NSX-T – How to Attach KVM VM to Logical Switch"
description: "Guide on attaching KVM VMs to NSX-T logical switches, with step-by-step instructions and troubleshooting tips"
date: 2018-04-15
slug: nsx-t-how-to-attach-kvm-vm-to-logical-switch
feature: "feature.png"
heroStyle: "background"
tags:
  - kvm
  - nsx-t
  - nsx-t-logical-switch
  - nsx-t-logical-port
draft: false
---


## Introduction

Having recently deployed NSX-T in our environment, I can say the deployment and configuration were straightforward using the installation docs. When it came time to attach a VM hosted on the KVM host it was a bit unclear how to accomplish this. VMware’s documentation mentions the following command `virsh dumpxml | grep interfaceid`.

![Terminal output showing a running virtual machine "web-05" with network interface details. A red arrow points to the absence of an "Interfaceid" line in the XML configuration.](image-01.png)

As you can see from the screenshot above I have no interfaceid, what gives??? Admittedly, when it comes to KVM and Openvswitch I’m a bit of a novice. The purpose of this post is to provide additional details around VM configuration on a KVM host for those in a similar situation.

After reviewing VMware’s documentation I decided to jump into VMware’s NSX-T Hands-on-labs to see how the VMs on the KVM hosts were configured:

![A terminal window displaying a command executed by a user on a virtual machine. The output shows an XML snippet of a network interface with highlighted parameters. The interface type is "bridge," and other details include MAC address, source bridge, and virtual port parameters.](image-02.png)

### Reconfigure web-05 Virtual Machine

1. Excellent, now the first thing to do is to dump the XML configuration of *web-05* to file with the following command:  `virsh dumpxml web-05 > web-05.xml`. After editing the XML to match that of VMware’s hands-on-labs we can proceed with shutting down web-05.
    
    ![A screenshot of a text editor displaying an XML configuration file with various hardware component settings. The highlighted section shows a network interface configuration with type 'bridge', a specified MAC address, and details of the source bridge and virtual port type.](image-03.png)
    
2. Shutdown web-05 using: `virsh shutdown web-05`
    
    ![A screenshot of a terminal session showing the process of shutting down a virtual machine named "web-05". The initial state is "running", followed by the shutdown command "virsh shutdown web-05", and finally, the state changes to "shut off".](image-04.png)
    
3. Create a domain from the XML file with: virsh create web-05.xml
    
    ![Terminal output showing the command "virsh create web-05.xml" was executed, creating the domain "web-05" from the XML file. The "virsh list --all" command displays the domain "web-05" with ID 19 in a running state.](image-05.png)
    
4. Confirm interfaceid with: `virsh dumpxml web-05 | grep “interface type” –after-context=10` Awesome we now have an interfaceid 😉
    
    ![Terminal screenshot showing the XML configuration of a network interface. The interface is of type "bridge" with a MAC address, source bridge, virtual port, target device, model type, alias name, and PCI address details. The "interfaceid" is highlighted in yellow.](image-06.png)
    
5. Proceed with creating a new logical port with the interfaceid:
    
    ![Screenshot of the "Add New Logical Port" dialog in a network management interface. It shows fields for Name, Description, Logical Switch, Admin Status, Attachment Type, and Attachment ID. The Logical Switch is set to "Web," Admin Status is "Up," and there are options to save or cancel.](image-07.png)
    
    ![Screenshot of a VMware NSX interface showing a list of logical ports, their IDs, admin status, operational status, switching profiles, attachments, and logical switches. The last row is highlighted, indicating a logical port named "web-05" with an admin status of "Up" and attached to the "Web" logical switch.](image-08.png)
    
6. Below demonstrates successful communication from web-04 (192.110.25.23) to web-05 (192.110.25.24):
    
    ![Network diagram showing connections between Web and App sections through a Tier 1 router. It includes two virtual machines, "pod-001-ubuntu" and "81d04280-3685" (TN-UBUNTUKVM and TN-ESXi), connected with a green link.](image-09.png)
    
    ![Terminal output of a ping test to IP address 192.110.25.24, showing 3 packets transmitted and received with 0% packet loss. Average round-trip time is 1.221 milliseconds.](image-10.png)
    

### Transport Node & OVS-VSCTL Show

Make sure the KVM host is part of the Transport Node as we will be using the “*nsx-manged*” bridge. If the host is not part of the Transport Node, `ovs-vsctl show` you will not see NSX-T bridges:

![Screenshot of a terminal showing two Open vSwitch configurations. The first is "Not attached to Transport Node" with basic details, while the second is "Attached to Transport Node" with a more complex configuration, including details of bridges and ports.](image-11.png)

![Screenshot of terminal output showing details of network ports and connections, with annotations marking "Overlay Network" and "VM Port" in a VMware NSX environment.](image-12.png)

Thanks for viewing! If you found this post helpful share it with the community. 😄
