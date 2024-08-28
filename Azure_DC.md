As a follow-up from my previous post where [I deployed a "edge" Kubernetes cluster onprem and connected it to Azure ARC](https://github.com/verboompj/arc_kubernetes/blob/main/arc_enabled_k3s.md), 
In this write-up I will deploy an Azure managed PostgreSQL database service & database on my onprem Kubernetes cluster.

Effectively I will have an ARC managed Kubertetes platform on an Edge location (my home) where I will deploy Azure Services to. The same Azure services I am familiar with on the public Azure platform, running in my home on my network. 

#### First up, quick recap- 

Previously I have deployed a 3 node cluster, each node has 12 Gib ram, 4 cores and access to a network that provides outbound internet connectivity. 
On this cluster I have deployed K3s Kubernetes and connected the cluster to Azure using Azure ARC.

Topics in this wrtite-up: 

[Azure ARC Data Controller](https://github.com/verboompj/arc_kubernetes/blob/main/Azure_DC.md#1-deploy-the-azure-data-controller) 

[Verify the Data Controller deployment](https://github.com/verboompj/arc_kubernetes/blob/main/Azure_DC.md#verify-deployment)

[Deploy Azure PostgreSQL](https://github.com/verboompj/arc_kubernetes/blob/main/Azure_DC.md#deploy-postgresql)

[Validating the deployment](https://github.com/verboompj/arc_kubernetes/blob/main/Azure_DC.md#validating-the-deployment)



### Deploy the Azure ARC Data Controller

In order to deploy Azure Data Services such as Azure SQL and Azure PostgreSQL, your ARC connected Kubernetes cluster needs to have a Data Controller running. Currently only one controller per cluster is supported. 

_In Kubernetes terminology, the data controller is a Kubernetes controller object that extends Kubernetes APIs to manage lifecycle of the data services. It is the heart of Arc-enabled data services and uses the underlying Arc-enabled Kubernetes for all aspects of managing the lifecycle of the data services like provisioning/deprovisioning, monitoring/logging to communication with Azure to bring down commands, policy and sending inventory, usage, metrics and billing information to Azure._

I'm not running any production workloads, but if you are, make sure you meet [all the resource prereqs.](https://learn.microsoft.com/en-us/azure/azure-arc/data/sizing-guidance#minimum-deployment-requirements)
Next, I moved over to [this great overview](https://learn.microsoft.com/en-us/azure/azure-arc/data/plan-azure-arc-data-services#deployment-steps) to prepare for my deployment. 
I won't use the quickstart from the Docs page, as it assumes an AKS deployment insterad of a "any" kubernetes approach. What we need is in the same doc, under the "how-to guides": [Any kubernetes how-to using Azure Portal.](https://learn.microsoft.com/en-us/azure/azure-arc/data/create-data-controller-direct-azure-portal) 
You can choose between direct and indirect see: https://learn.microsoft.com/en-us/azure/azure-arc/data/connectivity.

So onto the portal: 

![](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/portaldata.png)

My data controller will be installed in direct connectivity mode to an existing Azure Arc-enabled Kubernetes cluster. Hence, I ran the Direct connected, Azure Portal deployment wizzard. There are other options like the CLI or using Azure Data Studio. 

![](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/arcdc.png)

Supply the details, and make sure to create a new Custom Location where you select you ARC enabled Kubernetes cluster.

![](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/detailscontroller.png)

For the configuration Template, select `azure-arc-kubeadm`

![](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/kubeconfig.png)

The Details for your cluster are next, in my case I had to retrieve the Storage Class my cluster procides, that can be done running `kubectl get storageclass` on my 1st K3s node

![](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/storageclassk3s.png)

I choose Loadbalancer as Service Type, so the end result should look something like this:

![](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/kubedetailsarc.png)

In the next step I disabled metrics & monitoring (you may choose otherwise) and deployed by clicking Create. 

### Verify Deployment

Awesome, now thats all setup, we can track its progress in the Azure portal. You will see a new namespace under the name you provided earlier when creating the data controller.

![](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/dataservicecomplete.png)

And by browsing to the resource in the Azure portal, in the Resource Group you selected, you'll find your Data Controller Instance, ready, connected and all set to go to work.

![](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/dataconvalidate.png) 

### Deploy PostgreSQL 

As a last step I deploy a Postgres instance. Deployment is pretty straigt forward from the ARC pannel in the Azure portal, browse to PostgreSQL (preview) and click Create at the top left.

Select your Dataservice name as the Custom Location , size your instance and select the prefered Service Type. I selected Load-Balancer again. Type in your standard credentials ;-) , no I mean the secure ones ! And hit Create.

![](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/deploypostgresportal.png)

You will see it being deployed on your cluster shortly / immediately 

![](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/postgresdeploy.png)

And as a resource in the Azure portal as well. Mind the IP address as the external endpoint. That looks a lot like my local LAN ! (or in this case vlan) 

![](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/postgedeployed.png)

### Validating the deployment

Awesome! another succesfull deployment. Or is it ? we can check by creating a new database on our local Azure Postres instance, and query it maybe ? 

I used Azure Data Explorer and addedd the Postgress plugin / add-on. Using this great quickstart, I was able to connect and do some SQL queries: https://learn.microsoft.com/en-us/azure-data-studio/quickstart-postgres

The result is actually pretty cool, a local Azure Postgress instance on my local network, exposed on my local IP addresses. 

![](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/postgresquery.png)


