# STS

The Security Token Service generates temporary credentials (`sts:AssumeRole`) that can be used to access AWS resources.

STS tokens consist of several attributes:
- AccessKeyId: Unique ID of the credentials
- Expiration: The date and time of credential expiration.
- SecretAccessKey: Used to sign requests
- SessionToken: Unique token which must be passed with all requests.

STS tokens are temporary (expire) and do not belong to a specific identity.

In order to use the `sts:AssumeRole` call successfully, the identity must be in the trust policy of the role. STS then uses the permissions policy of the role to generate temporary credentials.

![STS](./static/images/sts.png)