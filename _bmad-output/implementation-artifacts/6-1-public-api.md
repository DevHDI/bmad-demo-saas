---
status: backlog
epic: "Epic 6: Public API & Integrations"
storyPoints: 13
---

# Story 6-1: Public REST API

## Description

Design and implement a versioned public REST API providing programmatic access to all CloudMetrics functionality for automation, custom integrations, and third-party tools. The API follows RESTful principles with predictable resource URLs, standard HTTP methods, and JSON payloads. Version 1 (v1) establishes the foundation with core resources: events, metrics, dashboards, alerts, and team management.

The API is fully documented using OpenAPI 3.0 specification enabling automatic generation of interactive documentation (Swagger UI), client SDKs, and integration tests. This documentation-first approach ensures API consistency and provides developers with accurate, always-up-to-date reference material. The OpenAPI spec also powers API mocking for frontend development before backend implementation.

Authentication uses API keys with scoped permissions, allowing fine-grained control over what each key can access. Rate limiting prevents abuse with tier-appropriate limits (Free: 100 requests/hour, Pro: 1000/hour, Enterprise: 10000/hour). The API is designed for backward compatibility with new versions adding endpoints without breaking existing integrations.

## Acceptance Criteria

- AC-1: Versioned API with /v1 prefix for all endpoints
- AC-2: RESTful resource design: GET (read), POST (create), PUT (update), DELETE (remove)
- AC-3: OpenAPI 3.0 specification documenting all endpoints, request/response schemas, and errors
- AC-4: Swagger UI hosted at /api/docs for interactive API exploration
- AC-5: API key authentication with API-Key header required on all requests
- AC-6: API key scopes restrict access to specific resources (read-only, full access)
- AC-7: Rate limiting enforced per tier with X-RateLimit-* headers in responses
- AC-8: Pagination for list endpoints with cursor-based pagination (not offset/limit)
- AC-9: Error responses follow RFC 7807 Problem Details format
- AC-10: Client SDK generation for JavaScript, Python, Go using OpenAPI generator

## Technical Notes

**API Resource Structure**:
```
/v1/events            # Event ingestion (already exists)
/v1/metrics           # Query metrics and aggregates
/v1/dashboards        # Dashboard CRUD operations
/v1/dashboards/:id/widgets  # Widget management
/v1/alerts            # Alert rule management
/v1/alerts/:id/history  # Alert firing history
/v1/team              # Team member management
/v1/team/invitations  # Invitation management
/v1/workspace         # Workspace settings
/v1/usage             # Usage statistics
```

**OpenAPI Specification Example**:
```yaml
openapi: 3.0.0
info:
  title: CloudMetrics API
  version: 1.0.0
  description: Programmatic access to analytics, dashboards, and alerts

paths:
  /v1/dashboards:
    get:
      summary: List dashboards
      parameters:
        - name: cursor
          in: query
          schema:
            type: string
        - name: limit
          in: query
          schema:
            type: integer
            default: 20
      responses:
        '200':
          description: Success
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/DashboardList'
```

**Authentication**:
- API keys generated per workspace with optional scopes
- API-Key header: `API-Key: cm_live_abc123xyz...`
- Scopes: `events:write`, `metrics:read`, `dashboards:full`, `alerts:full`, `team:admin`
- Unauthorized requests return 401 with problem details

**Rate Limiting**:
- Implementation: Token bucket in Redis per API key
- Limits: Free (100/hr), Pro (1000/hr), Enterprise (10000/hr)
- Response headers:
  - X-RateLimit-Limit: 1000
  - X-RateLimit-Remaining: 847
  - X-RateLimit-Reset: 1707507600 (Unix timestamp)
- Over limit: 429 Too Many Requests

**Pagination (Cursor-based)**:
```json
{
  "data": [ /* results */ ],
  "pagination": {
    "next_cursor": "eyJpZCI6MTIzfQ==",
    "has_more": true
  }
}
```

**Error Format (RFC 7807)**:
```json
{
  "type": "https://cloudmetrics.com/errors/validation-error",
  "title": "Validation Error",
  "status": 400,
  "detail": "Dashboard name is required",
  "instance": "/v1/dashboards",
  "errors": [
    {
      "field": "name",
      "message": "Name is required"
    }
  ]
}
```

**SDK Generation**:
- Use OpenAPI Generator for JavaScript, Python, Go clients
- Publish NPM package (@cloudmetrics/api-client), PyPI (cloudmetrics-api), Go module
- Auto-generate on API spec changes via CI/CD
- Include code examples in SDK documentation

**Edge Cases**:
- API version deprecation (announce 6 months ahead, maintain for 12 months)
- Breaking changes (introduce as new version /v2, not in v1)
- API key leaked (provide revocation and rotation, send security alert)
- Rate limit burst (allow short bursts, enforce average rate)
- Malformed API key (return 401, not 400, to avoid enumeration)

## Tasks

- [ ] Design RESTful API resource structure and URLs
- [ ] Create OpenAPI 3.0 specification for all endpoints
- [ ] Set up Swagger UI at /api/docs
- [ ] Implement API key generation and management
- [ ] Build API key authentication middleware
- [ ] Add scoped permission checking for API keys
- [ ] Implement token bucket rate limiter with Redis
- [ ] Create rate limit middleware with header injection
- [ ] Build cursor-based pagination utilities
- [ ] Implement RFC 7807 error response format
- [ ] Create /v1/metrics query endpoint
- [ ] Build /v1/dashboards CRUD endpoints
- [ ] Implement /v1/alerts management endpoints
- [ ] Add /v1/team and /v1/workspace endpoints
- [ ] Generate client SDKs for JavaScript, Python, Go
- [ ] Publish SDKs to package registries (NPM, PyPI, pkg.go.dev)
- [ ] Write API integration tests covering all endpoints
- [ ] Create API usage examples and tutorials
- [ ] Document API versioning and deprecation policy
- [ ] Conduct API security review

## Dependencies

- Story 2-1: Event Ingestion API (foundation)
- Story 1-2: Authentication System (API key model)

## Estimation

**Story Points**: 13

**Breakdown**:
- API design and OpenAPI spec: 3 points
- Authentication and rate limiting: 2 points
- Core resource endpoints: 5 points
- SDK generation and publishing: 2 points
- Testing and documentation: 1 point
