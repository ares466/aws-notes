# EBS

EBS (Elastic Block Storage) is a managed file storage service from AWS.

When provisioning an EB2 instance, you must select an instance type. There are several instance types, each with different characteristics, to choose from:
- General Purpose SSD (GP2)
- General Purpose SSD (GP3)
- Provisioned IOPS (IO1/IO2/BlockExpress)
- Hard Disk Drive (HDD)-based

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