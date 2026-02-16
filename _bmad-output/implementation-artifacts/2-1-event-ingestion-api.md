---
status: done
completedAt: "2026-01-30"
epic: "Epic 2: Data Ingestion Pipeline"
storyPoints: 8
---

# Story 2-1: Event Ingestion API

## Description

Build a high-throughput event ingestion system capable of handling millions of analytics events per day from client SDKs. The system provides both REST API endpoints for batch ingestion (up to 100 events per request) and WebSocket connections for real-time streaming scenarios. Event validation, rate limiting, and API key authentication ensure data quality and prevent abuse while maintaining sub-second latency.

The ingestion layer is designed for horizontal scalability with stateless API servers behind a load balancer. Events are validated against a strict schema using Zod, enriched with server-side metadata (ingestion timestamp, IP address), and immediately published to Kafka for asynchronous processing. This decoupling allows the ingestion API to remain responsive even under high load while ensuring zero data loss through Kafka's durability guarantees.

Rate limiting is critical to prevent both accidental misconfiguration (infinite loops sending events) and malicious abuse. Limits are enforced per API key using a token bucket algorithm implemented in Redis, allowing burst traffic while enforcing average rate limits. The system provides clear 429 responses with Retry-After headers when limits are exceeded.

## Acceptance Criteria

- AC-1: REST POST /v1/events endpoint accepts batch of up to 100 events with 201 response
- AC-2: WebSocket endpoint /v1/stream accepts real-time event streaming with backpressure handling
- AC-3: Event schema validation using Zod with clear error messages for invalid events
- AC-4: Rate limiting enforced per API key (Free: 100/min, Pro: 1000/min, Enterprise: 10000/min)
- AC-5: API key authentication with HMAC request signing for security
- AC-6: API key management UI for creating, rotating, and revoking keys
- AC-7: Event batching support with automatic flush on timeout or size limit
- AC-8: Ingestion latency p99 < 500ms from API receipt to Kafka publish
- AC-9: Graceful degradation with 503 responses when Kafka is unavailable
- AC-10: Comprehensive ingestion metrics (events/sec, validation errors, rate limit hits)

## Technical Notes

**Event Schema** (Zod validation):
```typescript
const EventSchema = z.object({
  type: z.string().min(1).max(100),
  timestamp: z.string().datetime(),
  session_id: z.string().uuid(),
  user_id: z.string().optional(),
  properties: z.record(z.unknown()),
  context: z.object({
    ip: z.string().ip().optional(),
    user_agent: z.string().optional(),
    locale: z.string().optional(),
    page_url: z.string().url().optional(),
    referrer: z.string().url().optional()
  }).optional()
});
```

**REST API Endpoint**:
- Method: POST /v1/events
- Authentication: API-Key header + HMAC-SHA256 signature
- Request: { events: Event[] } (max 100 events)
- Response: 201 Created with { accepted: number, rejected: number }
- Error codes: 400 (validation), 401 (auth), 429 (rate limit), 503 (unavailable)

**WebSocket Protocol**:
- Connection: WSS /v1/stream?api_key=xxx
- Message format: JSON with event array
- Backpressure: Server sends { type: 'slow_down' } when overwhelmed
- Heartbeat: Ping/pong every 30 seconds to detect dead connections

**Rate Limiting**:
- Algorithm: Token bucket with Redis-backed counters
- Window: 1-minute sliding window
- Burst allowance: 2x average rate for short bursts
- Response: 429 with Retry-After header indicating seconds to wait

**Edge Cases**:
- Malformed JSON in request body (return 400 with parse error)
- Event timestamp in future (accept with warning, use server time)
- Extremely large property values (truncate to 10KB with warning)
- WebSocket connection drop during transmission (client retry with deduplication)
- Kafka unavailable (buffer events in memory up to 10MB, then reject)

## Tasks

- [x] Design event schema with Zod validation
- [x] Create REST ingestion endpoint with batch support
- [x] Build WebSocket streaming endpoint with backpressure
- [x] Implement API key authentication with HMAC signing
- [x] Create Redis-backed token bucket rate limiter
- [x] Build API key management service and UI
- [x] Implement Kafka producer with retry logic
- [x] Add server-side event enrichment (timestamps, IP)
- [x] Create ingestion metrics and monitoring dashboard
- [x] Write integration tests for REST and WebSocket ingestion
- [x] Implement event deduplication using Kafka message keys
- [x] Add comprehensive error handling and logging
- [x] Build client SDK example code (JavaScript, Python)
- [x] Load test ingestion API with 10K events/sec
- [x] Document API endpoints with OpenAPI specification

## Dependencies

- Story 1-1: Platform Initialization (Next.js API routes)
- Story 1-3: Multi-Tenancy (tenant_id in events)

## Estimation

**Story Points**: 8

**Breakdown**:
- REST API implementation: 2 points
- WebSocket streaming: 2 points
- Validation and rate limiting: 2 points
- API key management: 1 point
- Testing and documentation: 1 point
