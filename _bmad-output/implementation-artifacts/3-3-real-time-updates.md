---
status: review
completedAt: "2026-02-09"
reviewedAt: "2026-02-10"
epic: "Epic 3: Dashboards & Visualizations"
storyPoints: 8
---

# Story 3-3: Real-Time Dashboard Updates

## Description

Implement real-time dashboard updates using Server-Sent Events (SSE) to stream metric changes from the server to connected clients. Unlike polling (which wastes bandwidth and server resources) or WebSockets (which are bidirectional and more complex), SSE provides a lightweight unidirectional streaming solution perfect for dashboard updates that flow server-to-client.

The implementation uses delta updates to minimize bandwidth: only metrics that have changed since the last update are transmitted, reducing payload sizes by 80-90% compared to full refreshes. The server maintains connection pools per tenant to efficiently broadcast updates to multiple users viewing the same dashboard. When new events are processed, the server calculates updated metrics and pushes deltas to all connected clients within 5 seconds.

Client-side connection management handles network interruptions gracefully with automatic reconnection using exponential backoff. The UI displays clear indicators for connection status: pulsing green dot for live connection, yellow "reconnecting" banner during recovery, and red "disconnected" warning after repeated failures. Users can toggle auto-refresh intervals or pause updates while analyzing specific time ranges.

## Acceptance Criteria

- AC-1: SSE endpoint /api/stream/dashboard/:id streams metric updates to connected clients
- AC-2: Delta calculation engine computes only changed metrics since last update
- AC-3: Automatic reconnection with exponential backoff (1s, 2s, 4s, 8s, max 30s)
- AC-4: Connection pooling groups clients by tenant and dashboard for efficient broadcasting
- AC-5: Visual indicator in header: green dot (live), yellow (reconnecting), red (disconnected)
- AC-6: Update flash animation briefly highlights metrics when values change
- AC-7: Auto-refresh interval selector (5s, 15s, 30s, 1m, Off) persists per user
- AC-8: Last-Event-ID tracking enables resume after disconnection without data loss
- AC-9: Server-side rate limiting prevents connection spam (max 10 connections per user)
- AC-10: Graceful degradation to 30-second polling if SSE unsupported (IE11 fallback)

## Technical Notes

**SSE Protocol**:
```typescript
// Server sends updates as SSE events
event: metric_update
id: 1707504320000
data: {"event_count": 1523, "unique_users": 342, "error_rate": 0.012}

event: heartbeat
data: {"timestamp": "2026-02-09T15:12:00Z"}
```

**Client Connection**:
```typescript
const eventSource = new EventSource('/api/stream/dashboard/abc123');

eventSource.addEventListener('metric_update', (event) => {
  const delta = JSON.parse(event.data);
  updateDashboardMetrics(delta);  // Apply delta to state
  flashUpdatedWidgets(Object.keys(delta));  // Visual feedback
});

eventSource.addEventListener('error', () => {
  // Automatic reconnection with backoff
  reconnectWithBackoff();
});
```

**Delta Calculation**:
- Server maintains previous metric values per connection
- On each update cycle (every 5 seconds), compute current metrics
- Compare with previous values, send only changed metrics
- Example: If only error_rate changed, send { error_rate: 0.013 }, not all metrics

**Connection Pooling**:
```typescript
// Group connections by tenant and dashboard
const connectionPools = new Map<string, Set<SSEConnection>>();
const poolKey = `${tenantId}:${dashboardId}`;

// Broadcast update to all clients in pool
function broadcastUpdate(poolKey: string, delta: MetricDelta) {
  const pool = connectionPools.get(poolKey);
  pool?.forEach(conn => conn.send(delta));
}
```

**Edge Cases**:
- Client disconnects during update transmission (connection cleanup, remove from pool)
- Server restart drops all connections (clients auto-reconnect, resume with Last-Event-ID)
- Multiple browser tabs open same dashboard (each gets own SSE connection)
- Network switches (mobile to WiFi) (connection drop triggers reconnect)
- User navigates away (close SSE connection in cleanup handler)

## Tasks

- [x] Create SSE streaming endpoint /api/stream/dashboard/:id
- [x] Implement connection authentication using JWT
- [x] Build delta calculation engine comparing current vs previous metrics
- [x] Create connection pool manager for efficient broadcasting
- [x] Implement Last-Event-ID tracking for resume after disconnection
- [x] Add automatic reconnection with exponential backoff on client
- [x] Build visual connection status indicator (pulsing green dot, yellow/red states)
- [x] Implement metric update flash animation when values change
- [x] Create auto-refresh interval selector with persistence
- [x] Add heartbeat mechanism (ping every 30s to detect dead connections)
- [x] Implement server-side rate limiting for connection spam
- [x] Build fallback polling mechanism for unsupported browsers
- [x] Add connection cleanup on client navigation
- [x] Write integration tests for SSE connection lifecycle
- [x] Load test SSE with 1000 concurrent connections per dashboard

## Dependencies

- Story 3-1: Dashboard Framework (dashboard layout)
- Story 3-2: Chart Widgets (widgets to update)
- Story 2-3: Data Storage Layer (metrics to stream)

## Estimation

**Story Points**: 8

**Breakdown**:
- SSE endpoint and protocol: 2 points
- Delta calculation and pooling: 2 points
- Client reconnection logic: 2 points
- Visual indicators and testing: 2 points
