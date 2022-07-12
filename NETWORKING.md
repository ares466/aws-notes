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

## Route Tables

- The VPC Router is an implicit virtual router within a VPC.
- VPC Routers are always highly available across all AZs in the region. It is also scalable so no performance management is required.
- The VPC Router is responsible for routing traffic between subnets, traffic from external networks into the VPC, and traffic from the VPC to external networks.
- The VPC Router has an interface in every subnet (subnet + 1 addresses) e.g., The `10.16.48.1` address in the `10.16.48.0/20` network.
- VPC routing logic is controlled via Route Tables.
- Every VPC is created with a main route table (RT), which is the default route table for every subnet in the VPC.
- Custom route tables can be created and associated with subnets (which removes the main route table).
- Subnets are associated with a single route table only.
- Best practice: Do not add routes to the main route table. Instead, create a custom route table.
- Route tables attempts to pick the most specific route first.
- Route tables can be associated with gateways.

![VPC Route Table](./static/images/networking_routetable.png)
