# Module 06: Sub-Minute Streaming Lakehouse Ingestion

## 1. Latency vs Cost Optimization Curves
- **Micro-Batch Mode**: Triggers every 10-60 seconds. Provides high compression, efficient file sizes, and low commit overhead.
- **Continuous Mode**: Sub-second event commits. Requires dedicated compaction engines to prevent the "million small files" catastrophe.
