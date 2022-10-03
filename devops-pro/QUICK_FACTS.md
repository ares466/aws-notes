# Quick Facts

Lessons learned from practice questions or review.  

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

**Elastic Beanstalk**
- When an instance is launched, Elastic Beanstalk runs `preinit`, `appdeploy`, and `postinit` (in that order). On subsequent deployments to running instances, Elastic Beanstalk runs `appdeploy` hooks.
- Elastic Beanstalk uses the following precedence for environment variables. This implies that values sent via the API/CLI and saved configurations will take precedence over *.ebextensions* configurations.
    1. Settings applied directly to the environment (API)
    2. Saved configurations
    3. Configuration files (*.ebextensions*)
    4. Default values


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

**SAM**
- SAM templates can also create non-serverless infrastructure.