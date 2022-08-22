# KMS

The Key Management Service is a regional and public service (accessible from the public zone). The service enables the creation, storage, and management of `symmetic` and `asymmetric` cryptography keys. It also supports functionality like `encryption` and `decryption`.

The KMS service is `FIPS 140-2 (L2)` compliant.

KMS `keys` are logical objects that are backed by physical key material. Keys also consist of metadata such as ID, date, policy, description, and state. Key material can be generated or imported.

KMS keys can be customer-owned (`CMKs`) or AWS-owned. CMKs are created and accessed explicitly by customers. AWS-managed keys are implicitly created and accessed by AWS services when encrypting and decrypting data.

KMS keys support `rotation` in which keys are replaced periodically. AWS-managed keys are automatically rotated about once per year. When rotated, AWS maintains the `backing keys` for backwards compatibility.

KMS keys can have `alias` - names by which they can be accessed.

Each KMS key supports a `key policy` that define the principals and permissions that are associated with the key.

```json
{
    "Id": "key-consolepolicy-3",
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Enable IAM User Permissions",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::1234567890:root"
            },
            "Action": "kms:*",
            "Resource": "*"
        },
        {
            "Sid": "Allow access for Key Administrators",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::1234567890:user/iamadmin"
            },
            "Action": [
                "kms:Create*",
                "kms:Describe*",
                "kms:Enable*",
                "kms:List*",
                "kms:Put*",
                "kms:Update*",
                "kms:Revoke*",
                "kms:Disable*",
                "kms:Get*",
                "kms:Delete*",
                "kms:TagResource",
                "kms:UntagResource",
                "kms:ScheduleKeyDeletion",
                "kms:CancelKeyDeletion"
            ],
            "Resource": "*"
        },
        {
            "Sid": "Allow use of the key",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::1234567890:user/iamadmin"
            },
            "Action": [
                "kms:Encrypt",
                "kms:Decrypt",
                "kms:ReEncrypt*",
                "kms:GenerateDataKey*",
                "kms:DescribeKey"
            ],
            "Resource": "*"
        },
        {
            "Sid": "Allow attachment of persistent resources",
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::1234567890:user/iamadmin"
            },
            "Action": [
                "kms:CreateGrant",
                "kms:ListGrants",
                "kms:RevokeGrant"
            ],
            "Resource": "*",
            "Condition": {
                "Bool": {
                    "kms:GrantIsForAWSResource": "true"
                }
            }
        }
    ]
}
```

**Crytopraphy keys never leave the KMS service**. By default, KMS keys never leave the region in which they are generated (exception being multi-region keys).

![KMS Operations](../static/images/kms_operations.png)

KMS keys can be used for up to **4KB** of data. To encrypt more than 4KB of data, a `data encryption key` is required. The `GenerateDataKey` operation can be used to generate a data key from a CMK. When generated, KMS provides the data key as `plaintext` and `ciphertext`. The plaintext version is used to encrypt the data and then immediately discarded. The ciphertext version of the data key is stored with the encrypted data. This process is called `envelope encyrption`.

To decrypt data, the service must request that KMS decrypt the data key ciphertext using the CMK from which it was generated. With the plaintext, the encrypted data can be decrypted.