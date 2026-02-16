---
status: done
completedAt: "2026-01-28"
epic: "Epic 1: Platform Foundation"
storyPoints: 13
---

# Story 1-3: Multi-Tenancy Architecture

## Description

Implement a robust multi-tenant architecture using PostgreSQL Row-Level Security (RLS) to ensure complete data isolation between organizations while maintaining query performance and operational simplicity. This approach allows all tenants to share the same database instance and application codebase while guaranteeing that no tenant can access another tenant's data, even in the event of application-level bugs.

Multi-tenancy is fundamental to CloudMetrics' SaaS model. Every table in the system includes a `tenant_id` column, and RLS policies automatically filter all queries to return only the current tenant's data. The tenant context is extracted from the user's JWT token, set as a PostgreSQL session variable, and enforced at the database level. This defense-in-depth approach provides security even if application code forgets to add tenant filtering.

The implementation includes comprehensive onboarding flows for creating new organizations, inviting initial team members, and configuring workspace settings. Resource limits are enforced per pricing tier (Free, Pro, Enterprise) including event ingestion rate limits, data retention policies, and feature access controls.

## Acceptance Criteria

- AC-1: Database schema includes tenant_id column on all user data tables with proper foreign key constraints
- AC-2: PostgreSQL RLS policies implemented on all tables with tenant isolation enforcement
- AC-3: Tenant-aware middleware extracts tenant_id from JWT and sets PostgreSQL session variable
- AC-4: Organization creation flow with workspace setup, billing tier selection, and initial configuration
- AC-5: User-to-tenant association with role mapping in junction table
- AC-6: Data isolation verification test suite covering all tables and query patterns
- AC-7: Tenant context propagation works correctly across all API routes and background jobs
- AC-8: Resource limit enforcement per pricing tier (events/month, retention, dashboards, alerts)
- AC-9: Tenant switching UI for users belonging to multiple organizations
- AC-10: Migration path for converting single-tenant data to multi-tenant schema

## Technical Notes

**RLS Implementation**:
```sql
-- Enable RLS on events table
ALTER TABLE events ENABLE ROW LEVEL SECURITY;

-- Policy for INSERT: tenant_id must match session variable
CREATE POLICY tenant_isolation_insert ON events
  FOR INSERT WITH CHECK (tenant_id = current_setting('app.current_tenant')::uuid);

-- Policy for SELECT/UPDATE/DELETE: only access own tenant data
CREATE POLICY tenant_isolation_select ON events
  FOR SELECT USING (tenant_id = current_setting('app.current_tenant')::uuid);
```

**Middleware Implementation**:
- Extract tenant_id from JWT claims after authentication
- Set PostgreSQL session variable: `SET app.current_tenant = 'tenant-uuid'`
- Use connection pooling with session state reset between requests
- Log tenant context in all application logs for debugging

**Resource Limits by Tier**:
| Resource | Free | Pro | Enterprise |
|----------|------|-----|------------|
| Events/month | 100K | 10M | Unlimited |
| Retention | 30 days | 90 days | 365 days |
| Dashboards | 3 | 20 | Unlimited |
| Alert rules | 5 | 50 | Unlimited |
| Team members | 3 | 20 | Unlimited |

**Database Schema**:
```sql
tenants: id, name, slug, billing_tier, created_at, settings
tenant_members: tenant_id, user_id, role, invited_at, accepted_at
tenant_limits: tenant_id, events_this_month, dashboards_count, alerts_count
```

**Edge Cases**:
- User belongs to multiple tenants (require explicit tenant selection)
- Tenant ID missing from JWT (reject request with 403 Forbidden)
- RLS policy bypass attempts (audit logging captures tenant context)
- Background jobs without user context (use service account with tenant_id)
- Tenant deletion with data retention requirements (soft delete with 90-day grace period)

## Tasks

- [x] Design multi-tenant database schema with tenant_id on all tables
- [x] Create migration to add tenant_id column to existing tables
- [x] Implement RLS policies for all tables (events, dashboards, alerts, etc.)
- [x] Build tenant-aware middleware for API routes
- [x] Create PostgreSQL session variable management utilities
- [x] Implement organization creation and onboarding flow
- [x] Build tenant membership association and role management
- [x] Create tenant switching UI component
- [x] Implement resource limit tracking and enforcement
- [x] Write comprehensive data isolation test suite
- [x] Add tenant context to background job execution
- [x] Create tenant usage dashboard for admins
- [x] Implement soft delete and data retention policies
- [x] Document multi-tenancy patterns for developers
- [x] Conduct security review of RLS implementation

## Dependencies

- Story 1-1: Platform Initialization (database setup)
- Story 1-2: Authentication System (JWT with tenant claims)

## Estimation

**Story Points**: 13

**Breakdown**:
- Database schema and RLS policies: 5 points
- Middleware and context propagation: 3 points
- Onboarding flow and UI: 3 points
- Testing and security review: 2 points
