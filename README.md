# velocloud-exoscale-virtual_edge
## Deployment of VeloCloud virtual Edge on Exoscale
\#A1 Digital Deutschland GmbH, 2019<br>
\#sd-wan@a1.digital 
## Requirements 
Prequisites for deploying the Virtual VeloCloud Edge on Exoscale are as follows:
- "A1 Digital SD-WAN" incl. valid license
- active exoscale account
- user with administrator privileges on both platforms
- egoscale CLI tool (optional)

## Introduction


Due to configuration limitations on both sides (VeloCloud Virtual Edge and Exoscale) 
it is not possible to run a vanilla(unaltered) VeloCloud Virtual Edge in a standalone scenario on Exoscale.

The reason for this are the strict interface configuration settings on Exoscale.
An Exoscale instance always uses the first interface(eth0) as WAN interface.
All subsequent configurable interfaces are utilized as privat network interfaces 
and are not ment to be used as an internetbreakout/WAN-Exit point.

This leaves us with 2 options. 

Either we are lossing support for VeloCloud by changing VMWare's VeloCloud Virtual Edge-Image or we deploy a seperate minimal virtual machine with a firewall that remaps the network interfaces for us.

> DO NOT GO WITH THE FIRST OPTION UNLESS YOU REALLY KNOW WHAT YOU ARE DOING

This readme explains only the second option as it is suitable for most customers. So we are going to deploy 2 seperate Exoscale instances. A VeloCloud Virtual Edge instance and a "router" instance which remaps the interfaces for us.

But first let's have a look on the network interfaces mappings of both VeloCloud Virtual Edge and Exoscale.

The VeloCloud Virtual Edge template is using the following network interface mapping.

| Interface Name by OS | Interface Name by VeloCloud | Description |
|----------------------|-----------------------------|-------------|
| Eth0 | GE1 | Switched Port LAN |
| Eth1 | GE2 | Switched Port LAN |
| Eth2 | GE3 | WAN Port (DHCP by default) |
| Eth3 | GE4 | WAN Port (DHCP by default) |



The Exoscale instance (compute instance) is using the following network interface mapping scheme:

| Interface Name | Descrition |
|----------------|------------|
| Eth0 | Default Interface with Public IP, deployed by Exoscale |
| Eth1 | Private Network Interface with or w/o DHCP Service |
| Eth2 | Private Network Interface with or w/o DHCP Service |

As mentioned we can sadly set neither of these settings with a parameter or configfile thus we are using a seperate Exoscale instance and assign them the same "Private Network". The "router"  instance will NAT everything from its first network device(eth0) to the third network device(eth2), which is VeloClouds first WAN Interface (GE3).

      Router                                 VeloCloud Edge

![Router Edge](.img/0001.jpg)

## Setup
As first creating the router instance. You can use the web portal of Exoscale or otherwise using the command-line tool Egoscale. The installation sources and documentation for Egoscale can be found on the website https://exoscale.github.io/egoscale/.

Start by configuring private networks for your zone. From the portal, click `Compute` -> `Private Networks` -> `Zone` -> `Allocate`

![Private Networks](.img/0002.png)

For deployment, you need 2 Private Networks. In the egoscale CLI you can create the private networks with the following command.
```
[user@host ~ ]$ exo privnet list
[user@host ~ ]$ exo privnet create privNet1ForVirtualEdge1 -z de-fra-1 
[user@host ~ ]$ exo privnet create privNet2ForVirtualEdge1 -z de-fra-1 
```

Next, we recommend creating a firewall rule to protect the instances. For administration purposes, SSH access to the router is used. Restrict this access to the source networks you are using. The VeloCloud Hub site functionality requires a DNAT for the VCMP protocol (UDP 2426). Initially the protocols HTTPS (tcp 443), NTP (tcp / udp 123) and VCMP (udp 2426) are required. The firewall configuration in Exoscale is cross-zone.
`Compute` -> `Firewalling` -> `Add`

![Add Rule](.img/0003.png)

In the egoscale CLI you can create the firewall rules with the following command.
```
[user@host ~ ]$ exo firewall list
[user@host ~ ]$ exo firewall create VelocloudEdge
[user@host ~ ]$ exo firewall add VelocloudEdge -c “0.0.0.0/0”  -p udp -P 2426 -d “From any to Instance allow UDP 2426 (VCMP)”
[user@host ~ ]$ exo firewall add VelocloudEdge -c “17.17.17.17/32”  -p tcp -P 22 -d “From my Network to Instance allow ssh”
```

Now create the instance as following. `Compute` -> `Instances` -> `Add`

![Create Instance](.img/0004.png)

The following informations are required.

| Properties | Values |
|------------|--------|
| Template: | Linux CentOS 7.6 64-bit |
| Instance Type: | Micro |
| Disk: | 10GB |
| Keypair: | your ssh public key or default |
| Private Networks: | PrivNet2ForVirtualEdge |
| Security Groups: | VeloCloudEdge |
| User Data: |  Finally to create a instance please use the [Cloud-Init Script](cloud-init/router_default.yml) from the A1 Digital. Edit the parameters (<foo bar>) in the script to your desired values. |

In the egoscale CLI use as following.
```
[user@host ~ ]$ exo vm create “vvcr1-defra1” –service-offering micro --template 296422c9-c15d-4cca-b599-8d8a48334096 --security-group  VelocloudEdge -–privnet PrivNet2ForVirtualEdge1 --zone de-fra-1 --disk 10 –-cloud-init-file ~/router_default.yml 
```

Deployment requires about 3-5 minutes due to the updates to the operating system of the created instance. After that, the system is already available and ready for use.

Let's move on to creating the virtual edge. To do this, create a instance as following.
`Compute` -> `Instances` -> `Add`

![Create Instance](.img/0005.png)


The following informations are required.

| Properties | Values |
|------------|--------|
| Hostname | e.g. vvce1-defra1 |
| Template: | edge-VC_KVM_GUEST-x86_64-3.2.1-174-R321-20181018-GA-updatable-ext4 |
| Instance Type: | Medium |
| Disk: | 10GB |
| Keypair: | your ssh public key or default |
| Private Networks: | PrivNet1ForVirtualEdge, PrivNet2ForVirtualEdge |
| Security Groups: | Default |
| User Data: | not working |

At egoscale CLI use as following.
```
[user@host ~ ]$ exo vm create “vvce1-defra1” –service-offering medium --template cdb8d70f-84ef-4c29-8ab3-e58212a99253 --security-group default -–privnet PrivNet1ForVirtualEdge1,PrivNet2ForVirtualEdge1 --zone de-fra-1 --disk 10 
```

Finally you need access to the Velocloud Edge console. Select `Compute` -> `Instances`. Enter the name of the newly created edge, e.g. vvce1-defra1.

![List Instances](.img/0006.png)

Select the **Name** of the instance filter list and then select `Console`.

![Instance Detail](.img/0007.png)

Use console window to login as user *root* with password *velocloud*.

![Console Window Login](.img/0008.png)
 
Activate the Edge with the *activate.py* script.

![Edge Activation](.img/0009.png)

The Edge is now enabled and can be administered through VeloCloud Orchestrator. The Edge has only one usable LAN port (GE2). You can connect your server instances to the SD WAN via the created private network PrivNet1ForVirtualEdge1.

## Additional Notice
Please check all script entries of their correctness. The IDs used here in the guide are also carefully checked for accuracy. If you have any questions or comments about this guide, please contact sd-wan@a1.digital.


