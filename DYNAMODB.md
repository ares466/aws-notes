# DynamoDB

## Frequently Updated Large Items

Due to the way WCUs work, large items are more expensive to update than smaller items. This situtaion is particularly cost-ineffective when the updates are small relative to the overall item size.

E.g., This item has a primary key, sort key, and three attributes with the following characteristics:
- Attr 1: Small, high velocity attribute
- Attr 2: Large, low velocity attribute
- Attr 3: Medium, low velocity attribute

The high velocity attribute (the attribute that changes often) is only about 10% of the total item size. Since DynamoDB writes the entire object on a `PutItem` or `UpdateItem` operation, the writer is charged for all 40KB of data.

![2022-11-09_16-39-38](https://user-images.githubusercontent.com/10566616/200947520-3284fca8-f2de-4658-b223-4ac5f60079f4.png)

One strategy to avoid this cost inefficiency is to use item collections. An item collection is the technique of splitting a single logical item into multiple physical items.

The high velocity items can be grouped into a single record and low velocity, but large, attributes are grouped into another record. Using this strategy, the cost of writes is limited to the total size of the small high velocity items.

![2022-11-09_16-44-37](https://user-images.githubusercontent.com/10566616/200948302-a5fab2ed-8bb8-4e8c-a0a1-3f8fc27e0687.png)
