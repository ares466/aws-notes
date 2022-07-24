# Notes on AWS

## Advanced Identities and Federation
- [Cognito](./COGNITO.md)
- [Directory Services](./DIRECTORYSERVICES.md)
- [Identity Federation](./IDENTITYFEDERATION.md)
- [Quotas](./QUOTAS.md)
- [Resource Access Manager](./RESOURCEACCESSMANAGER.md)
- [WorkSpaces](./WORKSPACES.md)

## Networking and Hybrid
- [Networking Foundations](./NETWORKING.md)
- [Direct Connect (DX)](./DIRECTCONNECT.md)
- [Endpoints](./ENDPOINTS.md)
- [Global Accelerator](./GLOBALACCELERATOR.md)
- [Private Link](./PRIVATELINK.md)
- [Route53](./ROUTE53.md)
- [S3 - Cross Account Access](./S3_CROSSACCOUNTACCESS.md)
- [Transit Gateway](./TRANSITGATEWAY.md)
- [VPNs in AWS](./VPN.md)

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
