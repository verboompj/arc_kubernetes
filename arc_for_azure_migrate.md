# Intro ARC inventory as base for a Azure Migrate Businesscase & Assessment

In this write-up I will dive into setting up a Azure ARC Edge deployment. 

Based on this deployment I will create a inventory file in CSV, based on the VM Insights and Connected Machine ARC extension.






## Intro into ARC
As a quick intro, what is Azure ARC ? 

Azure ARC is Azure's answer to enable customers to manage and deploy services in hybrid and/or multicloud environments. Azure ARC extends the Azure controlplane into any Virtual machine, selected Hypervisors and even other Cloud providers. 

I will use Azure ARC to extend the reach of Azure into my own datacenter, allowing me to operate and deploy workloads that I'm familiar with in the Azure Cloud, onto my own hardware.
And in this case, my own "datacenter" can be seen as an Edge site where local compute, storage and ingest can be offered as services against low latency, local performance.


### Target design 

This setup will use actual hardware, deployed onprem. I will be using my existing "datacenter cluster" (Proxmox VE) 
All devices are connected to a local LAN, the local lan offers communication between the physical servers, a NAS and allows outbound communication towards the Internet and selected public Azure services that I will be using.
