# DynamoDB

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

