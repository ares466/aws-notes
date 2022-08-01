# Trusted Advisor

AWS Trusted Advisor provides recommendations that help you follow AWS best practices. Trusted Advisor evaluates your account by using checks. These checks identify ways to optimize your AWS infrastructure, improve security and performance, reduce costs, and monitor service quotas.

AWS provides recommendations based on five categories:
- Cost Optimization
- Performance
- Security
- Fault Tolerance
- Service Limits

AWS Trusted Advisor is enabled at the account level and does not require any agent.

AWS Trusted Advisor offers a `free tier` that provides seven core checks. This tier comes with the *basic* and *developer* support plans.

Free tier checks:
- S3 bucket permissions (not objects)
- Security groups - specific ports unrestricted
- IAM use
- MFA on the root account
- EBS public snapshots
- RDS public snapshots
- 50 service limit checks

> [Exam Tip]
>
> Memorize the free tier checks.

AWS Trusted Advisor unlocks all functionality when enrolled in the *business* or *enterprise* support plans. Full functionality of AWS Trusted Advisor includes the seven core checks plus 115 additional checks.

In addition to the additional checks, business and enterprise support plan members have access to the AWS Support API to be able to pull out AWS Trusted Advisor findings or initiate a refresh via the API.

Using the premium version, you can implement CloudWatch Integration to react to findings discovered by AWS Trusted Advisor.