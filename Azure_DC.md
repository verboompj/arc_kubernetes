As a follow-up from my previous post where [I deployed a "edge" Kubernetes cluster onprem and connected it to Azure ARC](https://github.com/verboompj/arc_kubernetes/blob/main/arc_enabled_k3s.md), 
In this write-up I will deploy an Azure managed & supported PostgreSQL database service & database on my onprem cluster.

Effectively I will have an ARC managed Kubertetes platform on an Edge location (my home) where I will deploy Azure Services to. The same Azure services I am familiar with on the public Azure platform, running in my home on my network. Cant wait !

First up, quick recap- 

I have deployed a 3 node cluster, each node has 12 Gib ram, 4 cores and access to a network that provides outbound internet connectivity. On this cluster I have deployed K3s Kubernetes and connected the cluster to Azure using Azure ARC.







Prereqs  https://learn.microsoft.com/en-us/azure/azure-arc/data/sizing-guidance#minimum-deployment-requirements


Choose between direct and indirect see: https://learn.microsoft.com/en-us/azure/azure-arc/data/connectivity 
I chose Direct 


This data controller will be installed in direct connectivity mode to an existing Azure Arc-enabled Kubernetes cluster. This connectivity mode will allow you to create and manage Arc-enabled data services, such as SQL Managed Instance enabled by Azure Arc and PostgreSQL enabled by Azure Arc directly from the Azure portal.


kubectl get storageclass




https://learn.microsoft.com/en-us/azure/azure-arc/data/install-client-tools

Don't use the quickstart, as it assumes an AKS deployment insterad of a "any" kubernetes approach. 
Its in the same doc, under the "how-to guides": [Any kubernetes how-to using Azure Portal](https://learn.microsoft.com/en-us/azure/azure-arc/data/create-data-controller-direct-azure-portal)
I ran the Direct connected, Azure Portal deployment wizzard. There are other options like the CLI or using Azure Data Studio. 





Disabled metrics, monitoring in the next step and deployed. 





https://learn.microsoft.com/en-us/azure-data-studio/quickstart-postgres



