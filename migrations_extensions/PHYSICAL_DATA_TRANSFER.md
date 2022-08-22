# Physical Data Transfer

The `Snowball` services are designed to move large amounts of data in and out of AWS. When moving huge amounts of data, using a network to transfer data is not feasible.

All of these services involve loading the data onto a physical device (e.g., suitcase or truck), shipping it to an AWS site where AWS will load it into S3.

# Snowball

The Snowball Edge devices must be connected using `physical networking`.

Devices come in 50TB or 80TB capacity, but multiple devices can be ordered to support more data.

All data on a Snowball device is encrypted using KMS.

It is economical to use snowball when transferring data between 10TB and 10PB.

# Snowball Edge

`Snowball Edge` is similar to Snowball, but includes compute. Snowball Edge devices offer larger capacity and faster network connections.

There are a few versions of Snowball Edge:
- Storage Optimized (with EC2)
- Compute Optimized
- Compute Optimized (with GPU)

Snowball Edge is ideal for remote sites or where data processing on ingestion is required.

# Snowmobile

A `Snowmobile` is a portable data center within a shipping container on a truck.

Snowmobile is used for single data centers in which huge amounts of data (10 PB+) must be transferred into AWS. A single Snowmobile can support up to 100 PB of capacity.

Snowmobile is not economical for multi-site or sub-10PB use cases.
