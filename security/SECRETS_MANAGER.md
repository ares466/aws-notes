# Secrets Manager

AWS Secrets Manager is designed to enable secure and easy access to secrets from applications. Integration with the service can be done through the AWS console, AWS CLI, or AWS SDKs.

Secrets in Secrets Manager are encrypted using KMS keys.

Secrets Manager supports `automatic secret rotation` by periodically invoking a Lambda function. It can also directly integration with some AWS services (e.g., RDS) to update authentication information after a rotation.

![AWS Secrets Manager](../static/images/secretsmanager.png]
