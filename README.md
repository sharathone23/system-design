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

`Replication` In order to ensure durability and availability, Kafka replicates partitions using leader and follower mechanism where each partition will have a leader and group of followers(Depends on replication factor). Followers only act as a backup if needed, so events will be processed mainly on leader partition. Kafka ensures partitions for a given topic are spread across different brokers so that when a given broker is down partitions in another broker can still serve the events. (See below image) *Image credit: [Hello Interview](https://www.hellointerview.com/learn/system-design/deep-dives/kafka)*

![Logo](https://d248djf5mc6iku.cloudfront.net/excalidraw/ad17548cbc6fe72490ecd9a489a42aa3)


**Usage**  

**As Message Queue** :
Kafka can be configured as a message queue when you need to process tasks asynchronously and exactly once. In this setup, you create a single Consumer Group that distributes tasks across multiple worker nodes (consumers), each assigned to different partitions. This ensures that tasks are processed efficiently and only once.

**As Stream** :
When dealing with events that need to be processed by various consumers, each with a distinct purpose, Kafka functions as a stream processing system. Here, you define multiple Consumer Groups, each tailored to a specific processing task.

**Deep Dives in System Design Interview**  

**Scalability**  

**Constraints**  

1. Aim for <1MB per message and do not put blob data in message instead upload blob to S3 and put S3 url in the message.  

2. One broker up to 1TB data & 10k messages per second.  (If your use case is less than this then you dont really need to scale)

**How to scale?**  

1. Add More brokers.  

2. Choose a good partition key.  

3. Use managed services like Confluent Cloud Kafka, AWS MSK

**How to handle hot partitions?**  

1. Remove Key (if you dont need ordering, just remove the key and kafka will use round robin to assign poartitions)
2. Compound Key - key:random_valuewithinrange(1,10) or append userID to key (Again no ordering is supported if you go this route)
3. Backpressure - Slow down the producer

**Fault tolerance and Durability** (See above image) 

Relevant Settings:  

1. replication factor (3 default)
2. acks ( acks=all - maximum durability tradeoff here is performance) (acks = 2 slightly less durability but high performance)

**What happens if consumer goes down?**  

**Rebalacing**: Kafka rebalances the partitions accross the active consumers resulting in assigning partition of the failed consumer.  

When a down consumer is back up, Rebalancing occurs again and Consumer uses the last committed offset of the newly assigned partition and starts reading from there.  

**Note**: Kafka does not gaurantee assigning back the same partition during rebalancing when consumer comes back up. Regardless of which partitions the consumer is assigned after restarting, it will begin consuming messages from the last committed offset for each assigned partition.


**Errors and Retries**  


`Producer Retries`   
1. Kafka producer api supports retries to configure the number of retries and wait time when failed to write to kafka topic.


`Consumer Retries`  

1. Create a Retry Topic: Establish a separate topic, known as the retry topic, to handle failed events during consumer processing. When an event fails, publish it to this retry topic along with the current retry count.
   
2. Process Failed Events: Set up a dedicated consumer to process events from the retry topic. If the retry count is within the defined limit, attempt to reprocess the event. If the event fails again, increment the retry count and republish it to the retry topic. If the retry count exceeds the limit, move the event to a Dead Letter Queue (DLQ) topic for further analysis and troubleshooting. Usually DLQ topics do not have any consumers.

**Performance Optimizations**

1. Batch messages in producer. Send single request with multiple messages
2. Compress messages in producer using GZIP results in small payload size.

**Retention Policy**

Two main settings  

1. Time-Based Deletion (retention.ms)(default 7 days): If the time limit set by retention.ms is reached before the log size exceeds retention.bytes, Kafka will start deleting the oldest messages based on their age.

2. Size-Based Deletion (retention.bytes)(default 1GB): If the log size reaches the limit set by retention.bytes before the messages reach the age specified by retention.ms, Kafka will begin purging the oldest messages to keep the log within the specified size.

In practice, Kafka will delete messages when either of these conditions is met. This means that messages can be removed either because they are too old or because the log size is too large.



   
 
_________________________________________________________________________________________________________________________________________________________________________________________________________________


# Elasticsearch: Core Concepts, Use Cases, and Scaling Strategies

Elasticsearch is a distributed, RESTful search and analytics engine built on top of Apache Lucene. It is widely used for full-text search, log and event data analytics, and more, due to its scalability, speed, and real-time search capabilities. This blog will delve into the core concepts of Elasticsearch, explore its common use cases, and discuss scaling strategies to ensure optimal performance.

## Core Concepts of Elasticsearch

Understanding Elasticsearch's internal architecture is key to leveraging its power. Here are some of the core concepts:

### 1. **Index**
An **index** in Elasticsearch is a collection of documents that share similar characteristics. It is identified by a unique name and acts as a namespace for storing and retrieving documents. An index can be thought of as a database in relational databases.

### 2. **Document**
A **document** is the basic unit of information that can be indexed in Elasticsearch. It is represented in JSON format and consists of fields that are stored as key-value pairs. Each document belongs to an index and has a unique identifier (ID).

### 3. **Field**
Each document contains one or more **fields**. Fields are the attributes of the document, and each field can be of a different data type such as `text`, `keyword`, `integer`, `date`, etc.

### 4. **Node and Cluster**
A **node** is a single server that is part of an Elasticsearch cluster. A **cluster** is a collection of one or more nodes that collectively hold all the data and provide federated indexing and search capabilities. The cluster is identified by a unique name, and every node in the cluster must have the same cluster name to join it.

### 5. **Shard**
Elasticsearch allows you to subdivide your index into multiple pieces called **shards**. Each shard is a fully functional and independent index that can be hosted on any node in the cluster. Shards allow Elasticsearch to scale horizontally by distributing data and requests across multiple nodes.

There are two types of shards:
- **Primary Shard**: Holds the original data.
- **Replica Shard**: A copy of the primary shard that provides redundancy and high availability.

### 6. **Inverted Index**
At the core of Elasticsearch’s search functionality is the **inverted index**. An inverted index is a data structure that stores a mapping from content, such as words or terms, to their locations in a set of documents. This structure allows Elasticsearch to search very quickly across large datasets.

### 7. **Mappings and Analyzers**
- **Mapping**: Defines how a document, and its fields, are stored and indexed. It is similar to defining a schema in a relational database.
- **Analyzer**: Defines how a text field is processed during indexing and searching. It typically includes tokenization and filters (e.g., lowercasing).

## Common Use Cases for Elasticsearch

Elasticsearch is a versatile tool that can be applied to a wide range of use cases. Here are some of the most common:

### 1. **Full-Text Search**
Elasticsearch is highly optimized for full-text search. It provides features like tokenization, relevance scoring, fuzzy matching, highlighting, and more. Use cases include search functionality for websites, document management systems, e-commerce sites, and content-based applications.

### 2. **Log and Event Data Analytics**
With tools like the **Elastic Stack (formerly ELK Stack)** — consisting of Elasticsearch, Logstash, and Kibana — Elasticsearch is widely used for log and event data analytics. It can ingest, parse, and analyze log data from different sources, enabling real-time monitoring, alerting, and analysis.

### 3. **Real-Time Application Monitoring**
Elasticsearch can be used to monitor real-time data, such as application performance metrics, error rates, and server logs. Integrations with tools like **Beats** and **Logstash** make it easy to collect and visualize data in real-time.

### 4. **Geospatial Data Search**
Elasticsearch supports geospatial queries, making it a powerful tool for applications that require location-based search and analysis, such as finding nearby points of interest, geo-fencing, and routing.

### 5. **E-Commerce Search and Recommendations**
Many e-commerce platforms use Elasticsearch to provide fast and relevant product search results, including autocomplete, faceted navigation, filters, and personalized recommendations.

### 6. **Security Analytics**
Elasticsearch is commonly used in security information and event management (SIEM) to analyze logs and events for detecting security threats, monitoring user activity, and forensic analysis.

## Scaling Strategies for Elasticsearch

As data grows, scaling Elasticsearch becomes crucial for maintaining performance and reliability. Here are some key strategies for scaling Elasticsearch:

### 1. **Horizontal Scaling with Sharding**
Elasticsearch is designed for horizontal scaling through **sharding**. An index can be split into multiple shards, and each shard can be hosted on a different node in the cluster. This allows Elasticsearch to distribute data and search operations across multiple nodes, enhancing both storage capacity and search speed.

- **Shard Allocation Awareness**: You can control how shards are distributed across nodes to ensure even load balancing.
- **Shard Rebalancing**: Elasticsearch automatically balances shards across nodes when new nodes are added or removed.

### 2. **Replica Shards for High Availability**
Replica shards provide redundancy and high availability. By maintaining copies of primary shards on different nodes, Elasticsearch can continue to operate even if some nodes fail.

- **Scaling Read Operations**: Replica shards can also be used to scale read operations since they can handle search requests in parallel with primary shards.

### 3. **Indexing and Query Tuning**
Optimizing the way data is indexed and queried is critical for scaling Elasticsearch:

- **Use the Bulk API**: For high-volume indexing, the **Bulk API** reduces overhead by batching multiple indexing requests into a single request.
- **Use Filters Instead of Queries**: Filters are faster and more efficient than queries because they are cached and do not need to calculate relevance scores.
- **Tune Refresh Interval**: Adjust the `refresh_interval` setting to control how often Elasticsearch refreshes its index to make new data searchable. This can significantly impact performance for high-throughput indexing.

### 4. **Cluster Management and Monitoring**
Proper cluster management and monitoring are essential for maintaining Elasticsearch’s performance:

- **Monitoring Tools**: Use tools like **Kibana**, **Elastic Cloud**, or third-party solutions to monitor cluster health, node performance, and resource utilization.
- **Cluster State Management**: Keep an eye on the cluster state, shard allocation, and disk usage to avoid situations like "split brain" scenarios or out-of-memory errors.

### 5. **Caching Strategies**
Elasticsearch supports several caching mechanisms that can significantly improve search performance:

- **Node Query Cache**: Caches the results of frequently executed queries.
- **Field Data Cache**: Stores field values to optimize sorting and aggregations.
- **Shard Request Cache**: Caches the results of searches and aggregations at the shard level.

### 6. **Optimize Data Storage and Index Size**
- **Use Index Aliases**: Aliases allow for reindexing without downtime and enable zero-downtime upgrades.
- **Delete or Archive Old Data**: Implement strategies for archiving or deleting old data to save disk space and reduce index size.

## Conclusion

Elasticsearch is a powerful tool for search and analytics that is capable of handling a wide range of use cases, from full-text search to real-time data analytics. Understanding its core concepts, use cases, and scaling strategies is essential for designing efficient and scalable Elasticsearch clusters. Whether you're building an e-commerce search engine, a log analytics platform, or a real-time monitoring system, Elasticsearch provides the flexibility and power to meet your needs.

By applying the appropriate scaling strategies, you can ensure that your Elasticsearch cluster remains fast, reliable, and capable of handling growing data volumes and search demands.

 
_________________________________________________________________________________________________________________________________________________________________________________________________________________

