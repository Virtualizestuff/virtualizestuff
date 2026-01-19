---
title: "How to Create NSX Security Policies with Rules using PowerNSX"
description: "Automate NSX security policies using PowerNSX custom cmdlets for efficient policy creation and streamlined management"
date: 2017-08-07
slug: how-to-create-nsx-security-policies-with-rules-using-powernsx
feature: "feature.png"
heroStyle: "background"
tags:
  - powershell
  - powernsx
  - new-nsxsecuritypolicyfirewallrulespec
  - new-nsxsecuritypolicygisspec
  - new-nsxsecuritypolicy
  - add-nsxsecuritypolicyfwrule
  - add-nsxsgtospfwrule
  - add-nsxservicetospfwrule
draft: false
---

## Video Guide

{{< youtube j-THaTAvJ1k >}}

The video above demonstrates the cmdlets discussed in this post.

All code used in this demo can be found in my Github repository [here](https://github.com/Virtualizestuff/powernsx).

## Text Guide

In a recent consulting role I performed some PowerNSX development to help automate the creation of security policies with PowerNSX. The business wanted to reduce the time it took to create / edit / remove security policies within NSX. While the GUI is nice it can be very time-consuming when performing repetitive tasks like creating security policies with rules. The organization knew about PowerNSX, however, the ability to manipulate security policies with their rules wasn’t available. The organization relies heavily on Security Groups / Security Policies in their NSX environment.

So I knew I’d have to give it an “old college try” and develop cmdlets that address their concerns while also helping the PowerNSX community. It’s a **win** (business), **win** (PowerNSX community), **win** (personal development) situation!

A quick shout out to Nick Bradford for developing PowerNSX and the community for their continued contributions, Thank You!

The New-NsxSecurityPolicy cmdlet allows for the creation of a security policy with the associated firewall rules and Guest Introspection Services (Network Introspection Services coming soon). The New-NsxSecurityPolicy cmdlet by itself creates an empty security policy which isn’t particularly helpful. So it made sense to create additional functions for each of the services:

***Disclaimer:*** *The code shown in this post is not included in the PowerNSX module. There is still work to be done as I need to write Pester tests for these cmdlets to ensure everything works as expected and doesn’t break anything else. That said all code has been used in a production environment without issue.*

#### Spec Cmdlets:

* **New-NsxSecurityPolicyFirewallRuleSpec:**
    
    Creates the firewall rules in XML format using the following tags:
    
    * **Parameters**: *Name, Category, Description, Order, Enabled, Security Group(s), Service(s), Logging, Action, Direction*
        
* **New-NsxSecurityPolicyGISSpec:**
    
    Creates the Guest Introspection Service in XML format using the following tags:
    
    * **Parameters**: *Name, Category, Description, Order, Enabled, Enforced, ActionType*
        
* **New-NsxSecurityPolicyNISSpec (Coming Soon):**
    

---

#### Cmdlets:

* **New-NsxsecurityPolicy:**
    
    This function ensures the base XML shell needed to create an empty SP exists.
    
    * **Parameters**: *Name, Description, Precedence, FirewallRule\[\], GuestIntrospectionService\[\], AppliedTo, ReturnObjectIdOnly*
        
    * **Note:** The information passed through the FirewallRule\[\] and GuestIntrospectionService\[\] parameters are handed to their respective functions to generate the XML variable(s) which is then merged with the base XML to provide a completed XML formatted request which is then sent off to the NSX Manager.
        
* **Add-NsxSecurityPolicyFwRule:**
    
    Allows the ability to add additional rules to an existing Security Policy.
    
    * **Parameters**: *SecurityPolicy, FirewallRule*
        
* **Add-NsxSgToSpFwRule:**
    
    Add Security Group(s) to existing rules in a Security Policy.
    
    * **Parameters**: *SecurityPolicy*, SecurityGroup, ExecutionOrder
        
* **Add-NsxServiceToSpFwRule:**
    
    Add Service(s) to existing rules in a Security Policy.
    
    * **Parameters**: *SecurityPolicy, Service, ExecutionOrder*
        

For those like me and “new” to developing functions with PowerShell, I highly suggest reading the PowerNSX module itself, as it contains a wealth of information that has helped me wrap my head around functions.

Stay tuned for the [next post](https://virtualizestuff.com/how-to-edit-security-policies-apply-them-to-security-groups-with-powernsx) where we will discuss how to edit existing security policies with PowerNSX and applying said policy to security groups.


