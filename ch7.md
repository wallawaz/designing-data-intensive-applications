# Transactions

A way for an application to group several reads and writes together into a logical unit.
Either the entire transaction succeeds (commit); or it fails (rollback).

## ACID
  - Atomicity
    - cannot be broken down into smaller parts.
    - ability to abort a transaction on error and discard writes from within this transaction.
  - Consistency
    - db in a "good state"; data adheres to schema constraints.
  - Isolation
    - concurrently executing transactions are isolated from eachother.
  - Durability
    - promise that once a transaction committed; data will not be forgotten even if crash.

## Isolation levels
### Read Committed
  - weakest
  - reads will only see committed data.
    - no dirty reads
  - writes will only overwrite committed data.
    - no dirty writes
  - **read committed writes** usually implemented with **row-level-locks**
    - only one transaction can hold a lock on a row at at time; other transactions must wait for the commit to writes.
    - for every object written, db remembers the old existing value *and new value* set in ongoing transaction; only when transaction commits are reads forwarded to new value.

### Snapshot isolation
  - Also called **Repeatable Read**(PostgreSQL and MySQL), **Serializable**(Oracle)
  - each transaction reads from a consistent snapshot of the db.
  - readers never block writers. writers never block readers.
  - multi-version-concurrency control (MVCC).
    - postgres--> each writing transaction is given an increasing trans_id.
    - reading transactions can only view data: 
      - at the start_time of the reader's transaction; the transaction that created the object already committed.
      - the object is not marked for deletion, or if it is, it is the transaction that requested the deletion had not yet committed.

### lost updates
  - two clients concurrently {read-modify-write}. The second writer's write does not include the the first modification.
  - some dbs offer atomic write operations for you to handle this. or specific isolation levels will automatically detect these and abort an offending transaction.
  - or can be handled with explicit locking `SELECT * from figures WHERE name = 'robot' and game_id = 222 FOR UPDATE`.

### Write Skew
 - two transactions read the same objects; make a decision based on values of the object and writes. however once the write is commited, the premise to make the decision to write is no longer valid.

### Phantom
  - a write in one transactions change the result of a search in query in another transaction.
  - snapshot isolation avoids phantoms in read-only queries. however in read-write transactions can lead to write skew.
  - can remedy by **materializing conflicts**; i.e. create all unique candidate records in a lookup table first and require the modifying query to lock rows in there first.. *this method is error prone*.

### Serializable
  - strongest isolation level.
  - even though transactions execute in parallel, the end result is the same as if they had executed one at a time.
  - options:
### actual serial execution
  - remove concurency completely execute transactions only one at time on a single thread.
  - dont allow interactive multi-statement transactions; application code must submit entire transaction code as a **stored procedure**
  - only viable when each transaction is very fast to execute; or else bad performance impact in throughout.
  - limited to uses cases where active dataset can fit in memory; 
  - write throughput low enought to be handled by a single CPU core.

### Two-Phase Locking (2PL)
  - when write/modify/delete is attempted; exclusive access required.
  - writers do not only block other writes but also readers.
  - **shared** vs **exclusive** lock modes.
    - reads first grab a share lock; several transactions can grab **shared** on a given object; if any exclusive lock is held on the object, reads must wait to grab the shared.
    - writes must grab **exclusive** lock; no other transactions can hold any locks on the object.
    - once locks acquired; held until end of transaction; locks acquired then released.
  - throughput and response time significantly worse under 2pl than under weak isolation.

  - most dbs actually implement 2PL with **Index-Range Lock**; the predicate for the write operation is attained via an index on the relation... i.e. lock all rows with id=123; id being a PK.

## Serializable Snapshot Isolation Level (SSI)
  - Serializable Isolation Level in PostgreSQL
  - optimistic concurency control; when committing check if anything bad happened.
  - uses **Index-Range** lookups to update multiple values across different transactions.