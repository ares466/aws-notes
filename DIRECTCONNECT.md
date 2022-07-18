# Direct Connect

**Direct Connect (DX)** is a physical connection between a business premises, a DX location, and an AWS region.

DX allows for 1, 10, and 100 Gbps connections.

When you enable DX in AWS, you're simply reserving a port allocation at the DX location. It is the customers responsibility to physically connect to the DX location.

DX costs include hourly charges and outbound data transfer charges.

DX provides low and consistent latency. It also enables connection to AWS private services (VPCs) and AWS public services (e.g., S3, DynamoDB) without traversing the public internet.

![DX](./static/images/networking_dx.png)

DX locations are large regional data centers that are not owned by AWS, but rent space and equipment to AWS.

DX requires some specific configurations:
- auto-negotiation must be disabled
- port speed and full-duplex manually set
- Must support BGP and BGP MD5 authentication

Additionally, there are some cable requirements:
- 1 Gbps connection: 1000BASE-LX (1310 nm) transceiver
- 10 Gbps connection: 10GBASE-LR (1310 nm) transciever
- 100 Gbps connection: 100GBASE-LR4

## DX Connection Process

1. AWS customer lodges a DX request with AWS at some DX location.
2. AWS allocates a DX port on an AWS DX router at that DX location.
3. Customer downloads an LOA-CFA (Letter of Authorization - Customer Facility Access) form.
4. Customer lodges a request with the DX location data center owner to cross-connect the AWS and customer cages. The LOA-CFA form must be provided to the DX location owner.
5. DX location staff provision a cross-connect from AWS equipment to customer equipment.

> If you don't have space or equipment at a DX location, a comms partner can extend the DX port into your business premises.

## DX - BGP Session and VLANs

DX connections are layer 2 (data link) connections, but multiple types of layer 3 networks (e.g., VPCs, public zone) are required over a DX connection.

**VIFs (virtual interfaces)** allow multiple L3 networks to run over a L2 DX.

VLANs allow multiple isolated connections between the customer and AWS DX router.

VIFs consist of a:
- VLAN to isolate traffic
- BGP to exchange prefixes
- MD5 to authenticate BGP sessions

VIFs come in one of three types:
- [Public](#public-vifs)
- [Private](#private-vifs)
- Transit

Public VIFs are used to connect the public zone services.

A *dedicated* DX connection can have a total of 50 public and private VIFs, and one transit VIF.

A *hosted* DX connection can only have 1 VIF of any type.

### Public VIFs

Public VIFs are used to access AWS public zone services (e.g., Elastic IPs, S3, SNS, SQS). Private IP addresses cannot be accessed by public VIFs.

Public VIFs allow access to all public zone regions across the AWS global network (in contrast to private VIFs).

AWS advertises all AWS public IP ranges to the customer. The customer can advertise any public IPs owned over BGP. Advertised routes are not transitive; Customer-advertised prefixes do not leave AWS.

![Public VIF over DX](./static/images/dx_publicvif.png)

### Private VIFs

Private VIFs are created to access network resources within one AWS VPC via private IP addresses.

Private VIFs are attached to a *Virtual Private Gateway (VGW)* in the same region in which the DX location terminates.

Private VIFs can termninate at a VGW (default) or a *Direct Connect Gateway*.

There is no encryption on private VIFs, but apps can layer on encryption (e.g., HTTPS).

Private VIFs support standard (1500 MTU) or jumbo (9001 MTU) frames.

Route propagation is enabled by default in private VIFs when the VIF terminates at a VGW. AWS will advertise the VPC CIDR and the BGP Peer IPs and you can advertise specific corporate prefixes (100 max) or the default route (0.0.0.0/0).

![Private VIF over DX](./static/images/dx_privatevif.png)

> [Exam tip]   
>
> Private VIFs only support 100 advertised routes.

Key Learning Objectives:
- Private VIFs are used to access private AWS services (e.g., EC2) in VPCs via private IP addresses (NOT public IPs).  
- Private VIF => 1 VGW => 1 VPC  
- Private VIFs support IPv4 or IPv6.
- The customer configures an ASN on the VIF. The customer ASN can be publicly owned (granted by IANA) or private (64512 to 65535).
- An AWS ASN is configured on the VGW.

## DX Limitations
- DX uses physical cables so there is no inherent resilience based on a damage cable.
- DX only supports single-mode fibre (not copper wires).
- The time it takes AWS to allocate the port and the time it takes to physically connect your on-prem location to the DX location results in weeks to months of lead time.

## MACsec

By default, traffic flowing through a DX is unencrypted. Historically, the solution has been to run an end-to-end VPN over the DX connection, but speed suffers due to the heavy computational requirements of the VPN.

![DX Connection without MACsec](./static/images/networking_dxnomacsec.png)
*Caption: There are some security problems with a DX connection without MACsec.*  
*1. Plaintext data is flowing over a physical cable between AWS and a customer cage in the data center.*  
*2. Its common to use a 3rd party provider to establish the connection between the customer cage in the DX location and the customer premisis - another unknown party that must be trusted.*

DX with MACsec provides the confidentiality, data integrity, data origin authenticity, and replay protection lacking in DX without MACsec.

**MACsec** is a standard that extends the ethernet standard, allowing frames to be encrypted.

In order to use MACsec, two devices must be connected by switches or routers on the same L2 network (called *layer 2 adjacency*).

MACsec is designed to allow for super high speeds (e.g., terabit networks).

Components of MACsec:
- *Secure Channel* - unidirectional channels that are used to send data back and forth.
- *Secure Channel Identifiers (SCI)* - identifier for each channel.
- *Secure Associations* - communication over each channel takes place on transient sessions. Each secure channel generally only has one secure associations at a time.
- MACsec uses the *MACsec Key Agreement* protocol for key generation, discovery, and authentication.
- MACsec uses a *cipher suite* to determine how data is encrypted.

MACsec adds two 16 byte tags to the standard *802.1Q* ethernet frame:
- MACsec tag
- Integrity Check Value (ICV)

The original frame components are encapsulated, encrypted, and integrity protected.

![DX Connection with MACsec](./static/images/networking_macsecframe.png)

### MACsec vs VPN (IPSEC)

MACsec does not replace IPSEC over DX since it is not an end-to-end solution. 

| | MACsec | IPSEC |
| --- | --- | --- |
| Scope | MACsec provides protection for devices on the same L2 network (single hop). | IPSEC provides E2E protection. |
| Throughput | MACsec is very efficient and supports terabit-speed networks. | IPSEC requires cryptographic overhead which limits speed and throughput. |
| Support | Router must support MACsec standard. | Much wider vendor support since OpenVPN is ubiquitious. |
| OSI Layer | Layer 2 | Layer 3 |

## VPN over DX

A VPN provides end-to-end encryption and authentication over the public internet or a DX connection. When VPNs are run over DX, you get low and consistent latency. 

VPNs over DX use a public VIF and VGW/TGW public endpoints.

![VPN over DX](./static/images/dx_vpn.png)

Customers may choose to use a VPN over DX, or a VPN as a backup to the DX connection, or both.