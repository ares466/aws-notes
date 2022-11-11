# DynamoDB

## Architecture

All DynamoDB tables are deployed across three AZs within a region for high availability. DynamoDB requests first arrive at a cluster of *request routers* that determine the shard on which the data resides.

Strongly consistent reads are always handled by the primary storage nodes. Replication to secondary storage nodes is usually completed within single milliseconds, but may take up to a second.

## GSI

When a DynamoDB global secondary index’s write throttles are sufficient enough to create throttled requests, the behavior is called **GSI back pressure**.

Throttled requests result in `ProvisionedThroughputExceededException` errors in the AWS SDKs, generate `ThrottledRequests` metrics in CloudWatch, and appear as ’throttled write requests’ on the base table in the AWS console.

When GSI back pressure occurs, all writes to the DynamoDB table are rejected until space in the buffer between the DynamoDB base table and GSI opens up, regardless of whether the new item is eligible for the back-pressured GSI.

## Best Practices

### High-Velocity Attributes

Due to the way WCUs work, large items are more expensive to update than smaller items. This situtaion is particularly cost-ineffective when the updates are small relative to the overall item size.

E.g., This item has a primary key, sort key, and three attributes with the following characteristics:
- Attr 1: Small, high-velocity attribute
- Attr 2: Large, low-velocity attribute
- Attr 3: Medium, low-velocity attribute

The high-velocity attribute (the attribute that changes often) is only about 10% of the total item size. Since DynamoDB writes the entire object on a `PutItem` or `UpdateItem` operation, the writer is charged for all 40KB of data.

![2022-11-09_16-39-38](https://user-images.githubusercontent.com/10566616/200947520-3284fca8-f2de-4658-b223-4ac5f60079f4.png)

One strategy to avoid this cost inefficiency is to use item collections. An item collection is the technique of splitting a single logical item into multiple physical items. The attributes on the item are sharded into multiple items. 

In this scenario, the high-velocity attributes are grouped into an item and large low-velocity attributes are grouped into another item. Using this strategy, the cost of writes is constrainted to the total size of the high-velocity attributes.

Each of the items maintains the original primary key, but are suffixed with the shard id (e.g., A, B).

![2022-11-09_16-47-54](https://user-images.githubusercontent.com/10566616/200948874-c2e5b3f2-083b-46ed-bea4-445b28de144d.png)

### Key Decorators

Item keys can be decorated to improve readability.

Product Id: `pr-000001`  
Order Id: `or-324910`  

### Duplicate Attributes for Efficient Queries

Data within an item can be duplicated for efficient querying.

> E.g., The business would like to query orders data by quarter, month, or year.
>
> | PK | SK | OrderDate | Quarter | Month | Year |
> | --- | --- | --- | --- | --- | --- |
> | `or-000001` | `2022-11-01 12:31:00.000000` | `2022-11-01` | `Q4` | `11` | `2022` |
>
> The OrderDate, Quarter, Month, and Year represent different dimensions for the same year. Using GSIs that include these attributes allows for efficient querying.

### Overload GSI Key

The global secondary index key overloading design pattern is enabled by designating and reusing an attribute name (column header) across different item types and storing a value in that attribute depending on the context of the item type.

### Sparse GSIs

GSIs should be sparse to improve GSI scan performance, reduce cost, and improve efficiency.

### Use Item Versions for Concurrency Control

By defining and storing the version of an item as an attribute, developers can use conditional expressions to guarantee a change is being applied to the most recent version. This solution prevents unintended overwrites for a highly concurrent usecase.

### Parallel Scan

Even though DynamoDB distributes items across multiple physical partitions, a Scan operation can only read one partition at a time.

In order to maximize the utilization of table-level provisioning, use a parallel Scan to logically divide a table (or secondary index) into multiple logical segments, and use multiple application workers to scan these logical segments in parallel.

When a parallel scan is used, the application spawns three threads and each thread issues a Scan request, scans its designated segment, retrieving data 1 MB at a time, and returns the data to the main application thread.

### Logical Shards

If a scan is required on a large data set, logical shards can improve performance. Logically sharding the data involves saving an arbitrary shard id as an attribute on the item.

The number of shards can be determined based on the size of the data set. The shard id (e.g., 1-10) is randomly assigned to each record to ensure an even split.

*Caption: DynamoDB item that is assigned to logical shard #7*
| Key | Value |
| --- | --- |
| PK | request#662146 | 
| SK | shard#7 |

Creating a GSI with the shard attribute as the PK enables the application to run scans on each logical partition in parallel by using a KeyConditionExpression.

```python
ke = Key('GSI_1_PK').eq("shard#{}".format(shardid))
```

Doing the scan in parallel across multiple logical shards is much more efficient than doing a single scan.

## PartiQL

PartiQL is a SQL-like interface to DynamoDB. It will optimize query execution by running eligible query parts in parallel.
  - e.g., SELECT * FROM DemoTable07 (scan)
  - e.g., SELECT product_id, product_name, rating FROM DemoTable07
