# Module 01: Enterprise Lakehouse & Medallion Topology Design

## 1. Medallion Layer Topology & SLA Guarantees
- **Bronze (Raw Ingestion)**: Immutable, append-only landing zone. Captures raw JSON, CSV, or CDC logs with full payload and ingestion timestamps. Never queried by end-users.
- **Silver (Cleaned & Validated)**: Conformed schema, cleansed datatypes, deduplicated keys, and standard timezone timestamps. Serves enterprise data scientists and ad-hoc analysts.
- **Gold (Aggregated & Business Dimensional)**: Star-schema fact and dimension models, curated metrics, and KPI tables optimized for sub-second dashboard performance.

```mermaid
graph LR
    subgraph Bronze["Bronze Layer (Raw)"]
        B1["Raw CDC Streams"]
        B2["Batch Parquet Drops"]
    end
    subgraph Silver["Silver Layer (Cleansed)"]
        S1["Validated Orders"]
        S2["Conformed Customers"]
    end
    subgraph Gold["Gold Layer (Aggregated)"]
        G1["fct_revenue_daily"]
        G2["dim_customer_retention"]
    end
    Bronze -->|Schema Validation & Dedup| Silver
    Silver -->|Dimensional Modeling & KPI Rollups| Gold
```
