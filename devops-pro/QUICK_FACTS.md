# Quick Facts

Lessons learned from practice questions or review.  

**CloudFormation**
- CloudFormation supports the `cfn-init` and `cfn-hup` helper scripts to install and update software on EC2 instances. Helper scripts must be installed using the `yum install -y aws-cfn-bootstrap` command.
   - The `cfn-init` script runs on EC2 instance creation. It can fetch and parse metadata from CloudFormation, install packages, write files to disk, and enable/disable/start/stop services.
   - The `cfn-hup` script detects changes in resource metadata and runs user-specified actions when a change is detected. By default, updates are performed every 15 minutes, but this can be configured in the `cfn-hup.conf` file.

**CloudTrail**
- CloudTrail typically publishes events within 15 minutes of activity. There is no way to enable real-time logging.

**CloudWatch**
- As metrics age, they are aggregated into larger time granularities. After 15 months, all metrics expire.

| Granularity | Expiration |
| --- | --- |  
| One second or less | 3 hours |
| One minute or less | 15 days |
| 5 minutes or less | 63 days |
| One hour or less | 455 days (15 months) |

- CloudWatch Logs can be expored to S3 using the S3 Export feature (i.e., `Create-Export-Task`). Export tasks can take up to 12 hours and only support SSE-S3.

**CodeBuild**
- Builds can be configured using the `buildspec.yaml` file. By default, this file is in the root directory of the project, but the file name and location can be configured. `buildspec.yaml` can also be stored in an in-region S3 bucket.

The buildspec defines four main phases of a build:
- `install`: install packages into the build environment (e.g., frameworks)
- `pre_build`: Sign into services and install code dependencies
- `build`: commands are run during the build process
- `post_build`: create artifacts/packages, push docker images, explicit notifications

**CodeDeploy**
- CodeDeploy requires the CodeDeploy agent to be installed on the target server. CodeDeploy supports EC2, ECS, Lambda, and on-prem servers.
- CodeDeploy builds can be customized using the `appspec.yaml` configuraiton file.
- CodeDeploy supports lifecycle hooks for custom install, initialization, and validation operations via Lambda. Hooks can be added to the following phases of a deployment:
- `BeforeInstall`
- `AfterInstall`
- `AfterAllowTestTraffic`
- `BeforeAllowTraffic`
- `AfterAllowTraffic`

**EC2**
- EC2 instance metadata (available at *http://169.254.169.254/latest/meta-data*) contains information about when a spot instance will be terminated (*spot/termination_time*).

**ECS**
- Use an EventBridge rule to create notifications for stopped ECS tasks:
```json
{
   "source":[
      "aws.ecs"
   ],
   "detail-type":[
      "ECS Task State Change"
   ],
   "detail":{
      "lastStatus":[
         "STOPPED"
      ],
      "stoppedReason":[
         "Essential container in task exited"
      ]
   }
}
```

**ELB**
- ELBs add requests to the `SurgeQueue` when they are unable to establish a connection with a healthy instance. The SurgeQueue has a maximum size of 1024. The `SurgeQueueLength` CloudWatch metric represents the number of items on the queue. After the queue is full, requests will be rejected. The number of rejected requests is represented by the `SpillOverCount` CloudWatch metric.

**Elastic Beanstalk**
- When an instance is launched, Elastic Beanstalk runs `preinit`, `appdeploy`, and `postinit` (in that order). On subsequent deployments to running instances, Elastic Beanstalk runs `appdeploy` hooks.
- Elastic Beanstalk uses the following precedence for environment variables. This implies that values sent via the API/CLI and saved configurations will take precedence over *.ebextensions* configurations.
    1. Settings applied directly to the environment (API)
    2. Saved configurations
    3. Configuration files (*.ebextensions*)
    4. Default values

**GuardDuty**
- GuardDuty is an IDS service that monitors CloudTrail, VPC Flow Logs, and DNS query logs for anomolies.

**Lambda**
- Lambda code can be passed through CloudFormation via the `Code` attribute. The following types are supported:
    - `ImageUri`
    - `S3Bucket`, `S3Key`, `S3ObjectVersion`
    - `ZipFile`

**Networking**
- For high availability, NAT Gateways should be provisioned in each AZ within the region. The default route within each subnet should be pointed to the NAT Gateway.

**RDS**  
- RDS Automated Snapshots are taken once per day.

**S3**
- AWS recommends using bucket default encryption settings to enforce encryption at rest instead of bucket policies.
- Objects in the Glacier storage class are encrypted by default.

**SAM**
- SAM templates can also create non-serverless infrastructure.

**Service Quotas**
- Service Quotas is an AWS service that enables you to view and manage your quotas from a central location. Quotas, also referred to as limits, are the maximum value for your resources, actions, and items in your AWS account.
- Service Quotas can integrate with AWS Organizations, but trust must be explicitly enabled.
- When Service Quotas is associated with AWS Organizations, you can create a quota request template to automatically request quota increases when accounts are created.
