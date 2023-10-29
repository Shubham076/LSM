## Introduction

Log-Structured Merge-trees (LSM trees) are a key component in many NoSQL databases like Apache Cassandra, ScyllaDB, and RocksDB. They help optimize both read and write operations. This guide covers the core concepts, including Memtables, SSTables, and compaction strategies.

----------

### Memtable

A Memtable is an in-memory data structure where databases write new records first. It's organized as a balanced tree, hash table, or skip list.

#### Features:

-   **Fast Writes**: Since it's in-memory, writing to a Memtable is fast.
-   **Not Sorted**: Keys in Memtable are not inherently sorted.

### SSTable

Sorted String Table (SSTable) is a read-optimized, immutable data file stored on disk.

#### Features:

-   **Sorted Keys**: Keys are sorted, enabling binary search.
-   **Immutable**: Once written, SSTables are never modified, only compacted.

----------

## Write Path

1.  **Write to Memtable**: New writes are first recorded in the Memtable.
2.  **Flush to SSTable**: When Memtable reaches a certain size, it's flushed to disk as an SSTable.
3.  **Compaction**: As SSTables accumulate, they are compacted based on compaction strategies.

----------

## Read Path

1.  **Check Memtable**: Look for the key in the Memtable first.
2.  **Check SSTables**: If not found in Memtable, search in SSTables.
3.  **Merge**: Data from multiple SSTables may be merged for a complete view.

> **Note**: Reading may involve checking multiple SSTables, leading to higher read latency.

----------

## Compaction Strategies

### Size-Tiered Compaction


-   **Organization**: SSTables are grouped by size, not by levels or tiers. All SSTables co-exist without a tiered structure.
-   **Combining Logic**: When the number of SSTables in a specific size range reaches a threshold, they are combined.
-   **Result**: Produces fewer but larger SSTables. No inherent organization or hierarchy.
-   **Implication**: Good for write-heavy workloads but can result in higher read latency due to potential scanning of multiple large SSTables.

### Example
-   **Tier 1**: SSTables between 0-10 MB
-   **Tier 2**: SSTables between 10-100 MB
-   **Tier 3**: SSTables between 100-500 MB
-   **Tier 4**: SSTables larger than 500 MB

### Level-Tiered Compaction

Organize SSTables into levels based on size and age. Compact and promote SSTables to higher levels.

-   **Organization**: SSTables are arranged in levels or tiers. Each level has a size limit and contains sorted, non-overlapping SSTables.
-   **Combining Logic**: SSTables are promoted from one level to the next based on specific rules, often involving the SSTable's age and size.
-   **Result**: Each level contains SSTables of increasing size and decreasing age as you go up the levels.
-   **Implication**: Optimized for read operations, since fewer SSTables typically need to be scanned.

### Example 1
-   **Level 0**: Newly written SSTables, unsorted, up to 10 SSTables
-   **Level 1**: Up to 10 MB of sorted data
-   **Level 2**: Up to 100 MB of sorted data
-   **Level 3**: Up to 1 GB of sorted data
-   **Level 4**: Up to 10 GB of sorted data
-   **Level 5**: Up to 100 GB of sorted data

### Example 2:

-   **Before Compaction**:
    -   Level 0: SSTable_A, SSTable_B, SSTable_C
    -   Level 1: SSTable_D
-   **After Compaction**:
    -   Level 0: (Empty or fewer SSTables)
    -   Level 1: SSTable_D, SSTable_E (compacted and sorted version of SSTable_A, SSTable_B, and SSTable_C)

----------

## Examples

### Size-Tiered Example

Imagine you have a bucket where you throw in tennis balls (small SSTables). When the bucket is full, you take all those tennis balls and compress them into a basketball (larger SSTable).

### Level-Tiered Example

Imagine you have multiple shelves (levels). New tennis balls (small SSTables) start on the lowest shelf. When the shelf is full, you compress some tennis balls into a basketball and move it to the next higher shelf, making space on the lower shelf for new tennis balls.

----------

## FAQs

-   **Are keys in SSTable sorted?**  
    Yes, keys in an SSTable are sorted.
    
-   **Are newly written data initially at Level 0 in Level-Tiered Compaction?**  
    Yes, and they are promoted to higher levels upon compaction.
    

----------
