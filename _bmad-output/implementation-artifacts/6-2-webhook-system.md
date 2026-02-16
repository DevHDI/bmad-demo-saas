---
status: backlog
epic: "Epic 6: Public API & Integrations"
storyPoints: 8
---

# Story 6-2: Outgoing Webhook System

## Description

Build a reliable outgoing webhook system enabling CloudMetrics to push real-time notifications to external services when events occur (alert fires, dashboard created, incident resolved). Webhooks provide event-driven integration capabilities allowing customers to build custom workflows, update external systems, and trigger automation in response to CloudMetrics events.

The webhook delivery system implements industry best practices for reliability and security. Each webhook payload is signed with HMAC-SHA256 allowing recipients to verify authenticity and prevent tampering. Delivery includes automatic retries with exponential backoff for transient failures (network timeouts, 5xx errors), ensuring important events aren't lost due to temporary issues. Failed deliveries after all retries are logged for manual investigation.

Webhook management provides a self-service UI for registration, testing, and monitoring. Users can register multiple webhook endpoints per workspace, configure which events trigger each webhook, and test webhooks with sample payloads before going live. Delivery logs show complete history of all webhook attempts including response status, payload, and retry attempts for debugging integration issues.

## Acceptance Criteria

- AC-1: Webhook registration API and UI for creating webhook endpoints with URL, secret, and event filters
- AC-2: Event triggers: alert.fired, alert.resolved, dashboard.created, dashboard.updated, dashboard.deleted, incident.created
- AC-3: HMAC-SHA256 signature in X-Webhook-Signature header for payload verification
- AC-4: Exponential backoff retry for failed deliveries (1s, 2s, 4s, 8s, 16s, max 5 attempts)
- AC-5: Retry only for transient failures (timeouts, 5xx errors), not client errors (4xx)
- AC-6: Delivery timeout of 30 seconds per attempt to prevent hanging connections
- AC-7: Delivery logs showing all attempts with timestamps, status codes, response bodies
- AC-8: Webhook testing interface sends test payload and displays response
- AC-9: Webhook health status: healthy (>95% success), degraded (80-95%), unhealthy (<80%)
- AC-10: Automatic webhook disabling after 50 consecutive failures

## Technical Notes

**Webhook Schema**:
```typescript
interface Webhook {
  id: string;
  tenant_id: string;
  url: string;
  secret: string;  // Used for HMAC signature
  events: string[];  // ['alert.fired', 'dashboard.created']
  enabled: boolean;
  created_at: string;
  last_delivery_at?: string;
  success_rate: number;  // Last 100 deliveries
  consecutive_failures: number;
}

interface WebhookDelivery {
  id: string;
  webhook_id: string;
  event_type: string;
  payload: Record<string, unknown>;
  attempt: number;  // 1-5
  status_code?: number;
  response_body?: string;
  error?: string;
  delivered_at: string;
  duration_ms: number;
}
```

**Event Payload Example**:
```json
{
  "event": "alert.fired",
  "timestamp": "2026-02-16T10:32:15Z",
  "workspace_id": "ws-abc123",
  "data": {
    "alert_id": "alert-xyz789",
    "alert_name": "Error Rate Above Threshold",
    "severity": "critical",
    "metric": "error_rate",
    "value": 5.2,
    "threshold": 3.0,
    "dashboard_url": "https://app.cloudmetrics.com/dashboards/dash-123"
  }
}
```

**HMAC Signature Generation**:
```typescript
function generateSignature(payload: string, secret: string): string {
  const hmac = crypto.createHmac('sha256', secret);
  hmac.update(payload);
  return hmac.digest('hex');
}

// Recipient verifies:
const receivedSignature = req.headers['x-webhook-signature'];
const computedSignature = generateSignature(req.body, webhook.secret);
if (receivedSignature !== computedSignature) {
  throw new Error('Invalid signature');
}
```

**Delivery Queue**:
- Use Bull queue (Redis-backed) for async webhook delivery
- Job payload: webhook_id, event_type, event_payload
- Retry configuration: 5 attempts, exponential backoff
- Job timeout: 30 seconds
- Failed job handler: Log to delivery table, increment failure counter

**Retry Logic**:
```typescript
const retryDelays = [1000, 2000, 4000, 8000, 16000];  // milliseconds

async function deliverWebhook(webhook, payload, attempt = 1) {
  try {
    const response = await axios.post(webhook.url, payload, {
      timeout: 30000,
      headers: {
        'Content-Type': 'application/json',
        'X-Webhook-Signature': generateSignature(JSON.stringify(payload), webhook.secret),
        'X-Webhook-Delivery': deliveryId,
        'User-Agent': 'CloudMetrics-Webhooks/1.0'
      }
    });

    // Success (2xx response)
    logDelivery(webhook.id, 'success', response.status, attempt);
    return;
  } catch (error) {
    // Retry on timeout or 5xx
    if (shouldRetry(error) && attempt < 5) {
      await sleep(retryDelays[attempt - 1]);
      return deliverWebhook(webhook, payload, attempt + 1);
    }

    // Permanent failure
    logDelivery(webhook.id, 'failed', error.status, attempt, error.message);
    incrementFailureCounter(webhook.id);
  }
}
```

**Webhook Testing**:
- Test button sends sample payload for configured event type
- Shows request/response in modal for debugging
- Example: Test alert.fired sends mock alert payload to webhook URL
- Validates URL reachability and signature verification

**Edge Cases**:
- Webhook URL returns redirect (follow up to 3 redirects)
- Webhook URL unreachable (DNS failure, connection refused) (mark as failed, retry)
- Response body exceeds 1MB (truncate, log warning)
- Webhook secret rotated (use new secret for future deliveries, old deliveries fail)
- Webhook deleted during retry (cancel pending deliveries)

## Tasks

- [ ] Design webhook and delivery log database schemas
- [ ] Build webhook registration API (POST /v1/webhooks)
- [ ] Create webhook management UI (create, edit, delete)
- [ ] Implement HMAC-SHA256 signature generation
- [ ] Build webhook delivery queue with Bull
- [ ] Implement retry logic with exponential backoff
- [ ] Create delivery timeout and error handling
- [ ] Add delivery logging with status and response tracking
- [ ] Build webhook testing interface
- [ ] Implement delivery log viewer with filtering
- [ ] Add webhook health status calculation
- [ ] Create automatic disabling for unhealthy webhooks
- [ ] Implement event triggers for alert and dashboard events
- [ ] Write webhook delivery integration tests
- [ ] Document webhook integration guide with code examples
- [ ] Provide signature verification examples for multiple languages

## Dependencies

- Story 4-1: Alert Rules Engine (alert events to trigger webhooks)
- Story 3-1: Dashboard Framework (dashboard events)

## Estimation

**Story Points**: 8

**Breakdown**:
- Webhook registration and schema: 2 points
- Delivery queue and retry logic: 3 points
- HMAC signatures and security: 1 point
- Testing UI and logs: 2 points
