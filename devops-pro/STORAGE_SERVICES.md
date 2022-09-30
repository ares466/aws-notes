# EBS

EBS (Elastic Block Storage) is a persistent managed block storage service from AWS. EC2 instances are able to mount the block device to create a file system on the device.

Storage is provisioned in on AZ (resilient within that AZ only).

EBS volumes can only be attached to one EC2 instance **within the same AZ** at at time, but can be detached and reattached to other instances.

EBS supports snapshots in S3 that be used to create a new volume (across AZs).

When provisioning an EBS instance, you must select an instance type. There are several instance types, each with different characteristics, to choose from:
- General Purpose SSD (GP2)
- General Purpose SSD (GP3)
- Provisioned IOPS (IO1/IO2/BlockExpress)
- Hard Disk Drive (HDD)-based

EBS is billed based on a GB/month, plus configurable extras.

<img src="./static/images/ebs_arch.png" alt="EBS - Architecture" width="400"/>

## GP2 

General Purpose Storage SSD (GP2) can be as small as 1GB, or as large as 16TB.

GP2 uses a credit system for instance performance.

One `IO credit` is 16KB. IOPS is one chunk of 16KB in one second. For instance, if you transfer a 160KB file in one second, that represents 10 IOPS, or 10 IOPS credits.

EBS volumes have an `IO Credit Bucket` that has a capacity of 5.4 million IO credits.

Every EBS volume has baseline performance based on its size.  The buckets fills at a rate equal to the baseline performance of the instance. At a minimum (regardless of size), the IO credit bucket fills 100 IO credits per second.

Beyond the mimumum 100 IO credits, the bucket fills with an additional 3 IO credits per second per GB of volume size.

GP2 can burst IO up to 3,000 IOPS by depleting the credit bucket. 

All volumes start with an initial 5.4 million IO credits.

Volumes larger than 1 TB (1,000 GBs) have a baseline above burst so the credit system is not used and baseline performance is always used.

Any volumes above about 5 TB achieve the maximum IOPS - 16,000 IO credits per second.

GP2 is the default EBS instance type. GP2 should be used for boot volumes, low-latency interactive apps, and for dev and test environments.

### GP3

The IO credit system is not used in GP3 instance types. Instead, it is replaced with something much simpler.

Every GP3 instance, regardless of size, starts with 3000 IOPs and 125 MiB/s performance.

If you need better performance, you can buy up to 16000 IOPS or 1000 MiB/s.

The base price of GP3 is about 20% cheaper than GP2.

GP3 also offers a higher maximum throughput than GP2:
- GP3: 1000 MiB/s
- GP2: 250 MiB/s

The use case of GP3 is very similar to GP2 including virtual desktops, medium sized single instance databases, low-latency interactive apps, dev and test environments, and boot volumes.

## I01/I02/BlockExpress

`Provisioned IOPS` volumes are the highest performance Elastic Block Store (EBS) storage volumes designed for your critical, IOPS-intensive and throughput-intensive workloads that require low latency.

IO2 volumes, available since 2020, are designed to deliver up to a maximum of 64,000 IOPS and 1,000 MB/s of throughput per volume, 4x more IOPS than general-purpose volumes.

Provisioned IOPS volumes allow you to configure IOPS for the volume independent of volume (though some limits are imposed).

Provisioned IOPS offer consistently low latency and jitter.

| Type | Instance Size | Max IOPS | Throughput | Max Performance per EC2 Instance (requires multiple volumes) |
| --- | --- | --- | --- | --- |
| IO1 | 4 GB to 16 TB | 50 IOPS/GB | 1000 MB/s | 260,000 IOPS & 7500 MB/s |
| IO2 | 4 GB to 16 TB | 500 IOPS/GB | 1000 MB/s | 160,000 IOPS & 4,750 MB/s |
| BlockExpress | 4 GB to 64 TB | 1000 IOPS/GB | 4000 MB/s | 260,000 IOPS & 7500 MB/s |

Use IO1/IO2/BlockExpress for high performance, latency sensitive, I/O intense workloads such as NoSQL or relational databases.

## HHD

There are two types of HDD-based instance types:
- ST1 (throughput optimized)
- SC1 (cold storage)

ST1 is optiimized for serially written data (not randomly accessed).

ST1 supports volume sizes from 125 GB to 16 TBs, a maximum of 500 IOPS with a throughput of 500 MB/s.

SC1 is designed for infrequently used data storage, not performance workloads.

SC1 supports volume sizes from 125 GB to 16 TB, a maximum of 250 IOPS, and a max throughput of 250 MB/s.

## Exam Tips

- If the exam question highlights cost efficiency as a requirement, choose HHD-ST1 or HHD-SC1. 
    - If the exam question mentions throughput or streaming, choose HHD-ST1 over HHD-SC1.
    - If the exam question mentions a boot drive, ST1 and SC1 *cannot* be used.
- GP2/3 can achieve up to 16,000 IOPS per volume.
- IO1/2 can achieve up to 64,000 IOPS per volume.
- IO-BLOCKEXPRESS can achieve up to 256,000 IOPS per volume.
- You can combine volumes in a RAID0 configuration for up to 260,000 IOPS. This is the max IOPS per EC2 instance.
- If you need more than 260,000 IOPS, you must use an ephemeral instance store.

# Instance Stores

Instance store volumes provide block storage devices to EC2.

Unlike EBS, instance stores are physically connected to only one EC2 host. Because they're physically connected, they offer the highest performance block storage in AWS.

Instance stores are included in the price of the EC2 instance.

Instance stores can only be connected at launch. They cannot be added afterward.

Instance stores are ephemeral. Any data on an instance store will be losts if the instance is moved, resized, or there is a hardware failure.

# Storage Gateway

Storage Gateway is a family of services that can be used to assist in cloud migrations, extending on-prem storage into the cloud, storage tiering, disaster recovery, and replacement of backup systems.

A Storage Gateway is typically run on an on-prem VM, but can also be run as a hardware appliance.

Storage gateway presents storage using `iSCSI`, `NFS` (Linux environments), or `SMB` (Windows environments) and writes data to EBS, S3, and Glacier within AWS.

Storage Gateway is made up of three types of gateways:
- [Storage Gateway - Volume Gateway](#volume-stored)
- [Storage Gateway - Tape Gateway](#tape-gateway)
- [Storage Gateway - File Gateway](#file-gateway)

## Volume Stored

On-prem servers typically use the `iSCSI` protocol for network block storage on a `Network Attached Storage` (NAS) device.

A `Volume Gateway` can be used to replace on-prem NAS/SAM devices. Volume gateway presents NAS/SAM compliant devices to be used by on-prem servers, exactly the same way they would for on-prem NAS servers.

Storage gateway (volume) runs in two modes:
- [Stored Mode](#stored-mode)
- cached mode

### Stored Mode

In stored mode, all data is stored locally. All data written locally is also written to the `upload buffer`. Once on the upload buffer, the data is asynchronously copied as `EBS snapshots` into S3 via the public `Storage Gateway Endpoint`.

Storage gateway volume stored mode should be used for:
- Full disk backups of servers
- Assist with disaster recovery by creating EBS volumes from the EBS snapshots

**Stored mode does not improve database capacity.** All data is stored locally!

![Volume Gateway - Stored Mode](./static/images/storagegateway_volumestored.png)

### Cached Mode

Volume gateway `cached mode` shares the same basic architecture as stored mode, but the primary location of the data is no longer on-prem. Instead, all data is stored in an AWS-managed part of `S3` (cannot be viewed from the console).

The only data that is stored locally (`cached`) is the frequently accessed data.

Cached mode can be used for `data-center extension`, in which all data is stored in S3, but accessed as if it were on-prem.

![Volume Gateway - Cached Mode](./static/images/storagegateway_volumecached.png)

### Stored vs Cached Mode

Stored and Cached mode of a Volume Gateway share some similarities.
- In both modes, a Storage Gateway is deployed on-prem as a VM or hardware appliance. The Gateway is a iSCSI device that provides `raw block storage` over a network.
- In both modes, an `upload buffer` is used to temporarily hold day until it is `asynchronously` copied to S3.
- In both modes, the data is copied via the public Storage Gateway `endpoint`.
- In both modes, data is stored as an `EBS snapshot`. New EBS volumes can be created from the snapshots in the event of a disaster.

Storage and Cached mode differ in other ways:

| Stored Mode | Cached Mode |
| --- | --- |
| All data is stored locally. Data is backed up to the cloud. | All data is stored in S3. Only frequently accessed data is cached locally. |
| Data is stored in a customer-managed S3 bucket. | Data is stored in an AWS-managed S3 bucket. |
| Does not increase the capacity of a data center. | Increases the capacity of a data center. |
|  All data is accessed quickly since it is stored locally. | Only cached data is accessed quickly. |

## Tape Gateway

Tape Gateway is also known as `VTL` (Virtual Tape Library). 

Organizations typically backup data periodically. Backups can be used to restore functionality or data in the event of a disaster or security incident.

There are several ways to accomplish this, but one way is to backup to tape (e.g., LTO-9). Organizations write to tapes offsite.

Tapes are inserted into a `tape drive` to be read from or written to. Many drives are stored in a `tape library` or a `tape shelf`. `Tape loaders` are robot arms used to swap tapes into and out of drives.

Servers communicate with the tape drive using `iSCSI`.

![Storage Gateway - VTL](./static/images/storagegateway_tapes.png)

A `VTL Gateway` is used to act as a tape backup device using `iSCSI` to accept backups, write them to S3, and archive them into Glacier.

Backup data is written and read using the public `storage gateway endpoint`.

Backup data is initially written to S3, which acts an AWS-hosted tape library (i.e.,VTL). Each virtual tape can be anywhere from 100 GiB to 5 TiB.

Tapes are archived into the tape shelf (VTS) within Glacier. When archived backups are required, they are restored to the VTL.

![VTL Gateway](./static/images/storagegateway_vtl.png)

VTL enables consumers to gain benefits from cloud storage, but still be compatible with their existing tape backup infrastructure and processes.

## File Gateway

In `file mode`, Storage Gateway bridges on-prem file storage and S3. 

Files are accessed via `mount points` (shares) via `EFS` (Linux) or `SMB` (Windows). Each mount point is associated with a single S3 bucket. The mount point association with an S3 bucket is called a `bucket share`. A single file gateway supports up to 10 bucket shares.

When files are written to the mount point, they are visibile as objects in the S3 bucket.

Storage Gateway in file mode supports `read caching` and `write caching` to ensure LAN-like performance.

S3 objects are given a name based on the directory within the file share.  

`/winkie.jpg` -> `s3://a4l-fileshare/winkie.jpg`  

*Caption (below): Diagram shows a simple example in which on-prem servers are backed up to S3.*
![Storage Gateway - File Gateway](./static/images/storagegateway_filemode.png)

## Multiple Shares

Storage Gateway in file mode can also be used for sharing data between multiple on-prem sites.

By default, when data is written to S3 from one file gateway, the other file gateway is not notified. The other file gateway must perform a `listing` command to view the most recent data. 

The `NotifyWhenUploaded` API can be used to notify other gateways when objects are changed.

File Gateway does not support `object locking` to prevent write collisions. Instead, organizations must implement a read-only mode on other shares or tightly control file access.

*Caption (below): This architecture shows multiple on-prem contributors to the same file gateway.*
![File Gateway - Multiple Contributors](./static/images/storagegateway_file_multiple.png)

File Gateway can be partnered with S3 replication to ensure backups to another region for Disaster Recovery (DR).

![File Gateway - Disaster Recovery](./static/images/storagegateway_file_dr.png)

To make file storage more cost effective, use lifecycle rules to transition data through different storage classes.

![File Gateway - S3 Lifecycle Rules](./static/images/storagegateway_s3classes.png)

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

![KMS Operations](./static/images/kms_operations.png)

KMS keys can be used for up to **4KB** of data. To encrypt more than 4KB of data, a `data encryption key` is required. The `GenerateDataKey` operation can be used to generate a data key from a CMK. When generated, KMS provides the data key as `plaintext` and `ciphertext`. The plaintext version is used to encrypt the data and then immediately discarded. The ciphertext version of the data key is stored with the encrypted data. This process is called `envelope encyrption`.

To decrypt data, the service must request that KMS decrypt the data key ciphertext using the CMK from which it was generated. With the plaintext, the encrypted data can be decrypted.

**E.g., Cryptography Operations**

*Generate data*:
```bash
echo "find all the doggos, distract them with the yumz" > battleplans.txt
```

*Encrypt*:
```bash
aws kms encrypt \
    --key-id alias/catrobot \
    --plaintext fileb://battleplans.txt \
    --output text \
    --query CiphertextBlob \
    | base64 --decode > not_battleplans.enc
```

*Decrypt*:
```bash
aws kms decrypt \
    --ciphertext-blob fileb://not_battleplans.enc \
    --output text \
    --query Plaintext | base64 --decode > decryptedplans.txt

```

# S3

## Encryption

Encryption at rest in S3 is configured at the object level. S3 supports both client-side encryption and server-side encryption.

When client-side encryption is used, the objects are encrypted before reaching S3. When server-side encryption is used, the plain-text object is encrypted by the S3 service.

There are three types of server-side encryption that can be used:
- Server-Side Encryption with Customer-Provided Keys ([SSE-C](#sse-c))
- Server-Side Encryption with Amazon S3-Managed Keys ([SSE-S3](#sse-s3))
- Server-Side Encryption with KMS Keys stored in AWS KMS (SSE-KMS)

The S3 `PutObject` operation supports the ability to specify the encryption type in a request header (e.g., *x-amz-server-side-encryption*). If this header is not specified, the objects are not encrypted.

The encryption type can be set as a default on the bucket. If no encryption type is set in the *PutObject* operation, the default type is used.

The default type is *not* enforced, but the encryption type can be enforced via the bucket policy.

### SSE-C

In SSE-C, the customer provides the key and Amazon S3 does the encryption. 

S3 generates a hash of the key and stores it with the object. Subsequent read requests for the object compare the hash of the key sent in the request with the hash stored with the object. If the hashes do not match, the request fails.

After encrypting (*PutObject*) or decrypting (*GetObject*) the object, S3 will discard the key.

Use SSE-C when you need to manage the keys yourself.

### SSE-S3

*x-amz-server-side-encryption*: AES256

When using SSE-S3, Amazon S3 manages encryption and keys.

The S3 service creates a common `root key` that is used to encrypt all data keys.

When an object is uploaded to S3, S3 generates a `data key` specific to that object. The data key is used to encrypt the object. 

The root key is used to encrypt the data key. The ciphertext of the dat key is stored with the S3 object. The plain-text data key is discarded.

This process is called `envelope encryption`.

![S3 SSE-S3](./static/images/s3_sses3.png)

Use SSE-S3 when you have the following requirements:
- Do not need to manage keys
- Do not need role separation (encryption and data are managed by same role)

### SSE-KMS

*x-amz-server-side-encryption*: aws:kms

In this encryption mode, S3 will handle the encryption and KMS/S3 will manage the keys.

When you upload an object with SSE-KMS for the first time, S3 will request a new key from KMS. This is the root key that will be used for S3 encryption.

Customers can specify their own KMS key, which means fine-grained controls and auditing are available on the key used to encrypt S3 data.

As a result, if you don't have access to the KMS key, you cannot read the data in S3.

## Storage Classes

### S3 - Standard

The default storage class, `S3 - Standard`, replicates objects across at least three AZs in the same AWS region to achieve eleven 9s of durability. S3 uses `Content-MD5 Checksums` and `Cyclic Redundancy Checks` (CRCs) to detect and fix any data corruption. Customers are billed a storage fee, a transfer out fee, and a per-request fee.
- The storage fee is charged by GB
- The transfer fee is charged by GB
- The request fee is charged per 1,000 requests

S3 - Standard has a milliseconds first byte latency.

### S3 - Infrequent Access (IA)

`S3 - IA` shares a lot of the same characteristics as S3 - standard including the same durability and replication across at least three AZs.

S3-IA is billable by a storage fee, transfer fee, request fee, and a `retrieval fee`.
- The retrieval fee is charged per GB of data retrieved

Standard-IA has a `minimum duration charge` of 30 days. Objects can be stored for less time, but the minimum billing always applies.

Standard-IA has a `minimum capacity charge` of 128KB per object.

Standard-IA should be used for important long-lived data that is infrequently accessed.

### S3 - One Zone-IA

`S3 - One Zone-IA` shares most of the characteristics as S3-IA, but the data is not replicated across AZs. If the AZ fails, the data is gone.

S3 - One Zone-IA should be used for non-critical long-lived data that is infrequently accessed or data that can easily be replaced (e.g., cache data).

### S3 - Glacier Instant

`S3 - Glacier Instance` is similar to S3 Standard-IA, but has cheaper storage at the cost of a more expensive retrieval and longer billable minimums.

S3 - Glacier Instant has a `minimum duration charge` of 90 days and a `minimum capacity charge` of 128KB per object.

S3 - Glacier Instant should be used for long-lived data that is only accessed once per quarter in which retrieval times must be fast.

### S3 - Glacier Flexible

The `S3 - Glacier Flexible` was renamed from `S3 - Glacier`.

Objects stored in the S3 - Glacier Flexible storage class cannot be publicly accessible.

Objects in Glacier Flexible are retrieved to S3 Standard-IA temporarily in order to be accessed.

This storage classes offers flexible retrieval times that have different costs and SLAs:
- Expedited: 1-5 minutes
- Standard: 3-5 hours
- Bulk: 5-12 hours

The first byte latency for this storage class is minutes to hours.

This low cost storage class should be used for archival data where access is very infrequent (e.g., once per year) and minutes to hours of retrieval time is sufficient.

> To remember Glacier - Flexible, think of objects as being kept in `chilled storage`. These objects must be *thawed* before they become available.

### S3 - Glacier Deep Archive

`S3 - Glacier Deep Archive` is the most cost-effictive storage class at the cost of a high minimum duration and longer retrieval times.

Data in Glacier Deep Archive is retrieved to S3 Standard-IA temporarily for access.
- Standard: 12 hours
- Bulk: 48 hours

The first byte latency for this storage classes is hours to days!

Use this storage class for data that is rarely, if ever, accessed and an hours to days retrieval time is acceptable (e.g., legal or regulated data storage).

### S3 - Intelligent Tiering

`S3 - Intelligent Tiering` is not a storage class, but a storage class manager that can move objects between five different storage tiers:
- Frequent Access
- Infrequent Access
- Archive Instant Access
- Archive Access
- Deep Archive

Intelligent tiering monitors the usage of objects and automatically moves any objects not accessed for 30 days to a low cost infrequent access tier and eventually to archive instant access, archive access, and deep archive tiers.

The archive access and deep archive tiers are optional.

Intelligent Tiering has a monitoring and automation cost per 1,000 objects.

S3 Intelligent Tiering should be used for long-lived data with changing or unknown patterns.

### Comparing Storage Classes

| PH | Retrieval Time | Minimum Duration Charge | Minimum Capacity Charge | Durability | Replicated AZs | Use Case |
| --- | --- | --- | --- | --- | --- | --- |
| S3 - Standard | Instant | None | None | Eleven 9s | At least 3 AZs | Default | 
| S3 - Infrequent Access (IA) | Instant | 30 Days | 128KB per object | Eleven 9s | At least 3 AZs | Irreplacable data that is infrequently accessed. |
| S3 - One Zone IA | Instant | 30 Days | 128KB per object | Eleven 9s (assuming no AZ failure) | No replication - the data only exists in one AZ | Replaceable data that is infrequently accessed |
| S3 - Glacier Instant | Instant | 90 Days | 128KB per object | Eleven 9s | At least 3 AZs | Long-lived data that is accessed once per quarter in which instant retrieval is required |
| S3 - Glacier Flexible | Minutes to hours | 90 Days | 40KB per object | Eleven 9s | At least 3 AZs | Archived data that is very infrequently accessed (e.g., yearly) and minutes to hour retrieval time is sufficient. |
| S3 - Glacier Deep Archive | Hours to Days | 180 Days | 40 KB per object | Eleven 9s | At least 3 AZs | Archival data that is rarely, if ever, accessed in which hours to days retrieval time is acceptable. |

## Object Versioning

Objects in S3 are immutable. Without versioning, each object is solely accessed via its key. If you write an object to an S3 bucket with the same key as an existing object, the existing object is overwritten and unrecoverable.

Versioning enables storage of multiple versions of objects within a bucket. Operations that would overwrite objects generate a new version.

In a version-enabled bucket, each object contains a key and an id. These two fields are unique within the bucket. The most recently written version of an object is known as the `latest` version or `current` version. When requesting objects, developers can specify an id to access older versions. If no id is specified, the most recent version is assumed.

In a version-enabled bucket, the delete operation creates a new version of the object called a **delete marker**. The delete marker has the effect of hiding all versions of that object. All versions are still available if accessed by a key and version id within a *GET* operation.

To truly delete a specific version of an object, you must specify the key and version id in the delete command.

Object versioning is a bucket-level operation. By default, versioning is disabled. Once it is enabled on a bucket, it cannot be disabled again, though it can be suspended.

The **MFA Delete** feature is enabled within the versioning configuration of a bucket. When MFA is enabled on a bucket, object versions cannot be deleted, and versioning configuration cannot be changed, without providing a OTP generated from an MFA device. 

## S3 Lifecycle Configuration

A `lifecycle configuration` is a set of rules that consist of actions on a bucket or group of objects.

Lifecycle rules support `transition` actions and `expiration` actions.

Lifecycle transitions only flow "down". Transitions cannot be used to move an object to a higher storage tier.

*Caption (below): This chart shows the possible transitions of S3 object lifecycles.*
![S3 Lifecycle Transitions](./static/images/s3_lifecycleconfig.png)
*Note how there are some restrictions on transitions:*
- *Transitions can only flow "down". Transitions cannot be used to move data to a higher storage class.*
- *S3-Intelligent Tiering cannot move data to S3-One Zone-IA.*
- *S3 One Zone-IA cannot move data to S3-Glacier Instance Retrieval.*

Exam Tips:
- There is a `minmum 30 day duration` before you can use a lifecycle rule to transition an object from S3-Standard to S3-Standard-IA or S3-OneZone-IA.
- Be aware that smaller objects can actually cost more when transitioned to S3-Standard-IA, S3-Intellint-Tiering, or S3-OneZone-IA due to the minimum billable size requirements (40KB per object)  of those storage classes.
- A single rule cannot transition to Standard-IA or OneZone-IA and THEN to Glacier classes within 30 days due to the duration requirement of the Standard-IA and OneZone-IA storage classes. To transition to Glacier classes before 30 days, use a separate rule.

## S3 Replication

There are two types of S3 replication:
- Cross Region Replication (CRR)
- Single Region Replication (SRR)

Replication requires an IAM role that will be assumed to perform the replication. The IAM role must allow S3 to assume it in the trust policy, allow read access to the source bucket, and write access to the destination bucket.

If data is being replicated across accounts, the destination bucket policy must explicitly allow write access to the replication IAM role..

Replication can be applied to the entire bucket or a subset of objects.

The default is to write the data to the same storage class as the source bucket, but replication can be configured to write to a lower storage class in the destination bucket.

By default, the object ownership will remain with the source account, but replication can be configured to set the owner to the destination account.

The `Replication Time Control` (RTC) feature can be applied to replication. This will guarantee that data will be replicated within 15 minutes.

Exam tips:
- Replication is `not retroactive`. When replication is enabled on a bucket, no existing objects will be replicated.
- Replication requires `bucket versioning` to be enabled.
- Replication is `one-way`. If you write an object to the destination bucket, it will not be replicated back to the source bucket.
- Replication supports unencrypted and server-side encrypted objects. If `SSE-KMS` is being used, extra configuration is required. Replication does not support `SSE-C`.
- Replication requires that the source bucket owner has permissions to the objects.
- Replication only supports user events. Replication does not support system events like lifecycle transitions. It also does not support replication of objects in the Glacier or Glacier Deep Archive storage classes.
- Replication does not support deletes.

S3 replication can be used for the following use cases:
- Log aggregation (SRR)
- Sync PROD and TEST (SRR)
- Data residency requirements (SRR)
- Global resiliency improvements (CRR)
- Reduce latency (CRR)

## S3 - Cross Account Access

There are three ways to grant access to S3 buckets across AWS accounts.
- [Bucket and Object ACLs](#bucket-and-object-acls)
- [Bucket Policies](#bucket-policies)
- [Assume Role](#assume-role)

### Bucket and Object ACLs

Bucket ACLs enable you to manage access to buckets and objects. Each bucket and object has an ACL attached to it as a subresource. It defines which AWS accounts or groups are granted access, and the type of access.

> Bucket ACLs are a legacy concept that is no longer recommended by AWS.

Bucket ACLs cannot be used to manage access within the same AWS account as the bucket.

![S3 Bucket ACL](./static/images/s3_bucketacl.png)

Bucket ACLs grant access based on Cononical Account IDs (a legacy account identifier). The *Account canonical user ID* can be found on the *Security Credentials* console.

By adding a target account's canonical user id to a bucket or object's ACL, you can grant access to that account.

### Bucket Policies

A *bucket policy* is a resource-based policy that you can use to grant permissions to your bucket and the objects in it.

Bucket Policy permissions do not apply to objects owned by other AWS accounts.

```json
{
  "Version": "2012-10-17",
  "Statement": [
      {
          "Effect": "Allow",
          "Principal": {
              "AWS": "arn:aws:iam::1234567890:user/iamadmin"
          },
          "Action": [
              "s3:GetObject",
              "s3:PutObject",
              "s3:PutObjectAcl",
              "s3:ListBucket"
          ],
          "Resource": [
              "arn:aws:s3:::<bucket-name>/*",
              "arn:aws:s3:::<bucket-name>"
          ]
      }
  ]
}
```
*Caption: Bucket policy granting S3 access to the iamadmin user in the 1234567890 account.*

### Assume Role

S3 across-account access can also be setup using IAM roles.

Generally, cross account access using assume role looks like:

- The bucket exists in account A. 
- An IAM role exists in account A that has permissions to the S3 bucket. This IAM role grants trust permissions to another AWS account B.  
- An identity within B can assume the role and operate on the S3 bucket.  

The IAM role must grant `sts:AssumeRole` permissions to an identity in the other AWS account, or delegate access via the account root.

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "AWS": "arn:aws:iam::1234567890:root"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```
*Caption: Trust policy that grants the `sts:AssumeRole` permissions to the 1234567890 account.*

### Object Ownership

When S3 ACLs are enabled, bucket owners can choose the preferred object ownership strategy.

![S3 Object Ownership Setting](./static/images/s3_objectownershipenabled.png).

- **Bucket owner preferred** - if new objects written to the bucket specify the bucket-owner-full-control canned ACL, they are owned by the bucket owner. Otherwise, they ware owned by the object writer.
- **Object writer** - The object writer remains the object owner.

If the assume role strategy is being used for object writers, then the objects will be owned by the bucket account.

### Policy Evaluation

All relevant policies are considered when evaluating access to a bucket or objects including:
- Organizational SCP
- Bucket Policy or ACLs
- IAM Permission Boundaries
- Session Policies
- Identity Policies

Given an *effective policy* (the product of all relevant policy statements), access evaluation will follow these steps:

| Step # | Condition | Result if condition satisfied |
| --- | --- | --- |
| 1 | Check for explicit deny | DENY |
| 2 | Check for explicit allow | ALLOW |
| 3 | Implicit Deny | DENY |

When the request is coming from another AWS account, an explicit ALLOW is required from the bucket account *and* from the other AWS account.

## S3 Presigned URLs

Object owners can share objects with others by creating a `presigned URL`, using their own security credentials, to grant time-limited permission to download the objects.

When you create a presigned URL for your object, you must provide your security credentials and then specify a bucket name, an object key, an HTTP method (GET to download the object), and an expiration date and time. The presigned URLs are valid only for the specified duration.

If you created a presigned URL using a temporary token, then the URL expires when the token expires, even if the URL was created with a later expiration time.

When you access an object using a presigned URL, you are accessing the object using the credentials that were used to generate the URL. 

The permissions of the credentials are validated at access time, not at generation time. This means that if permissions to view the object are revoked from the role that generated the presigned URL, any access of the object through presigned URL will no longer work.

Anyone can create a presigned-URL, even if they do not have access to the object. In this case, the bearer of the URL will not be able to access the object either.

*Caption (below): AWS CLI command for generating a presigned URL.*  

```bash
aws s3 presign s3://animal4lifemedia29384/cat_pic.jpeg --expires-in 180
```  

*Caption (below): Presigned URL for accessing an S3 object. Notice the `X-Amz-Security-Token` query parameter.*
```
https://animal4lifemedia29384.s3.us-east-1.amazonaws.com/cat_pic.jpeg?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=ASIA3VOCFM2Z7FTRJF76%2F20220730%2Fus-east-1%2Fs3%2Faws4_request&X-Amz-Date=20220730T124307Z&X-Amz-Expires=180&X-Amz-SignedHeaders=host&X-Amz-Security-Token=IQoJb3JpZ2luX2VjENX%2F%2F%2F%2F%2F%2F%2F%2F%2F%2FwEaCXVzLWVhc3QtMSJGMEQCIGQQlino4PpMef61n5ZvKRkBFMuvD1umFzLfr8rEuO%2BmAiBdUZICdEidrGupnZnMw5yww8pyBAKoFeMw1eWevxNBOSqMAwguEAAaDDgwMTk1NTQ3MzA3NSIMIrcDu0U%2BSvAiVJ8aKukC3%2Fjbdp1DBZtIifYNj%2BkmvCM4%2FCJDLkpbAP%2BedkXLgsNZ9nzJSDv1oWKEgp0c1O1lmWNTkF%2BfXGZKNl0Aw6kIHgrvBSWnkvgkJZ%2FVeZBJ9V%2F2KkvNXAIKfAN7CZLp6MyN1gauBfj4odaIPYQDSMxhtoW0BhqU8Ich%2BdQB%2F34YQQSH66O4cD5TCGK2IZMeUg8ToiDAJYms68vq4trLuPpTwZdeea7H4O2BmZ%2FjRDuoh004G11qa4dtzZPkpMjA6QlpmDC2hxbepfX9UQ4apPuFIx%2BRp6hFHqY9sVAuR1zOjKZx0a3CW0oYvfqzAptrV%2BnLWkqLmPfCW8DJMKXCkfRXpmIfL9iXsIwNuPUdq3KCahmTBAoZDjy0fciT41LiOscR%2Baa9V11%2B5g1wobOGS2VLSXhPU0rv%2FxdbGaBeQRLHMoUwrhmbLS9Z9jZD5vVmRiYUG302QM%2B40BfwlyWaS%2FNJjPcRe6ipu%2BhjATCczJSXBjq0AkiC4ZCmysfXZSZ5OwZ6MAYuNV%2FZ52k9n3JdxjelKuWtf%2BfyluS1WmHnNBncBqPLklKmgkuXE37H3WYXaA%2FZb4HkNxpON5Q8FRBAJaC5rUYSPTwgBpfzmqGjJfShj2zCFWZo5LkO3EbsAL6zd3bMxKP2Olh%2B5u8WLIpEYHikoQJe9xzGmBX4UAlwvEYpjqT3q0PbCQsf20KzuaMuI7XUFViqjhG4zJHCREmmX0ubYglw8jge%2FdMznlCmKxHPbjm%2FhU4zgQQOEzBU%2BwW2zLL00QjQqA8%2F1BijYhejfmqKJ6LS9cpYBXPDQJkdlJwhJRT%2BI0hHBP7RofjFrtD9IrvQsTsDLfdcYUoRun8OUO8jN9u9gEa2tvLkLDPwhC2RS5JymeDth%2BBUv0pwjNW4KeZYOAz%2Fslm4&X-Amz-Signature=db4c7fe8d9f85264ad03e0d8c7b7fc678d2ec32a518e1e9620253a60d1069aec
```

## S3 Select & Glacier Select

S3 can store objects up to 5 TB in size. Often, data consumers want to retrieve the entire object. For large objects, this takes a lot of time.

If you only need to access part of an object, downloading the object and filtering on the client side still incurs the performance and cost penalty.

S3 and Glacier Select allow you to use SQL-like statements to select part of the object filtered on the server. 

S3 Select supports CSV, JSON, Parquet, and BZIP2 compression for JSON and CSV.

![S3 Select](./static/images/s3_select.png)

## S3 Access Points

`S3 Access Points` simplify management of access to S3 buckets and objects.

Each Access Point has different policies that cantrol who can use the access point and which data is available from that access point. Access Point resource policies support network access controls (e.g., from a specific VPC).

Each S3 Access Point has its own endpoint address.

Access Points can be created via the console or using the AWS CLI:
```bash
`aws s3control create-access-point` --name secretcats --account-id 123456789012 --bucket catpics
```

The bucket policy must grant the same access as the access point, or the bucket policy can `delegate` access by allowing all actions through the access point.

![S3 Access Points](./static/images/s3_accesspoints.png)

## S3 Inventory

The S3 Inventory feature helps you manage storage within S3. The service inventories objects and various optional fields (e.g., encryption, size, last modified, storage class, version id, replication status, object lock).

Inventory is analyzed daily or weekly. It cannot be run on-demand.

The result of the inventory analysis is a CSV, ORC, or Parquet report.

Multiple inventories can be setup to write to a destination bucket within the same account or different account.

## S3 Object Lock

`Object Lock` implements a `WORM` (write once read many) security policy in which individual versions cannot be modified or deleted.

Object Lock requires bucket versioning to be enabled.

A bucket can have default Object Lock settings.

Object Lock can only be enabled on *newly-created* buckets. In order to add object lock to an existing bucket, you must contact AWS support.

There are two types of Object Locks:
- Retention Period
    - Compliance Mode
    - Governance Mode
- Legal Hold

### Retention Period Object Lock

With `S3 Object Lock Retention Period`, you specify the days and years when the retention period ends. 
There are two modes in Retention Period:
- In `Compliance` mode, the retention period cannot be adjusted, deleted, or overwritten (including the account root user).
- In `Governance` mode, special permissions can be granted that allow modifying retention locking sessings. 

Governance mode requires the `s3:BypassGovernanceRetention` permission on a principal in order to modify a retention lock. Likewise, the request must contain the `x-amz-bypass-governance-retention:true` header. This header is defaulted on the AWS console.

### Legal Hold Object Lock

In Object Lock `Legal Hold`, a legal hold flag is set to either true or false on an object version. You cannot specify any kind of retention period.

When a legal hold is enabled, the object version cannot be modified or deleted.

The `s3:PutObjectLegalHold` permission is required to add or remove a legal hold.

![S3 Object Lock](./static/images/s3_objectlock.png)

## CORS

When a browser or other client makes a connection to a server, it receives and displays an HTML page. That HTML page references other resources such as images and JS scripts. 

The host that delivers the HTML response is called the *origin*. Any requests from this page back to the origin (for images or JS scripts) is allowed by default.

In some cases, the HTML may need resources that are served from other *origins*. By default, cross-origin requests (requests made to a different domain than origin) are restricted, but can be allowed using **Cross-Origin Resource Sharing** (CORS).

CORS is a set of HTTP response headers that instruct the browser to allow cross-origin requests for data for specific origins, or all origins.

S3 static websites enable developers to provide CORS configuration so the HTML served by S3 can optionally request resources from other domains.

CORS configuration is defined in JSON via multiple rules that are processes in order. The first rule that matches the requests is effective.

*Caption (below): This CORS document instructs S3 to allow any PUT, POST, and DELETE requests from `anotherdomain.io`, and allow GET requests from any domain.*
```json
{
    {
        "AllowedHeaders": ["*"],
        "AllowedMethods": ["PUT", "POST", "DELETE"],
        "AllowedOrigins": ["http://anotherdomain.io"],
        "ExposedHeaders": []
    },
    {
        "AllowedHeaders": ["*"],
        "AllowedMethods": ["GET"],
        "AllowedOrigins": ["*"],
        "ExposedHeaders": []
    },  
}
```

There are two types of requests:
- Simple requests: The browser requests the content from the other domain immediately.
- Preflight requests: The browser first sends a **preflighted** request to the domain to ensure its allowed.

| Header | Description | 
| -- | -- |
| `Access-Control-Allow-Origin` | Defines which origins should be enabled to serve content |
| `Access-Control-Max-Age` | Defines how long content from other origins should be cached |
| `Access-Control-Allow-Methods` | Defines which HTTP methods can be used to interact with other domains |
| `Access-Control-Allow-Headers` | Defines which HTTP headers are allowed to be sent in cross-origin requests |

## Amazon Macie

`Amazon Macie` is a service that discovers, monitors, and protects sensitive data within an S3 data.

Macie will automatically discover sensitive data including PII, PHI, and finance data.

Macie discovers sensitive data in S3 buckets via **Discovery Jobs**. Discovery Jobs result in **findings** in Security Hub, or by publishing events to EventBridge.

Macie uses **data identiers** to discover sensitive data. There are two types of data identifiers:
- **Managed data identifiers** are pre-defined identifiers that use machine learning and patterns to identify common types of sensitive data.
- **Custom data identifiers** are regex-based propietary identifiers. 

Custom identifiers support optional `keywords` that must be in some `proximity` to the regex. The `maximum match distance` setting is used to determine how close keyworks must be to the regex pattern.

Custom identifiers also support `ignore words`. If the regex contains an *ignore word*, the finding is ignored.

Macie uses a multi-account structure using AWS Organizations or by explictly inviting accounts.

![Amazon Macie](./static/images/macie.png)

Macie will generate two types of findings:
- `Policy findings` are generated when settings are discovered that reduce the security of an S3 bucket (e.g., *Policy:IAMUser/S3BlockPublicAccessDisabled*, *Policy:IAMUser/S3BucketEncryptionDisabled*).
- `Sensitive data findings` are generated when sensitive data is discovered in an S3 bucket (e.g., *SensitiveData:S3Object/Credentials*, *SensitiveData:S3Object/CustomIdentifier*).

## S3 Event Notifications

When enabled, S3 can create notifications as a result of S3 actions. Notifications can be in the form of SNS, SQS, EventBridge events, or Lambda Functions.

S3 supports notification on object creation (Put, Post, Copy, CompleteMultiPartUpload), object deletion (Delete, DeleteMarkerCreated), object restoration from Glacier (Post, Completed), and Replication events (e.g., OperationMissedThreshold, OperationReplicationAfterThreshold).

## S3 Access Logs

When enabled, S3 will log all bucket or object access to another S3 bucket. Access logs are usually available within the destination bucket within a few hours. 

Log files consist of log records. Log records are newline-delimited. Attributes within a log are space-delimited.

# EFS (Elastic File System)

`EFS` is a `NFSv4-based` file system that can be mounted into Linux. EFS uses `POSIX` file permissions.

A single EFS instance can be mounted into many EC2 instances at the same time.

EFS is a private service that is accessible from `mount targets` within a VPC. An EFS instance can also be accessed via VPN or DX as well.

To ensure high availability, a mount target should be deployed in multiple AZs.

EFS supports general purpose and max i/o `performance modes`. The general purpose mode is default and is sufficient for 99.9% of users.

EFS supports bursting and provisioned `throughput modes`.

EFS supports standard and infrequent access (IA) `storage classes`. Lifecycle policies can be used to automatically move data between storage classes.

*Caption (below): Screenshot showing the various configuration options available when creating an EFS instances.*
![EFS modes](./static/images/efs_modes.png)

*Caption (below): EFS is accessible from an on-prem network via mount targets over a VPN or DX connection.*
![EFS](./static/images/efs.png)

# FSx for Windows

`FSx for Windows File Server` is a fully-managed native Windows file server/shares. It is designed for integration with Windows environments.

FSx for Windows integrates with Directory Service or self-managed AD. This allows companies to integrate their existing on-prem AD with FSx.

FSx for Windows can be deployed in single or multi-AZ mode within a VPC.

FSx for Windows supports on-demand and scheduled backups.

FSx for Windows supports encryption at rest with KMS keys and can force encryption in transit.

FSx for Windows shares are accessible from within a VPC, a peered VPC, a VPN, or DX connection.

FSx for Windows key features and benefits:
- `VSS` (user-driven restores) allows users to restore files by interacting with the file share directly, rather than through AWS.
- FSx for Windows is a native file system accessible over `SMB` (in contrast to EFS which uses the NFS protocol).
- FSx for Windows uses the Windows permission model.
- FSx for Windows supports DFS (distributed file system) which allows scalable Windows shares.
- FSx for Windows is fully-managed and does not require a file server admin.

*Caption (below): FSx for Windows is a Windows file server.*
![FSx for Windows](./static/images/fsx_windows.png)
- *FSx for Windows can integrate with AWS services such as WorkSpaces.*
- *FSx for Windows supports integration with an AWS Directory Service or an existing on-prem AD.*

# FSx for Lustre

`FSx for Lustre` is a high-performing file system designed for Linux clients. Lustre supports `POSIX` style permissions.

Lustre offers 100s GB/s throughput and sub millisecond latency.

Lustre is used for machine learning, big data, and financial modelling.

There are type deployment types in Lustre - persistent or scratch.

`Scratch` is a deployment type that is highly optimized for fast access in the short term (no replication or data durability). Use scratch when you need pure performance, but do not need data durability or high availability.

`Persistent` is a deployment type designed for the longer team and considers requirements such as high availability, self-healing, and data durability. Data in a persistent deployment type is within one AZ only.

Both the scratch and persistent deployment types support manual or scheduled backups to S3.

FSx for Lustre is accessible over VPN or Direct Connect.

FSx stores file metadata into metadata targets (`MDTs`). 

Objects are stored on object storage targets (`OSTs`). By default, OSTs are 1.17 TiB each. By splitting the object stores into OSTs, Lustre can achieve high throughput.

The baseline performance of an instance is defined by its size:
- Minimum of 1.2TiB incremented by 2.4TiB.
- Scratch deployments baseline throughput at 200 MB/s per TiB of storage.
- Persistent deployments offers baseline throughput at 50 MB/s, 100 MB/s, and 200 MB/s per TiB of storage.
- FSx can burst up to 1,300 MB/s per TiB using a credit system.

![FSx for Lustre](./static/images/fsx_lustre.png)

### FSx for Lustre with an S3 Repository

*Caption (below): FSx for Lustre can be configured to use S3 as a repository.*
![FSx for Lustre with S3 as a repository](./static/images/fsx_lustre_s3.png)

- *FSx can be configured to `lazy load` data from S3 into the file system as its needed. Data can also be exported back to S3 at any point using `hsm_archive`.*

## FSx for Windows vs FSx for Lustre vs EFS

| | FSx for Windows | FSx for Lustre | EFS | 
| --- | --- | --- | --- |
| Type | File Share | High-performance file share | File Share |
| Target OS | Windows | Linux | Linux |
| File Share Protocol | SMB | Lustre | NFS |
| Use case | Windows integration and permission model | HPC including machine learning | Linux integration with POSIX permission model |
