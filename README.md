# AWS Certified Solutions Architect - Professional

## Advanced Permissions and Accounts
- [Quotas](./advanced_permissions_and_accounts/QUOTAS.md)
- [Resource Access Manager](./RESOURCEACCESSMANAGER.md)

## Advanced Identities and Federation
- [Cognito](./advanced_identities_federation/COGNITO.md)
- [Control Tower](./advanced_identities_federation/CONTROLTOWER.md)
- [Directory Services](./advanced_identities_federation/DIRECTORYSERVICES.md)
- [Identity Federation](./advanced_identities_federation/IDENTITYFEDERATION.md)
- [WorkSpaces](./advanced_identities_federation/WORKSPACES.md)

## Application Services, Containers, and Serverless
- [API Gateway](./appservices_containers/APIGATEWAY.md)
- [Containers](./appservices_containers/CONTAINERS.md)
- [EventBridge](./appservices_containers/EVENTBRIDGE.md)
- [GreenGrass](./appservices_containers/GREENGRASS.md)
- [IOT](./appservices_containers/IOT.md)
- [Lambda](./appservices_containers/LAMBDA.md)
- [Mechanical Turk](./appservices_containers/MECHANICALTURK.md)
- [MediaConvert](./appservices_containers/MEDIACONVERT.md)
- [MQ](./appservices_containers/MQ.md)
- [SAM](./appservices_containers/SAM.md)
- [SNS](./appservices_containers/SNS.md)
- [SQS](./appservices_containers/SQS.md)

## Compute, Scaling, & Load Balancing
- [EC2](./compute/EC2.md)
- [ELB](./compute/ELB.md)

## Data Analytics
- [AWS Batch](./data_analytics/BATCH.md)
- [EMR](./data_analytics/EMR.md)
- [Kinesis](./data_analytics/KINESIS.md)
- [QuickSight](./data_analytics/QUICKSIGHT.md)
- [Redshift](./data_analytics/REDSHIFT.md)

## Databases
- [Athena](./databases/ATHENA.md)
- [Aurora](./databases/AURORA.md)
- [DynamoDB](./databases/DYNAMODB.md)
- [Elasticsearch](./databases/ELASTICSEARCH.md)
- [Neptune](./databases/NEPTUNE.md)
- [QuantumDB](./databases/QUANTUMDB.md)
- [RDS](./databases/RDS.md)

## Disaster Recovery (DR)
- [Disaster Recovery](./dr/DISASTER_RECOVERY.md)

## Infrastructure as Code
- [CloudFormation](./infrastructure_as_code/CLOUDFORMATION.md)

## Migrations & Extensions
- [Cloud Migration](./migrations_extensions/CLOUDMIGRATION.md)
- [DataSync](./migrations_extensions/DATASYNC.md)
- [Storage Gateway](./migrations_extensions/STORAGEGATEWAY.md)

## Monitoring, Logging, and Cost Management
- [CloudTrail](./monitoring/CLOUDTRAIL.md)
- [CloudWatch](./monitoring/CLOUDWATCH.md)
- [Tags](./monitoring/TAGS.md)
- [Trusted Advisor](./monitoring/TRUSTEDADVISOR.md)
- [X-Ray](./monitoring/XRAY.md)

## Networking and Hybrid
- [Networking Foundations](./networking/NETWORKING.md)
- [Direct Connect (DX)](./networking/DIRECTCONNECT.md)
- [Endpoints](./networking/ENDPOINTS.md)
- [Global Accelerator](./networking/GLOBALACCELERATOR.md)
- [Private Link](./networking/PRIVATELINK.md)
- [Route53](./networking/ROUTE53.md)
- [Transit Gateway](./networking/TRANSITGATEWAY.md)
- [VPNs in AWS](./networking/VPN.md)

## Other
- [Miscellaneous Services](./other/OTHER.md)

## Security
- [ACM](./security/ACM.md)
- [CloudHSM](./security/CLOUDHSM.md)
- [Config](./security/CONFIG.md)
- [GuardDuty](./security/GUARDDUTY.md)
- [Inspector](./security/INSPECTOR.md)
- [KMS](./security/KMS.md)
- [Secrets Manager](./security/SECRETS_MANAGER.md)
- [SSM Parameter Store](./security/SSM_PARAMETERS.md)

## Storage Services
- [EFS](./storage_services/EBS.md)
- [File Shares](./storage_services/FILESHARES.md)
- [Instance Stores](./storage_services/INSTANCESTORE.md)
- [S3](./storage_services/S3.md)
- [Transfer Family](./storage_services/TRANSFERFAMILY.md)

## Resources

- [Cantrill](https://learn.cantrill.io/)
- [Cantrill Github](https://github.com/acantril)

## httpd_config.sh

Use the sh script in the EC2 UserData or through ssh to create and start an httpd server.

## ssmdoc-install-httpd.yaml

This is an example SSM document that can be used to install and start httpd on a fleet of EC2 instances.

## ec2-cfinit.yaml

CloudFormation template example of starting an EC2 server using AWS::CloudFormation:Init.
