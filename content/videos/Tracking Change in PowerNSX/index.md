---
title: "Tracking Change in PowerNSX"
description: "Learn how to track and audit NSX Manager API actions using PowerNSX with SSO credentials for improved security and accountability"
date: 2017-03-24
slug: tracking-change-in-powernsx
feature: "feature.png"
heroStyle: "background"
tags:
  - security
  - powershell
  - powercli
  - vmware-nsx
  - powernsx
  - nsx-automation
draft: false
---

## Video Guide

{{< youtube O9VMb-llh0g >}}

## Text Guide

**Important:**  
The code discussed in this post is an attempt to provide the ability to SSO credentials and can be obtained from [here](https://github.com/vmware/powernsx/pull/160/files).

After demoing the capabilities of PowerNSX to my colleagues and deploying it into the production environment. A question that popped up in my head was how do we track/audit who creates objects via API when using PowerNSX? Typically when you want to connect to NSX Manager via PowerNSX you leverage the Connect-NsxServer cmdlet as seen in *Figure-1* below:

![Screenshot of Visual Studio Code displaying a PowerShell script named "HomeLab.ps1" in the editor. The script connects to an NSX server using the command  with credentials prompted for NSX and vCenter. Arrows highlight parts of the script. The terminal is shown below with no output.](image-01.png)

Figure-1

Once connected all proceeding API request will use the NSX credentials. The sharing of this account is a BIG no-no from an auditing/security standpoint. *Figure-2* & *Figure-3* shows the creation of Security Tags, Groups, and Policies and as you can see they are all done with the NSX credential (admin).

![Coding session in Visual Studio Code displaying a PowerShell script with highlighted commands for creating NSX security tags, groups, and policies, indicated by orange arrows.](image-02.png)

Figure-2

![A screenshot of the VMware vSphere Web Client showing the "Audit Logs" tab under "Monitor." The log displays various user activities, focusing mainly on 'System' and 'admin' users with operations like 'Create' and 'Login' across different modules such as 'App Firewall' and 'Access Control.' The timestamps are from March 23, 2017.](image-03.png)

Figure-3

Users should have their own individual credentials with appropriate NSX role to perform their job duties this allows for an audit trail. My first thought was to create multiple CLI accounts on NSX Manager and this is a valid solution but a nightmare to manage. A simpler way is to leverage the user’s existing SSO credentials used for vCenter provided they have been assigned one of the NSX roles.

I began by diagnosing the functions below using *Write-Verbose* and *Set-Variable -Scope Global* to gain a better understanding of how they work:

* Connect-NsxServer
    
* Invoke-NsxRestMethod
    
* Invoke-NsxWebRequest
    

The first step was to add a new parameter to Connect-NsxServer called *SSOCredential* and a NoteProperty to the $connection called *SSOCredential*, *Figure-5* & *Figure-6* provides a visual:

![Screenshot of a GitHub code review showing changes made to a file. The updates include adding a new parameter called  to the  function. A highlighted comment points to the addition, explaining the update.](image-04.png)

Figure-5

![A GitHub pull request diff showing changes to a code file. Lines removed are highlighted in red, while added lines are in green. A new property, "SSOCredential," has been added, as indicated by an orange arrow and text annotation.](image-05.png)

Figure-6

As well as added logic to both Invoke functions to check if the following API calls are made:

1. /api/1.0/appliance-management/global/info
    
    * Requires ***local*** NSX Manager account with ***web-interface*** privilege (i.e. admin)
        
2. /api/2.0/services/vcconfig
    
    * Requires ***super\_user*** (System Administrator) assigned via API only or ***enterprise\_admin*** (Enterprise Administrator) or ***vshield\_admin*** (NSX Administrator)
        

**Note:** Removing this */api/1.0/appliance-management/global/info* call would allow the use of only SSO credentials, however, the \*/api/2.0/services/\*vcconfig would still require an account with at least NSX Administrator or higher role.

![Screenshot of a GitHub webpage showing code changes related to NSX and SSO credentials. It includes conditional statements with comments explaining when to use local NSX account or SSO credentials. The interface displays a split view with added lines highlighted in green and arrows pointing to specific lines of code.](image-06.png)

If we look at *Figure-7*, the $NSX variable contains our connection information notice the new SSOCredential property, it’s empty because we didn’t call the SSOCredential parameter.

![](image-07.png)

Figure-7

Let’s disconnect from the NSX Server and reconnect using the SSOCredential parameter. We’ll create some NSX objects using the same commands from earlier, see *Figures 8 – 11*. The benefit of using SSOCredential parameter is that we can now use SSO accounts with any NSX role assigned to them. If a user attempts to run a command that he/she doesn’t have permissions to an HTTP status of 403 will be returned.

**Note:** The ***secPolicy@virtualizestuff.lab*** account has a role of ***Security Administrator*** which is allowed to create Security Tags, Groups, and Policies.

![A screenshot of the VMware vSphere Web Client interface, displaying a list of users under the "Manage" tab. The list includes user names, their origin, role, status, and access scope. A row for "virtualizestuff.labecPolicy" user is highlighted, and a security administrator role is shown.](image-08.png)

Figure-8

![A Visual Studio Code interface showing a PowerShell script for connecting to an NSX server. The script includes credential requests for NSX, vCenter, and SSO. An arrow labeled "SSOCredential Stored" highlights a section in the terminal output, indicating successful credential handling and an existing PowerCLI connection.](image-09.png)

Figure-9

![Screenshot of a Visual Studio Code window showing a PowerShell script. The script includes commands to create NSX security objects: a security tag, a security group, and a security policy with a firewall rule. An arrow points to the script, labeled "Create the following objects." A terminal at the bottom displays related output.](image-10.png)

Figure-10

![](image-11.png)

Figure-11

The SSOCredential parameter is not mandatory so if your use case doesn’t require it simply don’t use that parameter. Hope this post was informative and if you have any questions please don’t hesitate to post a comment.