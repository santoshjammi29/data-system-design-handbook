# Module 05: High-Throughput Batch Topologies & Change Data Feed

## 1. Incremental Consumption via CDF
Change Data Feed (CDF) tracks row-level changes (`insert`, `update_preimage`, `update_postimage`, `delete`) in the transaction log. Downstream pipelines consume only mutated rows without re-scanning historical terabytes.
