# ElastiCache
Allows retrieval of frequently-accessed information from **fast, managed, in-memory caches** instead of relying entirely on reading from disk. Can decrease latency in read-heavy workloads e.g. fetching the results of IO-intensive DB queries; and compute-heavy workloads e.g. recommendation engines.

## Caching Engines
ElastiCache currently supports the **Memcached** and **Redis** caching engines.

### Memcached
ElastiCache manages Memcached nodes as a pool that can grow and shrink (similar to an EC2 Auto Scaling group); individual nodes are expendable and non-persistent.

Memcached provides a simple caching solution that best supports object caching and lets you scale out horizontally. Ideal for offloading a DB's contents into a cache.

### Redis
ElastiCache manages Redis more as a relational database, i.e. Redis clusters are managed as persistent, stateful entities that include using *multi-AZ redundancy* for handling failover (similar to RDS).

Redis supports complex data structures, hence would be ideal in cases where sorting and ranking datasets in memory are important (e.g. such as in leaderboards for games).

## Caching Strategies
### Lazy Loading
Only cache data when it is requested. Cache miss penalty on initial request. Chance to produce stale data; can be mitigated by setting a TTL. Shorter TTL = less stale data.

### Write-Through
Every database write will write to the cache as well. Data is never stale however there will be alot more operations to perform; and these resources are wasted if most of the data is never used.
