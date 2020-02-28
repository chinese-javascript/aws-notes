# DynamoDB Overview
DynamoDB is a fully-managed serverless NoSQL DB service. 

A database consists of *tables*, containing a set of *items*, which have *attributes*. Primary keys in DynamoDB have 2 types: *Partition* and *Composite*.

* A **Partition (Hash) Key** is a single unique attribute. It is hashed and the result determines its physical location in storage.
* A **Composite (Range) Key** is comprised of a Partition Key and a secondary *Sort (Range) Key*. Used in the event where not all partition keys may be unique.

DynamoDB supports both the document and key-value pair models. Supported data types are:
* HTML
* XML
* JSON

# Working with Tables
## Operations
### GetItem
`GetItem` returns an item for the given primary key.

### Query
`Query` returns items in a table based on the specified value for only the Primary Key attribute. Use the `ScanIndexForward` parameter (bool) to sort the  results based on sort key (default ascending).

### Scan
`Scan` examines every item in a table. Additional filters can then be applied to refine the results. Less efficient than query operations.

## Expressions
### Projection Expressions
By default, retrieving items from a table returns all data attributes. Use the `ProjectionExpression` parameter to control which attributes to retrieve. Kinda like a `SELECT` clause in SQL.

```
aws dynamodb get-item 
    --table-name ProductCatalog 
    --key '{"Id":{"N":"1"}}' 
    --projection-expression "Description, RelatedItems[0], ProductReviews.FiveStar"
```

### Condition Expressions
Conditional writes succeed only if the item attributes meet the expected conditions. Applies to `PutItem`, `UpdateItem`, and `DeleteItem`.

Example `UpdateItem` operation using conditional writing:
```
aws dynamodb update-item \
    --table-name ProductCatalog \
    --key '{"Id":{"N":"1"}}' \
    --update-expression "SET Price = :newval" \
    --condition-expression "Price = :currval" \
    --expression-attribute-values file://expression-attribute-values.json
```

The arguments for --expression-attribute-values are stored in the expression-attribute-values.json file:
```
{
    ":newval":{"N":"8"},
    ":currval":{"N":"10"} 
}
```

The item with key Id=1 will have its Price updated to 8, only if the current Price value is 10.

## Managing Throughput
DynamoDB *Provisioned Throughput* is measured in Capacity Units. A write capacity unit represents one write per second, for an item up to 1 KB in size. A read capacity unit represents one strongly consistent read per second, or two eventually consistent reads per second, for an item up to 4 KB in size.
* 1 * Write Capacity Unit = 1 * 1kB WPS
* 1 * Read Capacity Unit = 1 * 4kB Strongly Consistent RPS = 2 * 4kB Eventually Consistent RPS

**e.g. Calculating Strongly Consistent Read Capacity Units**
You want to read 80 items per second. 
Each item is 3kB in size.
You need Strongly Consistent reads.

1. Calculate how many Capacity Units required for each read; 3kB / 4kB = 0.75 (rounded up to 1)
2. Multiple by number of reads per second required; 80 * 1 = 80 Read Capacity Units required.

For Eventually Consistent reads, divide the result by 2.

**e.g. Calculating Write Capacity Units**
You want to write 420 items per second.
Each item is 512B in size.

1. Calculate how many Capacity Units required for each write; 512B / 1kB = 0.5 (rounded up to 1)
3. Multiply by number of writes per second required; 100 * 1 = 100 Write Capacity Units required.

DynamoDB also supports *On-Demand* capacity which instantly scales up and down throughput based on the application's activity. Pay-per-use pricing model, good for unpredictable workloads.

## Time to Live
Setting a *Time to Live* (TTL) for items in at able will define an item will expire and be marked for deletion. The value is stored as a user-defined attribute in the table, and the value format is in *epoch time* (number of seconds elapsed since 01/01/1970).

# Indexing
Upon creating a table the primary key must be specified (partition or composite). Primary keys must be unique; this will provide fast access to items when querying. To allow efficient access to data with attributes other than the primary key, secondary indexes can be used.

A *secondary index* is a data structure containing a subset of attributes from a table, along with an *alternate key* which differs from the primary key in the *base table*. When creating a secondary index, you specify which attributes to *project* to the index. There are 3 options:

* **KEYS_ONLY**: Only include the index's alternate key and the base table's primary key.
* **ALL**: Everything in the base table.
* **INCLUDE**: Manually choose which attributes to project.

DynamoDB supports 2 types of secondary indexes; *Local Secondary Indexes (LSI)* and *Global Secondary Indexes (GSI)*.

## Local Secondary Index (LSI)
Same partition key as the base table's with a different sort key. Limit of 5 per table. Must be provisioned during table creation.

## Global Secondary Index (GSI)
Different partition and sort keys compared to the base table. Limit of 20 per table. Can be provisioned any time. More expensive option.

# Read Consistency
DynamoDB supports eventually consistent and strongly consistent reads.
* **Eventually consistent** - Consistency across all copies of data is usually reached within a few seconds.
* **Strongly consistent** - Immediately reflects all successful write operations to the database.

# DynamoDB Accelerator (DAX)
In-memory caching for DynamoDB tables. Improves response times for **Eventually Consistent Reads** only. 

# DynamoDB Streams
DynamoDB can store a log of writes to a table in the form of *streams*. These are updated in real time, and can be used as a source trigger for Lambda functions. Stream data is stored for only 24 hours.

When an item in the table is modified, `StreamViewType` determines what information is written to the stream for this table. Valid values are:

* `KEYS_ONLY` - Only the key attributes of the modified item
* `OLD_IMAGE` - The entire item before it was modified
* `NEW_IMAGE` - The entire item after it was modified
* `NEW_AND_OLD_IMAGES` - Both the new and old item images

![](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/images/streams-endpoints.png)

Using the Amazon Kinesis Adapter is the recommended way to consume streams from Amazon DynamoDB. You can write applications for Kinesis Data Streams using the Kinesis Client Library (KCL). The KCL simplifies coding by providing useful abstractions above the low-level Kinesis Data Streams API. In both services, data streams are composed of shards, which are containers for stream records. Both services' APIs contain `ListStreams`, `DescribeStream`, `GetShards`, and `GetShardIterator` operations.

# Access
Access to DynamoDB is controlled using IAM policies. Fine-grained access can be provisioned in the "Condition" parameter, e.g. for comparing with the partition key, use `dynamodb:LeadingKeys`. Can be used for providing access only to records where the partition key matches with a user id, for example.

# Transactions
DynamoDB transactions incorporate operations across multiple items and tables into a single ACID (atomic, consistent, isolated, durable) transaction.

# Errors
## ProvisionedThroughputExceeded
The number of requests is too high. If using the AWS SDK this gets automatically resolved since it implements Exponential Backoff (keep retrying with longer waits between each attempt). If wait time starts to exceed 1 minute, consider increasing the RCUs/WCUs or using DAX.

