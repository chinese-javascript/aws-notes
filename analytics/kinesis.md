# Kinesis Overview
Amazon Kinesis is a service for capturing *streaming data* in large quantities in real-time. Streaming data is data that is generated continously from thousands of data sources, which typically send in the data simultaneously and in small sizes (order of kilobytes).

Examples include:
* Operating logs
* Financial transactions
* Social media feeds
* Geospatial data (e.g. Uber)
* Online game data

Kinesis offers multiple services for data streaming based on the use case:
* **Kinesis Video Streams** - Stream video from connected devices to AWS for analytics and machine learning.
* **Kinesis Data Streams** - Build custom applications that analyze data streams using popular stream-processing frameworks.
* **Kinesis Firehose** - Load data streams into AWS data stores.
* **Kinesis Analytics** - Process and analyze streaming data using SQL or Java.

# Kinesis Data Streams
Kinesis streams consist of shards. Each shard supports up to 5 reads per second with a total data read rate of 2MB/s and up to 1000 writes per second with a total data write rate of 1MB/s. Data in shards get delivered to consumers (e.g. EC2, Lambda, ElasticMapReduce, etc.) for processing.

A stream's *total capacity* is the sum of all the capacities of its shards.

A Kinesis data stream stores records from 24 hours by default. `IncreaseStreamRetentionPeriod` increases the retention period (maximum 168h). `DecreaseStreamRetentionPeriod` decreases the retention period (minimum 24h).

![](https://docs.aws.amazon.com/streams/latest/dev/images/architecture.png)

The *Kinesis Client Library* runs on each consumer instance and ensures that for each shard in the Kinesis stream, there is a record processor. Shards are distributed equally across all consumer instances e.g. for a Kinesis stream with 6 shards and 2 consumer instances, the KCL will create 3 record processors per consumer instance, each handling a separate shard. An Autoscaling group can be used to provision consumer instances based on instance CPU utilisation.

## Terminology and Concepts
*Producers* are responsible for putting records into a Kinesis data stream by calling the `client.PutRecord` or `client.PutRecords` API. E.g. A web server sends log data to a stream. 

The producer must specify a *partition key* string value with each sent record. The *partition key* gets hashed into a 128-bit integer value (aka the *sequence number*), which is used to map the associated data record to a *shard*. A *shard* is the sequence of data records which have been grouped together based on the records' sequence numbers. 

Each shard can support up to 5 reads per second at 2MB/s and 1000 writes per second at 1MB/s. Each shard also represents a fixed unit of capacity within a *Kinesis data stream*.

*Consumers* read the output from a Kinesis data stream for processing. These commonly run on a fleet of EC2 instances.

# Kinesis Firehose
No dealing with shards/consumers, Firehose sends stream data directly to destinations such as S3, Redshift, Elasticsearch, or Splunk.

![](https://docs.aws.amazon.com/firehose/latest/dev/images/fh-flow-s3.png)

To deliver stream data to Redshift, the data must first be sent to S3. Firehose then issues an Amazon Redshift COPY command to load data from the S3 bucket to a Redshift cluster.

![](https://docs.aws.amazon.com/firehose/latest/dev/images/fh-flow-rs.png)

# Kinesis Analytics
Allows analysing stream data inside Kinesis using SQL-type queries, and store the results in S3, Redshift or ElasticSearch.

![](https://docs.aws.amazon.com/kinesisanalytics/latest/dev/images/kinesis-app.png)
