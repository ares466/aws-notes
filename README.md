# Notes on AWS

## Advanced Permissions and Accounts
- [Quotas](./advanced_permissions_and_accounts/QUOTAS.md)
- [Resource Access Manager](./RESOURCEACCESSMANAGER.md)

## Advanced Identities and Federation
- [Cognito](./advanced_identities_federation/COGNITO.md)
- [Directory Services](./advanced_identities_federation/DIRECTORYSERVICES.md)
- [Identity Federation](./advanced_identities_federation/IDENTITYFEDERATION.md)
- [WorkSpaces](./advanced_identities_federation/WORKSPACES.md)

## Networking and Hybrid
- [Networking Foundations](./networking/NETWORKING.md)
- [Direct Connect (DX)](./networking/DIRECTCONNECT.md)
- [Endpoints](./networking/ENDPOINTS.md)
- [Global Accelerator](./networking/GLOBALACCELERATOR.md)
- [Private Link](./networking/PRIVATELINK.md)
- [Route53](./networking/ROUTE53.md)
- [S3 - Cross Account Access](./networking/S3_CROSSACCOUNTACCESS.md)
- [Transit Gateway](./networking/TRANSITGATEWAY.md)
- [VPNs in AWS](./networking/VPN.md)

## Storage Services

## Resources

- [Cantrill](https://learn.cantrill.io/)
- [Cantrill Github](https://github.com/acantril)

## httpd_config.sh

Use the sh script in the EC2 UserData or through ssh to create and start an httpd server.

## ssmdoc-install-httpd.yaml

This is an example SSM document that can be used to install and start httpd on a fleet of EC2 instances.

## ec2-cfinit.yaml

CloudFormation template example of starting an EC2 server using AWS::CloudFormation:Init.
