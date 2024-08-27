# Intro into ARC enabled Kubernetes

In this write-up I will dive into setting up an onprem kubernetes environment to serve as an Edge deployment. 

Based on this deployment I will deploy (data)services to this "edge"  using Azure ARC. I will deploy services such as Azure PostgreSql, Azure IOT Operations and a WebApp, for example. 


[Intro into ARC](https://github.com/verboompj/EntraGSA/blob/main/README.md#intro-into-arc)

[Setup Entra Private Access](https://github.com/verboompj/EntraGSA/blob/main/README.md#setup-entra-private-access)

- [1. Connector Service](https://github.com/verboompj/EntraGSA/blob/main/README.md#2-connector-service)




## Intro into ARC
As a quick intro, what is Azure ARC ? 

Azure ARC is Azure's answer to enable customers to manage and deploy services in hybrid and/or multicloud environments. Azure ARC extends the Azure controlplane into any Virtual machine, selected Hypervisors and even other Cloud providers. 

I will use Azure ARC to extend the reach of Azure into my own datacenter, allowing me to operate and deploy workloads that I'm familiar with in the Azure Cloud, onto my own hardware.


### Target design 

This setup will use actual hardware, deployed onprem. I will be using my existing "datacenter cluster" (3 HPe mini desktop PC's and a Synology NAS, to serve as a iSCSI & NFS storage appliance). 
All devices are connected to a local LAN, the local lan offers communication between the physical servers, the iSCSI storage pool as served by the NAS and allows outbound communication towards the Internet and selected public Azure services that I will be using.
Important as ever, is DNS. I will be depending on a lot of (inter, intra) cluster communication, and don't want to be typing IP addresses nor keeping track in hostfiles ;-). In my case I already run a Pihole DNS instance that I will use for local DNS resolution.

![](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/overview_hw.png)

### Hardware Setup

As a hypervisor I chose to run Proxmox VE, it is available for free and offers a very rich featureset, including HA, FT, Hardware Pass-through, etc. 
I don't actually need most of these features for this deployment, buit it does serve other purposes next to this case. [Proxmox VE Website](https://www.proxmox.com/en/downloads)

On top of proxmox, as virtual machines, I use Ubuntu Server LTS 22.04 Its been around and is widely supported. For this deployment I need 3 instances. These instances will be deployed with the Kubernetes runtime.
In proxmox, I created 3 servers, I chose to run an UEFI BIOS (mark-out pre-enroll keys in the wizard), 2 cores, 12 Gb ram, 60Gib disk. 

![](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/proxmox_host.png)

### Kubernetes

For Kubernetes I chose to deploy K3s. K3s offers a light-weight kubernetes solution ideal for Edge scenario's. K3s is a certified Kubernetes solution by [Rancher](https://www.rancher.com/products/k3s) and is also free to use. 
I followed the basic step-by-step [to deploy k3s.](https://learn.microsoft.com/en-us/azure/iot-operations/deploy-iot-ops/howto-prepare-cluster?tabs=ubuntu#create-a-cluster) 

Followed by adding a 2nd and 3rd node to the cluster using :  `curl -sfL https://get.k3s.io | K3S_URL=https://myserver:6443 K3S_TOKEN=mynodetoken sh -`
The value to use for K3S_TOKEN is stored at /var/lib/rancher/k3s/server/node-token on your server node. Replace the `myserver:6443` with the actual server name (thanks DNS) 

Validate all nodes are up and running: `kubectl get nodes`
![](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/kubectl.png)

Success ! 

### Azure ARC

Next up, ARC enabling the thing. Again following a [step-by-step](https://learn.microsoft.com/en-us/azure/iot-operations/deploy-iot-ops/howto-prepare-cluster?tabs=ubuntu#arc-enable-your-cluster)

I did run into a (dns?) glitch with 22.04 LTS; whenever i use/add the `az extension` to the local Bash when IPv6 is enabled. Running any `az` command litterly takes ages. 
I chose to disable it on the 3 Ubuntu nodes for now, and the issue is resolved. Surely some DNS issue on my side, but a topic for another day. 

Do this by changing the line from `GRUB_CMDLINE_LINUX_DEFAULT="" ` to `GRUB_CMDLINE_LINUX_DEFAULT="ipv6.disable=1" ` in the /etc/default/grub file , followed by a `sudo update-grub`

after installing de Azure CLI onto my master (initial) K3s node, i ran the steps documented and verified succes by issuing : 

[](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/kubectl_arc.png)



#### Key features ( amongst others) 
- Prevent stolen tokens from being replayed with the compliant network check in Conditional Access.
- Apply universal tenant restrictions to prevent data exfiltration to other tenants or personal accounts including anonymous access.
- Enriched logs with network and device signals currently supported for SharePoint Online traffic.
- Protect user access to the public internet while leveraging Microsoft's cloud-delivered, identity-aware SWG solution.
- Enable web content filtering to regulate access to websites based on their content categories and domain names.
- Apply universal Conditional Access policies for all internet destinations, even if not federated with Microsoft Entra ID, through integration with Conditional 
  Access session controls.

i.e. an Identity (& Device) based upstream Secure Web Gateway as part of the Entra platform, leveraging CASB and Conditional Access components of that very suite, tunneling specified traffic over a SWG to its destination using a secured and conditioned path. 

### Entra Private Access - 
Microsoft Entra Private Access provides your users - whether in an office or working remotely - secured access to your private, corporate resources. Microsoft Entra Private Access builds on the capabilities of Microsoft Entra application proxy and extends access to any private resource, port, and protocol.

#### Key features
- Quick Access: Zero Trust based access to a range of IP addresses and/or FQDNs without requiring a legacy VPN.
- Per-app access for TCP apps (UDP support in development).
- Modernize legacy app authentication with deep Conditional Access integration.
- Provide a seamless end-user experience by acquiring network traffic from the desktop client and deploying side-by-side with your existing third-party SSE solutions.

i.e. Azure AD ( Entra) App Proxy on Steroids. This is a service of particular interest to me personally, as I love to phase out any form of (Client)VPN service wherever I see them. This is that very VPN killer service that promises to accomplish just that. 
It also leverages Entra's other services such as conditional access framework, MFA, RBAC, etc. Lets explore.

###
###

## Setup Entra Private Access

The Entra Private Access service consists of 3 main components:
  
1. A Connector - This is a (group of) server(s) that has a line of sight to the service one wants to expose - a web, rds, ssh, whatever service you want your client to be able to connect to. On these servers one installs the Connnector Service to reverse-connect into the Entra platform. 

2. An Entra Application registration, representing the service you'd like to expose, the conditional access policy and the user asignment. This is the " traditional" Entra Enterprise Application as we know it + Network Access properties.

3. A Client - Global Secure Access Client - installers available for Windows, Android, IOS and macOS - installed on the client device.

![Screenshot](https://github.com/verboompj/EntraGSA/blob/main/Pictures/private-access-diagram-quick-access3.png)

