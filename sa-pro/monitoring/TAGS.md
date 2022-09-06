# Cost Allocation Tags

Cost Allocation Tags must be enabled individually in each account, or for the AWS Organization from the primary account.

Cost Allocation Tags can be either be:
- AWS-generated
- User-defined

`AWS-generated` tags are defined and populated by AWS (e.g., `aws:createdBy`, `aws:cloudformation:stack-name`). These tags are added to resources *after* the cost allocation is enabled. It is not retroactive.

`User-defined` tags are supported as well (e.g., user:something).

Both AWS-generated and user-defined tags are visible in cost reports within 24 hours.