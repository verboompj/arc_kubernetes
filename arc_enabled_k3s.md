# Intro into ARC enabled Kubernetes

In this write-up I will dive into setting up an onprem kubernetes environment to serve as an Edge deployment. 

Based on this deployment I will deploy (data)services to this "edge"  using Azure ARC. I will deploy services such as Azure PostgreSql, Azure IOT Operations and a WebApp, for example. 

[Intro into Azure ARC](https://github.com/verboompj/arc_kubernetes/blob/main/arc_enabled_k3s.md#intro-into-arc)

[Target design](https://github.com/verboompj/arc_kubernetes/blob/main/arc_enabled_k3s.md#target-design)

[K3s Kubernetes](https://github.com/verboompj/arc_kubernetes/blob/main/arc_enabled_k3s.md#kubernetes)

[ARC Enabling](https://github.com/verboompj/arc_kubernetes/blob/main/arc_enabled_k3s.md#azurearc)

## Intro into ARC
As a quick intro, what is Azure ARC ? 

Azure ARC is Azure's answer to enable customers to manage and deploy services in hybrid and/or multicloud environments. Azure ARC extends the Azure controlplane into any Virtual machine, selected Hypervisors and even other Cloud providers. 

I will use Azure ARC to extend the reach of Azure into my own datacenter, allowing me to operate and deploy workloads that I'm familiar with in the Azure Cloud, onto my own hardware.
And in this case, my own "datacenter" can be seen as an Edge site where local compute, storage and ingest can be offered as services against low latency, local performance.


### Target design 

This setup will use actual hardware, deployed onprem. I will be using my existing "datacenter cluster" (3 HPe mini desktop PC's and a Synology NAS, to serve as a iSCSI & NFS storage appliance). 
All devices are connected to a local LAN, the local lan offers communication between the physical servers, the NAS and allows outbound communication towards the Internet and selected public Azure services that I will be using.

Important as ever, is DNS. I will be depending on a lot of (inter, intra) cluster communication, and don't want to be typing IP addresses nor keeping track in hostfiles ;-). In my case I already run a Pihole DNS instance that I will use for local DNS resolution.

![](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/overview_hw.png)

### Hardware Setup

As a hypervisor I chose to run Proxmox VE, it is available for free and offers a very rich featureset, including HA, FT, Hardware Pass-through, etc. 
I don't actually need most of these features for this deployment, buit it does serve other purposes next to this case. [Proxmox VE Website](https://www.proxmox.com/en/downloads)

On top of proxmox, as virtual machines, I use Ubuntu Server LTS 22.04. 22.04 has been around for a while now and is widely supported for all types of hardware and deployments. For this deployment I need 3 instances, and all 3 instances will be deployed with a Kubernetes runtime later on.
In proxmox, I created 3 servers, I chose to run an UEFI BIOS (mark-out pre-enroll keys in the wizard), 2 cores, 12 Gb ram, 60Gib disk. 

![](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/proxmox_host.png)

## Kubernetes

For Kubernetes I chose to deploy K3s. K3s offers a light-weight kubernetes solution ideal for Edge scenario's. K3s is a certified Kubernetes solution by [Rancher](https://www.rancher.com/products/k3s) and is also free to use. 
I followed the basic step-by-step [to deploy k3s.](https://learn.microsoft.com/en-us/azure/iot-operations/deploy-iot-ops/howto-prepare-cluster?tabs=ubuntu#create-a-cluster) 

Followed by adding a 2nd and 3rd node to the cluster using :  `curl -sfL https://get.k3s.io | K3S_URL=https://myserver:6443 K3S_TOKEN=mynodetoken sh -`
The value to use for K3S_TOKEN is stored at /var/lib/rancher/k3s/server/node-token on your server node. Replace the `myserver:6443` with the actual server name (thanks DNS) 

Validate all nodes are up and running: `kubectl get nodes`

![](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/kubectl.png)
Great Success ! 

## AzureARC

Next up, ARC enabling the thing. Again following a [step-by-step](https://learn.microsoft.com/en-us/azure/iot-operations/deploy-iot-ops/howto-prepare-cluster?tabs=ubuntu#arc-enable-your-cluster)

I did run into a (dns?) glitch with 22.04 LTS; whenever i use/add the `az extension` to the local Bash when IPv6 is enabled on the machine I'm using: Running any `az` command litterly takes ages. 
I chose to disable ipv6 on the 3 Ubuntu nodes for now, and the issue is resolved. Surely some DNS issue on my side, but a topic for another day. 

Do this by changing the line from `GRUB_CMDLINE_LINUX_DEFAULT="" ` to `GRUB_CMDLINE_LINUX_DEFAULT="ipv6.disable=1" ` in the /etc/default/grub file , followed by a `sudo update-grub`

after installing de Azure CLI onto my master (initial) K3s node, i ran the steps [documented](https://learn.microsoft.com/en-us/azure/iot-operations/deploy-iot-ops/howto-prepare-cluster?tabs=ubuntu#arc-enable-your-cluster) and verified succes by issuing: `kubectl get deployments,pods -n azure-arc`

![](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/kubectl_arc.png)

Awesome. A number of deployments each consisting of one or more pods. The cluster is now ARC enabled and ready to be managed using familiar Azure toolsets. Lets quickly glance over some of it. But before we do some additional tips for Kubernetes:

1. I choose to mark my Master node (control-plane node) as cordoned. Doing so prevents any further scheduling of pods on the node itself, isolating it to do management tasks only. `kubectl cordon #hostname`
2. To avoid having to issue a sudo before any kubectl issue:  `sudo chmod 644 /etc/rancher/k3s/k3s.yaml` 

In the azure portal, search for ARC and find your new cluster as one of the connected Kubernetes Clusters:

![](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/arcportal.png)

Clicking on it one level deeper shows the details of the cluster. From top to bottom we can see the Overview, the Kubernetes resources, Settings, Monitoring and Automations.
Monitoring allows you to deploy familiar Azure Monitor solutions such as Container Insights, but also managed Prometheus and Grafana. 

![](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/arccluster.png)

I am zooming in on the Kubernetes Resources just a little bit next, to show how it maps the kubernetes logic into a portal experience.

First setup a bearer token to authenticate into the cluster using the steps [provided here](https://learn.microsoft.com/en-us/azure/azure-arc/kubernetes/cluster-connect?tabs=azure-cli%2Cagent-version#service-account-token-authentication-option) 
Its ran from the az cli enabled (1st) node of your K3s cluster. 

Copy the token output 

Now, back to the Azure portal, click on Kubernetes Resources (preview) and select Namespaces. You will be presented with a screen that asks you to paste in the token you just created.

After an auto-refresh you will see the actual namespaces active on the cluster. I am mostly interested in the 2nd menu item, Workloads. Click Workloads and under the 1st tab, Deployments, click on the drop-down marked "Filter by namespace".
Select "azure-arc" and see that the list of Deployments actually matches what we've also seen when issuing the `kubectl get deployments,pods -n azure-arc` command earlier. 

![](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/arcworkloads.png)

Swithcing over to the Pods tab we can see the pods that make up the Deployments in a little more detail. Click on any of them to find more details, such as pod IP and the node its currently running on.

![](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/poddetails.png)

Adding any new namespace / service is done using the same YAML files as one would submit using kubectl. 
And I'll leave it at that for now. As you can see , ARC enabling allows for a portal experience that enables you to do the same and more things with your cluster whilst being side-by-side with existing tools such as kubectl.
Next time I'll explore some extensions such as IOT Operations and Arc enabled Dataservices.


