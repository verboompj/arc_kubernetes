

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
