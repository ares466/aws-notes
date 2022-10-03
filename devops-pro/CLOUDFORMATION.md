# CloudFormation

`CloudFormation templates` can be written in YAML or JSON.

Templates are used to create one or more CloudFormation `stacks`. The goal of the template is to create physical resources from the logical definition of the resource.

CloudFormation contains logical resources. In a CloudFormation template, you define what needs to be created, and the CloudFormation services determines how to create them.

When a CloudFormation template is deployed, the logical resources in the template are now tied to the physical resources that were created. Therefore, if the logical resources within a template are changed (deleted), the physical resources are changed (deleted) as well.

![CloudFormation Stack Behaviors](./static/images/cf_behaviors.png)

## Parameters

CloudFormation supports parameters sourced from SSM.

```yaml
Parameters:
  LatestAmiId:
    Description: "AMI for EC2"
    Type: 'AWS::SSM::Parameter::Value<AWS::EC2::Image::Id>'
    Default: '/aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2'
```

## Intrinsic Function

The `!GetAZs` returns a list of AZs in the current region. The function accepts an optional parameter to specify a different region.

The `!Select` function returns an object from a list.

*Caption (below): Using `GetAZs` and `Select` to select an AZ.*
```yaml
Instance:
    Type: 'AWS::EC2::Instance'
    Properties:
        AvailabilityZone: !Select [0, !GetAZs 'us-east-1']
```

The `!Split` function splits a single String into a list of Strings based on some delimiter.

The `!Join` function concatenates a list of individual Strings into one String.

The `!Base64` function base64-encodes plain text. This is especially useful when defining `UserData` within an EC2 instance.

The `!Sub` function substitutes variables into the input. The `!Sub` function cannot be used to reference itself.

```yaml
UserData:
    Fn::Base64: !Sub |
        #!/bin/bash -xe
        yum -y update
```

The `!Cidr` function returns an array of CIDR address blocks based on the `count` parameter.

## Mappings

CloudFormation templates support a `Mappings` section which allows for a key-value pair map that can be used to lookup values based on input.

```yaml
Mappings:
    RegionMap:
        us-east-1:
            HVM64: 'ami-0'
            HVMG2: 'ami-1'
        us-west-1:
            HVM64: 'ami-2'
            HVMG2: 'ami-3'
        eu-west-1:
            HVM64: 'ami-4'
            HVMG2: 'ami-5'
```
*Caption (above): Defines a `RegionMap` that defines a top level key corresponding to the AWS region. It also defines second-level keys based on the architecture set of the AMI.*

```yaml
!FindInMap ['RegionMap', !Ref 'AWS::Region', 'HVM64']
```
*Caption (above): The `FindInMap` intrinsic function can be used to get information out of the map.*

## Outputs

CloudFormation templates support the use of optional *Outputs*. Values can be declared in the Outputs section. By doing so, they become visible as outputs in the CLI and console UI. 

Outputs are also accessible from a parent stack when using Nested Stacks.

Outputs can be exported, allowing for cross-stack references.

```yaml
Outputs:
    WordPressUrl:
        Description: "Instance Web URL"
        Value: !Join ['', ['https://', !GetAtt Instance.DNSName]]
```

## Conditions

CloudFormation templates support the ability to change which resources are created, and how they're configured, based on conditions.

Conditions can be defined in the `Conditions` section.

Conditions are evaluated to *true* or *false*.

Conditions are processed before resources are created.

*Caption (below): By defining a parameter called `EnvType`, we can use it in the condition below.*
```yaml
Parameters:
    EnvType:
        Default: 'dev'
        Type: String
        AllowedValues:
            - 'dev'
            - 'prod'
```

*Caption (below): Define a condition called `IsProd` that evaluates to true if the `EnvType` parameter is `prod`.*
```yaml
Conditions:
    IsProd: !Equals
        - !Ref EnvType
        - 'prod'
```

*Caption (below): Use the `Condition` property of a resource to conditionally create the resource based on the `IsProd` condition.*
```yaml
Resources:
    Wordpress:
        Type: 'AWS::EC2::Instance'
        Properties:
            ImageId: 'ami-0'
    Wordpress2:
        Type: 'AWS::EC2::Instance'
        Condition: IsProd
        Properties:
            ImageId: 'ami-0'
```

## DependsOn

CloudFormation attempts to create resources concurrently to reduce the total deployment time of a stack.

Some resources are dependent on other resources so cannot be created concurrently.

CloudFormation tries to determine the creation order of resources, but the `DependsOn` property allows authors to explitly define dependencies.

*Caption (below): This template defines several resources. Some of the resources (e.g., InternetGatewayAttachment) have an explicit dependency on others (e.g., InternetGateway) by using the !Ref function. Others do not and may require the DependsOn property.* 
```yaml
Resources:
    VPC:
        Type: AWS::EC2::VPC
        Properties:
            CidrBlock: 10.16.0.0/16
    InternetGateway:
        Type: AWS::EC2::InternetGateway
    InternetGatewayAttachment:
        Type: AWS::EC2::VPCGatewayAttachment
        Properties:
            VpcId: !Ref VPC
            InternetGatewayId: !Ref InternetGateway
    # An elastic IP requires an IGW attached to VPC in order to work. Since there is no dependency between EIP and InternetGateway or InternetGatewayAttachment, this may fail without a DependsOn property.
    EIP:
        Type: AWS::EC2::EIP
        DependsOn: InternetGatewayAttachment
```

> [Exam Tip]
>
> One well known dependency that requires the `DependsOn` property within CloudFormation involves Elastic IPs. Elastic IP addresses require an IGW Gateway to be attached to the VPC or else they will fail. However, since this dependency is not explicit, success depends on the random order the CloudFormation service chooses. To avoid this trap, use `DependsOn` as seen in the example above.

## Wait Conditions

CloudFormation creates physical resources based on the logical resources within a CloudFormation template. Once a physical resource has completed creation, the status of the logical resources updates to `CREATE_COMPLETE`.

For some resources, like EC2 instances, the physical resource may be created and therefore the logical resources set to `CREATE_COMPLETE`, but the EC2 instance is not ready.

The CloudFormation `cf-signal` feature can be used to eliminate this issue. When using *cf-signal*, the CloudFormation stack will wait for the specified number of success signals before updating the status of a logical resource to *CREATE_COMPLETE*. 

If no success signals are received, the stack will fail after reaching the specified timeout value (12 hour max).

*cf-signal* also supports receiving failure signals. If the stack receives a failure signal, the stack sets the status of the logical resource to `FAILED`.

There are two logical resources within a template that support receiving signals:
- `CreationPolicy`
- `WaitCondition`

A *CreationPolicy* resource should be used for EC2 instances and auto scaling groups.

```yaml
AutoScalingGroup:
    Type: AWS::AutoScaling::AutoScalingGroup
    Properties:
        DesiredCapacity: 3
    CreationPolicy:
        ResourceSignal:
            Count: 3
            Timeout: PT15M

LaunchConfig:
    Type: AWS::AutoScaling::LaunchConfiguration
    Properties:
        UserData:
            Fn::Base64:
                !Sub |
                    #!/bin/bash -xe
                    yum update -y aws-cfn-bootstrap
                    ...
                    /opt/aws/bin/cfn-signal -e $? --stack ${AWS::StackName} --resource AutoScalingGroup --region ${AWS::Region}
```
*Caption (above): This auto scaling group relies on a CreationPolicy in order to succeed. Specifically, it requires three success signals before moving to a `CREATE_COMPLETE` status (one from each instance). If it does not receive three signals within 15 minutes, the resource and stack will fail.*

`WaitConditions` operate in a similar manner to a *CreationPolicy* in that it must wait for the specified number of success signals, or timeout if none are received.

*WaitConditions* can depend on other resources, and  resources can depend on the *WaitCondition* making it a versatile approach to implementing waits.

A *WaitCondition* relies on a logical resource called a `WaitHandle`. A *WaitHandle* generates a pre-signed URL for resource signals.

```yaml
WaitCondition:
    Type: AWS::CloudFormation::WaitCondition
    DependsOn: SomeOtherResource
    Properties:
        Handle: !Ref WaitHandle
        Timeout: 300
        Count: 1

WaitHandle:
    Type: AWS::CloudFormation::WaitConditionHandle
```

AWS resources can send back signals to the WaitHandle.

```json
{
    "Status": "SUCCESS",
    "Reason": "Something Important",
    "UniqueId": "ID1337",
    "Data": "Something happened!"
}
```

Using the `GetAtt` function and the `WaitHandle` logical resource, we can pull data from the signal.

```yaml
!GetAtt WaitCondition.Data
```
*Caption (above): The `GetAtt` can be used to pull data from the signal from the `WaitCondition`.*

## Cross Stack Architectures

Thus far, we've discuss CloudFormation stacks in the context of a single template. There are some limitations with a single template strategy:
- There is a limit of 500 resources within a template
- All resources within a single stack share the same lifecycle

For complex architectures, CloudFormation supports two approaches to a multi-stack strategy:
- Nested Stacks
- Cross-Stack References

**Nested Stacks** are stacks created as part of other stacks. You create nested stacks within another stack by using the `AWS::CloudFormation::Stack` resource.

Nested stacks start with a **root stack**, which defines the first `AWS::CloudFormation::Stack` resource. Any stack that creates a `AWS::CloudFormation::Stack` resource is also called a **parent stack**.

When the nested stack is created, the outputs are available to reference from within the parent stack.

*Caption (below): The `VpcStack` resource defines a new CloudFormation stack. Stack outputs from `VpcStack` (e.g., VpcId) can be passed into other nested stacks (e.g., AppStack).*
```yaml
Resources:
    VpcStack:
        Type: AWS::CloudFormation::Stack
        Properties:
            TemplateURL: https://someurl.com/vpctemplate.yaml
            Parameters:
                Param1: !Ref SomeParam1
                Param2: !Ref SomeParam2
            TimeoutInMinutes: 10
    AppStack:
        Type: AWS::CloudFormation::Stack
        Properties:
            TemplateURL: https://someurl.com/apptemplate.yaml
            Parameters:
                Param1: !Ref VpcStack.VpcId
```

Use cases:
- to make modular templates for code reuse
- when all resources are lifecycle linked
- when you need to overcome the 500 reource limit in a stack

> [Exam Tip]
>
> - It is advisable to manage all nested stacks from the root stack, though nested stacks can be deleted individually.
> - When you delete a root stack, it will delete all nested stacks as well.

**Cross-stack references** serves a similar objective as nested stacks, but works differently which means its better suited for some use cases.

Normally, outputs are not visible from other stacks unless the stacks are nested. Outputs can be **exported** which makes them visible from other stacks. Exports must have a unique name within the region.

`Fn::ImportValue` can be used within a template to use an exported value.

*Caption (below): A stack can export an output value so it is available to other stacks.*
```yaml
Outputs:
    SharedVpcId:
        Description: Shared Services VPC
        Value: !Ref VPC
        Export:
            Name: SharedVpcId
```

*Caption (below): The `!ImportValue` function can be used to reference the exported value.*
```yaml
!ImportValue SharedVpcId
```

Stacks can only reference values from within the same region. It cannot access values in other regions.

> [Exam Tip]
>
> *Nested Stacks* can be used to share templates. *Cross-stack references* can be used to share resources.

## Stack Sets

**AWS CloudFormation StackSets** extends the capability of stacks by enabling you to create, update, or delete stacks across multiple accounts and AWS Regions with a single operation.

StackSets are containers that are deployed within an **admin account**. These containers contain stack instances which reference stacks. Stack instances and stacks are within **target accounts**.

StackSets can deploy stacks using *self-managed* roles or *service-managed* roles.

After you've defined a stack set, you can create, update, or delete stacks in the target accounts and AWS Regions you specify. 

![StackSets](./static/images/cf_stacksets.png)

When you create, update, or delete stacks, you can also specify operation preferences. For example, include the order of Regions you want to perform the operation, the **failure tolerance threshold** before stack operations stop, and the number of accounts performing **stack operations concurrently**.

By default, StackSets will delete all stacks that it manages. You can configure it to retain stacks in certain accounts or regions, while deleting the rest.

## Deletion Policies

If you delete a logical resource from a template and deploy it, the physical resource will be deleted. This can cause data loss or a loss of service.

You can optionally define a **deletion policy** on each resource. The deletion policy supports three options:
- Delete (default)
- Retain
- Snapshot (supported for some AWS services such as EBS, ElastiCache, Neptune, RDS, Redshift)

The **Snapshot** deletion policy will create a snapshot of the resource before deleting it. The created snapshot will continue to exist even after the stack is deleted. It must be manually cleaned up if desired. Notably, EC2 is not a supported resource for the snapshot deletion policy.

Deletion policies only apply to delete operations, not replace operations.

## Stack Roles

Everything that happens within AWS require permissions via IAM roles, including creating resources in CloudFormation. 

By default, CloudFormation uses the permissions of the principal that initiated a stack operation. As a result, the principal initiating a stack operation must have the correct permissions to create, update, or delete the AWS resources contained in the stack.

Alternatively, CloudFormation can assume a role to gain the necessary permissions to create and manage AWS resources. **Stack Roles** allows users to implement role separation so that the identity creating the stack does not need resource permissions - only the `PassRole` permission. 

![CloudFormation Stack Roles](./static/images/cf_stackroles.png)

## Init

To initiate an EC2 instance after creation, the `UserData` property within the `AWS::EC2::Instance` resource can be used to perform procedural commands.

Alternatively, the **CloudFormation Init** feature allows you to boostrap an EC2 instance by defining a desired state in a `AWS::CloudFormation::Init` property within an `AWS::EC2::Instance` resource.

CloudFormation Init is idempotent.

The target EC2 instance must have the `cfn-init` helper script installed. This script is responsible for performing the operations defined in the `AWS::CloudFormation::Init` resource.

```yaml
EC2Instance:
    Type: AWS::EC2::Instance
    Metadata:
        AWS::CloudFormation::Init:
            configSets:
            install_cfn:
            software_install:
            configure_instance:
            install_wordpress:
            configure_wordpress:
    Properties:
        UserData:
            !Base64 !Sub |
                #!/bin/bash -xe
                yum -y update
                /opt/aws/bin/cfn-init -v --stacak ${AWS::StackId} --resource ECInstance --configsets wordpress_install --region ${AWS::Region}
                /opt/aws/bin/cfn-signal -e $? --stack ${AWS::StackId} --resource EC2Instance --region ${AWS::Region}
```
*Caption (above):*
- *In the `Metadata` property of the EC2 instance, we define **Config Keys** that should be preformed as part of the EC2 initialization process.*
- *In the `UserData` section of the EC2 instance, we must make a call to the `cfn-init` and `cfn-signal` scripts to start the initiation process and provide necessary info.*

Each `ConfigKey` in `AWS::CloudFormation::Init` contains several sections that perform different types of operations:

```yaml
AWS::CloudFormation::Init:
    configure_wordpress:
        packages: # packages to install
        groups: # local group management
        users: # local user management
        sources: # download and extract archives
        files: # files to create
        commands: # commands to execute
        services: # services to enable
```

The output logs of the UserData component can be found in `/var/log/cloud-init-output.log`, 
If using CloudFormation Init, there is also a `/var/log/cfn-init-cmd.log`, `var/log/cfn-init.log`.

These files are useful for diagnosing issues with the EC2 bootsrapping process.

## Cfn Hup

`cfn-init` is a helper tool on EC2 instances that is only run once as part of the bootstraping process. If you update `AWS::CloudFormation::Init` on a running instance, it is not rerun.

The `cfn-hup` helper is a daemon which can be installed to detect changes in resources metadata and run configurable actions based on those changes. When an update stack operation is performed with new metadata, the running EC2 instance is updated.

![CloudFormation Hup](./static/images/cf_hup.png)
*Caption (above): The `cfn-hup` utility on the EC2 service periodically checks the CF metadata provided by a CloudFormation deploy. When it detects a change, it updates the instance accordingly.*

## Change Sets

When updating a stack, the update can have one of three impacts on the AWS resource:
- No interruption (No impact to the resource)
- Some interruption (e.g., EC2 reboot)
- Replacement (Create a new copy of the resource which may result in data loss)

`Change Sets` allow you to preview changes to validate which operations will occur on the resource, and how it will impact the resource. If you wish to proceed, you can initiate the update operation from the change set to actually make the changes.

## Custom Resources

CloudFormation supports most AWS services, but not every service and not every feature of every service.

CloudFormation `Custom Resources` allow developers to create resources that are not yet supported, or in a configuration that is not supported by CloudFormation.

For example, custom resources will allow you to populate (or empty) an S3 bucket after creation.

Custom resources are backed by Lambda functions or an SQS Queue.

![CloudFormation](./static/images/cf_customresources.png)

*Caption (below): You can define a custom resource that calls a Lambda function and any number of parameters.*
```yaml
CopyS3Objects:
    Type: "Custom::S3Objects"
    Properties:
      ServiceToken: !GetAtt MyFunction.Arn # Determines where to send the create/update/delete request
      SourceBucket: "cl-randomstuffforlessons" # Param 1
      SourcePrefix: "customresource" # Param 2
      Bucket: !Ref animalpics # Param 3
```

When a CloudFormation operation is initiated, CloudFormation invokes the Lambda. It passes in a JSON object with contextual data.

*Caption (below): The `ResponseURL` is used by the custom resource target to register success or failure.
```json
{
    "RequestType": "Create",
    "ServiceToken": "arn:aws:lambda:us-east-1:801955473075:function:S3BUCKETCUSTOM-CopyS3ObjectsFunction-w5I8gQBSiIwR",
    "ResponseURL": "https://cloudformation-custom-resource-response-useast1.s3.amazonaws.com/arn%3Aaws%3Acloudformation%3Aus-east-1%3A801955473075%3Astack/S3BUCKETCUSTOM/20f18af0-0f7e-11ed-9097-0ec03a295bd3%7Ccopyanimalpics%7C90581a31-cb85-45b1-b89c-b5cae6bca709?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20220729T203732Z&X-Amz-SignedHeaders=host&X-Amz-Expires=7200&X-Amz-Credential=AKIA6L7Q4OWTTE4NZIXU%2F20220729%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Signature=1e2ec308d4c5de616345f97a16b16b8c1a74a6b813645e260735153422c91390",
    "StackId": "arn:aws:cloudformation:us-east-1:801955473075:stack/S3BUCKETCUSTOM/20f18af0-0f7e-11ed-9097-0ec03a295bd3",
    "RequestId": "90581a31-cb85-45b1-b89c-b5cae6bca709",
    "LogicalResourceId": "copyanimalpics",
    "ResourceType": "Custom::S3Objects",
    "ResourceProperties": {
        "ServiceToken": "arn:aws:lambda:us-east-1:801955473075:function:S3BUCKETCUSTOM-CopyS3ObjectsFunction-w5I8gQBSiIwR",
        "SourceBucket": "cl-randomstuffforlessons",
        "SourcePrefix": "customresource",
        "Bucket": "s3bucketcustom-animalpics-14m7mbmcw7zzv"
    }
}
```

From within the Lambda function, we must signal to CloudFormation whether the custom operation was completed successfully.

```python
import cfnresponse

def handler(event, context):
    
    # DO CUSTOM THINGS

    # result is either cfnresponse.FAILED or cfnresponse.SUCCESS
    cfnresponse.send(event, context, result, {})
```

## Stack Policy

The Stack Policy is the IAM style policy statement which governs which resources in the stack can be changed and who is authorized to change them.
