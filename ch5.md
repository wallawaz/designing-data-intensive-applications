# Replication

Keeping a copy of the same data on multiple machines that are connected via network.
Each node storing a copy of data - replica. Need them to be all in sync.

## Leader based replication (master/slave)
1. Client --write--> Leader.
2. Leader --write--> Replica(s)
          --write--> Replica(s)
    - replication log
3. Client --read--> Leader OR Replica(s)

## Synchronous vs Asynchronous
### Synchronous - leader waits for follower(s) to be finished.
 - Ensures replica always has up-to-date data before proceeding.
 - If follower(replica) has problem responding... system becomes blocked.
### Aynchronous - leader sends write to followers; does not wait for confirmation; immediately responds.
 - faster repsonse; however unable to guarantee successful write on replica.
 - Ensures replica always has up-to-date data before proceeding
### Semi-Synchronous (Best)
 - Only 1 Synchronous replica; all other async.
 - Ensures up to date data on first replica. all others are async.
 - If sync replica faces issues, an async is promoted to be the sync replica.
## Adding a new replica
1. Take db snapshot (dump).
2. Copy snapshot to new replica.
3. Replica connect to leader and request all changes it is missing since snapshot via replication log (log sequence number).
4. replica process changes until caught up.
## Outages
  - Follower failure simple; create a new one, or reboot the node and it will catch up via replication log.
  - Leader failure; a replica needs to be promoted to leader (FAILOVER).
    - clients need writes forwared to new leader.
    - other followers need to read from new leader.
  - Detect a failure via heartbeat.
  - Choose new leader via consensus.
  - Problems:
    - with async replication no guarantee that promoted leader has all writes.
    - multiple nodes taking Leader role (split-brain)
    - heartbeat timeout config 30seconds? 1min?

## Repliation logs
### Statement based replication
 - leader forwards literal SQL statements INSERT, UPDATE, DELETE etc to followers.
 - problematic with non-determinanistic functions... `now()`
  - triggers/stored procedures may not 100% behave the same way on replica.
### WAL shipping
- Write ahead log (append only) of leader is forwarded to all followers to consume.
- Uses low level data structure from the db storage engine (could make engine migration difficult).
### Logical (Row-Based)
- uses differnt log format specifically for replication.
- decoupled from storage engine, easier to parse by external systems.

### Problem with Read-Scaling architecture (followers++)
  - only really works with async replication (avoid blocking).
  - will lead to eventual consistency in reads from replica vs leader due to **replication lag**.
  - **Reading your own writes**
    - user may see outdated info if they are reading from a lagging follower shortly after making write.
  - Need **read-after-write-consitency**; user always will see their recent updates.
    - when reading something the user has modified read from **ledader** (only applicable if a user's writes can be segmented)
    - prevent client from reading from follower for `n seconds` after they write.
  - Prevent user from seeing thing **backwards in time**
    - **Monotomic Reads** - if one user makes reads in sequence they will see the same results.
    - each user only read from one replica in their session.
  - **Consitent Prefix reads** - if a sequence of writes happens in an order; anyone reading these writes will see the same order.

### Mutli-Leader Replication
  - allow more than 1 node to accept writes. each leader simulatenously acts as a follower to other leaders.
  - may be used in multi-data center enviornment (leader in each data-center.)
  - BDR (postgres); Tungsten Replicator (MySQL)
  - DOWNSIDE: same daata may be concurrently modified in 2 diff datacenters.
    - need conflict resolution.
    - avoid conflicts by forcing all writes in a transaction to a specific data center. (within the application code)

  - convergance: All replicas must arrive at the same final value
    - last-write-wins (LWW); each write given unique id; pick the write with the highest.
    - each replica given a unique ID; writes from higher replicas take presidence.
    - merge writes together ???
    - record the conflict; resolve at a later point in time.
  
  - custom conflict resolution on immediate WRITE OR READ.


### Multi-Leader Topologies
  - **Replication Toplogy** -> the communication paths along which the writes propagated from node to node.
  - **all-to-all** (most common) - every leader sends writes to every other leader. (most densely connected).
  - **star**
  - **circular** (MySQL)
  - circular and star have problems when 1 node fails ; interupt the flow to others.

### Leaderless Replication
  - gaining popularity again after Amazon Dynamo.
  - possible to get stale data on a read from a node that missed a write.
    - to solve this; clients send requests from several nodes in parallel and accepts data with the most recent version number.
  - **Repairing stale data**
    - Read repair - only when a client request determines that there is stale data on a node is this stale data ever updated.
      - values that are rarely read may be stale on some replicas.
    - Anti-entropy - background process that looks for differences in data between replicas.

### Quorums
  - `n replicas`
  - every write must be confirmed by `w nodes`
  - must query at least `r nodes` for each read.
  - `w + r > n`
  - make `n` an odd number; `w = r = (n+1) / 2`; ex: `n=3 w=2 r=2` or `n=5 w=3 r=3`
  - `w-- or r--`; more likely to read stale values; more likely that the read didn't hit node with latest value. however as we decrease these we have lower latenct and higher availability.

### Sloppy Quorum
  1. `n=[A,B,C]`
  2. `C` becomes becomes transiently unreachable.
  3. Allow --write--> `[B, C, D]` to be considered a "sloppy" replica set.
  4. Write to `D` would be stored with a "hint" that it should ideally have been sent to `A`.
  5. Once `A` has recovered, `D` --write-back--> `A`.
  6. "hinted handoff" is one of the mechanisms by which such a database can maintain expected levels of availability and durability across failure states.

### Replication lag
  - subtract a follower's current position form the leader's current position in the replication log.
  - not possible to determine in **Leaderless**.

#### Concurrentcy "happens before"
  - An operation `A` happens before operation `B` if `B` knows about `A`, or depends on `A`, or builds upon `A`.
  - 2 operations are concurrent if neigher happens before one another (neighter knows about each other).
  - cannot direcly rely on **time** to define concurrency bc of problems with clocks in distributed systems.