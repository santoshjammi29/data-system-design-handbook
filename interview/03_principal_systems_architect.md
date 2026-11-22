# 🎓 The Data System Design & Lakehouse Architecture Handbook — Tier 3: Principal Systems Architect (Distributed Storage, PACELC & Resilience)

> **Focus Areas**: Distributed consensus (Raft/Paxos), PACELC theorem, cross-region replication, BCDR, Data Mesh domain decomposition.
> **Target Roles**: Principal Systems Architect, Enterprise Data Architect

---

### Q3.1: How do you resolve file system read timeouts ($403$ SlowDown errors) when thousands of concurrent Spark tasks evaluate file statistics across millions of immutable objects concurrently?

**Question & Production Scenario:**
In an enterprise modern data system design and lakehouse architecture, how do you resolve the following:
> How do you resolve file system read timeouts ($403$ SlowDown errors) when thousands of concurrent Spark tasks evaluate file statistics across millions of immutable objects concurrently?

**Detailed Technical Solution & Production Implementation:**
1. Rate Limiting: $503$ (or $403$) SlowDown errors indicate request throttling on the S3 prefix. This happens when Spark tasks concurrently query object metadata.

2. Resolution: (a) Increase the S3 request limit by adding random prefixes to paths. (b) Configure Spark query planning to rely on table formats (Iceberg/Delta) which store file statistics in metadata files, avoiding calling `listStatus` on S3 during query planning, eliminating S3 API timeouts.

**Production Implementation:**
```sql
-- Modern lakehouse table partitioning and retention configuration
ALTER TABLE prod_catalog.gold.fact_sales
SET TBLPROPERTIES (
    'delta.logRetentionDuration' = 'interval 30 days',
    'delta.deletedFileRetentionDuration' = 'interval 7 days'
);
OPTIMIZE prod_catalog.gold.fact_sales ZORDER BY (customer_id, transaction_date);
```

**Key Invariants & Architectural Gotchas:**
- Enforce atomic commits and ACID guarantees to eliminate dirty reads across multi-engine readers.
- Optimize file size layout (128MB - 512MB) to prevent object storage API rate limits.
- Design data pipelines around idempotent writes to enable deterministic disaster recovery replays.

**Certification Alignment:** `Enterprise Data Architect / AWS & Azure Solutions Architect Professional`

---

### Q3.2: How do you handle schema evolution at the storage layer when changing a field type from int32 to int64 inside an Avro-encoded landing zone before it transforms into a Parquet-based structured lake layer?

**Question & Production Scenario:**
In an enterprise modern data system design and lakehouse architecture, how do you resolve the following:
> How do you handle schema evolution at the storage layer when changing a field type from int32 to int64 inside an Avro-encoded landing zone before it transforms into a Parquet-based structured lake layer?

**Detailed Technical Solution & Production Implementation:**
1. Schema mapping: Avro supports type promotion (int32 is compatible with int64). The reader schema must define the promoted type as int64.

2. ETL Transformation: In the Spark/ETL pipeline, read the Avro files using the new schema configuration. Cast the column explicitly to long (`bigint`) during the load phase before writing to the Parquet table, ensuring the Parquet storage layer maintains consistent datatype definitions across partitions.

**Production Implementation:**
```sql
-- Modern lakehouse table partitioning and retention configuration
ALTER TABLE prod_catalog.gold.fact_sales
SET TBLPROPERTIES (
    'delta.logRetentionDuration' = 'interval 30 days',
    'delta.deletedFileRetentionDuration' = 'interval 7 days'
);
OPTIMIZE prod_catalog.gold.fact_sales ZORDER BY (customer_id, transaction_date);
```

**Key Invariants & Architectural Gotchas:**
- Enforce atomic commits and ACID guarantees to eliminate dirty reads across multi-engine readers.
- Optimize file size layout (128MB - 512MB) to prevent object storage API rate limits.
- Design data pipelines around idempotent writes to enable deterministic disaster recovery replays.

**Certification Alignment:** `Enterprise Data Architect / AWS & Azure Solutions Architect Professional`

---

### Q3.3: Design a storage layout and hashing strategy for a data lake storing high-frequency IoT timeseries data to guarantee uniform distribution across underlying physical storage partitions while keeping query ranges bounded by time windows.

**Question & Production Scenario:**
In an enterprise modern data system design and lakehouse architecture, how do you resolve the following:
> Design a storage layout and hashing strategy for a data lake storing high-frequency IoT timeseries data to guarantee uniform distribution across underlying physical storage partitions while keeping query ranges bounded by time windows.

**Detailed Technical Solution & Production Implementation:**
1. Storage Layout: Use a multi-tier pathing structure: `s3://iot-lake/hash_bucket={0-99}/year={yyyy}/month={mm}/day={dd}/`.

2. Hashing: Calculate the hash bucket as `hash(device_id) % 100`. This distributes devices uniformly across 100 prefixes, avoiding S3 rate limits.

3. Query Optimization: Queries filtering on time windows will only scan the date directories within the hash buckets. The query engine reads buckets in parallel, ensuring balanced I/O distribution without scanning unneeded folders.

**Production Implementation:**
```sql
-- Modern lakehouse table partitioning and retention configuration
ALTER TABLE prod_catalog.gold.fact_sales
SET TBLPROPERTIES (
    'delta.logRetentionDuration' = 'interval 30 days',
    'delta.deletedFileRetentionDuration' = 'interval 7 days'
);
OPTIMIZE prod_catalog.gold.fact_sales ZORDER BY (customer_id, transaction_date);
```

**Key Invariants & Architectural Gotchas:**
- Enforce atomic commits and ACID guarantees to eliminate dirty reads across multi-engine readers.
- Optimize file size layout (128MB - 512MB) to prevent object storage API rate limits.
- Design data pipelines around idempotent writes to enable deterministic disaster recovery replays.

**Certification Alignment:** `Enterprise Data Architect / AWS & Azure Solutions Architect Professional`

---

### Q3.4: Explain how the omission of the _SUCCESS file or atomic marker mechanisms in modern cloud object stores impacts data lineage validation tools that hook into low-level file listing APIs.

**Question & Production Scenario:**
In an enterprise modern data system design and lakehouse architecture, how do you resolve the following:
> Explain how the omission of the _SUCCESS file or atomic marker mechanisms in modern cloud object stores impacts data lineage validation tools that hook into low-level file listing APIs.

**Detailed Technical Solution & Production Implementation:**
1. Omission Impact: Hadoop legacy tools check for a `_SUCCESS` file before initiating downstream processing. Cloud native writing bypasses this step, committing transactions directly to a catalog.

2. Lineage Break: If lineage validation tools rely on scanning for these files, they will fail to trigger. To resolve this, hook lineage tools directly into the catalog (e.g., OpenLineage or Unity Catalog Event Logs) rather than scanning object storage file structures.

**Production Implementation:**
```sql
-- Modern lakehouse table partitioning and retention configuration
ALTER TABLE prod_catalog.gold.fact_sales
SET TBLPROPERTIES (
    'delta.logRetentionDuration' = 'interval 30 days',
    'delta.deletedFileRetentionDuration' = 'interval 7 days'
);
OPTIMIZE prod_catalog.gold.fact_sales ZORDER BY (customer_id, transaction_date);
```

**Key Invariants & Architectural Gotchas:**
- Enforce atomic commits and ACID guarantees to eliminate dirty reads across multi-engine readers.
- Optimize file size layout (128MB - 512MB) to prevent object storage API rate limits.
- Design data pipelines around idempotent writes to enable deterministic disaster recovery replays.

**Certification Alignment:** `Enterprise Data Architect / AWS & Azure Solutions Architect Professional`

---

### Q3.5: What are the structural and execution consequences of configuring parquet.enable.dictionary to true for high-cardinality UUID columns in a multi-petabyte dataset?

**Question & Production Scenario:**
In an enterprise modern data system design and lakehouse architecture, how do you resolve the following:
> What are the structural and execution consequences of configuring parquet.enable.dictionary to true for high-cardinality UUID columns in a multi-petabyte dataset?

**Detailed Technical Solution & Production Implementation:**
1. Dictionary Bloat: High-cardinality UUID columns contain unique values for every row. Configuring dictionary encoding forces the engine to build a massive key dictionary in memory.

2. Performance Degradation: The dictionary will quickly exceed the page size limit (1MB). Parquet will fall back to plaintext encoding, but the memory allocated to the dictionary is lost. This causes garbage collection pressure, high memory consumption, and slower write/read execution speeds.

**Production Implementation:**
```sql
-- Modern lakehouse table partitioning and retention configuration
ALTER TABLE prod_catalog.gold.fact_sales
SET TBLPROPERTIES (
    'delta.logRetentionDuration' = 'interval 30 days',
    'delta.deletedFileRetentionDuration' = 'interval 7 days'
);
OPTIMIZE prod_catalog.gold.fact_sales ZORDER BY (customer_id, transaction_date);
```

**Key Invariants & Architectural Gotchas:**
- Enforce atomic commits and ACID guarantees to eliminate dirty reads across multi-engine readers.
- Optimize file size layout (128MB - 512MB) to prevent object storage API rate limits.
- Design data pipelines around idempotent writes to enable deterministic disaster recovery replays.

**Certification Alignment:** `Enterprise Data Architect / AWS & Azure Solutions Architect Professional`

---

### Q3.6: How do you design an optimal multi-tier data lake storage architecture (Hot/Cool/Cold) that preserves query capabilities via external metadata pointers without incurring catastrophic data hydration latency penalties?

**Question & Production Scenario:**
In an enterprise modern data system design and lakehouse architecture, how do you resolve the following:
> How do you design an optimal multi-tier data lake storage architecture (Hot/Cool/Cold) that preserves query capabilities via external metadata pointers without incurring catastrophic data hydration latency penalties?

**Detailed Technical Solution & Production Implementation:**
1. Table Format Pointers: Use Iceberg/Delta formats which separate metadata from data files. Store the table metadata files in the central catalog.

2. Lifecycle Rules: Configure storage lifecycle rules to move physical Parquet files to Cool or Cold tiers after a specific duration (e.g., 90 days).

3. Query Optimization: The catalog continues pointing to the cold files. When a query scans cold data, engines like Trino use range-requests to read only the page footer index, avoiding full file download (hydration) and minimizing retrieval latency.

**Production Implementation:**
```sql
-- Modern lakehouse table partitioning and retention configuration
ALTER TABLE prod_catalog.gold.fact_sales
SET TBLPROPERTIES (
    'delta.logRetentionDuration' = 'interval 30 days',
    'delta.deletedFileRetentionDuration' = 'interval 7 days'
);
OPTIMIZE prod_catalog.gold.fact_sales ZORDER BY (customer_id, transaction_date);
```

**Key Invariants & Architectural Gotchas:**
- Enforce atomic commits and ACID guarantees to eliminate dirty reads across multi-engine readers.
- Optimize file size layout (128MB - 512MB) to prevent object storage API rate limits.
- Design data pipelines around idempotent writes to enable deterministic disaster recovery replays.

**Certification Alignment:** `Enterprise Data Architect / AWS & Azure Solutions Architect Professional`

---

### Q3.7: Detail the synchronization and conflict resolution process of Delta Lake's Multi-Cluster Writes via a transactional log store (e.g., DynamoDB or native cloud storage conditions). How does it handle a split-brain scenario where two independent clusters attempt to commit version $N+1$ simultaneously?

**Question & Production Scenario:**
In an enterprise modern data system design and lakehouse architecture, how do you resolve the following:
> Detail the synchronization and conflict resolution process of Delta Lake's Multi-Cluster Writes via a transactional log store (e.g., DynamoDB or native cloud storage conditions). How does it handle a split-brain scenario where two independent clusters attempt to commit version $N+1$ simultaneously?

**Detailed Technical Solution & Production Implementation:**
1. Lock coordination: Delta Lake relies on atomic write operations (put-if-absent) in the object store. If the store does not support this natively (like S3), Delta uses a external coordinator (e.g., DynamoDB).

2. Split-Brain Resolution: When two clusters attempt to commit version `N+1`, they write a lease to the coordinator. The coordinator grants the commit to the first request. The second request receives a commit conflict exception. The second cluster must reload the latest log state (`N+1`), verify if its write overlaps with the committed change (schema validation), and retry by committing as version `N+2`.

**Production Implementation:**
```sql
-- Delta Lake 3.x table with Liquid Clustering and Deletion Vectors
CREATE TABLE prod_catalog.gold.customer_events (
    event_id STRING,
    user_id BIGINT,
    event_timestamp TIMESTAMP,
    payload STRING
)
USING DELTA
CLUSTER BY (user_id, event_timestamp)
TBLPROPERTIES (
    'delta.enableDeletionVectors' = 'true',
    'delta.autoOptimize.optimizeWrite' = 'true',
    'delta.autoOptimize.autoCompact' = 'true'
);
```

**Key Invariants & Architectural Gotchas:**
- Enforce atomic commits and ACID guarantees to eliminate dirty reads across multi-engine readers.
- Optimize file size layout (128MB - 512MB) to prevent object storage API rate limits.
- Design data pipelines around idempotent writes to enable deterministic disaster recovery replays.

**Certification Alignment:** `Enterprise Data Architect / AWS & Azure Solutions Architect Professional`

---

### Q3.8: What are the internal failure mechanics when a Delta Lake table undergoes a massive schema migration (e.g., changing a complex struct column's internal data types) while a Spark Structured Streaming application is concurrently executing against that table with mergeSchema enabled?

**Question & Production Scenario:**
In an enterprise modern data system design and lakehouse architecture, how do you resolve the following:
> What are the internal failure mechanics when a Delta Lake table undergoes a massive schema migration (e.g., changing a complex struct column's internal data types) while a Spark Structured Streaming application is concurrently executing against that table with mergeSchema enabled?

**Detailed Technical Solution & Production Implementation:**
1. Streaming Schema Collision: The active streaming job encounters unmatched schema structures. It attempts to merge the schema. If another transaction locks the catalog, the stream fails with a commit conflict, requiring stream restarts.

**Production Implementation:**
```sql
-- Delta Lake 3.x table with Liquid Clustering and Deletion Vectors
CREATE TABLE prod_catalog.gold.customer_events (
    event_id STRING,
    user_id BIGINT,
    event_timestamp TIMESTAMP,
    payload STRING
)
USING DELTA
CLUSTER BY (user_id, event_timestamp)
TBLPROPERTIES (
    'delta.enableDeletionVectors' = 'true',
    'delta.autoOptimize.optimizeWrite' = 'true',
    'delta.autoOptimize.autoCompact' = 'true'
);
```

**Key Invariants & Architectural Gotchas:**
- Enforce atomic commits and ACID guarantees to eliminate dirty reads across multi-engine readers.
- Optimize file size layout (128MB - 512MB) to prevent object storage API rate limits.
- Design data pipelines around idempotent writes to enable deterministic disaster recovery replays.

**Certification Alignment:** `Enterprise Data Architect / AWS & Azure Solutions Architect Professional`

---

### Q3.9: Contrast the architectural implementation of Time Travel queries in Iceberg (via snapshot IDs and timestamps in the metadata tree) with Delta Lake (via file log history and checkpoint parsing). What is the maximum scalability ceiling of metadata-driven time travel?

**Question & Production Scenario:**
In an enterprise modern data system design and lakehouse architecture, how do you resolve the following:
> Contrast the architectural implementation of Time Travel queries in Iceberg (via snapshot IDs and timestamps in the metadata tree) with Delta Lake (via file log history and checkpoint parsing). What is the maximum scalability ceiling of metadata-driven time travel?

**Detailed Technical Solution & Production Implementation:**
1. Iceberg: Queries lookup target snapshot IDs in metadata. Scalable. Delta: parses JSON log files back to the target version. Scale limit is metadata size; listing millions of versions causes planning out-of-memory errors.

**Production Implementation:**
```sql
-- Apache Iceberg table definition with hidden partitioning and Z-Order
CREATE TABLE prod_catalog.sales.orders (
    order_id BIGINT,
    customer_id STRING,
    order_ts TIMESTAMP,
    order_amount DECIMAL(18, 2)
)
USING iceberg
PARTITIONED BY (days(order_ts))
TBLPROPERTIES (
    'write.format.default' = 'parquet',
    'write.parquet.compression-codec' = 'zstd',
    'commit.retry.num-retries' = '10'
);
```

**Key Invariants & Architectural Gotchas:**
- Enforce atomic commits and ACID guarantees to eliminate dirty reads across multi-engine readers.
- Optimize file size layout (128MB - 512MB) to prevent object storage API rate limits.
- Design data pipelines around idempotent writes to enable deterministic disaster recovery replays.

**Certification Alignment:** `Enterprise Data Architect / AWS & Azure Solutions Architect Professional`

---

### Q3.10: How do you troubleshoot and eliminate the "Metadata Bloat" phenomenon in an Iceberg table that experiences continuous micro-batch streaming appends every 10 seconds, causing query planning time to exceed actual data execution time?

**Question & Production Scenario:**
In an enterprise modern data system design and lakehouse architecture, how do you resolve the following:
> How do you troubleshoot and eliminate the "Metadata Bloat" phenomenon in an Iceberg table that experiences continuous micro-batch streaming appends every 10 seconds, causing query planning time to exceed actual data execution time?

**Detailed Technical Solution & Production Implementation:**
1. Bloat: Thousands of snapshots block planning. Eliminate by calling the `expireSnapshots` procedure periodically to purge old snapshots and manifest files, keeping the metadata tree slim.

**Production Implementation:**
```sql
-- Apache Iceberg table definition with hidden partitioning and Z-Order
CREATE TABLE prod_catalog.sales.orders (
    order_id BIGINT,
    customer_id STRING,
    order_ts TIMESTAMP,
    order_amount DECIMAL(18, 2)
)
USING iceberg
PARTITIONED BY (days(order_ts))
TBLPROPERTIES (
    'write.format.default' = 'parquet',
    'write.parquet.compression-codec' = 'zstd',
    'commit.retry.num-retries' = '10'
);
```

**Key Invariants & Architectural Gotchas:**
- Enforce atomic commits and ACID guarantees to eliminate dirty reads across multi-engine readers.
- Optimize file size layout (128MB - 512MB) to prevent object storage API rate limits.
- Design data pipelines around idempotent writes to enable deterministic disaster recovery replays.

**Certification Alignment:** `Enterprise Data Architect / AWS & Azure Solutions Architect Professional`

---

### Q3.11: Explain how Apache Hudi’s Metadata Table (mdt) internally replaces the traditional file listing operations, how it uses an internal HFile format, and how you recover from an out-of-sync or corrupted metadata table index.

**Question & Production Scenario:**
In an enterprise modern data system design and lakehouse architecture, how do you resolve the following:
> Explain how Apache Hudi’s Metadata Table (mdt) internally replaces the traditional file listing operations, how it uses an internal HFile format, and how you recover from an out-of-sync or corrupted metadata table index.

**Detailed Technical Solution & Production Implementation:**
1. Metadata Table: Stores file lists in HFile format, bypassing S3 listings. If corrupted, delete the `.hoodie/metadata` directory and rebuild it from the commit logs.

**Production Implementation:**
```sql
-- Modern lakehouse table partitioning and retention configuration
ALTER TABLE prod_catalog.gold.fact_sales
SET TBLPROPERTIES (
    'delta.logRetentionDuration' = 'interval 30 days',
    'delta.deletedFileRetentionDuration' = 'interval 7 days'
);
OPTIMIZE prod_catalog.gold.fact_sales ZORDER BY (customer_id, transaction_date);
```

**Key Invariants & Architectural Gotchas:**
- Enforce atomic commits and ACID guarantees to eliminate dirty reads across multi-engine readers.
- Optimize file size layout (128MB - 512MB) to prevent object storage API rate limits.
- Design data pipelines around idempotent writes to enable deterministic disaster recovery replays.

**Certification Alignment:** `Enterprise Data Architect / AWS & Azure Solutions Architect Professional`

---

### Q3.12: What are the internal locking and state transition mechanisms when converting a massive, unmanaged legacy Parquet directory structure into an Apache Iceberg table in-place using the migrate vs. add_files procedures?

**Question & Production Scenario:**
In an enterprise modern data system design and lakehouse architecture, how do you resolve the following:
> What are the internal locking and state transition mechanisms when converting a massive, unmanaged legacy Parquet directory structure into an Apache Iceberg table in-place using the migrate vs. add_files procedures?

**Detailed Technical Solution & Production Implementation:**
1. Migrate: Overwrites the old table, creating new metadata and deleting the old table schema. `add_files`: Metadata-only commit that appends existing Parquet paths to the Iceberg manifest list, preserving original files in-place.

**Production Implementation:**
```sql
-- Apache Iceberg table definition with hidden partitioning and Z-Order
CREATE TABLE prod_catalog.sales.orders (
    order_id BIGINT,
    customer_id STRING,
    order_ts TIMESTAMP,
    order_amount DECIMAL(18, 2)
)
USING iceberg
PARTITIONED BY (days(order_ts))
TBLPROPERTIES (
    'write.format.default' = 'parquet',
    'write.parquet.compression-codec' = 'zstd',
    'commit.retry.num-retries' = '10'
);
```

**Key Invariants & Architectural Gotchas:**
- Enforce atomic commits and ACID guarantees to eliminate dirty reads across multi-engine readers.
- Optimize file size layout (128MB - 512MB) to prevent object storage API rate limits.
- Design data pipelines around idempotent writes to enable deterministic disaster recovery replays.

**Certification Alignment:** `Enterprise Data Architect / AWS & Azure Solutions Architect Professional`

---

### Q3.13: How does Delta Lake implement Liquid Clustering, how does it physically break away from traditional hive-style partitioning, and what are the deterministic runtime rules it uses to determine when and how data blocks are clustered?

**Question & Production Scenario:**
In an enterprise modern data system design and lakehouse architecture, how do you resolve the following:
> How does Delta Lake implement Liquid Clustering, how does it physically break away from traditional hive-style partitioning, and what are the deterministic runtime rules it uses to determine when and how data blocks are clustered?

**Detailed Technical Solution & Production Implementation:**
1. Liquid Clustering: Bypasses standard partition folders. Groups data using dynamic clustering keys. Background processes read file statistics, map keys on a Z-Order curve, and merge matching ranges into balanced Parquet files.

**Production Implementation:**
```sql
-- Delta Lake 3.x table with Liquid Clustering and Deletion Vectors
CREATE TABLE prod_catalog.gold.customer_events (
    event_id STRING,
    user_id BIGINT,
    event_timestamp TIMESTAMP,
    payload STRING
)
USING DELTA
CLUSTER BY (user_id, event_timestamp)
TBLPROPERTIES (
    'delta.enableDeletionVectors' = 'true',
    'delta.autoOptimize.optimizeWrite' = 'true',
    'delta.autoOptimize.autoCompact' = 'true'
);
```

**Key Invariants & Architectural Gotchas:**
- Enforce atomic commits and ACID guarantees to eliminate dirty reads across multi-engine readers.
- Optimize file size layout (128MB - 512MB) to prevent object storage API rate limits.
- Design data pipelines around idempotent writes to enable deterministic disaster recovery replays.

**Certification Alignment:** `Enterprise Data Architect / AWS & Azure Solutions Architect Professional`

---

### Q3.14: How do you handle schema evolution tracking within a data catalog when an upstream system sends incompatible structural modifications (e.g., changing a column from a primitive to a struct), ensuring downstream consumer pipelines do not break?

**Question & Production Scenario:**
In an enterprise modern data system design and lakehouse architecture, how do you resolve the following:
> How do you handle schema evolution tracking within a data catalog when an upstream system sends incompatible structural modifications (e.g., changing a column from a primitive to a struct), ensuring downstream consumer pipelines do not break?

**Detailed Technical Solution & Production Implementation:**
1. Schema quarantine: Configure Glue/metastore schema validation gates. If an incompatible change is detected, block catalog updates, write the new data to a quarantine folder, and alert data engineers for manual mapping.

**Production Implementation:**
```sql
-- Apache Iceberg table definition with hidden partitioning and Z-Order
CREATE TABLE prod_catalog.sales.orders (
    order_id BIGINT,
    customer_id STRING,
    order_ts TIMESTAMP,
    order_amount DECIMAL(18, 2)
)
USING iceberg
PARTITIONED BY (days(order_ts))
TBLPROPERTIES (
    'write.format.default' = 'parquet',
    'write.parquet.compression-codec' = 'zstd',
    'commit.retry.num-retries' = '10'
);
```

**Key Invariants & Architectural Gotchas:**
- Enforce atomic commits and ACID guarantees to eliminate dirty reads across multi-engine readers.
- Optimize file size layout (128MB - 512MB) to prevent object storage API rate limits.
- Design data pipelines around idempotent writes to enable deterministic disaster recovery replays.

**Certification Alignment:** `Enterprise Data Architect / AWS & Azure Solutions Architect Professional`

---

### Q3.15: Explain how catalog-level caching mechanisms in distributed query engines interact with the physical freshness of open table format metadata files. How do you prevent stale metadata reads without completely disabling caching?

**Question & Production Scenario:**
In an enterprise modern data system design and lakehouse architecture, how do you resolve the following:
> Explain how catalog-level caching mechanisms in distributed query engines interact with the physical freshness of open table format metadata files. How do you prevent stale metadata reads without completely disabling caching?

**Detailed Technical Solution & Production Implementation:**
1. Metadata Freshness: Query engines cache table metadata files to speed up planning. To prevent stale reads, configure TTL thresholds (e.g., 10 seconds) or configure the catalog to push update notifications to the engine cache.

**Production Implementation:**
```sql
-- Apache Iceberg table definition with hidden partitioning and Z-Order
CREATE TABLE prod_catalog.sales.orders (
    order_id BIGINT,
    customer_id STRING,
    order_ts TIMESTAMP,
    order_amount DECIMAL(18, 2)
)
USING iceberg
PARTITIONED BY (days(order_ts))
TBLPROPERTIES (
    'write.format.default' = 'parquet',
    'write.parquet.compression-codec' = 'zstd',
    'commit.retry.num-retries' = '10'
);
```

**Key Invariants & Architectural Gotchas:**
- Enforce atomic commits and ACID guarantees to eliminate dirty reads across multi-engine readers.
- Optimize file size layout (128MB - 512MB) to prevent object storage API rate limits.
- Design data pipelines around idempotent writes to enable deterministic disaster recovery replays.

**Certification Alignment:** `Enterprise Data Architect / AWS & Azure Solutions Architect Professional`

---

### Q3.16: What are the structural and network failure scenarios when a cross-account data share via Delta Sharing protocol or Iceberg Catalog federation encounters transient network partition events between the provider and consumer tenants?

**Question & Production Scenario:**
In an enterprise modern data system design and lakehouse architecture, how do you resolve the following:
> What are the structural and network failure scenarios when a cross-account data share via Delta Sharing protocol or Iceberg Catalog federation encounters transient network partition events between the provider and consumer tenants?

**Detailed Technical Solution & Production Implementation:**
1. Share Failure: Reader compute nodes cannot resolve metadata tokens or contact the provider's token vendoring API, causing execution timeouts. Mitigate by implementing local token caching on consumer coordinators.

**Production Implementation:**
```sql
-- Apache Iceberg table definition with hidden partitioning and Z-Order
CREATE TABLE prod_catalog.sales.orders (
    order_id BIGINT,
    customer_id STRING,
    order_ts TIMESTAMP,
    order_amount DECIMAL(18, 2)
)
USING iceberg
PARTITIONED BY (days(order_ts))
TBLPROPERTIES (
    'write.format.default' = 'parquet',
    'write.parquet.compression-codec' = 'zstd',
    'commit.retry.num-retries' = '10'
);
```

**Key Invariants & Architectural Gotchas:**
- Enforce atomic commits and ACID guarantees to eliminate dirty reads across multi-engine readers.
- Optimize file size layout (128MB - 512MB) to prevent object storage API rate limits.
- Design data pipelines around idempotent writes to enable deterministic disaster recovery replays.

**Certification Alignment:** `Enterprise Data Architect / AWS & Azure Solutions Architect Professional`

---

### Q3.17: How do you design a disaster recovery strategy for an enterprise data catalog that guarantees catalog-state point-in-time recovery to exactly match the underlying point-in-time state of the physical files in immutable storage buckets?

**Question & Production Scenario:**
In an enterprise modern data system design and lakehouse architecture, how do you resolve the following:
> How do you design a disaster recovery strategy for an enterprise data catalog that guarantees catalog-state point-in-time recovery to exactly match the underlying point-in-time state of the physical files in immutable storage buckets?

**Detailed Technical Solution & Production Implementation:**
1. Backup Alignment: Snapshot the catalog relational database (Postgres) and the storage bucket concurrently. During recovery, restore both to the matching timestamp, ensuring catalog schemas align with physical files on disk.

**Production Implementation:**
```sql
-- Apache Iceberg table definition with hidden partitioning and Z-Order
CREATE TABLE prod_catalog.sales.orders (
    order_id BIGINT,
    customer_id STRING,
    order_ts TIMESTAMP,
    order_amount DECIMAL(18, 2)
)
USING iceberg
PARTITIONED BY (days(order_ts))
TBLPROPERTIES (
    'write.format.default' = 'parquet',
    'write.parquet.compression-codec' = 'zstd',
    'commit.retry.num-retries' = '10'
);
```

**Key Invariants & Architectural Gotchas:**
- Enforce atomic commits and ACID guarantees to eliminate dirty reads across multi-engine readers.
- Optimize file size layout (128MB - 512MB) to prevent object storage API rate limits.
- Design data pipelines around idempotent writes to enable deterministic disaster recovery replays.

**Certification Alignment:** `Enterprise Data Architect / AWS & Azure Solutions Architect Professional`

---

### Q3.18: How does the choice of catalog implementation influence the efficiency of data discovery, automated data profiling, and data quality metric collection across a multi-petabyte data lake environment?

**Question & Production Scenario:**
In an enterprise modern data system design and lakehouse architecture, how do you resolve the following:
> How does the choice of catalog implementation influence the efficiency of data discovery, automated data profiling, and data quality metric collection across a multi-petabyte data lake environment?

**Detailed Technical Solution & Production Implementation:**
1. Catalog Impact: Catalogs with built-in metadata APIs (like Unity Catalog) allow profiling tools to scan file statistics directly from logs, bypassing expensive data scan jobs on S3, reducing profiling costs.

**Production Implementation:**
```sql
-- Apache Iceberg table definition with hidden partitioning and Z-Order
CREATE TABLE prod_catalog.sales.orders (
    order_id BIGINT,
    customer_id STRING,
    order_ts TIMESTAMP,
    order_amount DECIMAL(18, 2)
)
USING iceberg
PARTITIONED BY (days(order_ts))
TBLPROPERTIES (
    'write.format.default' = 'parquet',
    'write.parquet.compression-codec' = 'zstd',
    'commit.retry.num-retries' = '10'
);
```

**Key Invariants & Architectural Gotchas:**
- Enforce atomic commits and ACID guarantees to eliminate dirty reads across multi-engine readers.
- Optimize file size layout (128MB - 512MB) to prevent object storage API rate limits.
- Design data pipelines around idempotent writes to enable deterministic disaster recovery replays.

**Certification Alignment:** `Enterprise Data Architect / AWS & Azure Solutions Architect Professional`

---

### Q3.19: Detail how you would implement an automated, policy-driven data retention and purging engine directly inside the data lake catalog that scans for expired datasets and executes cryptographic deletion of corresponding storage keys.

**Question & Production Scenario:**
In an enterprise modern data system design and lakehouse architecture, how do you resolve the following:
> Detail how you would implement an automated, policy-driven data retention and purging engine directly inside the data lake catalog that scans for expired datasets and executes cryptographic deletion of corresponding storage keys.

**Detailed Technical Solution & Production Implementation:**
1. Crypto-Deletion Engine: Schedule a daily job in the catalog. It scans metadata for tables exceeding retention limits. It calls Key Vault to destroy the cryptographic keys associated with the target tables, rendering data unreadable instantly.

**Production Implementation:**
```sql
-- Apache Iceberg table definition with hidden partitioning and Z-Order
CREATE TABLE prod_catalog.sales.orders (
    order_id BIGINT,
    customer_id STRING,
    order_ts TIMESTAMP,
    order_amount DECIMAL(18, 2)
)
USING iceberg
PARTITIONED BY (days(order_ts))
TBLPROPERTIES (
    'write.format.default' = 'parquet',
    'write.parquet.compression-codec' = 'zstd',
    'commit.retry.num-retries' = '10'
);
```

**Key Invariants & Architectural Gotchas:**
- Enforce atomic commits and ACID guarantees to eliminate dirty reads across multi-engine readers.
- Optimize file size layout (128MB - 512MB) to prevent object storage API rate limits.
- Design data pipelines around idempotent writes to enable deterministic disaster recovery replays.

**Certification Alignment:** `Enterprise Data Architect / AWS & Azure Solutions Architect Professional`

---

### Q3.20: Detail how you would design a multi-tenant resource allocation strategy using Trino Resource Groups or Spark Capacity Schedulers to guarantee data engineering pipeline SLAs while accommodating unpredictable ad-hoc data scientist queries.

**Question & Production Scenario:**
In an enterprise modern data system design and lakehouse architecture, how do you resolve the following:
> Detail how you would design a multi-tenant resource allocation strategy using Trino Resource Groups or Spark Capacity Schedulers to guarantee data engineering pipeline SLAs while accommodating unpredictable ad-hoc data scientist queries.

**Detailed Technical Solution & Production Implementation:**
1. Resource Groups: Configure Trino resource groups. Assign production pipelines to a high-priority group with guaranteed CPU shares (e.g., 70%). Assign ad-hoc queries to a low-priority group with a hard concurrency limit (e.g., 5 queries), protecting SLAs.

**Production Implementation:**
```sql
-- Modern lakehouse table partitioning and retention configuration
ALTER TABLE prod_catalog.gold.fact_sales
SET TBLPROPERTIES (
    'delta.logRetentionDuration' = 'interval 30 days',
    'delta.deletedFileRetentionDuration' = 'interval 7 days'
);
OPTIMIZE prod_catalog.gold.fact_sales ZORDER BY (customer_id, transaction_date);
```

**Key Invariants & Architectural Gotchas:**
- Enforce atomic commits and ACID guarantees to eliminate dirty reads across multi-engine readers.
- Optimize file size layout (128MB - 512MB) to prevent object storage API rate limits.
- Design data pipelines around idempotent writes to enable deterministic disaster recovery replays.

**Certification Alignment:** `Enterprise Data Architect / AWS & Azure Solutions Architect Professional`

---

### Q3.21: Architect a multi-cloud lakehouse deployment where compute engines on AWS and GCP simultaneously read and write to the same central Iceberg tables without data duplication?

**Question & Production Scenario:**
In an enterprise modern data system design and lakehouse architecture, how do you resolve the following:
> Architect a multi-cloud lakehouse deployment where compute engines on AWS and GCP simultaneously read and write to the same central Iceberg tables without data duplication?

**Detailed Technical Solution & Production Implementation:**
In enterprise multi-cloud architectures, running redundant pipelines in both AWS and Google Cloud Platform (GCP) doubles storage costs and introduces data divergence.

### Multi-Cloud Zero-Copy Architecture:
1. **Authoritative Storage & Regional Anchor**:
   - Anchor authoritative data storage in a single primary cloud with lowest egress rates (e.g., AWS S3 in `us-east-1`).
2. **Open REST Catalog with Cross-Cloud IAM (Apache Polaris / Nessie)**:
   - Host an Iceberg REST Catalog accessible over private interconnects (AWS Direct Connect + GCP Cloud Interconnect).
   - Use Cloud STS (Security Token Service) to exchange GCP service account identities for temporary presigned AWS S3 read/write URLs dynamically.
3. **Concurrent Multi-Cloud Writes**:
   - Google Cloud Dataproc / BigQuery and AWS EMR / Databricks commit transactions to the central REST Catalog.
   - The catalog serializes commits using atomic CAS (Compare-And-Swap) operations on metadata pointers.
4. **Mitigating Inter-Cloud Egress**:
   - For latency-sensitive analytical queries on GCP BigQuery, deploy an automated replication daemon utilizing **Delta/Iceberg Shallow Clones** or cloud storage cross-region batch caching to avoid incurring egress fees on repetitive BI scans.

**Production Implementation:**
```sql
-- Apache Iceberg table definition with hidden partitioning and Z-Order
CREATE TABLE prod_catalog.sales.orders (
    order_id BIGINT,
    customer_id STRING,
    order_ts TIMESTAMP,
    order_amount DECIMAL(18, 2)
)
USING iceberg
PARTITIONED BY (days(order_ts))
TBLPROPERTIES (
    'write.format.default' = 'parquet',
    'write.parquet.compression-codec' = 'zstd',
    'commit.retry.num-retries' = '10'
);
```

**Key Invariants & Architectural Gotchas:**
- Enforce atomic commits and ACID guarantees to eliminate dirty reads across multi-engine readers.
- Optimize file size layout (128MB - 512MB) to prevent object storage API rate limits.
- Design data pipelines around idempotent writes to enable deterministic disaster recovery replays.

**Certification Alignment:** `Enterprise Data Architect / AWS & Azure Solutions Architect Professional`

---

### Q3.22: How do you design a custom catalog implementation for Apache Iceberg optimized for billions of partitions and ultra-low latency metadata resolution?

**Question & Production Scenario:**
In an enterprise modern data system design and lakehouse architecture, how do you resolve the following:
> How do you design a custom catalog implementation for Apache Iceberg optimized for billions of partitions and ultra-low latency metadata resolution?

**Detailed Technical Solution & Production Implementation:**
Standard relational-backed catalogs (JDBC Catalog on PostgreSQL) degrade when managing tables with billions of partitions and millions of manifest files.

### Custom Catalog Blueprint:
1. **Adopting the Iceberg REST Catalog Specification**:
   - Implement the standard Iceberg REST OpenAPI specification, decoupling query engines from database internals.
2. **Distributed Distributed Key-Value Metadata Store (FoundationDB / DynamoDB)**:
   - Store table metadata pointers in FoundationDB or Amazon DynamoDB.
   - Use atomic conditional transactions (`AttributeExists`, `ConditionExpression`) to implement sub-millisecond atomic commit swapping (`commitTable`).
3. **Hierarchical Manifest Caching in Redis**:
   - Cache serialized manifest lists and file partition statistics in an in-memory Redis cluster.
   - When a query engine requests metadata planning, the REST catalog resolves partition pruning server-side and streams back only the pruned file list via gRPC/HTTP/2.
4. **Bloom Filter Metadata Skipping**:
   - Embed Bloom filters into custom catalog manifest files to resolve high-cardinality point lookups in under 10 milliseconds.

**Production Implementation:**
```sql
-- Apache Iceberg table definition with hidden partitioning and Z-Order
CREATE TABLE prod_catalog.sales.orders (
    order_id BIGINT,
    customer_id STRING,
    order_ts TIMESTAMP,
    order_amount DECIMAL(18, 2)
)
USING iceberg
PARTITIONED BY (days(order_ts))
TBLPROPERTIES (
    'write.format.default' = 'parquet',
    'write.parquet.compression-codec' = 'zstd',
    'commit.retry.num-retries' = '10'
);
```

**Key Invariants & Architectural Gotchas:**
- Enforce atomic commits and ACID guarantees to eliminate dirty reads across multi-engine readers.
- Optimize file size layout (128MB - 512MB) to prevent object storage API rate limits.
- Design data pipelines around idempotent writes to enable deterministic disaster recovery replays.

**Certification Alignment:** `Enterprise Data Architect / AWS & Azure Solutions Architect Professional`

---

### Q3.23: Design a real-time, highly concurrent fraud detection system utilizing Merge-On-Read tables optimized for millisecond-level point lookups within a petabyte-scale lakehouse?

**Question & Production Scenario:**
In an enterprise modern data system design and lakehouse architecture, how do you resolve the following:
> Design a real-time, highly concurrent fraud detection system utilizing Merge-On-Read tables optimized for millisecond-level point lookups within a petabyte-scale lakehouse?

**Detailed Technical Solution & Production Implementation:**
Fraud detection requires real-time evaluation (<50ms) of incoming credit card transactions against historical petabyte-scale lakehouse behavioral profiles.

### Architecture Blueprint:
1. **Storage Format**: Apache Iceberg with Merge-On-Read (MOR) format v2 and equality delete files.
2. **Two-Tier Engine Topology**:
   - **Ingestion & Feature Engine**: Apache Flink processes real-time transaction streams from Kafka, updating user risk profiles in the Iceberg table using lightweight delete files.
   - **Point-Lookup Query Layer**: Deploy an ultra-fast distributed OLAP engine (StarRocks / Trino) reading Iceberg metadata directly with in-memory caching.
3. **Local Secondary Indexing (Puffin Files)**:
   - Generate Apache Iceberg Puffin files storing Bloom filters on `credit_card_hash` and `account_id` per Parquet data file.
4. **Execution Flow**:
   - When a swipe occurs, the fraud engine issues a point query:
     `SELECT * FROM user_profiles WHERE account_id = 'A987654'`
   - The query engine uses the Bloom filter to pinpoint the exact 1 Parquet file out of 500,000 files in 8 milliseconds, merges the in-memory delete vector, and returns the profile in sub-30ms.

**Production Implementation:**
```sql
-- Modern lakehouse table partitioning and retention configuration
ALTER TABLE prod_catalog.gold.fact_sales
SET TBLPROPERTIES (
    'delta.logRetentionDuration' = 'interval 30 days',
    'delta.deletedFileRetentionDuration' = 'interval 7 days'
);
OPTIMIZE prod_catalog.gold.fact_sales ZORDER BY (customer_id, transaction_date);
```

**Key Invariants & Architectural Gotchas:**
- Enforce atomic commits and ACID guarantees to eliminate dirty reads across multi-engine readers.
- Optimize file size layout (128MB - 512MB) to prevent object storage API rate limits.
- Design data pipelines around idempotent writes to enable deterministic disaster recovery replays.

**Certification Alignment:** `Enterprise Data Architect / AWS & Azure Solutions Architect Professional`

---

### Q3.24: Architect a branch-and-merge framework using Project Nessie or Delta Lake branching to allow data science teams to perform isolated experiments on production data structures?

**Question & Production Scenario:**
In an enterprise modern data system design and lakehouse architecture, how do you resolve the following:
> Architect a branch-and-merge framework using Project Nessie or Delta Lake branching to allow data science teams to perform isolated experiments on production data structures?

**Detailed Technical Solution & Production Implementation:**
Allowing data scientists to test complex feature transformations or model training pipelines against production data historically required duplicating terabytes of data into sandbox environments.

### Branch-and-Merge Architecture (Project Nessie / Git-for-Data):
1. **Create Isolated Zero-Copy Branch**:
   - A data scientist creates a virtual branch off `main` in Project Nessie:
```sql
-- Create an isolated experimental branch
CREATE BRANCH experiment_churn_v2 IN nessie FROM main;
USE REFERENCE experiment_churn_v2 IN nessie;
```
   - Operates in sub-second time; zero data is copied.
2. **Isolated Transformation Execution**:
   - The ML pipeline modifies schemas, updates columns, and appends synthetic feature tables on the `experiment_churn_v2` branch.
   - Production users on `main` continue reading unaltered production data with zero interference.
3. **Automated Data Quality Validation**:
   - CI/CD tests run against the experimental branch to verify statistical drift, null ratios, and metric accuracy.
4. **Atomic Fast-Forward Merge**:
   - Once validated, merge the branch back into `main`:
```sql
MERGE BRANCH experiment_churn_v2 INTO main IN nessie;
```
   - Updates the production table metadata pointer atomically; production users instantly see the new validated features with zero downtime.

**Key Invariants & Architectural Gotchas:**
- Enforce atomic commits and ACID guarantees to eliminate dirty reads across multi-engine readers.
- Optimize file size layout (128MB - 512MB) to prevent object storage API rate limits.
- Design data pipelines around idempotent writes to enable deterministic disaster recovery replays.

**Certification Alignment:** `Enterprise Data Architect / AWS & Azure Solutions Architect Professional`

---

### Q3.25: How do you engineer a comprehensive disaster recovery and cross-region replication architecture for a massive data lakehouse with zero data loss (RPO=0)?

**Question & Production Scenario:**
In an enterprise modern data system design and lakehouse architecture, how do you resolve the following:
> How do you engineer a comprehensive disaster recovery and cross-region replication architecture for a massive data lakehouse with zero data loss (RPO=0)?

**Detailed Technical Solution & Production Implementation:**
Achieving RPO=0 (zero data loss) across multi-region data lakes during cloud provider regional disasters requires coordinating physical storage replication with transaction log consensus.

### Comprehensive DR Architecture:
1. **Synchronous Cloud Storage Replication**:
   - Configure AWS S3 Multi-Region Access Points (MRAP) with bidirectional Cross-Region Replication (CRR) and replication metrics/alerts enabled, or Azure GRS (Geo-Redundant Storage).
2. **Distributed Transaction Consensus**:
   - Standard asynchronous file replication cannot guarantee RPO=0 because metadata commit files can arrive before underlying Parquet data files.
   - Deploy an active-active cross-region catalog (e.g., Apache Polaris backed by multi-region CockroachDB / AWS Spanner).
   - Writes commit to the transaction catalog using Raft consensus across both regions before acknowledging the client.
3. **Delta Deep Clone Incremental Sync**:
   - For semi-synchronous disaster recovery (RPO < 5m), schedule continuous Delta Deep Clone jobs:
```sql
CREATE OR REPLACE TABLE secondary_lake.sales.orders
DEEP CLONE primary_lake.sales.orders;
```
4. **Automated Health Probes & DNS Failover**:
   - Route 53 health checks monitor primary region endpoints.
   - Upon confirmed regional outage, trigger automated DNS failover to the DR region, unpausing standby compute clusters instantly.

**Key Invariants & Architectural Gotchas:**
- Enforce atomic commits and ACID guarantees to eliminate dirty reads across multi-engine readers.
- Optimize file size layout (128MB - 512MB) to prevent object storage API rate limits.
- Design data pipelines around idempotent writes to enable deterministic disaster recovery replays.

**Certification Alignment:** `Enterprise Data Architect / AWS & Azure Solutions Architect Professional`

---

---

### System Design: Enterprise GenAI & RAG Platform on Lakehouse (Mosaic AI Architecture)

#### 1. Functional & Non-Functional Requirements
- **Functional**:
  - Ingest 50M enterprise documents (PDFs, Confluence, Jira, ServiceNow) across 20 global business domains.
  - Sub-second hybrid retrieval (dense vector embeddings + sparse keyword search).
  - Multi-turn AI Agent capable of tool-calling enterprise SQL databases and returning verifiable source citations.
  - Multi-tenant data isolation with strict role-based and attribute-based access control (ABAC).
- **Non-Functional**:
  - P95 query latency $< 800\text{ms}$ under 500 concurrent users.
  - Zero PII leakage across enterprise boundaries.
  - High availability ($99.95\%$) with scale-to-zero serverless compute during off-peak hours.

#### 2. End-to-End Architectural Blueprint
```
[Unstructured Data Sources] (PDF, Docs, Tickets)
                │
                ▼
[Auto Loader Streaming Ingestion] ──▶ [Bronze Delta Table]
                │
                ▼
[Semantic Text Chunking & Normalization] ──▶ [Silver Delta Table] (CDF Enabled)
                │
                ▼
[Mosaic AI Vector Search Delta Sync] ──▶ [Managed Vector Search Endpoint] (HNSW)
                │                                    │
                │                                    ▼
                │                  [Unified Governance: Unity Catalog]
                │                   (ABAC Tags, Row Filters, Column Masks)
                │                                    │
                ▼                                    ▼
     [Mosaic AI Gateway] ◀── [Databricks Model Serving] (Llama 3 / DBRX)
     (PII Masking & Rate Limits)                     │
                ▲                                    ▼
                │                  [Mosaic AI Agent Framework]
                │                   (Tool Calling against DBSQL)
                │                                    │
                ▼                                    ▼
[Enterprise Chat App / Slack] ◀─── [MLflow 3.x Agent Evaluation & Review App]
```

#### 3. Deep Architectural Tradeoff: Managed Mosaic AI vs Dedicated Vector DBs

| Decision Criteria | Managed Mosaic AI Vector Search | Dedicated Vector DB (Pinecone, Milvus, Qdrant) |
| :--- | :--- | :--- |
| **Data Synchronization** | **Native Delta Sync**: Reads Delta Change Data Feed directly. No ETL code needed to sync updates/deletes. | **Custom Sync Pipelines**: Requires writing Kafka/Flink/Airflow pipelines to mirror CRUD events into vector database. |
| **Security & Governance** | **Single Pane of Glass**: Inherits Unity Catalog row filters, column masks, and tags. Zero data copies outside compliance ring. | **Fragmented Governance**: Vector DB maintains independent API keys and access lists; ACL sync lag is a constant vulnerability. |
| **Embedding Generation** | **Managed Embeddings**: Endpoints automatically compute vectors via Foundation Model APIs during ingestion. | **Client-Side Embeddings**: Client must compute embeddings and manage vector dimensions prior to insertion. |
| **Hybrid SQL Joins** | **In-Engine `vector_search()`**: Join vector similarity scores with relational dimension tables in a single SQL query plan. | **Two-Phase Architecture**: App must query vector DB first, extract IDs, and send secondary `WHERE id IN (...)` query to data warehouse. |

#### 4. Production Code: Deploying Governed Agent with Vector Search
```python
# Production Agent Definition using MLflow and Vector Search
import mlflow
from databricks.vector_search.client import VectorSearchClient

class EnterpriseRAGAgent(mlflow.pyfunc.PythonModel):
    def load_context(self, context):
        self.vsc = VectorSearchClient()
        self.index = self.vsc.get_index(
            endpoint_name="prod_vector_search_endpoint",
            index_name="prod_catalog.ai_schema.enterprise_docs_idx"
        )

    def predict(self, context, model_input):
        question = model_input["query"].iloc[0]
        # Retrieve context from Vector Search index
        results = self.index.similarity_search(
            query_text=question,
            columns=["chunk_content", "source_url"],
            num_results=3
        )
        chunks = [row[0] for row in results["result"]["data_array"]]
        prompt = f"Answer using only these sources: {chunks}\nQuestion: {question}"
        # Execute guarded query via Foundation Model API
        return mlflow.deployments.get_deploy_client("databricks").predict(
            endpoint="databricks-meta-llama-3-70b-instruct",
            inputs={"messages": [{"role": "user", "content": prompt}]}
        )
```
