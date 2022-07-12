# Networking

## Public and Private AWS Services

- **Public services** are accessed via public endpoints (e.g., S3).  
- **Private services** run in a private VPC and can only be accessed by devices that have been specifically allowed (e.g., EC2).

![AWS Public and Private Zones](./static/images/networking_privatepubliczone.png)

## DHCP

- **DHCP** stands for Dynamic Host Configuration Protocol. It is used to automatically configure network devices.
- The DHCP protocol starts with a layer 2 broadcast to request info from the DHCP server. At a minimum, the server responds with the IP address, subnet mask, and default gateway of the network.
- DHCP from within a VPC also contains:
  - DNS servers
  - Domain names
  - NTP server configuration
  - NetBios name servers & node type
- By default, VPCs use the Amazon provided DNS, but support custom DNS.
- **DHCP option sets** are used to control DHCP settings within a VPC.
- DHCP option sets are immutable.
- DHCP option sets can be assigned to zero or more VPCs. A VPC can only be assigned one option set.
- When a DHCP option set is assigned to a VPC, the association is immediate but the network changes require a DHCP Renew to occur (which may take some time).
- By default, AWS provides public and private DNS names.

`ip-private-ipv4-address.region.compute.internal`

- To use a custom domain, you must use custom DNS servers.

