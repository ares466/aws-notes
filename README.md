# Notes on AWS

## Notes

- [Amazon Cognito](./COGNITO.md)
- [Direct Connect (DX)](./DIRECTCONNECT.md)
- [Directory Services](./DIRECTORYSERVICES.md)
- [Global Accelerator](./GLOBALACCELERATOR.md)
- [Identity Federation](./IDENTITYFEDERATION.md)
- [Networking Foundations](./NETWORKING.md)
- [Quotas](./QUOTAS.md)
- [Resource Access Manager](./RESOURCEACCESSMANAGER.md)
- [Route53](./ROUTE53.md)
- [S3 - Cross Account Access](./S3_CROSSACCOUNTACCESS.md)
- [Transit Gateway](./TRANSITGATEWAY.md)
- [VPNs in AWS](./VPN.md)
- [WorkSpaces](./WORKSPACES.md)


## Resources

- [Cantrill](https://learn.cantrill.io/)
- [Cantrill Github](https://github.com/acantril)

## httpd_config.sh

Use the sh script in the EC2 UserData or through ssh to create and start an httpd server.

## ssmdoc-install-httpd.yaml

This is an example SSM document that can be used to install and start httpd on a fleet of EC2 instances.

## ec2-cfinit.yaml

CloudFormation template example of starting an EC2 server using AWS::CloudFormation:Init.
