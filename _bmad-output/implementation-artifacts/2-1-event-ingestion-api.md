---
status: done
---
# Story 2.1: Event Ingestion API

## Description
Build high-throughput REST and WebSocket APIs for ingesting analytics events from client SDKs with validation and rate limiting.

## Acceptance Criteria
- REST endpoint for batch event ingestion
- WebSocket endpoint for real-time streaming
- Event schema validation with Zod
- Rate limiting per API key (10K events/min)
- API key management for clients

## Tasks
- [x] Design event schema and validation rules
- [x] Create REST ingestion endpoint
- [x] Build WebSocket streaming endpoint
- [x] Implement Zod schema validation
- [x] Add rate limiting middleware
- [x] Create API key management system
