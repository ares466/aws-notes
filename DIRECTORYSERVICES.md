# Directory Services

- [AWS-Managed Microsoft AD](#aws-managed-microsoft-ad)
- [AD Connector](#ad-connector)
- [Simple AD](#simple-ad)

## AWS-Managed Microsoft AD

AWS Directory Service lets you run Microsoft Active Directory (AD) as a managed service. Since it is running Microsoft AD, it supports all Microsoft AD features, such as:
- Managed using standard AD tools and processes
- Create and apply **group policy**
- Provide SSO to applications and services
- Supports schema extensions for Microsoft AD-aware applications (e.g., Sharepoint, SQL Server, Distributed File System (DFS)).
- Enable MFA by integrating with your **RADIUS-based MFA infrastructure**
- Supports one-way, two-way, and forest trust relationships with other directory services.

AWS Managed Microsoft AD is built using Microsoft Active Directory 2012 R2.

AWS-Managed Microsoft AD comes in two sizes:
- Standard - supports up to 30,000 objects
- Enterprise - supports up to 500,000 objects

AWS-Managed Microsoft AD is highly available by default, with domain controllers spanning across at least 2 AZs.

The AD is deployed in an AWS-managed VPC, but two ENIs are injected into two subnets in two AZs in the customer-managed VPC.

Since it is a managed service, it automatically comes with monitoring, snapshots, replication, and recovery.

Best choice for more than 5000 users and if you need trust relationships bewteen AWS and your on-prem directories.

Since the directory is hosted on AWS, users can continue to operate through a network link failure to any connected on-prem systems.

Workloads suited to AWS-Managed Microsoft AD:
- Requirement to establish trusts
- Requirement to use schema extensions
- Requirement for a fully-managed AD

## AD Connector

AD Connector is a directory gateway with which you can redirect directory requests to your on-premises Microsoft Active Directory without caching any information in the cloud.

AD Connector comes in two sizes, small and large. The size of the connector determines allocated compute. There are no enforced user or connection limits.

AD Connector endpoints (ENIs) are deployed into two subnets in different AZs within the customer-managed VPC. You can spread application loads across multiple AD Connectors to scale to your performance needs.

Instead of handling requests directly, AD Connector redirects requests to existing directory services on-prem. No directory data is stored in AWS. Therefore, a **working network connection is required for AD Connector to function**. If the connection (Direct Connect or VPN) between AWS and on-prem is not functioning, AD Connector service will not work.

Example workloads suited to AD Connector:
- Small proof of concepts
- Workloads in which legal or compliance requirements prevent storing AD data in the cloud

## Simple AD

Simple AD is a standalone managed directory that is powered by a Samba 4 Active Directory Compatible Server.

It is available in two sizes.
- Small - Supports up to 500 users (approximately 2,000 objects including users, groups, and computers)
- Large - Supports up to 5,000 users (approximately 20,000 objects including users, groups, and computers)

Simple AD provides a subset of the features offered by AWS Managed Microsoft AD, including:
- the ability to manage user accounts and group memberships
- create and apply group policies
- securely connect to Amazon EC2 instances
- provide Kerberos-based single sign-on (SSO)

Simple AD does not support:
- MFA
- trust relationships with other domains
- schema extensions

## Comparison

| | Simple AD | AWS-Managed Microsoft AD | AD Connector |
| --- | --- | --- | --- |
| Short Desc | Simple AD instance with basic features | Fully managed Microsoft AD on AWS | Connects to an existing on-prem directory |
| Type | Samba 4 | Microsoft AD | N/A |
| Full-fledged AD | Yes | No | Yes | 
| Supports trust relationships | No | Supports one-way, two-way, and forest trust relationships | No |
| Supports schema extensions for MS AD-aware applications | No | Yes | No |  

> Several AWS services support authentication and authorization via a directory service:
> - AWS Console
> - WorkSpaces
> - RDS
> - WorkDocs
> - QuickSight
> - WorkMail
> - Connect
> - Chime
