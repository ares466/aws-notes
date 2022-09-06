# IAM, Accounts, and Organizations

## ARNs

Amazon resource names uniquely identify resources within an AWS account.

ARNs have the following composition:

`arn:partition:service:region:account-id:resource-id`
`arn:partition:service:region:account-id:resource-type/resource-id`
`arn:partition:service:region:account-id:resource-type:resource-id`

## IAM Identities & Policies

**IAM identities** consist of IAM users, IAM groups, and IAM roles. 

**IAM policies** can be attached to IAM identities to grant permissions to AWS services.

An IAM policy consists of an optional *SID*, an *action*, an *effect*, a *resource*, and a set of *conditions*.

When evaluating an API request, IAM will collate all applicable policies (e.g., identity policies, resource policies, SCP policies, session policies) into a comprehensive effective policy. The effective policy is evaluated based on a set of ordered rules:
1. explicit DENY
2. Explicit ALLOW
3. Implicit DENY

IAM policies can be *managed* or *inline*. 
- Managed IAM policies can be reused and require less management overhead. 
    - **AWS managed policies** are defined by AWS and cannot be changed.
    - **Customer managed policies** are defined by customers and can be changed.
- **Inline IAM policies** are not reusable and therefore should only be used for special exception cases.

Limitations of IAM users:
- AWS only allows 5,000 IAM users per account.
- An IAM user can only be a member of 10 groups.

## IAM Groups

**IAM groups** are containers for IAM users. IAM groups do not have credentials. IAM groups reduce the administrative overhead of managing IAM users.

IAM groups can have managed or inline policies attached to grant permissions to all users within the group. When an IAM user is a member of more than one group, they are granted the composite of all policies that belong to those groups.

IAM groups do not support nesting (groups within groups).

> [Exam Tip]
>
> IAM groups are not true identities! As a result, IAM groups cannot be referenced as a principal within an IAM or resource policy.

## IAM Roles

IAM roles consist of two types of policies:
- **Trust policy** - defines what principals or services can assume the role.
- **Permissions policy** - defines the AWS permissions available to the principal or service that assumes the role.

## AWS Organization

An AWS Organization consists of one **management account** and one or more **member accounts**.

Accounts can be grouped into **organizational units** (OUs) underneath the **organizational root**.

*Caption (below): OUs are hierarchical.*
```
Organizational Root
-- Development OU
   -- Marketing OU
   -- Finance OU
-- Prod OU
   -- Marketing OU
   -- Finance OU
```

One of the primary use cases for using an AWS Organization is consolidated billing. All usage is pooled and billed through the **billing account** (management account).

> [Best Practice]
>
> Use a single AWS account in an AWS organization for all user logins or identity federation. From that account, users can *role switch* to other accounts to perform work.

## SCPs

Service control policies can be assigned to accounts or AWS Organization OUs to restrict which services and actions are available within the account.

SCPs can be written as an *ALLOW* list or *DENY* list.

*Caption (below): The default SCP does not restrict any services.*
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "*",
            "Resource": "*"
        }
    ]
}
```

*Caption (below): The following SCP will result in an account or OU that only has access to S3 and EC2. Actions performed on any other services will be denied.*
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:*",
                "ec2:*"
            ],
            "Resource": "*"
        }
    ]
}
```

SCPs do not grant permissions.

> [Exam Tip]
>
> The management account cannot be restricted by SCPs, but the root user within applicable accounts are restricted.