# system-design
Designing Data-Intensive Applications Notes

### Reliable
  System should continue to work correctly even when things go wrong due to hardware/software/human errors.
### Scalable
  System should perform well even when load increases.
### Maintainable
  System should be simple for new engineers (to make enhancements),ops team (to keep things run smoothly)
  
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

                                


