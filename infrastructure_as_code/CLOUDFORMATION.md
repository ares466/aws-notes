# CloudFormation

`CloudFormation templates` can be written in YAML or JSON.

Templates are used to create one or more CloudFormation `stacks`. The goal of the template is to create physical resources from the logical definition of the resource.

CloudFormation contains logical resources. In a CloudFormation template, you define what needs to be created, and the CloudFormation services determines how to create them.

When a CloudFormation template is deployed, the logical resources in the template are now tied to the physical resources that were created. Therefore, if the logical resources within a template are changed (deleted), the physical resources are changed (deleted) as well.

![CloudFormation Stack Behaviors](../static/images/cf_behaviors.png)

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

