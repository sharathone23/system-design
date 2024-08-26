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
1. **Cache**: Use as cache in Application layer, where requests first go to cache, if exists read from cache else go to Database and return back data and write to cache.
   
   Cache Expiration (TTL):
      
    You can set an expiration time for a key in Redis, after which the key will be automatically deleted. This is known as TTL (Time-To-Live).
    `EXPIRE key seconds` : Set an expiration time on an existing key.

    Eviction Policies:
   
    `noeviction`: Returns errors when the memory limit is reached and new keys are added. No keys are evicted.
   
    `allkeys-lru`: Removes the least recently used (LRU) keys among all keys in the database.
   
    `allkeys-random`: Removes random keys among all keys in the database.
   
    `volatile-lru`: Removes the least recently used (LRU) keys among the keys with an expiration time.
   
    `volatile-random`: Removes random keys among the keys with an expiration time.
   
    `volatile-ttl`: Removes keys with the shortest remaining TTL among the keys with an expiration time.
   
    LRU config:
   `CONFIG SET maxmemory-policy allkeys-lru`

   Differences(Expiration vs Eviction):
   
   Expiration: If a key has a TTL set, it will be removed from Redis once the TTL expires, regardless of the memory usage.
   
   Eviction: If Redis reaches its memory limit, it will evict keys according to the configured eviction policy. This may include keys with TTL, but the eviction policy determines which keys are chosen.

2. **Rate Limiter**: When we want to guard a expensive service or when an external service cannot accepts more than 5 requests per second, in these scenarios we can use Redis as rate limiter.
   Add an entry with key
   
   `INCR expensive_service_name_rate_limit` - (increment a key if exists or sets to 1) and return the latest value.
   
   if(returnedValue > limit) don't make the call Else proceed with call and make the key expire after 60 seconds by running `EXPIRE` command.
   `EXPIRE expensive_service_rate_limit 60 LT` Removes the key which is similar to setting it to zero.
   
   This is the most basic setup, but a lot can be customized based on use case like using a window or letting the clients know when they are eligible to send requests again.

3. **Stream**: Redis Streams are ordered list of Items.
   
    Usecase - if we want to process all items in async job queue, if Item is in Stream then it is eventually processed(no data loss)
   
   Each Stream will have a Consumer Group with the pointer to current Item which is to be processed next. At any given moment only one of the Worker can claim that Item, If worker fails ,other worker picks up that Item and Pointer is incremented and     moved to next Item in Stream. Worker will continue the heart-beat to let Consumer Group know that its still working, But what if there is a network connectivity issue in between worker and Consumer group? Consumer group assumes the worker failed      and assigns the same Item to another worker causing duplicate processing of the same Item. Hence, Redis stream only gaurantees at least once processing/delivery but does not guarantee exactly once processing/delivery.

4. **Leaderboard**: Redis Sorted Sets are collections of unique elements, each associated with a score. The elements are automatically ordered by their score, making Sorted Sets ideal for leaderboards where you need to rank members based on their     
   scores.
     
   Use the ZADD command to add members to a sorted set with their associated scores.
   
   `ZADD leaderboard 100 "player1"`
   
   `ZADD leaderboard 200 "player2"`
   
   `ZADD leaderboard 150 "player3"`
   
    To get the top N members, use the ZRANGE command with the WITHSCORES option to include scores in the result:
   
   `ZRANGE leaderboard 0 9 WITHSCORES`

5. **Geo Spatial Index** Use case - When you want to search items based on a location. The way this works is while adding an item you should provide the lat and long of the item. While search you should provide the lat and long and it will return the 
   list of items which are in that region.
  
    Example: Add few stores first with their location  
    
    `GEOADD stores:locations -73.935242 40.730610 "Store 1"`  
    
    `GEOADD stores:locations -74.0060 40.7128 "Store 2"`  
    
    `GEOADD stores:locations -73.9772 40.7831 "Store 3"`  
    
     Find all stores within 5 kilometers of a specific point usually users current location(longitude, latitude):  
     
    `GEOSEARCH stores:locations FROMLONLAT -73.935242 40.730610 BYRADIUS 5 km WITHDIST ASC`

6. **Pub/Sub** : Redis Pub/Sub (Publish/Subscribe) is a messaging pattern supported by Redis that allows for real-time messaging and event notification between different parts of a system. It is useful for applications that need to broadcast messages to multiple subscribers or react to events in real time. (data could be **LOST** if consumer is down)
   
   Use Cases:
   
   Real-time Notifications.
   
   Chat applications with multiple users in a channel.
   
 
_________________________________________________________________________________________________________________________________________________________________________________________________________________

# Kafka
Kafka is a popular event streaming platform and it can be used either as a Message Queue or as a Stream processing system. 

**Key Terminology** (Bottom Up)  

`Offset` A unique identifier for a record within a Kafka partition. Offsets are used to track the position of a consumer in the log and ensure that records are processed in order. Consumers periodically commits offset to kafka because in case of failure or restarts of consumers they can start from where they left off using offset.

`Partition` A partition is a physical single log on disk that stores a portion of the data in a Kafka topic. Each partition is ordered, and messages within a partition are assigned a unique sequential ID called an offset. Its an immutable Append only log file which makes writes fast.  

`Topic` A Topic is a logical grouping of partitions that holds the same type of events.  

`Producer` An application that writes or publishes records (events/messages) to one or more Kafka topics.  

`Consumer` An application that reads or subscribes to records (events/messages) from one or more Kafka topics.  

`Consumer Group` A group of consumers that work together to consume records from a Kafka topic. Each consumer in the group is assigned a partition during startup, No two consumers within the same group read from the same partition at the same time.  

`Broker`  A Kafka server that stores data and serves client requests. Brokers receive records from producers, assign offsets to them, and store them in correct partition on disk. They also serve records to consumers.  

`ZooKeeper` A centralized service for maintaining configuration information, naming, providing distributed synchronization, and providing group services. Kafka uses ZooKeeper to manage and coordinate the Kafka brokers.  

`Kafka Cluster`  Kafka Cluster is a collection of multiple Kafka brokers that work together to manage the distribution, storage, and processing of data across a distributed system. Each broker within the cluster handles a portion of the data (organized into partitions), and together, they ensure high availability, fault tolerance, and scalability. The cluster allows Kafka to manage large volumes of data by spreading the load across multiple servers, ensuring that the system remains resilient even if individual brokers fail. The Kafka cluster acts as the backbone of the Kafka ecosystem, enabling efficient data streaming and processing in distributed environments.  

`Replication` In order to ensure durability and availability, Kafka replicates partitions using leader and follower mechanism where each partition will have a leader and group of followers(Depends on replication factor). Followers only act as a backup if needed, so events will be processed mainly on leader partition. Kafka ensures partitions for a given topic are spread across different brokers so that when a given broker is down partitions in another broker can still serve the events. (See below image)

![Logo](https://d248djf5mc6iku.cloudfront.net/excalidraw/ad17548cbc6fe72490ecd9a489a42aa3)


**Usage**  

**As Message Queue** :
Kafka can be configured as a message queue when you need to process tasks asynchronously and exactly once. In this setup, you create a single Consumer Group that distributes tasks across multiple worker nodes (consumers), each assigned to different partitions. This ensures that tasks are processed efficiently and only once.

**As Stream** :
When dealing with events that need to be processed by various consumers, each with a distinct purpose, Kafka functions as a stream processing system. Here, you define multiple Consumer Groups, each tailored to a specific processing task.
   
 
_________________________________________________________________________________________________________________________________________________________________________________________________________________

