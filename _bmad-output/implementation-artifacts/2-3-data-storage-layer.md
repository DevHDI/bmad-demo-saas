---
status: in-progress
startedAt: "2026-02-04"
epic: "Epic 2: Data Ingestion Pipeline"
storyPoints: 13
---

# Story 2-3: Data Storage & Query Layer

## Description

Build the data storage and query infrastructure optimized for time-series analytics workloads. TimescaleDB hypertables provide automatic time-based partitioning for billions of events while maintaining PostgreSQL compatibility. Continuous aggregates pre-compute common dashboard queries (event counts, unique users, p95 latencies) for sub-second response times. Redis caches hot data (last 24 hours) to reduce database load and provide real-time dashboard updates.

The query engine abstracts the complexity of querying across hypertables, continuous aggregates, and cache layers. It automatically routes queries based on time range and freshness requirements: recent data (< 5 minutes) from Redis, recent historical (< 24 hours) from continuous aggregates, and older data from raw hypertables. All queries are tenant-aware and respect RLS policies for data isolation.

Data retention policies automatically drop old chunks based on pricing tier: Free tier retains 30 days, Pro 90 days, Enterprise 365 days. Before deletion, cold data is exported to S3 for compliance and potential rehydration. Compression is applied to older chunks to reduce storage costs while maintaining query performance through intelligent chunk exclusion.

## Acceptance Criteria

- AC-1: TimescaleDB hypertables created for events table with time-based partitioning (7-day chunks)
- AC-2: Continuous aggregates configured for 1-minute, 1-hour, and 1-day time buckets
- AC-3: Redis caching layer stores aggregated metrics with 30-second TTL
- AC-4: Query engine with parameterized query builder prevents SQL injection
- AC-5: Automatic query routing based on time range (cache vs aggregates vs raw data)
- AC-6: Data retention policies configured per tenant pricing tier (30/90/365 days)
- AC-7: Compression enabled on chunks older than 7 days (50%+ space savings)
- AC-8: S3 export job for cold data before deletion (Parquet format)
- AC-9: Query performance monitoring with slow query logging and CloudWatch metrics
- AC-10: Dashboard query response time p95 < 1 second for 7-day time range

## Technical Notes

**TimescaleDB Hypertable Configuration**:
```sql
CREATE TABLE events (
  time TIMESTAMPTZ NOT NULL,
  tenant_id UUID NOT NULL,
  event_id UUID NOT NULL,
  type VARCHAR(100) NOT NULL,
  session_id UUID,
  user_id VARCHAR(255),
  properties JSONB,
  context JSONB
);

SELECT create_hypertable('events', 'time', chunk_time_interval => INTERVAL '7 days');
CREATE INDEX ON events (tenant_id, time DESC);
CREATE INDEX ON events (tenant_id, type, time DESC);
```

**Continuous Aggregates**:
```sql
CREATE MATERIALIZED VIEW events_1min
WITH (timescaledb.continuous) AS
SELECT
  time_bucket('1 minute', time) AS bucket,
  tenant_id,
  type,
  COUNT(*) as event_count,
  COUNT(DISTINCT user_id) as unique_users
FROM events
GROUP BY bucket, tenant_id, type;

-- Refresh policy: Every 1 minute for last 1 hour
SELECT add_continuous_aggregate_policy('events_1min',
  start_offset => INTERVAL '1 hour',
  end_offset => INTERVAL '1 minute',
  schedule_interval => INTERVAL '1 minute');
```

**Redis Caching Strategy**:
- Key pattern: `metrics:{tenant_id}:{metric}:{window}`
- TTL: 30 seconds for real-time metrics
- Data structure: Sorted sets for time-series, hashes for aggregates
- Eviction: LRU (Least Recently Used) when memory limit reached

**Query Engine Routing**:
| Time Range | Data Source | Reason |
|-----------|-------------|---------|
| Last 5 min | Redis cache | Real-time, low latency |
| Last 24 hours | events_1min aggregate | Pre-computed, fast |
| Last 7 days | events_1hour aggregate | Balanced detail/speed |
| Older | events_1day aggregate | Historical trends |
| Raw events | events hypertable | Ad-hoc analysis |

**Edge Cases**:
- Query spans cache boundary (merge Redis + database results)
- Continuous aggregate refresh lag (serve slightly stale data with indicator)
- Cache miss during high load (fallback to database with circuit breaker)
- Tenant exceeds retention period (gracefully return empty results)
- Chunk deletion in progress (queries may fail, retry with backoff)

## Tasks

- [x] Configure TimescaleDB extension on RDS PostgreSQL
- [x] Create events hypertable with time-based partitioning
- [x] Design and create continuous aggregate views (1min, 1hr, 1day)
- [x] Set up continuous aggregate refresh policies
- [x] Implement Redis caching layer with TTL management
- [ ] Build query engine with automatic routing logic
- [ ] Create parameterized query builder with Zod validation
- [ ] Implement data retention policies with chunk dropping
- [ ] Build S3 export pipeline for cold data archival
- [ ] Add compression policies for chunks older than 7 days
- [ ] Create query performance monitoring dashboard
- [ ] Implement slow query logging and alerting
- [ ] Write query engine integration tests
- [ ] Load test query engine with concurrent dashboard queries
- [ ] Document query patterns and optimization guide

## Dependencies

- Story 2-2: Stream Processing (writes to events table)
- Story 1-1: Platform Initialization (TimescaleDB, Redis setup)

## Estimation

**Story Points**: 13

**Breakdown**:
- TimescaleDB hypertable setup: 2 points
- Continuous aggregates: 3 points
- Redis caching layer: 2 points
- Query engine and routing: 4 points
- Retention and compression: 2 points
