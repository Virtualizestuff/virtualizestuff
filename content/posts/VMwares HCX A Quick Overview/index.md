---
title: "VMware’s HCX – A Quick Overview"
description: "VMware’s HCX enables seamless app mobility, cloud migration, infrastructure hybridity, optimized performance, and simplified disaster recovery"
date: 2018-11-05
slug: vmwares-hcx-a-quick-overview
feature: "feature.png"
heroStyle: "background"
tags:
  - vmware
  - workload-migration-to-cloud
  - vmware-hcx
draft: false
---


As more organizations leverage the capabilities of VMware Cloud on AWS, it’s essential to understand the connectivity options: VPN, Direct Connect, and Hybrid Cloud Extension (HCX). I recently had the privilege of deploying HCX in our Technical Solutions Center (TSC). Today’s discussion aims to provide a high-level overview of HCX and the associated components. Before doing so, it’s important to highlight some of the migration challenges experienced by organizations today:

* Dispersed versions of vSphere along with a mixture of legacy/new hardware across sites
    
* Difficult post-migration testing
    
* The potential for routing/firewall misconfiguration
    
* Legacy applications with hard-coded IP addresses and reliance on on-premises infrastructure components
    

## HCX Introduction

To address these challenges, HCX provides an abstraction layer allowing for vSphere on-premises and cloud resources to be presented to the application as a single resource regardless of vSphere version (vSphere 5.5 +). VMware refers to this as “infrastructure hybridity.” That allows application mobility across multiple clouds without the need to reconfigure virtual machines or infrastructure. HCX also packs a capable disaster recovery solution that’s easy to set up, manage and allows organizations to scale their DR capabilities. For organizations that currently leverage VMware Cloud providers like IBM, OVH you too can also utilize HCX, however, for the purposes of this post we’ll focus on VMC on AWS implementation of HCX.

## HCX Cloud vs HCX Enterprise

Before we jump into the components its best to clarify HCX Cloud vs. HCX Enterprise:

* HCX Cloud (**Target**) – HCX Management VM deployed into VMC on AWS SDDC.
    
* HCX Enterprise (**Source**) – HCX management VM deployed into our on-premises datacenter.
    

If you’re a VMC on AWS customer, then you already have access to HCX at no additional cost. To automatically provision the HCX Cloud VM into your SDDC instance simply press the “Deploy” button from the VMC console. Once deployed you can into the HCX cloud web console where you can download the HCX Enterprise OVA for use with the on-premises data center.

HCX Enterprise is responsible for the following:

* Integration with on-premises vCenter instance
    
* Site pairing with HCX Cloud
    
* Deployment of additional HCX service appliances:
    
    * HCX WAN Interconnect
        
    * HCX WAN Optimization
        
    * HCX Network Extension
        
* Restful API and the HCX API documentation ([https:///hybridity/docs](https:///hybridity/docs))
    

**Pro Tip #1:** Deployment of HCX services into the on-prem site automatic initiates deployment of their “peer” counterparts into the SDDC instance, as shown in step 4 of the above diagram. 😊

## Infrastructure Hybridity Components

The additional HCX service appliances mentioned above provide the “infrastructure hybridity” so let’s explore each of the components.

**HCX WAN Interconnect –** Handles the migration and cross-cloud vMotion capabilities over the internet or private lines to the target site. The WAN Interconnect also provides strong encryption, traffic engineering, and virtual machine mobility. **Pro Tip #2**: The WAN Interconnect appliance also shows up as a fictitious ESXi host in vCenter at both sites acting as a secure proxy for cross-cloud vMotions.

**HCX WAN Optimization –** Allows organizations to onboard to the cloud faster by leveraging existing internet connectivity for migrations, until their preferred connectivity option (Direct Connect/MPLS circuits) is available. Regardless of connectivity options, HCX WAN Optimization improves performance by utilizing techniques like de-duplication and compression.

**HCX Network Extension** – Extends L2 networks from on-premises to the cloud without the need to change the virtual machine’s IP or MAC addresses or on-premises infrastructure. **Pro Tip #3:** Extension of NSX universal wires are not currently supported but is on the roadmap.

A great feature on the horizon for VMC customers is proximity routing (HCX-PR) which allows for optimized routing that eliminates the need for hairpinning between sites. There are a couple of caveats:

1. HCX-PR requires dynamic routing between both sites.
    
2. HCX-PR isn’t supported yet for VMC customers but is on the roadmap
    

Those currently using VMware cloud providers like IBM, OVH you can take full advantage of HCX-PR. 😉

**Pro Tip #4:** The configuration/connectivity of the IPsec VPN is automatic between the source and target sites for their respective service (HCX WAN Interconnect and HCX Network Extension). For a visual reference step 5 in the above diagram.

That’s going to wrap up this post on HCX Overview hope it was helpful.
