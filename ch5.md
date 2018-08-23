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
 - Leader failer; a replica needs to be promoted to leader (FAILOVER).
  - clients need writes forwared to new leader.
  - other followers need to read from new leader.
  - Detect a failure via heartbeat.
  - Choose new leader via consensus.

- Problems
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