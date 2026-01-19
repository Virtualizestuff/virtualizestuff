---
title: "How to Edit Security Policies & Apply them to Security Groups with PowerNSX"
description: "Learn how to edit firewall rules and apply security policies to groups with PowerNSX. Quick guide with parameters and useful cmdlets"
date: 2017-08-11
slug: how-to-edit-security-policies-apply-them-to-security-groups-with-powernsx
feature: "feature.png"
heroStyle: "background"
tags:
  - powershell
  - nsx
  - powernsx
  - edit-nsxsecuritypolicyfwrule
  - add-nsxapplysptosg
draft: false
---

## Video Guide

{{< youtube KaeoZm_VRhk >}} 

The video above demonstrates the cmdlets discussed in this post.

***Disclaimer:*** *The code shown in the video is not included in the PowerNSX module. There is still work to be done as I need to write Pester tests for these cmdlets to ensure everything works as expected and doesn’t break anything else. That said all code has been used in a production environment without issue.*

All code used in this demo can be found in my Github repository [here](https://github.com/Virtualizestuff/powernsx).

## Text Guide

In the previous post found [here](https://virtualizestuff.com/how-to-create-nsx-security-policies-with-rules-using-powernsx), we discussed how to create security policies with PowerNSX. In this post, I’ll demonstrate how to edit existing security policy firewall rules then apply security policies to security groups with PowerNSX.

* **Edit-NsxSecurityPolicyFwRule:**  
    Edit existing Security Policy firewall rules
    
    * **Parameters**: *SecurityPolicy, FirewallRule, ExecutionOrder, ReturnObjectIdOnly*
        
* **Add-NsxApplySpToSg:**  
    Apply security policy to security group(s)
    
    * **Parameters**: *SecurityPolicy, SecurityGroup\[\], ReturnObjectIdOnly*

That it! This is a short and sweet post. Stay tuned for the [last post](https://virtualizestuff.com/how-to-remove-nsx-security-policies-with-rules-using-powernsx) in this series where we’ll discuss how to remove NSX security policy objects using PowerNSX:

1. Remove a specific NSX security policy rule (***Remove-NsxSecurityPolicyFwRule***)
    
2. Remove a security group(s) from an NSX security policy rule (***Remove-NsxSgFromSpFwRule***)
    
3. Remove a service from an NSX security policy rule (***Remove-NsxServiceFromSPFwRule***)
    
4. Remove an applied security policy from a security group(s) (***Remove-NsxApplySpFromSg***)
    
5. Remove entire NSX security policy (***a native PowerNSX cmdlet: Remove-NsxSecurityPolicy***)
    

As well as how to properly decommission NSX Security Groups in order to prevent sync issues.
