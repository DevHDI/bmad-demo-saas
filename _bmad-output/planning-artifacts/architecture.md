---
status: done
completedAt: "2026-01-20"
---
# CloudMetrics - Architecture Document

## 1. System Overview

CloudMetrics is a real-time analytics platform designed for monitoring application performance, user behavior, and business metrics. The system ingests millions of events per day and provides sub-second query responses for dashboard visualizations.

### 1.1 Architecture Style

The platform follows an **event-driven architecture** with a clear separation between the ingestion pipeline, processing layer, and query/presentation layer. This allows independent scaling of each component based on load characteristics.

### 1.2 Key Design Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Frontend | Next.js 14 (App Router) | Server components, real-time streaming, SSR |
| Backend | Node.js + Fastify | High-throughput event handling, WebSocket support |
| Time-Series DB | TimescaleDB | PostgreSQL-compatible, automatic partitioning, continuous aggregates |
| Stream Processing | Apache Kafka | Durable event log, consumer groups, exactly-once semantics |
| Cache | Redis 7 (Cluster) | Hot data caching, session management, rate limiting |
| Search | Elasticsearch | Log search, full-text queries across events |
| Auth | Custom + Okta SAML | Multi-tenant SSO, fine-grained permissions |
| Real-time | Server-Sent Events (SSE) | Simpler than WebSocket for unidirectional updates |
| Infrastructure | AWS (ECS + Terraform) | Container orchestration, infrastructure as code |
| Monitoring | OpenTelemetry + Grafana | Vendor-neutral observability, internal dogfooding |

## 2. System Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                      Client SDKs                             │
│   (JavaScript, Python, Go, Ruby, iOS, Android)               │
└──────────────────┬──────────────────────────────────────────┘
                   │ HTTPS / WebSocket
┌──────────────────▼──────────────────────────────────────────┐
│                  Ingestion Layer                              │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────────────┐│
│  │  REST API     │  │  WebSocket   │  │  Rate Limiter      ││
│  │  (Fastify)    │  │  Gateway     │  │  (Redis)           ││
│  └──────┬───────┘  └──────┬───────┘  └────────────────────┘│
└─────────┼──────────────────┼────────────────────────────────┘
          │                  │
┌─────────▼──────────────────▼────────────────────────────────┐
│                  Processing Layer                             │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────────────┐│
│  │  Kafka        │  │  Stream      │  │  Enrichment        ││
│  │  (Event Bus)  │  │  Processors  │  │  (Geo-IP, Device)  ││
│  └──────────────┘  └──────────────┘  └────────────────────┘│
└─────────────────────────┬───────────────────────────────────┘
                          │
┌─────────────────────────▼───────────────────────────────────┐
│                  Storage Layer                                │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────────────┐│
│  │  TimescaleDB  │  │  Redis       │  │  S3                ││
│  │  (Time-series)│  │  (Hot cache) │  │  (Cold storage)    ││
│  └──────────────┘  └──────────────┘  └────────────────────┘│
└─────────────────────────┬───────────────────────────────────┘
                          │
┌─────────────────────────▼───────────────────────────────────┐
│                  Presentation Layer                           │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────────────┐│
│  │  Next.js App  │  │  Query       │  │  SSE Streaming     ││
│  │  (Dashboard)  │  │  Engine      │  │  (Real-time)       ││
│  └──────────────┘  └──────────────┘  └────────────────────┘│
└─────────────────────────────────────────────────────────────┘
```

## 3. Multi-Tenancy Architecture

### 3.1 Isolation Strategy

CloudMetrics uses **row-level security (RLS)** in PostgreSQL/TimescaleDB for tenant data isolation:

```sql
-- Every table includes tenant_id
ALTER TABLE events ENABLE ROW LEVEL SECURITY;
CREATE POLICY tenant_isolation ON events
  USING (tenant_id = current_setting('app.current_tenant')::uuid);
```

### 3.2 Tenant Context Propagation

```
Request → Auth Middleware → Extract tenant_id from JWT
       → Set PostgreSQL session variable
       → RLS policies automatically filter queries
```

### 3.3 Resource Limits per Tier

| Resource | Free | Pro | Enterprise |
|----------|------|-----|------------|
| Events/month | 100K | 10M | Unlimited |
| Retention | 30 days | 90 days | 365 days |
| Dashboards | 3 | 20 | Unlimited |
| Alert rules | 5 | 50 | Unlimited |
| Team members | 3 | 20 | Unlimited |
| API rate limit | 100/min | 1000/min | 10000/min |

## 4. Data Pipeline Architecture

### 4.1 Ingestion Flow

1. **Client SDK** sends batch of events (up to 100 per request)
2. **Ingestion API** validates schema (Zod), checks rate limits
3. Events published to **Kafka topic** (`events.raw`)
4. **Stream Processors** consume events:
   - Enrichment: Add geo-IP location, device info, user agent parsing
   - Aggregation: Compute time-window aggregates (1min, 5min, 1hr)
   - Routing: Fan-out to appropriate storage based on event type
5. **Storage**: Raw events → TimescaleDB hypertable, Aggregates → materialized views

### 4.2 Event Schema

```typescript
interface AnalyticsEvent {
  id: string;           // UUID v7 (time-sortable)
  tenant_id: string;    // Organization UUID
  type: string;         // "pageview" | "click" | "custom" | ...
  timestamp: string;    // ISO 8601
  session_id: string;
  user_id?: string;
  properties: Record<string, unknown>;
  context: {
    ip?: string;
    user_agent?: string;
    locale?: string;
    page_url?: string;
    referrer?: string;
  };
}
```

### 4.3 Storage Strategy

| Data Type | Store | Retention | Query Pattern |
|-----------|-------|-----------|---------------|
| Raw events | TimescaleDB hypertable | Per-tier | Time-range scans |
| 1-min aggregates | TimescaleDB continuous aggregate | 90 days | Dashboard queries |
| 1-hr aggregates | TimescaleDB continuous aggregate | 365 days | Trend analysis |
| Daily summaries | PostgreSQL table | Forever | Billing, reports |
| Hot aggregates | Redis sorted sets | 24 hours | Real-time dashboards |

## 5. Query Engine

### 5.1 Query Optimization

- **Continuous aggregates** pre-compute common time-window queries
- **Chunk exclusion** via TimescaleDB automatically skips irrelevant partitions
- **Redis cache** for frequently-accessed dashboard data (30s TTL)
- **Parameterized query builder** prevents SQL injection and enables plan caching

### 5.2 Real-time Updates

Dashboard real-time updates use Server-Sent Events (SSE):

```
Client → GET /api/stream/dashboard/:id
Server → SSE connection established
       → Initial state sent
       → Delta updates every 5 seconds
       → Only changed metrics included in updates
       → Automatic reconnection with last-event-id
```

## 6. Security Architecture

- **Authentication**: JWT with RSA-256 signing, refresh token rotation
- **SSO**: SAML 2.0 (Okta, Azure AD), OAuth 2.0 (Google Workspace)
- **Authorization**: RBAC (Owner, Admin, Editor, Viewer) + resource-level ACLs
- **Data Isolation**: PostgreSQL RLS, Kafka topic-per-tenant
- **Encryption**: TLS 1.3 in transit, AES-256 at rest
- **API Security**: API key + HMAC signature for SDK ingestion
- **Rate Limiting**: Token bucket algorithm per API key, Redis-backed
- **Audit Trail**: All write operations logged with actor, timestamp, diff

## 7. Deployment & Infrastructure

- **Compute**: AWS ECS Fargate (auto-scaling containers)
- **Database**: AWS RDS (TimescaleDB on PostgreSQL 16)
- **Cache**: AWS ElastiCache Redis Cluster (3 shards)
- **Streaming**: Amazon MSK (Managed Kafka)
- **Storage**: S3 for cold data, CloudFront for static assets
- **IaC**: Terraform modules per environment
- **CI/CD**: GitHub Actions → ECR → ECS rolling deployments
- **Monitoring**: OpenTelemetry → Grafana Cloud (internal dogfooding)
- **Environments**: dev, staging, production (isolated AWS accounts)
