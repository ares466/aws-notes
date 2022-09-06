# IAM, Accounts, and Organizations

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

## ARNs

Amazon resource names uniquely identify resources within an AWS account.

ARNs have the following composition:

`arn:partition:service:region:account-id:resource-id`
`arn:partition:service:region:account-id:resource-type/resource-id`
`arn:partition:service:region:account-id:resource-type:resource-id`

