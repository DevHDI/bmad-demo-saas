---
status: backlog
epic: "Epic 5: Team Collaboration & Workspace Management"
storyPoints: 8
---

# Story 5-2: Workspace Management

## Description

Build comprehensive workspace management capabilities enabling organizations to configure settings, monitor usage against plan limits, and manage billing subscriptions. The workspace settings page centralizes all organization-level configuration including workspace name, logo, timezone, data retention policies, and integrations. This provides a single source of truth for workspace administration.

The usage dashboard provides real-time visibility into resource consumption across all pricing tier limits: events ingested this month, dashboard count, active alert rules, team member count, and data retention status. Visual progress bars show consumption relative to limits with warnings when approaching thresholds (80% used) and hard stops when limits reached. This transparency prevents surprise overage charges and enables proactive plan upgrades.

Stripe integration handles all billing operations: plan selection (Free, Pro, Enterprise), payment method management, subscription upgrades/downgrades, and invoice history. The system implements proper subscription lifecycle management with proration on plan changes, grace periods for failed payments, and automatic account suspension after repeated payment failures.

## Acceptance Criteria

- AC-1: Workspace settings page with sections: General, Team, Billing, Integrations, Advanced
- AC-2: General settings: workspace name, slug (subdomain), logo upload, timezone selection
- AC-3: Usage dashboard shows current consumption vs limits for all resources (events, dashboards, alerts, members)
- AC-4: Visual progress bars with color coding: green (< 80%), yellow (80-95%), red (> 95%)
- AC-5: Stripe integration for payment method management (card on file)
- AC-6: Plan selection UI showing Free, Pro, Enterprise tiers with feature comparison
- AC-7: Subscription upgrade/downgrade with immediate effect and prorated billing
- AC-8: Invoice history with PDF download for each billing period
- AC-9: Workspace-level configuration: retention policy (30/90/365 days), IP allowlist, SSO enforcement
- AC-10: Billing alerts when approaching limits (email at 80%, 95%, 100% usage)

## Technical Notes

**Workspace Settings Schema**:
```typescript
interface WorkspaceSettings {
  tenant_id: string;
  name: string;
  slug: string;  // Subdomain: slug.cloudmetrics.com
  logo_url?: string;
  timezone: string;  // IANA timezone (America/New_York)
  billing_tier: 'free' | 'pro' | 'enterprise';
  stripe_customer_id?: string;
  stripe_subscription_id?: string;
  retention_days: 30 | 90 | 365;
  ip_allowlist?: string[];  // CIDRs
  sso_enforced: boolean;
  created_at: string;
  updated_at: string;
}
```

**Usage Tracking**:
```typescript
interface UsageStats {
  tenant_id: string;
  period_start: string;  // Billing period start
  events_this_month: number;
  events_limit: number;
  dashboards_count: number;
  dashboards_limit: number;
  alerts_count: number;
  alerts_limit: number;
  members_count: number;
  members_limit: number;
}
```

**Stripe Integration**:
- Product IDs: prod_free, prod_pro, prod_enterprise
- Price IDs: price_pro_monthly ($49), price_enterprise_monthly (custom)
- Webhook events: customer.subscription.updated, invoice.payment_succeeded, invoice.payment_failed
- Subscription metadata: tenant_id, upgrade_source (self-service vs sales)

**Plan Comparison Table**:
| Feature | Free | Pro | Enterprise |
|---------|------|-----|------------|
| Events/month | 100K | 10M | Unlimited |
| Retention | 30 days | 90 days | 365 days |
| Dashboards | 3 | 20 | Unlimited |
| Alerts | 5 | 50 | Unlimited |
| Team members | 3 | 20 | Unlimited |
| Price | $0 | $49/mo | Custom |

**Billing Lifecycle**:
1. **Free Tier**: Default on signup, no payment required
2. **Upgrade to Pro**: Select plan → Add payment method → Create Stripe subscription → Activate Pro features
3. **Downgrade to Free**: Confirm downgrade → Prorate remaining time → Deactivate Pro features → Archive data beyond Free limits
4. **Payment Failure**: Retry payment after 3 days → Email warnings → 7-day grace period → Suspend account (read-only) → Delete after 30 days

**Edge Cases**:
- Workspace slug already taken (suggest alternatives: slug1, slug-2, slug-new)
- Downgrade would exceed limits (show warning, require deletion of excess resources)
- Payment method update during active subscription (update without interruption)
- Multiple failed payment retries (send escalating notifications, then suspend)
- Enterprise plan inquiry (redirect to sales contact form, no self-service)

## Tasks

- [ ] Design workspace settings page layout with tabbed sections
- [ ] Build general settings form (name, slug, logo, timezone)
- [ ] Create logo upload with image validation and S3 storage
- [ ] Implement usage tracking aggregation queries
- [ ] Build usage dashboard with progress bars and limits
- [ ] Integrate Stripe Elements for payment method collection
- [ ] Create plan selection UI with feature comparison table
- [ ] Implement subscription creation on plan upgrade
- [ ] Build subscription update for downgrades with proration
- [ ] Create invoice history page with PDF downloads
- [ ] Implement Stripe webhook handlers (subscription, payment events)
- [ ] Add billing alerts at usage thresholds (80%, 95%, 100%)
- [ ] Build workspace-level config (retention, IP allowlist, SSO)
- [ ] Write billing integration tests with Stripe test mode
- [ ] Document billing lifecycle and troubleshooting guide

## Dependencies

- Story 1-3: Multi-Tenancy (workspace concept)
- Story 5-1: Team Roles (Owner permission for billing)

## Estimation

**Story Points**: 8

**Breakdown**:
- Workspace settings UI: 2 points
- Usage tracking dashboard: 2 points
- Stripe billing integration: 3 points
- Configuration and testing: 1 point
