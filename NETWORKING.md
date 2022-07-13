# Networking

## Public vs Private AWS Services

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

## Firewalls

- Every connection has two parts: (1) REQUEST and (2) RESPONSE.
> Generally, TCP communication looks like:
> 1. The client picks a temporary (ephemeral) source port 1024-65535 (depends on OS).
> 2. [REQUEST] The client initiates a connection to the server on a well-known destination port (e.g., HTTPS tcp/443).
> 3. [RESPONSE] The server responds using the source port of tcp/443 and the destination ephemeral port picked by the client.
> ![TCP](./static/images/networking_tcp.png)

- The **directionality** of TCP connection depends on the perspective of the sender or receiver. Traffic can either be *inbound* traffic or *outbound* traffic.
- A stateful firewall is intelligent enough to identify the request and response components as being related. As a result, you only need to explicitly allow traffic in one direction.

### NACLs

- **Network ACLs** are *stateless* firewalls that operate at the subnet level. NACLs will filter traffic crossing the subnet boundary, but will not evaluate traffic within the subnet.
- NACLs consist of *INBOUND* and *OUTBOUND* rules. INBOUND rules match traffic entering the subnet and OUTBOUND rules match traffic leaving the subnet.
- NACLs support explicit ALLOW and explicit DENY actions.
- NACL rules are processed in order based on the rule number in ascending order (lowest number first). Once a match occurs, processing stops.
- Since NACLs are stateless, you must allow traffic in both directions (INBOUND and OUTBOUND).
- VPCS are created with a default NACL with two *INBOUND* rules and two *OUTBOUND* rules. The result is that all traffic is ALLOWED (the NACL has no effect).

![Default NACL](./static/images/networking_nacldefault.png)

- When you create a NACL manually, it is created with a single `DENY ALL` rule. NACLs are not automatically associated with VPCs.
- NACLs can be used to block traffic based on IPs/CIDR, port, and protocol. They cannot be used to block logical resources.
- Each subnet can only be associated to a single NACL, but a NACL can be associated wtih multiple subnets.

### Security Groups

- **Security groups** are *stateful* firewalls that detect response traffic automatically.
- Security groups support an explicit ALLOW or an implicit DENY (not explicitly allowing traffic), but do not support explicit DENY. As a result, you cannot block specific bad actors.
- Security groups support IP/CIDRs and logical resources, including other security groups.
- Security groups are attached to ENIs (not the instance).

> BEST PRACTICE: In workload will well defined traffic patterns, reference security groups within the rules rather than an IP CIDR. Security groups are also able to reference themselves.

## Border Gateway Protocol (BGP)

- BGP consists of a number of networks called autonomous systems (AS) that is controlled by a single router (called the BGP router).
- Each AS is assigned an ASN (Autonomous System Number) by IANA. ASNs range from 0 - 65,535. 
  - 0 - 64511 are public ASNs that must be assigned by IANA to be used.
  - 64512 - 65534 are private and can be used within private peering arrangements that can be used without official assignment.
- BGP operates on port `tcp/179`.
- BGP connections are not automatic - peering must be manually configured.
- BGP is a *path-vector* protocol which means it exchanges the best path to a destination between peers. The path is called the **ASPATH** (Autonomous System Path).
- iBGP (internal BGP) focuses on routing within an AS. eBGP (external BGP) focuses on routing among different AS.
- BGP advertises the shortest ASPATH between peers, regardless of the latency characteristics of the path. **ASPATH prepending** can be used to artificially make a path seem longer. With ASPATH prepending, you can configure BGP routers to advertise certain paths.

![BGP Example](./static/images/networking_bgp.png)

