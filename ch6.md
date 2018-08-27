# Partitioning

A Shard/region/tablet/vnode,vBucket.
Each parition is effectively a small db of its own; although some operations may run across partitions.
+ main reason is for scalability.

## key-value based
 - spread evenly across nodes
 - __unfair__ partioning leading to skew --> **hot spot**

### key range
 - ranges of keys not necessarily evenly spaced; because data may not be evenly distributed.
 - if keys are time-based; may want an additional prefix before the time-based key to avoid majority of writes targeting `today's` key

## hash key based
 - good hash function --skewed-data--> uniform distributed.
 - each partition receives a range of hashes.
 - `-` lose key-based partioning.

 ## secondary indexes
  - do not map neatly to partitions.

### document-based
 - local index; each partition maintains it own set of 2ndary indexes
 - query for indexed term must hit **all** partitions to reach each index structure.
    - scatter/gather.
### global-based
 - covers all partitions
 - **term-partitioned** - the term we're loooking for determines the the which partition the index lies on.
 - can make reads more efficent; however makes writes more complicated.
  - updated to global-secondary-indexes often are asychronous.

## Rebalancing
 - afterwords, {storage, reads, and writes} should all be shared fairly evenly between nodes.
 - while rebalancing db should continue to accept reads & writes.
 - minimum data should be shuffled between nodes to minimize network and disk I/O.

### strategies
  - Fixed number of partitions
    - create many more partitions than there are nodes; assign several partitions to each node.
    - new nodes can *steal* partitions from every other node until evenly balanced again.
    - most define the fixed number of partitions at onset of db.
  - Dynamic partitioning
    - key-range based create their partitions automatically.
    - each partition assigned to one node. node can have multiple partitions.
    -  number of partitions adapts to total data volume.
  - Proportion partitions to number of nodes (Casandra, Ketama).
    - fixed number of partitions; per node.

## Service Discovery
  - a. round robin
  - b. requests first fowarded through **routing tier**
  - c. require clients be aware of the partioning. connect directly.

