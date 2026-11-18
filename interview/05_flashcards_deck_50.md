# 🗂️ The Data System Design & Lakehouse Architecture Handbook — Technical Flashcard Deck (50 Active Recall Questions)

> **Coverage**: Modules 01 through 10 of The Data System Design Handbook
> **Focus**: 50 Rapid-Fire Questions & Answers for Lakehouse Storage, Open Table Formats, Distributed Systems, and Architecture

---

### 1. What is a Modern Data Lakehouse?

**Answer:**  
An architecture that combines the flexibility and cost-effectiveness of cloud data lakes with the ACID transactions and governance of data warehouses.

---

### 2. What is Apache Parquet and why is it columnar?

**Answer:**  
Open-source columnar file format designed for efficient data storage and retrieval, organizing data by column to optimize read query I/O.

---

### 3. What is a Row Group in Parquet?

**Answer:**  
A logical horizontal partitioning of data into rows, containing column chunks for each column in the table (typically 128MB - 512MB).

---

### 4. What is dictionary encoding in Parquet?

**Answer:**  
Replaces repetitive string values with small integer keys, drastically reducing storage size and accelerating filter operations.

---

### 5. Difference between Snappy, GZIP, and ZSTD compression codecs?

**Answer:**  
Snappy prioritizes decompression speed; GZIP prioritizes compression ratio; ZSTD provides the best combination of high compression and fast decode.

---

### 6. What is the Medallion Architecture (Bronze, Silver, Gold)?

**Answer:**  
Data design pattern where data is progressively refined: Bronze (raw append-only), Silver (cleansed/deduplicated), Gold (aggregated business marts).

---

### 7. What is an Open Table Format?

**Answer:**  
A metadata specification (Delta Lake, Apache Iceberg, Apache Hudi) that brings ACID transactions and snapshot isolation to raw files on cloud storage.

---

### 8. How does the Delta Lake transaction log work?

**Answer:**  
Stores ordered JSON commit files (`000000.json`) under `_delta_log/`, periodically compacted into Parquet checkpoint files every 10 commits.

---

### 9. How does Apache Iceberg metadata tree work?

**Answer:**  
Hierarchical tree: Catalog points to current Snapshot metadata file, which points to Manifest List, which points to Manifest Files, which point to Data Files.

---

### 10. Difference between Delta Lake and Apache Iceberg metadata layout?

**Answer:**  
Delta uses an append-only linear JSON commit log with checkpoint files; Iceberg uses a tree of immutable Avro manifest snapshots.

---

### 11. What is Apache Hudi and what are its two storage types?

**Answer:**  
Table format with Copy-on-Write (CoW, rewrites Parquet on write) and Merge-on-Read (MoR, writes delta Avro files and merges at read time).

---

### 12. What are Deletion Vectors in Delta Lake and Iceberg?

**Answer:**  
Roaring bitmaps that track deleted rows within existing Parquet files without rewriting the entire data file (eliminating Copy-on-Write amplification).

---

### 13. What is Optimistic Concurrency Control (OCC)?

**Answer:**  
Concurrency protocol where transactions execute without locking, checking for conflicts only at commit time and failing/retrying if conflicts exist.

---

### 14. What constitutes a write-write conflict in Delta Lake?

**Answer:**  
Two concurrent transactions attempting to modify or delete the same files or overlapping partition slices simultaneously.

---

### 15. What is Liquid Clustering in Delta Lake?

**Answer:**  
Dynamic clustering algorithm that replaces rigid Hive partitioning and Z-Order, automatically clustering data along multiple columns incrementally.

---

### 16. Why does Hive-style partitioning (e.g. `/year/month/day/hour`) fail at scale?

**Answer:**  
Leads to the small-file problem (millions of tiny files) and massive metadata directory traversal bottlenecks on cloud object storage.

---

### 17. What is Z-Ordering and how does it multidimensionally cluster data?

**Answer:**  
Interleaves the bits of multiple column values along a Space-Filling Peano-Hilbert curve to preserve spatial locality across multiple columns.

---

### 18. What is the Small File Problem in data lakes?

**Answer:**  
Excessive tiny files (<10MB) creating enormous metadata overhead, slow directory listings, and severe object storage GET/LIST API latency.

---

### 19. What does `OPTIMIZE` / Bin-Packing do?

**Answer:**  
Compacts many small files into uniform large files (typically 1GB target size) to optimize query read performance.

---

### 20. What does `VACUUM` do in Delta Lake / Iceberg?

**Answer:**  
Permanently removes historical data files that have been deleted or superseded and fall outside the retention threshold (e.g. 7 days).

---

### 21. What happens if you run `VACUUM` with retention set to zero hours?

**Answer:**  
Active concurrent readers reading previous snapshots will experience crash failures (`FileNotFoundException`), and time travel is destroyed.

---

### 22. What is Time Travel in lakehouse engines?

**Answer:**  
Querying historical table snapshots using `VERSION AS OF` or `TIMESTAMP AS OF` by replaying the metadata state at that point in time.

---

### 23. What is Apache Iceberg Hidden Partitioning?

**Answer:**  
Partition transforms (e.g. `day(ts)`, `bucket(16, id)`) are tracked automatically by the engine without users having to explicitly query partition columns.

---

### 24. What is an Iceberg REST Catalog?

**Answer:**  
Standardized HTTP API specification allowing multi-engine compute (Spark, Trino, Snowflake, Flink) to securely access and mutate Iceberg metadata.

---

### 25. What is Unity Catalog in Databricks?

**Answer:**  
Universal governance and metadata catalog providing a 3-level namespace (`catalog.schema.table`), unified ACLs, and automated data lineage.

---

### 26. What is the PACELC theorem?

**Answer:**  
In a partitioned system (P), choose between Availability (A) and Consistency (C); else (E), choose between Latency (L) and Consistency (C).

---

### 27. What is the CAP theorem?

**Answer:**  
A distributed system can guarantee at most two out of Consistency, Availability, and Partition Tolerance simultaneously.

---

### 28. What is Raft consensus and how does it ensure consistency?

**Answer:**  
Leader-based consensus protocol where state changes are replicated to a majority quorum before being committed to the distributed log.

---

### 29. What is a Split-Brain scenario in distributed systems?

**Answer:**  
When network partition divides a cluster into two disconnected groups that both believe they are the leader, causing data divergence and corruption.

---

### 30. How do quorums prevent split-brain?

**Answer:**  
Requires strict majority vote (`N/2 + 1`); two disjoint partitions cannot both achieve a strict majority simultaneously.

---

### 31. What is Enterprise Data Mesh?

**Answer:**  
Decentralized sociotechnical data architecture treating data as a product, organized around domain-driven autonomous teams with federated governance.

---

### 32. What are the 4 core principles of Data Mesh?

**Answer:**  
Domain ownership, Data as a product, Self-serve data infrastructure platform, and Federated computational governance.

---

### 33. What is a Data Product in Data Mesh?

**Answer:**  
An autonomous, discoverable, trustworthy, and secure node on the mesh encapsulating code, data, metadata, infrastructure, and SLAs.

---

### 34. What is a Data Contract and what are its key components?

**Answer:**  
Contract between producer and consumer specifying Schema, Semantics, SLA (freshness, availability), and Quality constraints.

---

### 35. Difference between Data Mesh and Data Fabric?

**Answer:**  
Data Mesh is an organizational/architectural decentralization paradigm; Data Fabric is an automated technology-driven integration layer.

---

### 36. What is disaster recovery RPO (Recovery Point Objective)?

**Answer:**  
The maximum acceptable amount of data loss measured in time (e.g. 15 minutes of lost data) following an unplanned outage.

---

### 37. What is disaster recovery RTO (Recovery Time Objective)?

**Answer:**  
The maximum acceptable amount of downtime required to restore normal system operations following an outage.

---

### 38. Difference between Active-Passive and Active-Active lakehouse replication?

**Answer:**  
Active-Passive replicates to a standby DR region with failover lag; Active-Active serves reads and writes across multiple regions simultaneously.

---

### 39. How do cloud object storage eventual consistency models impact lakehouses?

**Answer:**  
Historically, eventual consistency caused stale reads; modern S3/ADLS Gen2 offer strong read-after-write consistency for PUTs and DELETEs.

---

### 40. What is Object Immutability and WORM (Write Once, Read Many) in cloud storage?

**Answer:**  
Retention policy (e.g. S3 Object Lock) that mathematically prevents files from being modified or deleted for a mandatory compliance duration.

---

### 41. How does partition pruning work in columnar query engines?

**Answer:**  
Reads metadata partition values to entirely skip scanning directories that do not match the query predicate.

---

### 42. How does file-level data skipping work (Min/Max statistics)?

**Answer:**  
Column statistics stored in metadata allow the engine to skip scanning entire Parquet files whose `min_val` and `max_val` do not overlap the predicate.

---

### 43. What is Bloom Filter indexing in modern lakehouses?

**Answer:**  
Probabilistic data structure that rapidly tests whether an element is definitely NOT in a file, eliminating unnecessary file reads for high-cardinality keys.

---

### 44. What is the difference between Logical Plan and Physical Plan in query execution?

**Answer:**  
Logical plan represents relational algebra transformations abstractly; physical plan dictates exact execution algorithms (e.g. Hash Join vs SortMerge Join).

---

### 45. What is Catalyst Optimizer in Apache Spark?

**Answer:**  
Extensible query optimizer that performs analysis, logical optimization, physical planning, and code generation for Spark SQL and DataFrames.

---

### 46. What is Adaptive Query Execution (AQE)?

**Answer:**  
Re-optimizes physical query plans at runtime based on accurate stage runtime statistics (coalescing shuffle partitions, switching to broadcast joins).

---

### 47. What is data skew in distributed processing?

**Answer:**  
Uneven distribution of data across partitions, causing a few straggler tasks to process orders of magnitude more data than others.

---

### 48. How does two-phase key salting resolve severe join skew?

**Answer:**  
Appends random salt (0..N-1) to skewed keys in table A and replicates rows in table B across all salt values, distributing the hot key evenly.

---

### 49. What is Shuffle Spill in Apache Spark and how do you diagnose it?

**Answer:**  
Occurs when memory execution buffers exceed executor RAM, forcing data to spill to local disk; diagnosed via Spark UI `Spill (Memory)` and `Spill (Disk)`.

---

### 50. What is the golden architectural rule for lakehouse file sizing?

**Answer:**  
Target Parquet file sizes between 128MB and 512MB: large enough to maximize columnar compression and I/O throughput, small enough for nimble tasks.

---
