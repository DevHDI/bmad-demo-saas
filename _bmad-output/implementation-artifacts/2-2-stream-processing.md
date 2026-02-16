---
status: done
completedAt: "2026-02-03"
epic: "Epic 2: Data Ingestion Pipeline"
storyPoints: 13
---

# Story 2-2: Stream Processing Pipeline

## Description

Implement the stream processing layer that consumes events from Kafka, performs real-time aggregations, enriches events with contextual data (geo-location, device information), and routes processed events to appropriate storage systems. This layer transforms raw events into queryable analytics data while maintaining exactly-once processing semantics and sub-second latency.

The stream processing architecture uses Kafka consumer groups for parallel processing with automatic partition rebalancing. Multiple processor types handle different workloads: aggregation processors compute time-window statistics (1-minute, 5-minute, 1-hour windows), enrichment processors add geo-IP data and parse user agents, and routing processors fan out events to TimescaleDB, Elasticsearch, and Redis based on event type and query patterns.

Error handling is robust with a dead letter queue (DLQ) pattern. Events that fail processing after retries are sent to a separate Kafka topic for manual inspection and replay. Processing lag is continuously monitored with CloudWatch alarms triggering when consumer groups fall behind, enabling auto-scaling of processor instances before user-facing queries are impacted.

## Acceptance Criteria

- AC-1: Kafka consumer groups configured with partition assignment strategy for balanced processing
- AC-2: Time-window aggregation processors compute 1-minute, 5-minute, and 1-hour statistics
- AC-3: Geo-IP enrichment adds city, region, country, and coordinates based on IP address
- AC-4: User agent parsing extracts browser, OS, device type, and version information
- AC-5: Event routing logic directs events to appropriate storage (TimescaleDB, Elasticsearch, Redis)
- AC-6: Dead letter queue captures failed events with error details for debugging
- AC-7: Processing lag monitoring with CloudWatch metrics and alarms
- AC-8: Exactly-once processing semantics using Kafka transactional producers
- AC-9: Automatic consumer group rebalancing on processor scaling or failure
- AC-10: Processing throughput of 100K events/second across consumer group

## Technical Notes

**Kafka Configuration**:
- Topic: events.raw (12 partitions, replication factor 3)
- Consumer groups: aggregation-workers, enrichment-workers, storage-writers
- Partition assignment: Sticky assignor for reduced rebalancing overhead
- Offset commit: Auto-commit every 5 seconds after successful processing
- Session timeout: 30 seconds with max poll interval 5 minutes

**Time-Window Aggregation**:
```typescript
// Example: Count events by type in 1-minute windows
const aggregator = new TimeWindowAggregator({
  windowSize: 60000, // 1 minute
  aggregateFn: (events) => ({
    event_type: events[0].type,
    count: events.length,
    unique_users: new Set(events.map(e => e.user_id)).size,
    window_start: roundToMinute(events[0].timestamp)
  })
});
```

**Enrichment Services**:
- Geo-IP: MaxMind GeoIP2 database (updated weekly)
- User agent: ua-parser-js library for device/browser detection
- Session enrichment: Merge events by session_id to compute session duration

**Storage Routing**:
| Event Type | Storage | Reason |
|-----------|---------|--------|
| pageview, click | TimescaleDB | Time-series queries, dashboards |
| error, exception | Elasticsearch | Full-text search, stack traces |
| realtime metrics | Redis | Hot data, < 24hr retention |
| All events | S3 (async) | Cold storage, compliance |

**Edge Cases**:
- Out-of-order events (use event timestamp, not processing time)
- Late-arriving events outside aggregation window (drop or re-aggregate)
- Consumer group rebalancing during processing (commit offsets before release)
- Enrichment API failures (retry 3x with backoff, then send to DLQ)
- Kafka broker failure (consumer auto-discovers new leader)

## Tasks

- [x] Set up Amazon MSK (Managed Kafka) cluster with 3 brokers
- [x] Create Kafka topics with appropriate partitions and replication
- [x] Build base consumer group framework with error handling
- [x] Implement time-window aggregation processor
- [x] Integrate MaxMind GeoIP2 database and enrichment logic
- [x] Add user agent parsing with ua-parser-js
- [x] Create event routing logic based on type and query patterns
- [x] Build dead letter queue handler with retry mechanism
- [x] Implement exactly-once semantics with Kafka transactions
- [x] Add CloudWatch metrics for processing lag and throughput
- [x] Create auto-scaling configuration for consumer instances
- [x] Write integration tests for all processor types
- [x] Load test stream processing with 100K events/sec
- [x] Document processor architecture and troubleshooting guide

## Dependencies

- Story 2-1: Event Ingestion API (Kafka producers)
- Story 1-1: Platform Initialization (Kafka infrastructure)

## Estimation

**Story Points**: 13

**Breakdown**:
- Kafka consumer setup: 2 points
- Aggregation processors: 4 points
- Enrichment services: 3 points
- Routing and DLQ: 2 points
- Monitoring and testing: 2 points
