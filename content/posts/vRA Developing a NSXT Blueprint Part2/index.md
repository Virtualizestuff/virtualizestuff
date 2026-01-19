---
title: "vRA: Developing a NSX-T Blueprint – Part2"
description: "Explore how to create and publish an NSX-T Blueprint in vRA, covering network profiles, software components, and blueprint design strategies"
date: 2018-06-20
slug: vra-developing-a-nsx-t-blueprint-part2
feature: "feature.png"
heroStyle: "background"
tags:
  - vrealize-automation
  - vra
  - nsx-t
  - vrealize-operations
  - pks
draft: false
---


In the previous [post]({{< ref "vRA Developing a NSXT Blueprint Part1" >}}), I discussed how to setup a development environment, configuration settings made to the virtual machines and converting virtual machines to templates. This post assumes vRA is operational as I'll be discussing the following topics as it relates to creating/publishing the NSX-T Blueprint that will be used to deploy PKS.

## Network Profiles & Reservation

Network profiles contain pre-defined IP information like gateway, subnet, DNS, and IP address ranges. When a virtual machine is provisioned vRA will assign an IP address based on the information in the network profile. There are three types of Network Profiles as depicted in Figure1. I have opted for NAT network profile (One-to-Many) in order to support multiple deployments. In Figure2 are the network profiles I created for the blueprint. Next, we have to tie the external network profile with our existing port group as shown in Figure3.

![Diagram depicting a virtual distributed switch network setup. It shows three sections: External Network Profile, Routed Network Profile, and NAT Network Profile. The first section features a VM linked to a port group. The second section includes perimeter edge and distributed logical router connections with blueprints and existing networks. The third section illustrates vRA Deploy NSX Edge with options for one-to-one or one-to-many IP assignment. A key clarifies symbols for components created in NSX and vRA.](image-01.jpeg)

Figure1 – vRA Network Profiles

![Screenshot of the "Network Profiles" section displaying network settings with options for new, edit, copy, and delete. The list includes profiles like "dVS-Prod-Mgmt_VLAN12" (External) and "vSphere - Transit (NAT)."](image-02.jpeg)

Figure2 – Network Profiles

![A screenshot of the "Edit Reservation" page in vSphere (vCenter), focused on the "Network" tab. It shows a panel with network adapter options, listing various networks and associated network profiles. Some options are checked, with "dVS-Prod-Mgmt_VLAN12" selected. Tabs include General, Resources, Network, Properties, and Alerts.](image-03.jpeg)

Figure3 – Apply Network Profile to Network Adapter

## Software Component

Custom specification will destroy any user profile (tsc\\administrator) custom configurations we make like *MTputty, WinSCP, Chrome/Firefox bookmarks, desktop image, etc*. So in order to access the ControlCenter, we need a way to assign the IP address of the external interface.

During a Livefire training event for Hybrid Cloud in Boston a couple weeks ago I struck up a conversation with classmate [Chris Smith](https://www.linkedin.com/in/chris-smith-virt-arcade/) who focuses on the Cloud Automation. I explained the situation above to him and wanted to get his perspective. He mentioned the easiest way is to use Software Components with bindings. After the completing our session for the day I went back to the hotel and began playing with Software Components/bindings and to my surprise, it just worked! So a huge thank you to Chris for sharing the knowledge. Please check out his [site](http://virt-arcade.com/) here as he has excellent content around vRA.

#### SetExternalIP

The first software component I created was SetExternalIP and created a property called “cc\_ip”. We will use this property later when we configure binding. Then proceeded to Actions where I created a basic script that sets the IP address of the External interface. I take the IP address based on the cc\_ip property and assign it to \[string\]$IP. Line 1 of the code below ensure the variable is a string then passes it to the -IPAddress property of  New-NetIPAddress cmdlet. The -InterfaceAlias property references the network adapter “External” that was changed in the previous [post]({{< ref "vRA Developing a NSXT Blueprint Part1" >}}).

![Edit Software interface showing the "SetExternalIP" properties. The table includes a property named "cc_ip" with a description "External IP for ControlCenter VM," and its type is "String." Options for encryption, override, requirement, and computation are set to 'No' or 'Yes' as displayed. Navigation buttons are at the bottom.](image-04.jpeg)

Figure4 – SetExternalIP Software Component: Properties

```powershell
[string]$ip = $cc_ip
New-NetIPAddress –InterfaceAlias “External” -IPAddress $ip `
                 –PrefixLength 24 -DefaultGateway 10.100.12.x `
                 -AddressFamily IPv4 -Confirm:$false
```

![A software editing interface for "SetExternalIP" displaying actions in different lifecycle stages. The "Install" script is shown in a pop-up, written in PowerShell, for setting an external IP address with specified parameters.](image-05.jpeg)

Figure5 – SetExternalIP Software Component: Action

**SetStaticRoute**

The SetStaticRoute software component is simpler only having an action that references the External interface again. I increased the metric to ensure traffic goes through the internal interface. During the Blueprint Design phase, we will attach both software components to the nsxt-controlcenter machine.

```powershell
Set-NetIPInterface -InterfaceAlias External -InterfaceMetric 2000
```

![Screenshot of software configuration interface titled "Edit Software: SetStaticRoute," displaying lifecycle stages with script types (Powershell and Bash). The highlighted "Install" stage shows a Powershell script command for setting a network interface. A dialog box allows script editing.](image-06.jpeg)

Figure6 – SetStaticRoute Software Component: Action

## Blueprint Design

With the networking and software component portion complete we can now start working on the blueprint design. To ensure our vSphere templates are imported into vRA lets perform an inventory sync as shown in Figure7. Now click on “Design” tab &gt; Blueprints &gt; New give the blueprint a name shown in Figure8. Select the transport zone and reservation policy Figure9.

![A screenshot of an inventory interface showing the last completion date as 5/24/2018 at 5:08 PM UTC-04:00, status as "Succeeded," data collection toggled to "On," and an option to request now. There is a field for frequency in hours.](image-07.jpeg)

Figure7 – Sync Inventory

![Screenshot of a "New Blueprint" form with fields filled for creating an "NSX-T Environment." It includes fields for name, ID, description, deployment limit, and lease duration, with options to set minimum and maximum values. Buttons for "OK" and "Cancel" are at the bottom.](image-08.jpeg)

Figure8 – New Blueprint

![Dialog box titled "New Blueprint" showing settings for NSX. It includes options for "Transport zone" and "Edge and routed gateway reservation policy," with dropdown menus and an app isolation checkbox. "OK" and "Cancel" buttons are at the bottom.](image-09.jpeg)

Figure9 – Select Transport Zone and Reservation

We’ll start with a blank canvas shown in **Figure10**.

![User interface of a blueprint design tool titled "NSX-T Environment" with a grid canvas on the right and a categories list on the left, including options like "Network & Security." A prompt suggests dragging components to the canvas to start modeling.](image-10.jpeg)

Figure10 – Blank canvas

The first thing to do is drag some network constructs to the canvas Figure11:

* 1 x Existing Network

* 2 x On-Demand NAT Network

![Diagram showing a network blueprint for an NSX-T environment. Three lines connect "On-Demand NAT Network" to entities labeled "dVSProd/Mgmt_V...," "vSpherevPodRout...," and "vSphereTransiNAT." Categories are listed on the left, including Network & Security.](image-11.jpeg)

Figure11 – Add Networks

Next up is the virtual machines we will focus on the vPodRouter and the ControlCenter VMs (Figure12 – Figure16) as all other VMs will be attached to the vSphereTransitNAT network. We drag over a vSphere (vCenter) Machine then configure the machine to use the vPodRouter template we created in the previous post and attach it to the required networks. We have two virtual machines attached to their corresponding networks (Figure17). Now to attach the software components we created earlier to nsxt-controlcenter machine (Figure18) and then bind property “cc\_ip” to “\_resource~nsxt-controlcenter~ip\_address” (Figure19). This will pass the IP address from the nsxt-controlcenter

#### vPodRouter

![Screenshot of a blueprint design interface for an NSX-T environment. The interface shows a design canvas with a vSphere setup featuring a machine type selection on the left and a design layout with connected elements on the right. Options to save, finish, or cancel the design are at the bottom.](image-12.jpeg)

Figure12 – Create a vSphere (vCenter) Machine

![Screenshot of a software interface for creating a new blueprint in an NSX-T environment. The left panel displays categories such as Machine Types and Software Components. A design canvas is at the top center. The right panel shows a "Select Template" window with a list of templates featuring details like name, endpoint, CPUs, memory, and storage.](image-13.jpeg)

Figure13 – Assign vSphere template nsxt-vPodRouter

![User interface of a software application for designing a new NSX-T environment. It shows a design canvas on the right and various menu categories on the left, including Machine Types and Network & Security. A configuration for a network called "vSpherePodRouterNAT" is displayed, with options for assignment type and address.](image-14.jpeg)

Figure14 – Assign Networks to vPodRouter

#### ControlCenter

![Screenshot of editing a blueprint in an NSX-T environment. The interface shows categories like Machine Types, Software Components, and Others. A "Select Template" window lists various server configurations with details like name, endpoint, CPUs, memory, and storage. Options to save, finish, or cancel are at the bottom.](image-15.jpeg)

Figure15 – Assign vSphere template nsxt-ControlCenter

![Screenshot of an NSX-T environment blueprint editor. The design canvas shows components and connections, with a focus on network settings. Options for adding and editing network configurations are visible at the bottom. Categories and machine types are listed on the left sidebar.](image-16.jpeg)

Figure 16 – Assign Networks to ControlCenter

![Diagram labeled "Edit Blueprint: NSX-T Environment" showing a design canvas with two elements, "nsxt-vPodRouter" and "nsxt-controlcenter," connected to three lines: "dVSProd1Mgmt_VL," "vSpherePodRout," and "vSphereTransitNAT." Save, Finish, and Cancel buttons are at the bottom.](image-17.jpeg)

Figure17 – Design canvas with 3 networks and 2 virtual machines

![Diagram of an NSX-T environment blueprint with interconnecting components like `nsxt-vPodRouter` and `nsxt-controlcenter`. The interface shows various software components and design elements with lines and arrows indicating connections.](image-18.jpeg)

Figure18 – Attach SetExternalIP and SetStaticRoute to nsxt-controlcenter machine

![Screenshot of an "Edit Blueprint: NSX-T Environment" interface. It shows a design canvas with components like "nsxt-tvPodRouter" and "nsxt-controlcenter.” A properties panel below displays a field for setting an "External IP" for ControlCenter VM with a string value. Options to save or finish are visible.](image-19.jpeg)

Figure19 – Software Component Binding

#### Rinse and Repeat

Simply repeat the process for the remaining virtual machines that make up the NSX-T blueprint. Your design canvas should look similar to Figure20 once finished.

![A network diagram on a design canvas for an NSX-T environment, displaying interconnected components with labels such as nsxt-vcRouter, nsxt-controlCenter, and various nodes labeled with vSphere (vCenter) Manager. There are connections between these components, representing the network setup.](image-20.jpeg)

Figure20 – Design Canvas with all required networks, virtual machines and software components

Below is a breakdown of the dependency order for the virtual machines inside the blueprint:

1. nsxt-vPodRouter

2. nsxt-dc

    * nsxt-esxi-01

    * nsxt-esxi-02

    * nsxt-esxi-03

    * nsxt-esxi-04

    * nsxt-esxi-05

    * nsxt-esxi-06

3. nsxt-vcsa-01

4. nsxt

    1. nsxt-ctrl-01

    2. nsxt-ctrl-02

    3. nsxt-ctrl-03

        * nsxt-edge-01

        * nsxt-edge-02

5. nsxt-controlcenter

    * SetExternalIP

    * SetStaticRoute

## vRO PowerShell Host / Event Subscription

Since this is a nested environment we will need to enable the following security policies; *Promiscuous Mode*, *Forged Transits*, and *MAC Address Changes* in order for the virtual machines to communicate. To accomplish we will leverage Event Subscription to initiate a PowerCLI script via a Powershell host in VRO due to my lack of javascript experience but is something I’m working on it  ;).

1. Add PowerShell Host

    ![Screenshot of VMware vRealize Orchestrator showing a task to "Add a PowerShell host" with a message indicating successful addition. The interface includes a schema workflow with steps like "Is SSL?," "Construct URL," and "Import a certificate." Various folders and tasks are displayed on the left pane.](image-21.png)

2. Invoke an External Script  
    In our case, it’s just a simple .ps1 file with `Write-Output “Testing ;)”` to validate things work. You can see in the screenshot above Invoke an external script ran successfully.  
    **Important:** In order to get the scripts to execute successfully I had to update the PowerShell plugin from *1.0.11* to ***1.0.13*** after doing so the scripts ran without issue.

![Screenshot of Windows PowerShell ISE with a script that outputs "Testing ;)".](image-22.png)

Figure22 – Simple script to test functionality

\* Create a vRO workflow that will process the vRA payload looking for the two on-demand networks by finding the port group ids then passing that information to the PowerCLI script that will enable Promiscuous Mode, Forged Transits, and MAC Address Change.

![Screenshot of VMware vRealize Orchestrator showing the "SetSecurityPolicyOnPortGroups" workflow. The interface includes a tree view with folders such as AMQP and PowerShell, and an attribute section listing various settings like ExternalScript and Password with corresponding values.](image-23.png)

Figure23 – SetSecurityPolicyOnPortGroups Workflow – Attributes

![Screenshot of the VMware vRealize Orchestrator interface, showing a script for a workflow named "SetSecurityPolicyOnPortGroups." The interface includes tabs for General, Inputs, Outputs, and more, as well as a navigation panel on the left displaying workflow components and a script editor on the right with highlighted JavaScript code.](image-24.png)

Figure24 – SetSecurityPolicyOnPortGroups Workflow – Script to extract port group ids created by vRA

![Screenshot of VMware vRealize Orchestrator interface displaying the workflow "SetSecurityPolicyOnPortGroups." The left pane shows a hierarchical library of tasks, and the central area features a schema with visual workflow elements connected by arrows. Below is a panel for configuring input and output parameters for an external script invocation.](image-25.png)

Figure25 – SetSecurityPolicyOnPortGroups Workflow – Visual Bindings from script to external script

![A Windows PowerShell Integrated Scripting Environment (ISE) window displaying a script named "SetPortSecurity.ps1". The script connects to a vCenter server using provided credentials and updates security policies for specified port groups.](image-26.png)

Figure26 – SetSecurityPolicyOnPortGroups Workflow – PowerCLI script used to change security policy settings on discovered port groups.

\* Create an Event Subscription making sure to define conditions, select workflow (SetSecurityPolicyOnPortGroups) and enable blocking.

![Screenshot of "Edit Workflow Subscription" interface showing conditions for running a workflow based on component ID, lifecycle state, and blueprint name. Includes options to add expressions.](image-27.png)

Figure27 – Event Subscription conditions. Only need to affect the vPodRouter logical switches (port groups) as all virtual traffic flows on these two networks.

![Screenshot of the vRO workflow subscription interface. The "SetSecurityPolicyOnPortGroups" workflow is selected, with its input and output parameters displayed. The interface includes navigation tabs and buttons for workflow actions.](image-28.png)

Figure28 – Select vRO workflows to execute

![Screenshot of an administration interface showing the details of a workflow subscription named "SetSecurityPolicyOnPortGroups." It includes fields for priority set to 7, timeout set to 15 minutes, and a description that enables certain network security policies for on-demand networks created by vRA. A "Blocking" option is checked.](image-29.png)

Figure29 – Set blocking and define priority

Confirm workflow executed successfully and settings have been changed.

![Screenshot of VMware vRealize Orchestrator showing a workflow named "SetSecurityPolicyOnPortGroups" with steps like printProperties, Scriptable task, and Invoke an external. The left panel lists scripts and folders, while the bottom panel displays log messages.](image-30.png)

Figure30 – vRO workflow success

![A Windows PowerShell ISE interface displaying a script and its resulting output. The script retrieves and formats security policies for named port groups. The output below lists settings for "AllowPromiscuous," "ForgedTransmits," and "MacChanges," all set to "True" for two distinct port groups.](image-31.png)

Figure31 – Validate via PowerCLI

## Publishing Time

Our blueprint is ready to be published simply click Finish and click Publish (Figure21).

In order to be selectable from the catalog, we need to associate the blueprint with a service assuming Service and Entitlements have already been set up. Navigate to Administration &gt; Catalog Items &gt; Select NSX-T Environment &gt; Configure &gt; Service. In my case, the service is called “VMware \[Nested\]”.

**Note:** You can also add an icon here for your blueprint. 😉

If we go to our catalog &gt; VMware \[Nested\]  we should see our new shiny blueprint (Figure22). In our environment, this blueprint takes an hour to deploy (Figure23).

FYI the Blueprint name is “\[NAT\] NSX-T Environment” is the original blueprint. The “NSX-T Environment” blueprint was created for this post as you can see it’s a popular one (Figure24). 😉

To access the environment I simply get the IP address for the nsxt-controlcenter from the Items tab (Figure25).

![Screenshot of a web interface showing a "Success" message indicating selected items were published. The interface displays a list of blueprints with columns for name, description, status, and last modified date.](image-32.jpeg)

Figure32 – NSX-T Environment blueprint published

![Service catalog interface showing VMware nested services, including a 3-node vSAN cluster and NSX-T environment, with request buttons. Side menu lists other categories like Containers, DellEMC, and Microsoft.](image-33.jpeg)

Figure33 – Published NSX-T Blueprint

![Screenshot of a request for the deployment of an NSX-T environment, showing execution information with a table listing components, their status, details, and times. The status is approved, and the requester is Dave Davis.](image-34.jpeg)

Figure34 – Deployment Time

![A software interface displaying a list of deployments with details like name, description, owner, expiration dates, and creation dates. Some deployments show warnings. The left sidebar contains options for Deployments, Machines, and Network.](image-35.jpeg)

Figure35 – Popular Blueprint

![A software interface displaying a list of deployments with details such as name, description, owner, IP address, and dates. The list includes various entries provisioned by VMware, showing different status icons and information.](image-36.jpeg)

Figure36 – Get IP Address

## Results

Below are screenshots of various components vSphere, NSX-T, and PKS (Figure37 -Figure41). This has been an awesome learning experience I hope you enjoyed the two posts series on developing an NSX-T blueprint in vRealize Automation.

If your interested in seeing a video demonstration of the vRA/NSX-T environment just leave a comment below and I’d be happy to put something together.

![A network diagram titled "NSX-T/PKS (vRA Blueprint)" shows various network components and connections, including vSpherePodRouter, gateways, VLANs, and IP subnets. It highlights sections like Infrastructure, NSX-T, and Pivotal with VM instances and a DHCP pool. A table at the bottom right lists IP addresses and corresponding contexts. Desktop icons are visible on the left.](image-37.jpeg)

Figure37 – ControlCenter Desktop

![Dashboard interface of VMware NSX showing the overview of fabric components and clusters. It displays circular graphs for deployment connectivity, transport nodes, transport zones, and clusters. Backup settings indicate automatic backups are not enabled and not configured.](image-38.jpeg)

Figure38 – NSX-T Dashboard

![Screenshot of a VMware NSX interface showing a list of logical routers. Columns include Logical Router, ID, Type, Connected Tier-0 Router, High Availability Mode, Transport Zone, and Edge Cluster. Options for adding, editing, deleting, or taking actions are available. The interface also displays a navigation panel on the left.](image-39.jpeg)

Figure39 – NSX-T Routers

![Screenshot of a network management interface displaying a list of logical switches. The table includes columns for the switch name, ID, admin status, logical ports, traffic type, config state, and transport zone. All switches are marked with an "Up" status and "Success" config state.](image-40.jpeg)

Figure40 – NSX-T Switches

![A screenshot displays a vSphere client interface on the left, indicating the successful deployment of PKS with a list of virtual machines. On the right is a webpage titled "Yelb," offering healthy food recommendations with voting options for IHOP, Chipotle, Outback, and Buca di Beppo.](image-41.jpeg)

Figure41 – vCenter and PKS components deployed with a functional application
