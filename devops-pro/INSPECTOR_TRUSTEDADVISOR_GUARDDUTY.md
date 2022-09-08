# Amazon GuardDuty

Amazon GuardDuty is a continous security monitoring service that uses AI/ML and threat intelligence feeds to analyze supported data sources in order to identify unexpected or unauthorized activity as it occurs.

When a `finding` is discovered, GuardDuty can be configured to notify via SNS or trigger an event-driven process via EventBridge. 

GuardDuty supports multiple accounts within an AWS Organization.

GuardDuty supports DNS logs, VPC Flow logs, CloudTrail logs.

![Amazon GuardDuty](../static/images/guardduty.png)

# Amazon Inspector

Amazon Inspector scans EC2 instances, instance OS, and containers for known vulnerabilities or deviations from best practices.

Inspector `assessments` are run periodically (e.g., 15 minutes, 1 hour, 1 day).

Inspector provides a `report of findings` ordered by priority.

Inspector supports a few types of assessments:
- `Network assessment` (agentless or with agent)
- `Host assessment` (agent)

An Inspector `rules package` determines which vulnerabilities and best practices should be analyzed.
- The `network reachability` package (runs with an agent or agentless) analyzes the end-to-end reachability of a system. The network reachability results in findings such as *RecognizedPortWithListener*, *RecognizedPortNoListener*, or *RecognizedPortNoAgent* (used when no agent is available to check listeners), *UnrecognizedPortWithListener*.
- The `CVE` package analyzes against known security vulnerabilities (agent required).
- The `CIS` (Center for Internet Security) Benchmarks package requires an agent.
- The `Security Best Practices` for Amazon Inspector requires an agent.

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