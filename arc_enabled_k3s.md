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

This setup will use actual hardware, deployed onprem. I will be using 3 HPe mini desktop PC's and a Synology NAS to serve as a storage appliance. 
All devices are connected to a local LAN, the local lan offers communication between the physical "servers", the iSCSI storage pool as served by the NAS and allows outbound communication towards the Internet and public Azure services I will be using.

As a hypervisor I chose to run Proxmox VE, it is available for free and offers a very rich featureset, including HA, FT, Hardware Pass-through, etc. 
I don't actually need most of these features for this deployment, buit it does serve other purposes next to this case. [Proxmox VE Website](https://www.proxmox.com/en/downloads)

On top of proxmox, as virtual machines, I use Ubuntu Server LTS 22.04 Its been around and is widely supported. For this deployment I need 3 instances. These instances will be deployed with the Kubernetes runtime.

In my case, I chose to deploy K3s as Kubernetes deployment. K3s offers a light-weight kubernetes solution ideal for Edge scenario's. K3s is a certified Kubernetes solution by [Rancher](https://www.rancher.com/products/k3s) and is also free to use. 
I followed the basic step-by-step [to deploy k3s:](https://learn.microsoft.com/en-us/azure/iot-operations/deploy-iot-ops/howto-prepare-cluster?tabs=ubuntu#create-a-cluster) 

followed by adding a seciond and 3rd node to the cluster using : 

'curl -sfL https://get.k3s.io | K3S_URL=https://myserver:6443 K3S_TOKEN=mynodetoken sh -'
The value to use for K3S_TOKEN is stored at /var/lib/rancher/k3s/server/node-token on your server node.



![](https://github.com/verboompj/arc_kubernetes/blob/main/pictures/overview_hw.png)


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

