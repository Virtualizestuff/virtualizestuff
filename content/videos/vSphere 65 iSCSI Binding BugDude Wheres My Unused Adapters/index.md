---
title: "vSphere 6.5 iSCSI Binding Bug…Dude, Where’s My Unused Adapters?"
description: "Resolve the vSphere 6.5 iSCSI binding bug with simple ESXCLI commands. Learn how to fix your unused adapter issue efficiently"
date: 2017-01-02
slug: vsphere-65-iscsi-binding-bugdude-wheres-my-unused-adapters
feature: "feature.png"
heroStyle: "background"
tags:
  - bug
  - vmnic
  - iscsi-binding
  - esxcli
draft: false
---

## Video

{{< youtube eSuxUi_XD0k >}}

## Problem

Tonight I was going through an install of ESXi 6.5 (No vCenter yet) and I wanted to setup iSCSI binding to provide multiple paths to the iSCSI array. This requires setting the other vmnic into an “Unused” State. I need to set vmnic5 to unused, however I quickly realized there was no Unused Adapter option in the GUI as shown in *Figure-1*, only Mark Active or Standby.

![Image showing the "Edit port group - iSCSI-1" settings in a network configuration interface. The focus is on the "Failover order" section, where "vmnic1" is active and "vmnic5" is on standby, both with 1000 Mbps, full duplex connection.](image-01.png)

Figure-1

I then SSH into the host and ran the following command:  
`esxcli network vswitch standard portgroup policy failover get -p iSCSI-1`

Notice in *Figure-2,* there is an “Unused Adapter” so I thought maybe there is an ESXCLI command to set the vmnic adapter to “Unused” so I ran the following command:  
`esxcli network vswitch standard portgroup policy failover set –help`  
As you can see in *Figure-3* there is only “active-uplink” and “standby-uplink”.

![Terminal output showing ESXi network vSwitch settings for portgroup policy failover. Details include load balancing set to explicit, network failure detection to link, active adapter as vmnic1, standby adapter as vmnic5, and several override settings.](image-02.png)

Figure-2

![Text showing command options for configuring the failover policy of a port group using  in vSwitch. Options include settings for active uplinks, failback, failure detection, load balancing, notifying switches, portgroup name, standby uplinks, and using vSwitch settings.](image-03.png)

Figure-3

Like most I googled it and came across a bug report on VMware’s Fling site found [here](https://labs.vmware.com/flings/esxi-embedded-host-client/bugs/298) were other users experienced the same issue.

---

## Workaround

Continuing to troubleshooting the issue and stumbled across a workaround and figured why not share it. To workaround this issue we need to set the standby adapter (vmnic5) to Unused by issuing the following ESXCLI command:  
`esxcli network vswitch standard portgroup policy failover set -p iSCSI-1 -a vmnic5`. The results are shown in *Figure-4*. This moves vmnic5 to active while also moving vmnic1 to Unused. In my case I need to have vmnic5 set to Unused so I run the command again but this time using vmnic1, as shown in *Figure-5*.

![Command-line window showing configuration of a vSwitch standard portgroup policy in ESXi, displaying load balancing, network failure detection, active and unused adapters, and override settings.](image-04.png)

Figure-4

![Terminal screenshot showing command results for configuring network vswitch settings, with "vmnic1" as an active adapter. A turquoise arrow points to the text, "This is the result we are after!"](image-05.png)

Figure-5

There you have it a quick workaround around for this issue! If you found this helpful please share it.


