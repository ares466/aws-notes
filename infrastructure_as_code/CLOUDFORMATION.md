# CloudFormation

`CloudFormation templates` can be written in YAML or JSON.

Templates are used to create one or more CloudFormation `stacks`. The goal of the template is to create physical resources from the logical definition of the resource.

CloudFormation contains logical resources. In a CloudFormation template, you define what needs to be created, and the CloudFormation services determines how to create them.

When a CloudFormation template is deployed, the logical resources in the template are now tied to the physical resources that were created. Therefore, if the logical resources within a template are changed (deleted), the physical resources are changed (deleted) as well.

![CloudFormation Stack Behaviors](../static/images/cf_behaviors.png)

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