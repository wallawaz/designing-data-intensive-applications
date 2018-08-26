# Storage and Retrieval

## Hash Indexes
Compaction - throwing away duplicate keys across segments within a log.
  - enable merging segments
  - Hash table must first in memory.
  - Range queries inefficent.

## SSTables
  - same as hash map, many segment files containing keys with pointers to addresses.
  - main difference: Sequence of k/v pairs **sorted by key**
  - each key only appear once within each segment.
  - still need a sparse index in memory to point to each segment.
  - writes:
    - increase in-memory balance tree data structure.
    - when large enough; write this data structure as newest segment.
    - new writes can continue to a new in-memory memtable while dumping the last segment
    - reads scan --memtable-->latest_segment-->next_segment--> etc.
    - periodically merge and compact segments.
      - can interfere with performance if occurs during high activity.
      - if write throughput is high enough and compaction not configured correctly; compaction may not be able to keep up with rate of writes.

## Log-structured Merge Tree LSM-Tree
  - keeping cascade of SSTabled merged within background.
  - *Bloom Filters* to detect if a given key exists within the entire set.
    - avoid scanning every segment if a key is missing.

## B-Trees
  - break database down into fixed-size blocks (pages) of sorted keys.
    - pointers to rows in the **heap file**.
  - tree hierarchy with pointers from block to block.
  - move down the tree until **leaf page** containing the value for the key inline or final reference to value.
  - balanced tree algorithm; `n` keys always has a depth of `O(log n)`.
  - writes are overwriting the tree strucure *in place* then rebalancing the tree.
  - crash resiliance:
    - all writes also write to Write Ahead Log (WAL) (append only).

General rule: LSM-tree faster for writes; B-Tree faster for reads.

## Transactional vs Analytical Processing
  - Online Transactional Processing (OLTP)
    - Application looks up small number of records with index; records need to be quickly inserted/updated back into db based on user input.
  - Online Analytical Processing (OLAP)
    - analytic query scan over large # of records; reading only a few columns to perform aggregates.
    - Business Intelligence.
    - trend to move these analysics on separate **data warehouse**.
      - read only copy of the data
      - Extracted from OLTP databases --> Transformed into analysis friendly schema --> Cleaned --> Loaded into DataWarehouse.
      - ETL
    - Redshift; SQL-on-Hadoop: Apache Hive, Spark SQL, Cloudera Impala, Facebook Presto, Google Dremel..

### Star Schema
  - fact table in the center of the schema as an "already joined together *wide* relation of other dimensions"
### Column-Oriented Storage
  - store values of columns together
  - leads well to compression (bitmap encoding).
  - usually no sort order, although one can be set if this will help access patterns. i.e. timestamp..