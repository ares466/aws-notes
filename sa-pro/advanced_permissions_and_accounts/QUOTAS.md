# Account & Region Quotas

Most AWS services have a default per-region quota. Others have account-level limits.

AWS Quotas can either be soft limits or hard limits.
- **Hard limit quotas** cannot be increased (e.g., Number of IAM users in an account).
- **Soft limit quotas** can be increased. Quotas can be increased via the *Service Quota* console or the Support console.

Review the [Service endpoints and quotas](https://docs.aws.amazon.com/general/latest/gr/aws-service-information.html).

The AWS Service Quota dashboard displays a personalized live overview of currently applied quotas with respect to your AWS account.

**Quota request templates** automatically request up to 10 quota increases for newly-created accounts in your organization.

**CloudWatch Alarms** can be used to monitor the used capacity of an applied quota within your AWS account.

Quotas can be viewed and managed via the AWS CLI:
- [list-service-quotas](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/service-quotas/list-service-quotas.html) (specific to the account)
- [list-aws-default-quotas](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/service-quotas/list-aws-default-service-quotas.html) (AWS defaults)
- [request-service-quota-increase](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/service-quotas/request-service-quota-increase.html)
