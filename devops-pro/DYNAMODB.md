# DynamoDB

DynamoDB is a NoSQL public DBaaS service that supports key/values & documents.

DynamoDB is regionally resilient and can be made globally resilient with DynamoDB Global Tables.

DynamoDB provides single-digit millisecond latency for first byte retrieval.

DynamoDB tables can use a **primary key** of a single partition key or a composite key which consists of a partition and sort key. Each item in the table must have a unique primary key.

Each item in the table must be 400KB or less. There are no limits on the number of items in a DynamoDB table.

Billing in DynamoDB is based on RCU, WCU, storage, and additional features. Customers can purchase reservations to reduce cost.

## Performance (Throughput)

DynamoDB supports two capacity modes: on-demand and provisioned.

The **On-demand** mode is used for unknown, unpredictable, or low admin workloads in which the service will be automatically scale throughput to match demand. On-demand mode is charged per million RCU/WCU.

The **provisioned** mode requires customers to set the number of RCU/WCUs on a table by table basis.

DynamoDB capacity (throughput) is measured in RCUs (read capacity units) and WCUs (write capacity units).
- RCU = up to 4KB per second
- WCU = up to 1KB per second

Each DynamoDB table has an RCU and WCU burst pool (300 seconds) that is used when additional RCU/WCUs are used.

## Operations

Data can be retrieved from a DynamoDB table using queries or scans.

The **query** operation accepts a single PK value and an optional SK value or range. The query operation will return zero or more items with the specified primary key. 

The RCUs consumed equals the size of all returned items. Conditional filtering or projects discard data *after* capacity is calculated.

The **scan** operation is the most flexible read operation, but also the least efficient. The scan operation reads every item in the table, consuming the capacity of *every item*. 

## Consistency

## Backups

DynamoDB offers on-demand and point-in-time recovery (PITR) backups.

**On-demand backups** create a full copy of the table and are retained until explicitly removed. On-demand backups can be encrypted using KMS, with or without indexes, and restored to the same or another region.

When **point-in-time recovery** is enabled on a DynamoDB table, the service continously records changes that can be replayed at any point (1 second interval) in the 35-day recovery window.

