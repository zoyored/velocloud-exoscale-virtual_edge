# velocloud-exoscale-virtual_edge
## Deployment of VeloCloud virtual Edge on Exoscale 
A1 Digital Deutschland GmbH, 2019 
sd-wan@a.digital 
### Base Case
Exoscale Instances having a ordered interface setup by default. The sequence isn't customizable. First interface is the public IF eth0, all privat networks whos configured by user, starts at eth1 up to bottom. Hypervisor on Exoscale is KVM.
The VeloCloud virtual Edge having a default interface map for KVM Hypervisor. The sequence isn't customizable. The two first interfaces was used for LAN IF (switched ports). The WAN IF (routed ports) are using eth2 and eth3. So you can't register a virtual Edge hwo has deployed on Exoscale.

### Hypervisor      VeloCloud Edge
```
eth0 Public     eth0 - GE1 LAN
eth1 Privat     eth1 - GE2 LAN
eth2 Privat     eth2 - GE3 WAN 
eth3 Privat     eth3 - GE4 WAN
``` 
