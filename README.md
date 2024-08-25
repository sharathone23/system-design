# system-design notes
_________________________________________________________________________________________________________________________________________________________________________________________________________________

### Reliable
  System should continue to work correctly even when things go wrong due to hardware/software/human errors.
### Scalable
  System should perform well even when load increases.
### Maintainable
  System should be simple for new engineers (to make enhancements),ops team (to keep things run smoothly)

_________________________________________________________________________________________________________________________________________________________________________________________________________________

# Database Internals: Write and Read Mechanisms

There are two major ways how data is written and read back later in databases:

1. **Write-Ahead Logging (WAL) - Immutable**
2. **Update-in-Place Pattern - Mutable**

## Write-Ahead Logging (WAL) - Immutable

Write-Ahead Logging (WAL) is a method used by databases to ensure durability and support crash recovery. Here’s how it works:

- **Durability**: Before modifying data pages on disk, changes are first written to a dedicated log file known as the WAL log. New data is always appended at the end of the file.
- **Sequential Logging**: WAL ensures that changes are sequentially logged before the corresponding data modifications, providing a reliable record of all transactions.
- **Recovery**: In the event of a crash or system failure, the database can replay the WAL log to restore the database to its last consistent state.

### Implementation

- **Log-Structured Merge Trees (LSM Trees)**: LSM Trees are a popular immutable on-disk storage structure that is write-efficient since writes are done sequentially without modifying the earlier values.

## Update-in-Place Pattern - Mutable

Update-in-Place is a strategy where databases directly modify existing data at its original storage location. Key aspects include:

- **Efficiency**: Enables efficient data updates by avoiding the overhead of copying data to new locations.
- **Direct Modification**: Data modifications (inserts, updates, deletes) are performed directly within the existing data structure, such as B-trees or hash tables.
- **Transaction Support**: Supports transactional operations where changes are immediately visible and committed once completed.

### Implementation

- **B-trees**: Used in relational databases for indexing and managing data pages efficiently. Unlike LSM Trees, B-Trees update the data in place, which is not as write-efficient since it requires performing lookups before overriding the value.

## Conclusion

- **LSM Trees**: Preferred for databases designed for write-heavy operations. Commonly used in databases like Cassandra and LevelDB, LSM Trees efficiently handle a high volume of writes.
- **B-Trees**: Preferred for databases designed for read-heavy operations. Used in databases like MySQL and PostgreSQL, B-Trees provide efficient indexing and query performance. When designing a database that prioritizes read performance, B-Trees are often the preferred choice.

_________________________________________________________________________________________________________________________________________________________________________________________________________________

# Replication
Database replication is a technique used to maintain multiple copies of data from a database, and keep it updated in real-time. This is crucial for ensuring data availability, improving performance, and protecting against data loss.

### Types of Database Replication
There are several types of database replication, including:

1. **Master-Slave Replication**: In this model, one database server (the master) is responsible for all write operations, while the other servers (the slaves) handle read operations. The master server replicates its data to the slave servers.

2. **Multi-Master Replication**: In this model, multiple database servers are able to handle both read and write operations. Any changes made to the data on one server are automatically replicated to the other servers.

3. **Peer-to-Peer Replication**: In this model, all changes to the database are replicated to all servers. This model is more complex but provides high availability and load balancing.

### Benefits

Database replication offers several benefits, including:

- **Improved data availability**: By maintaining multiple copies of data, replication ensures that data is always available, even if one server fails.

- **Improved performance**: Replication allows for load balancing. Read operations can be distributed across multiple servers, reducing the load on any single server and improving performance.

- **Data protection**: Replication provides a form of backup. If data is lost or corrupted on one server, it can be recovered from another server.

_________________________________________________________________________________________________________________________________________________________________________________________________________________

# Sharding & Partitioning

# Sharding
Sharding is a database architecture pattern that involves dividing a large database into smaller, more manageable segments, known as *shards*. Each shard contains a subset of the total data and is stored on separate database servers. This division can be based on various criteria, such as geographic location, customer group, or data type.

### **Benefits**
- **Improved Performance**: By distributing the data across multiple servers, sharding reduces the load on any single server and improves overall performance.
- **Scalability**: Sharding makes it easier to scale horizontally by adding more servers.
- **High Availability**: It increases the availability of applications, as a failure in one shard doesn’t affect the others.

### **Challenges**
- **Complexity**: Sharding increases the complexity of database management and application logic.
- **Data Distribution**: Ensuring even data distribution across shards can be challenging.


# Partitioning
Partitioning, on the other hand, refers to dividing a database into smaller parts, but these partitions remain within the same database instance. It’s more about organizing data within a single database server rather than spreading it across multiple servers.

### **Benefits**
- **Performance Improvement**: It can improve performance on large tables by reducing index size and improving query efficiency.
- **Maintenance Ease**: Easier maintenance tasks, as operations can be performed on individual partitions.

### **Challenges**

- **Limited Scalability**: Unlike sharding, partitioning doesn’t inherently support horizontal scaling.
- **Partitioning Logic**: Implementing the right partitioning strategy requires careful planning.

# Sharding vs Partitioning: When to Use Which?
- **Use Sharding** when dealing with very large databases where scalability and performance across multiple servers are crucial. Ideal for globally distributed applications.
- **Use Partitioning** for improving performance and maintenance within a single server environment, especially when working with large tables.

_________________________________________________________________________________________________________________________________________________________________________________________________________________

# Deep Dives
# Redis
Redis is popular for cache but it can also be used as durable database. Redis stores all of its data in memory which is the reason for its ligtning fast response times. Apart from database, it can also be used as a distributed lock, leader boards, replacement for kafka(in some instances using Redis Streams).

### Details
Redis is a single threaded, in memory data structure server. Single Threaded simplifies things a lot from conflicts perspective, as requests are processed in the order they are received.
Redis core is a key-value dict, values can be strings, numbers, binary blobs, sorted sets, hashes, geo-spatial indexes and bloom filters.

### Use Cases
1. **Cache**: Use as cache in application layer, where requests are first go to cache, if exists read from cache else go to database and return back data and write to cache.
   Expiration policy: Use EXPIRE command while setting the value to set TTL(Time to Live)
   Eviction policy: Least Recently Used(LRU), LFU(Least Frequently used) - in this setup cache is filled until you run out of memory and starts evicting based on the strategy LRU/LFU to free up space

2. **Rate Limiter**: When we want to gaurd a expensive service or external service cannot accepts more than 5 requests per second, in these scenarios we can use Redis as rate limiter, Add an entry with key
   INCR expensive_service_name_rate_limit (increment a key if exists or sets to 1) and return the latest value
   if(value is >limit) don't make the call else proceed with call and make sure expire the key after 60 seconds(EXPIRE expensive_service_rate_limit 60 LT) which will remove the key which is similar to setting to zero.
   this is the most basic setup, but a lot can be customized based on use case like using a window or letting the clients know when they are eligible to send requests again.

3. **Stream**: Redis Streams are ordered list of Items, if we want to process all items in async job queue, it Item is in Stream then it is eventually processed.
   Stream will have a Consumer Group with the pointer to current Item which is to be processed next. At any given moment only one of the Worker can claim that Item, If worker failes other worker picks up that Item and Pointer is incremented and moved    to next Item in Stream. Worker will continue the heart-beat to let Consumer Group know that its still working, But waht if net work connectivity is lost in between worker and Consumer group, Hence, Redis stream only gaurantees at least once processing but not guarantee exactly once because when network issue, CG can assume the worker failed and assign the same Item to another worker causing in processing the same Item twice.

4. **Leaderboard**: Using Sorted Set
5. **Geo Spatial Index** Use case - When you want to search items based on a location. The way this works is while adding an item you should provide the lat and long of the item. While search you should provide the lat and long and it will return the list of items which are in that region.


_________________________________________________________________________________________________________________________________________________________________________________________________________________


