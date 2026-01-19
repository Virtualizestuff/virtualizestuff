---
title: "WSFC and Long ESXi Boot Times"
description: "Learn how to fix long ESXi boot times caused by WSFC configurations with practical solutions and PowerCLI scripting"
date: 2019-12-16
slug: wsfc-and-long-esxi-boot-times
feature: "feature.png"
heroStyle: "background"
tags:
  - sql
  - vmware
  - powercli
  - esxi
  - rdms
  - sql-wsfc
  - powercli-script
draft: false
---


{{< youtube YVHicssmxao >}} 

Hey, what’s up everyone! Today’s video we are going to talk about Windows Server Failover Cluster (WSFC) and how to fix long ESXi boot times. So, if your experiencing this, stick around, and I’ll show you how to resolve this. Gist is located at the end for your convenience.

## WSFC Topology

Before we jump into the lab let's talk about the topology. As you can see there are a couple SQL nodes participating in the WSFC:

* Both nodes have a public and private network:
    
    * The public network allows the applications to access the database
        
    * While the private network is used for WSFC heartbeat traffic
        
* 12 physical RDMs ( LUNs 110 – 101 ) are also attached to the nodes
    
* Node01 is on ESXi-06, while Node02 is on ESXi-07
    

## LAB Time

Let’s pull up the console for Node01 to verify our WSFC is working. As you can see the SQL role is running and Node02 is the current owner. Let’s make Node01, the owner node by right clicking on SQL Server selecting Move and click on the Best Possible Node. We can go ahead and shutdown Node02 as we will be rebooting ESXi-07 to demonstrate the long boot times.

Once ESXi-07 has restarted I’ll start the stopwatch to see how long it takes to come up.  
See how it gets stuck on vmw\_vaaip\_cx ( 45 seconds )…let's view the vmkernel logs by pressing ALT + F12. The host is attempting to examine each and every LUN it sees one by one. Notice error “**D:0x18**”, under LUN 110, this means there is an ISCSI reservation conflict. The reason for this is Node01 has locked all physical RDMs attached to it preventing ESXi07 from examining those LUNs and will eventually timeout and move onto the next LUN. It’s this process that causes the long ESXi boot times.  
As you can see it took around 18 minutes to boot up, in my homelab. I’ve seen it take several hours to reboot a host, in large production environments.

[Now that ESXi07 is back up let's ju](https://gist.github.com/Virtualizestuff/a111597aa6ed9f3cc54032fddd9e29d0#file-setperenniallyreservedondatastores-ps1)mp into Visual Studio Code, in order to fix this! We need a way to tell ESXi to skip over LUNs associated with a WSFC. Well, there happens to be a property related to LUNs called “Perennially Reserved” and by default this is set to “false”. This script allows us to set the value to “true” on all LUNs attached to our WSFC, across both ESXi hosts. The first step is to identify physical RDMs that are shared between VMs by providing an output containing the ScsiCanonicalName, VM, ESXiHost, and Datastore where the RDM is located.

The next bit of information is whether perennially reserved is enabled on these LUNs. As you can see, they are set to false which is why our ESXi07 host takes forever to boot.

Let’s scroll down to Phase 2 and remove the # from the setconfig command. Note, the last value in parenthesis sets the perennially reserved to true. Now it’s time execute the highlighted code, notice as it loops through each ESXi host its setting the perennially reserved value to “true” for each LUN found in the WSFC.

Ok with that taken care of let’s reboot ESXi07 again and see how long it takes to boot up. This time taking about 1 1/2 minutes to boot up. As you can see it’s very important to enable perennially reserved on LUNs used in a WSFC.

### **Gist: PowerCLI Scripts**

#### **SetPerenniallyReservedonRDMs.ps1**

{{< gist Virtualizestuff b541fbd2a61d2986345df52086cd62d1 >}}
