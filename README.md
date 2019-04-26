# velocloud-exoscale-virtual_edge
## Deployment of VeloCloud virtual Edge on Exoscale 
A1 Digital Deutschland GmbH, 2019<br>
sd-wan@a.digital 
## Base Case
Exoscale Instances having a ordered interface setup by default. The sequence isn't customizable. First interface is the public IF eth0, all privat networks whos configured by user, starts at eth1 up to bottom. Hypervisor on Exoscale is KVM.
The VeloCloud virtual Edge having a default interface map for KVM Hypervisor. The sequence isn't customizable. The two first interfaces was used for LAN IF (switched ports). The WAN IF (routed ports) are using eth2 and eth3. So you can't register a virtual Edge hwo has deployed on Exoscale.

### Interface Map
```
Hypervisor Instance       VeloCloud Edge
eth0 Public               eth0 - GE1 LAN
eth1 privnet1             eth1 - GE2 LAN
eth2 privnet2             eth2 - GE3 WAN 
eth3 privnet3             eth3 - GE4 WAN
``` 
## Solution
Create a simple Router (instance type micro) to hiding the Velocloud Edge.<br>
```
Hypervisor Instance       VeloCloud Edge        Router
eth0 Public               eth0 - GE1 LAN        
eth1 privnet1             eth1 - GE2 LAN
eth2 privnet2             eth2 - GE3 WAN        eth1 privnet2
eth3 privnet3             eth3 - GE4 WAN        eth2 privnet3
                                                eth0 Public
``` 
On your Edge you will lost the first LAN interface GE1 for using. Interface will never receive a public IP from Exoscale. It's a dead IF. GE 3 and GE 4 can now communicate over privnet2 and privnet3 via router to Internet. You can register this Edge and using it to reach your Exoscale instances behind privnet1. 
 
