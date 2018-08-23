# Notes on Designing Data Intensive Applications
[Martin Kleppmman](https://dataintensive.net/)

## Part 2 (Distributed Data)
### Reasons why to distrubite onto multiple machines.
### Scalability
  - (volume|reads|write) grows bigger than singel machine can handle.
### Fault Tolerance/high availablity
  - allow system to continue working even if a node goes down.
### Latency
  - position nodes distrubuted closer to end-users to avoid network latency.

### Scaling to a higher load
  - Shared Memory
    - ++ memory/CPUs on to be treated as single machine
    - $$$
  - Shared-Disk
  - serveral machines sharing same disk.
  - contention and overhead of locking.

- Shared Nothing (horizontal scaling)
  - Data is Distibuted across nodes via Replication OR Patitioning
  - Replication: copy of same data across machines
  - Paritioning: Spliting db into smaller subsets (partitions)
5. [Replication](./ch5.md)