---
status: backlog
epic: "Epic 4: Alerting & Incident Management"
storyPoints: 8
---

# Story 4-2: Notification Channels

## Description

Implement a multi-channel notification delivery system that routes alert notifications to the appropriate communication platforms based on severity and team preferences. The system supports email for low-urgency alerts, Slack for team collaboration, PagerDuty for on-call incident response, and custom webhooks for integration with other tools. Each channel is configurable with routing rules, templates, and delivery retry logic.

The notification dispatcher acts as a central hub receiving alert firing events from the rules engine and fanning out to configured channels. Each channel implements its own delivery protocol: SMTP for email, Slack Incoming Webhooks API, PagerDuty Events API v2, and HTTP POST for custom webhooks. Failed deliveries are retried with exponential backoff and logged for troubleshooting.

Escalation policies ensure critical alerts reach the right people even if initial notifications are unacknowledged. For example, a critical alert might first notify the primary on-call engineer via PagerDuty; if unacknowledged after 5 minutes, escalate to the secondary engineer and post to the #incidents Slack channel. This tiered approach balances notification fatigue with ensuring urgent issues receive attention.

## Acceptance Criteria

- AC-1: Email channel sends HTML-formatted notifications with alert details, metric values, and dashboard links
- AC-2: Slack integration posts to configured channels with interactive buttons (Acknowledge, View Dashboard, Mute)
- AC-3: PagerDuty integration creates incidents with severity mapping (critical, warning, info)
- AC-4: Custom webhook channel POSTs JSON payload to user-defined URLs with HMAC signature
- AC-5: Notification routing rules: route to different channels based on alert severity and tags
- AC-6: Delivery retry with exponential backoff (1s, 2s, 4s, 8s, 16s, max 5 attempts)
- AC-7: Escalation policies define multi-tier notification sequences with timeout intervals
- AC-8: Notification templates customizable per tenant with variable substitution
- AC-9: Delivery status tracking: pending, delivered, failed, with retry count
- AC-10: Rate limiting prevents notification spam (max 100/hour per channel per tenant)

## Technical Notes

**Notification Dispatcher Architecture**:
```typescript
interface NotificationEvent {
  alert_id: string;
  tenant_id: string;
  severity: 'critical' | 'warning' | 'info';
  title: string;
  message: string;
  metric_value: number;
  dashboard_url: string;
  fired_at: string;
}

class NotificationDispatcher {
  async dispatch(event: NotificationEvent) {
    // Fetch configured channels for tenant
    const channels = await getChannelsForTenant(event.tenant_id);

    // Apply routing rules
    const relevantChannels = channels.filter(ch =>
      this.matchesRoutingRules(ch, event)
    );

    // Dispatch to all matching channels in parallel
    await Promise.allSettled(
      relevantChannels.map(ch => this.sendToChannel(ch, event))
    );
  }
}
```

**Email Channel**:
- SMTP configuration per tenant (or shared SendGrid account)
- HTML templates with CloudMetrics branding
- Inline CSS for email client compatibility
- Unsubscribe link for non-critical alerts
- Attachment support for error logs (optional)

**Slack Channel**:
```typescript
// Slack message format with interactive buttons
{
  text: "🚨 Critical Alert: Error rate above threshold",
  blocks: [
    {
      type: "section",
      text: { type: "mrkdwn", text: "*Error Rate*: 5.2% (threshold: 3%)" }
    },
    {
      type: "actions",
      elements: [
        { type: "button", text: "Acknowledge", action_id: "ack" },
        { type: "button", text: "View Dashboard", url: "https://..." }
      ]
    }
  ]
}
```

**PagerDuty Integration**:
- Events API v2 for incident creation
- Severity mapping: critical → P1, warning → P2, info → P3
- Automatic incident resolution when alert resolves
- Deduplication using alert_id to prevent duplicate incidents

**Escalation Policy Example**:
```yaml
escalation_tiers:
  - tier: 1
    channels: [pagerduty_primary]
    timeout: 5 minutes
  - tier: 2
    channels: [pagerduty_secondary, slack_incidents]
    timeout: 10 minutes
  - tier: 3
    channels: [email_leadership]
```

**Edge Cases**:
- Slack webhook returns 410 Gone (webhook deleted, disable channel and notify admin)
- Email bounces (mark email invalid, disable channel after 3 bounces)
- PagerDuty rate limit hit (queue notifications, process when limit resets)
- Webhook timeout (abort after 30s, retry with backoff)
- Alert resolves during escalation (cancel escalation, send resolution notification)

## Tasks

- [ ] Build notification dispatcher service with event queue
- [ ] Create email channel with SMTP and HTML template engine
- [ ] Design email templates for alert firing and resolution
- [ ] Implement Slack integration with Incoming Webhooks API
- [ ] Build Slack interactive message handlers (acknowledge, mute)
- [ ] Integrate PagerDuty Events API v2 for incident management
- [ ] Create custom webhook channel with HMAC signature generation
- [ ] Implement delivery retry logic with exponential backoff
- [ ] Build escalation policy engine with tier progression
- [ ] Create notification routing rules UI
- [ ] Add delivery status tracking and logging
- [ ] Implement rate limiting per channel per tenant
- [ ] Write integration tests for all channel types
- [ ] Build notification testing interface (send test alert)
- [ ] Document channel configuration and escalation policies

## Dependencies

- Story 4-1: Alert Rules Engine (firing events to notify)

## Estimation

**Story Points**: 8

**Breakdown**:
- Email channel: 2 points
- Slack and PagerDuty: 2 points
- Custom webhooks: 1 point
- Escalation policies: 2 points
- Retry and testing: 1 point
