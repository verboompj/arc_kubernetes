# Intro ARC inventory , Dependency Mapping & create a export for an Azure Migrate Businesscase & Assessment

In this write-up I will dive into setting up a Azure ARC Edge deployment. 

Based on this deployment I will create a inventory of all things OnPrem and a export in CSV, based on the VM Insights and Connected Machine ARC extension.
The export I can use to create a businesscase in Azure Migrate.


#### !!Important!! 

Azure ARC now supportd a direct integration with Azure Migrate through a preview feature, check it out here: https://learn.microsoft.com/en-us/azure/migrate/concepts-arc-resource-discovery?view=migrate 

<br><br>


[Intro ino ARC](https://github.com/verboompj/arc_kubernetes/blob/main/arc_for_azure_migrate.md#intro-into-arc)

[Log Analytics Workspace](https://github.com/verboompj/arc_kubernetes/blob/main/arc_for_azure_migrate.md#log-analytics-workspace)

[Deploy ARC](https://github.com/verboompj/arc_kubernetes/blob/main/arc_for_azure_migrate.md#azure-arc)

[VM Insights](https://github.com/verboompj/arc_kubernetes/blob/main/arc_for_azure_migrate.md#azure-vm-insights)

[Depndency Agent](https://github.com/verboompj/arc_kubernetes/blob/main/arc_for_azure_migrate.md#optional-dependency-agent)

<br><br>

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

We will enable VM Insigts, a custom DCR, (optional) the Dependency AGent and run KQL queries.

To keep it lean and mean, i will be using Public Endpoints for all Azure Services, and Europe as my service and data boundry. 

### Hardware Setup

As a hypervisor I chose to run Proxmox VE, it is available for free and offers a very rich featureset, including HA, FT, Hardware Pass-through, etc. 
I don't actually need most of these features for this deployment, buit it does serve other purposes next to this case. [Proxmox VE Website](https://www.proxmox.com/en/downloads)

On top of proxmox, as virtual machines, i deployed 2 simple Windows server instances : `winserver01` & `winserver02` 

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

Next, tagging is of use to display richer context of this deployment, as an example ass a `Datacenter` : `nameofDC` tag and a `City` and or `Country` as example. 
This will be applied to the ARC resources next. 

Next, click Download and Run script. The easiest way for me is to simply copy the script to my clipboard and execute the script in Powershell (ISE) on the target server(s) 

After awhile, a popup or browser will launch, asking you to authenticate to your Azure subscription. Doing so enables the final registration of this resource into the Azure ARC enviromnent. If all went wll, the end result should look something like this:

![](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/ARC_On3.png)

<br><br>

## Azure VM Insights

Allright, lets get some Monitoring Logs and Metrics in for our 2 servers. By default, ARC enables many services for free. The VM Insights we are after are metered services, billed by the amount of logs ingested. It won't be much for a demo environment , so lets go ahead to set it up.

VM Insights are different from VM Performance Logs through a custom DCR. Be ware that VMInsights enabled the VM Insights data based on a predefined set of Metrics. 
Adding more Metrics is possible, using a second DCR for capturing those specific ones. I will explain DCR's in a minute. 

Simply click Monitor Insights , configure your Log Analytics Workspace created earlier and done. 

<br><br>
![](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/ARC_Res.png)
<br><br>

Using this method , in the dialog click on `Customize Infrastructure Monitoring` , uncheck the preview and select our LAW we created before as target. Click Save and Enable. 
If all went well, the AMA agent will be pushed to your ARC enabled VM through a newly created Data Collection Rule (DCR). Nice one ! 


![](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/ARC_Mon4.png)


Wile we wait for it to complete, lets see what we actually triggerd by this quick enablement --> A DCR.

Lets see that new Data Collection Rule.

For that we go to Azure Monitor in the Azure Portal. Use search bar at the top again and search for Monitor.
When in Monitor, scroll down in the left column to Settings and click on Data Collection Rules.

Here you see our auto created rule. Click on it to explore it. 

<br><br>
![](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/ARC_Mon5.png)
<br><br>

The rule consists of "routing logic" in this case for Performance Counters to Log Analytics , see `View Data Sources` 
The rule also triggers the deploymant of the AMA Agent for all selected Resources. Click `Resources` to find the server selected.

<br><br>
![](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/ARC_Mon6.png)
<br><br>

Try not to modify this auto generated rule, again if you want to capture more metrics or custom fields, simply create an additional DCR.

For the sake of doing so, lets create such rule, give it a name, select your source systems and as target deifine Metrics and make sure you route them to your LAW

<br><br>
![](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/DCR6.png) 

<br>

![](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/DCR7.png)

<br><br>


Ok back to our VM on the ARC blade - By now the AMA agent should be deployed. Lets check the Extensions on the right column: 

<br><br>
![](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/ARC_Mon7.png)
<br><br>


### What you should see in your LAW now 
The default tables are these : 
<br><br>
![](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/ARCNEW.png)
<br><br>

If you only enabled VM Insights and the custom DCR, you should see these tables:
<br><br>
![](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/ARCNEW2.png)
<br><br>



## Dependency Agent

We may want to have the Dependency Agent deployed. It is a extension of the AMA agent and enables deeper insights on connections and dependencies through MAP or KQL visualizations.
It is set for retirement in June of 2028 - thats OK for now. See : https://learn.microsoft.com/en-us/azure/azure-monitor/vm/vminsights-dependency-agent#manually-install-or-upgrade-dependency-agent-on-windows 

There are many ways to deploy this agent, in this demo we deploy it just as we did the AMA agent itself, through a DCR ( the rule ;-) ) 
However, i'd like to click through once more to deploy this rule for me . 

On the Server in the ARC blade, click Insights once more. Here you'll see results comming in on Performance Metrics captured by our fiorst DCR rule. 
Click on Map at the top 
<br><br>
![](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/ARC_Mon11.png)
<br><br>

Immidiately a popup appears on the right side where we can deploy the Dependency Agent , however we need to enable a additional DCR for it and cannot combine it with our first. 
For that click on Create New 

<br><br>
![](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/ARC_Mon9.png)
<br><br>

Add a name and select the Dependency Agent ( Map feature) This will trigger the deployment of the agenton top of the AMA Extension using a new DCR.


<br><br>
![](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/ARC_Mon10.png)
<br><br>

After completing we should get some additional tables in the Log Analytics Workspace we can query in KQL in our next adventure.

<br><br>
![](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/LAW3.png)
<br><br>

