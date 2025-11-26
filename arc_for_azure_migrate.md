# Intro ARC inventory as base for a Azure Migrate Businesscase & Assessment

In this write-up I will dive into setting up a Azure ARC Edge deployment. 

Based on this deployment I will create a inventory file in CSV, based on the VM Insights and Connected Machine ARC extension.






## Intro into ARC
As a quick intro, what is Azure ARC ? 

Azure ARC is Azure's answer to enable customers to manage and deploy services in hybrid and/or multicloud environments. Azure ARC extends the Azure controlplane into any Virtual machine, selected Hypervisors and even other Cloud providers. 

I will use Azure ARC to extend the reach of Azure into my own datacenter, allowing me to operate and deploy workloads that I'm familiar with in the Azure Cloud, onto my own hardware.
And in this case, my own "datacenter" can be seen as an Edge site where local compute, storage and ingest can be offered as services against low latency, local performance.

ream more on Azure Arc right here : https://learn.microsoft.com/en-us/azure/azure-arc/overview 

### Target design 

This setup will use actual hardware, deployed onprem. I will be using my existing "datacenter cluster" (Proxmox VE) 
All devices are connected to a local LAN. The local lan offers communication between the physical servers and allows outbound communication through a NAT-firewall towards the Internet and selected public Azure services that I will be using.

2 demo Windows Server VM's will be used, both deployed with Windows Sevrer 2022 Datacenter Edition. 
I will deploy the "ARC agent" first, then, using a Data Collection Rule deploy the "AMA agent" (Azure Monitor Agent) and lastly enable enrollment of the "Dependency Agent" , as part of the AMA agent.

Fort this demo I will deploy a new Log Analytics workspace just for the 2 VM's and we will explore some tables and run som KQL queries to extract the data we need in order to create a Azure Migrate Assessment and Business Case.

To keep it lean and mean, i will be using Public Endpoints for all Azure Services, and Europe as my service and data boundry. 

### Hardware Setup

As a hypervisor I chose to run Proxmox VE, it is available for free and offers a very rich featureset, including HA, FT, Hardware Pass-through, etc. 
I don't actually need most of these features for this deployment, buit it does serve other purposes next to this case. [Proxmox VE Website](https://www.proxmox.com/en/downloads)

On top of proxmox, as virtual machines, i deployed 2 simple Windows server instances : `winserver01` & `winserver02` 

![](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/pve.png)


<br><br><br>

## Log Analytics Workspace

A fresh LAW (Log  Analytics Workspace) is usefull if you really want to track all updates the AMA agent and Dependency Agent create as Tables in your workspace, so lets deploy one:
Create a new Resouce Group `ARC_Zwolle` in my case , and give your WS a name `ARCws` for example(make sure it is unique in your RG). Next, Next, Create: 

![](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/LAW.png)

<br><br><br>



## Azure ARC

Azure ARC next: In the portal type ARC in the top field and click on Azure ARC: 

![](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/ARC.png)

<br><br>
We will create a onboarding script for the 2 OnPrem servers. 
Click on Infrastructure, Machines in the left column and click the " `+ Onboard/Create` " to onboard new resources.
Since we are only onboarding a few resources, a manual onboarding is OK for us. 

![](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/ARC_On.png)

<br><br>

Fill in the required fields: Resource group (same as before), a Region ( West Europe for me), Operating System (in this case Windows), I left the SQL checkmark unchecked, a Public Endpoint for this Demo and finally the Authentication, here we select Manual as we are only onboarding 2 VM's.

![](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/ARC_On2.png)

<br><br>

Next, tagging is of use to display richer context of this deployment, as an example ass a `Datacenter` : `nameofDC` tage and a `City` and or `Country` as example. This will be applied to the ARC resources next. 
Next, click Download and Run script. The easiest way for me is to simply copy the script to my clipboard and execute the script in Powershell (ISE) on the target server(s) 

After awhile, a popup or browser will launch, asking you to authenticate to your Azure subscription. Doing so enables the final registration of this resource into the Azure ARC enviromnent. If all went wll, the end result should look something like this:

![](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/ARC_On3.png)

<br><br>




