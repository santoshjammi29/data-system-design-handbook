# Module 06: Sub-Minute Streaming Lakehouse Ingestion

![Real-Time Event Streaming & Kafka Architecture](../../assets/whiteboards/kafka_event_streaming_pipeline.webp)
> *Architectural Blueprint: [Real-Time Event Streaming & Kafka Architecture](../../docs/whiteboard_diagrams/README.md#kafka-event-streaming-pipeline). View full multi-persona breakdown and engine mechanisms in the master index.*

## 1. Latency vs Cost Optimization Curves
- **Micro-Batch Mode**: Triggers every 10-60 seconds. Provides high compression, efficient file sizes, and low commit overhead.
- **Continuous Mode**: Sub-second event commits. Requires dedicated compaction engines to prevent the "million small files" catastrophe.
