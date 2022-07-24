# Amazon Cognito

Cognito handles authentication, authorization, and user management for web or mobile apps.

There are two primary features of Cognito:
- [User Pools](#user-pools)
- [Identity Pools](#identity-pools)

[User and identity pools](#user-and-identity-pools-together) can be used together.

## User Pools

A user pool is a user directory in Amazon Cognito. 

With a user pool, your users can sign in to your web or mobile app through Amazon Cognito.

User Pools also support:
- Sign-up and sign-in services
- A built-in, customizable web UI to sign in users.
- Social sign-in with Facebook, Google, Amazon, Apple, as well as SAML identity providers from your user pool.
- User directory management and user profiles.
- Security features such as MFA

After successfully authenticating a user, Amazon Cognito issues JSON web tokens (JWT).

This JWT can be used to:
- Authorize access to your own APIs
- Authorize access through AWS API Gateway
- Exchange for AWS credentials using an *Identity Pool*

Whether your users sign in directly or through a third party, all members of the user pool have a directory profile that you can access through a Software Development Kit (SDK).

> With the exception of AWS API Gateway, AWS services do not support JWT authentication. To access an AWS service, you must swap the JWT for AWS credentials using an Identity Pool.

![Amazon Cognito User Pool](./static/images/cognito_userpool.png)

1. The user signs into the User Pool and receives a JWT.
2. The user can now use the JWT to call self-managed APIs, or services through AWS API Gateway.

## Identity Pools

Amazon Cognito identity pools (federated identities) enable you to create unique identities for your users and federate them with identity providers.

Amazon Cognito identity pools support the following identity providers:
- Amazon, Facebook, Google, Apple
- Amazon Cognito User Pools
- Open ID Connect Providers
- SAML identity providers
- Developer authenticated identities

With an identity pool, you can obtain temporary AWS credentials with permissions you define to directly access other AWS services or to access resources through Amazon API Gateway.

When Amazon Cognito receives a user request, the service will determine if the request is either authenticated or unauthenticated, determine which role is associated with that authentication type, and then use the policy attached to that role to respond to the request. 

![Amazon Cognito Identity Pool](./static/images/cognito_identitypool.png)

1. Users initiate authentication with an identity pool.
2. The user is redirected to the sign-in page of the configured identity provider.
3. After successful authentication, the user receives an idP-specific token.
4. The token is passed into a Cognito identity pool. Based on the characteristics of the user, Cognito assumes the predefined IAM role and returns temporary AWS credentials.
5. The temporary AWS credentials can then be used to access AWS services. I

The roles used to provide access to resources must have the proper trust policy confired.
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Federated": "cognito-identity.amazonaws.com"
            },
            "Action": "sts:AssumeRoleWithWebIdentity",
            "Condition": {
                "StringEquals": {
                    "cognito-identity.amazonaws.com:aud": "us-east-1:1cd55b65-2d71-441c-83c0-a5de48b83f77"
                },
                "ForAnyValue:StringLike": {
                    "cognito-identity.amazonaws.com:amr": "authenticated"
                }
            }
        }
    ]
}
```

## User and Identity Pools Together

Cognito user and identity pools can be used together to create a complete authentication and authorization solution that enables user management, user login, and authorization to AWS resources.

![Cognito User and Identity Pools together](./static/images/cognito_userandidentitypool.png)
1. Users obtain a JWT from a Cognito user pool.
2. The JWT is swapped for temporary AWS credentials.
3. The AWS credentials can be used to access AWS resources.