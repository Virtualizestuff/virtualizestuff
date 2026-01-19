---
title: "Why NSX Name Fields Should Be Case-Insensitive"
description: "Why NSX name fields should be case-insensitive to prevent duplicate entries and ensure consistent security group management in VMware environments"
date: 2017-05-20
slug: why-nsx-name-fields-should-be-case-insensitive
feature: "feature.png"
heroStyle: "background"
tags:
  - powershell
  - nsx
  - powernsx
  - nsx-security-group
  - case-insensitive
draft: false
---


## Creating Security Groups

The example below is why NSX name fields should be case-insensitive. When creating security groups with the names: “SG-01”, “Sg-01”, “sG-01”, and “sg-01” one might think this won’t be allowed as they’re the same name. Actually, NSX allows this due to the fact that the name field is case-sensitive resulting in four security groups created as shown in *Figure-1*:

![A winking emoji with a white cross and open mouth is next to four file icons labeled differently with variations of "SG-01".](image-01.png)

Figure-1

Again, the above example shows why NSX name fields should be case-insensitive. This would prevent such behavior, for example, security group “SG-01” already exists and when attempting to create another security group with the name “sg-01” would fail with a message similar to “Another object with the same name already exists in the current scope: Global”.

This issue also occurs in the following objects:

* Security Groups
    
* Security Policies
    
* Custom Services
    
* IP Sets
    
* MAC Sets
    
* Service Groups
    
* IP Pools
    
* Logical Switches
    
* Creating NSX Edges
    

Surprisingly, Security Tags prevents the above problem, as shown in *Figure-2*:

![Screenshot of a VMware vSphere Web Client interface showing a list of security tags. A warning message indicates a name conflict: "Another object with the same name: st-01 already exists in the current scope: Global." An emoji with a smiling face is added for emphasis.](image-02.png)

Figure-2

Granted, this issue “could” be resolved with standardized naming schemes, however, simply forgetting to capitalize or not capitalize a letter could accidentally create an unintended object. When attempting to filter for “SG-01” in the GUI or in PowerNSX all four security groups are returned, as seen in *Figure-3* which is not what the Emoji wants.

Luckily with PowerNSX, we can leverage PowerShell’s -cmatch comparison operator to help narrow down our selection as shown in *Figure-4*. That’s going to wrap up this post.

![A side-by-side comparison of a GUI and PowerNSX command results showing different cases of "SG-01". The GUI search highlights the correct "SG-01" while PowerNSX lists several variations, with only "SG-01" being correct. A thought bubble mentions "Only want SG-01".](image-03.png)

Figure-3

![Command prompt displaying details of a security group named "SG-01" with a smiling emoji and arrow pointing to the name filter.](image-04.png)

Figure-4
