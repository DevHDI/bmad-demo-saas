---
status: backlog
epic: "Epic 6: Public API & Integrations"
storyPoints: 13
---

# Story 6-3: Third-Party Integrations

## Description

Develop integrations with popular developer tools and monitoring platforms enabling CloudMetrics to function as part of a broader observability and collaboration ecosystem. These integrations expand CloudMetrics' reach by connecting with tools teams already use daily: Datadog for unified metrics, Grafana for visualization freedom, Jira for incident tracking, and CSV/JSON import for migrating from other analytics platforms.

Each integration is bidirectional where appropriate: export CloudMetrics data to external systems for consolidated monitoring, and import data from external sources into CloudMetrics for unified analytics. The integrations follow OAuth 2.0 patterns for secure authorization, storing only encrypted access tokens and requesting minimal necessary permissions.

An integration marketplace provides discovery and installation of all available integrations with configuration wizards guiding users through setup. Each integration includes clear documentation, setup instructions, and troubleshooting guides. Usage analytics help prioritize which integrations to build next based on customer demand.

## Acceptance Criteria

- AC-1: Datadog integration exports CloudMetrics metrics to Datadog via Datadog API with custom tags
- AC-2: Grafana data source plugin allows querying CloudMetrics metrics in Grafana dashboards
- AC-3: Jira integration creates Jira issues from CloudMetrics incidents with bidirectional sync
- AC-4: CSV data import parses time-series CSV files and ingests as custom events
- AC-5: JSON data import supports structured event data with schema validation
- AC-6: Integration marketplace UI lists all integrations with setup status (not connected, connected)
- AC-7: OAuth 2.0 flow for integrations requiring external authentication (Datadog, Jira)
- AC-8: Integration configuration UI with credential management and test connection
- AC-9: Integration status monitoring shows last sync time and error rate
- AC-10: Integration templates for common use cases (Datadog dashboards, Jira workflows)

## Technical Notes

**Datadog Integration**:
```typescript
// Export CloudMetrics metrics to Datadog
interface DatadogExport {
  workspace_id: string;
  datadog_api_key: string;  // Encrypted at rest
  datadog_site: 'datadoghq.com' | 'datadoghq.eu';
  metric_mapping: {
    cloudmetrics_metric: string;
    datadog_metric: string;
    tags: Record<string, string>;
  }[];
  export_interval: '1m' | '5m' | '15m';
  enabled: boolean;
}

// Send to Datadog Metrics API
POST https://api.datadoghq.com/api/v2/series
{
  "series": [
    {
      "metric": "cloudmetrics.error_rate",
      "type": "gauge",
      "points": [
        {
          "timestamp": 1707507600,
          "value": 5.2
        }
      ],
      "tags": ["env:production", "service:api"]
    }
  ]
}
```

**Grafana Data Source Plugin**:
- TypeScript plugin following Grafana plugin SDK
- Query editor for selecting CloudMetrics metrics, time ranges, filters
- Data frame format conversion for Grafana visualization
- Variable support for dynamic dashboard filtering
- Alerting support (query CloudMetrics from Grafana alerts)

**Jira Integration**:
```typescript
interface JiraIntegration {
  workspace_id: string;
  jira_url: string;  // https://company.atlassian.net
  jira_email: string;
  jira_api_token: string;  // Encrypted
  project_key: string;  // PROJ
  issue_type: string;  // Incident, Bug
  auto_create_on_alert: boolean;
  bidirectional_sync: boolean;  // Sync Jira status back to CloudMetrics
}

// Create Jira issue from incident
POST https://company.atlassian.net/rest/api/3/issue
{
  "fields": {
    "project": { "key": "PROJ" },
    "summary": "[CloudMetrics] Error Rate Above Threshold",
    "description": "Alert fired at 2026-02-16T10:32:15Z\nMetric: error_rate\nValue: 5.2%\nThreshold: 3.0%",
    "issuetype": { "name": "Incident" },
    "labels": ["cloudmetrics", "production"]
  }
}
```

**CSV/JSON Import**:
```typescript
// CSV format for time-series data
timestamp,metric,value,tags
2026-02-16T10:00:00Z,page_views,1234,page:/home;browser:chrome
2026-02-16T10:01:00Z,page_views,1289,page:/home;browser:chrome

// JSON format for structured events
{
  "events": [
    {
      "timestamp": "2026-02-16T10:00:00Z",
      "type": "page_view",
      "properties": {
        "page": "/home",
        "browser": "chrome",
        "duration_ms": 234
      }
    }
  ]
}
```

**Integration Marketplace**:
- Grid of integration cards with logo, name, description
- Status badge: Not Connected, Connected, Error
- Setup wizard modal for each integration
- Configuration panels for connected integrations
- Usage stats: last sync, events synced, error rate

**OAuth 2.0 Flow** (Datadog example):
1. User clicks "Connect Datadog" → Redirect to Datadog OAuth consent
2. User approves → Datadog redirects back with authorization code
3. Exchange code for access token → Store encrypted token
4. Use token for API requests with refresh when expired

**Integration Templates**:
- Datadog Dashboard: Pre-built Datadog dashboard querying CloudMetrics metrics
- Jira Workflow: Automated workflow for incident → issue creation → resolution sync
- Grafana Dashboards: Sample Grafana dashboards using CloudMetrics data source

**Edge Cases**:
- External service downtime (queue sync operations, retry with backoff)
- Token revocation (detect 401 errors, prompt user to reconnect)
- Data format mismatch on import (validate schema, show errors, skip invalid rows)
- Rate limit on external API (respect rate limits, throttle exports)
- Integration deleted while sync in progress (cancel pending operations)

## Tasks

- [ ] Design integration configuration database schema
- [ ] Build Datadog export connector with API client
- [ ] Implement metric mapping configuration UI
- [ ] Create scheduled export job for Datadog sync
- [ ] Develop Grafana data source plugin with query editor
- [ ] Build Grafana plugin backend for CloudMetrics API queries
- [ ] Publish Grafana plugin to Grafana plugin marketplace
- [ ] Implement Jira OAuth 2.0 authentication flow
- [ ] Create Jira issue creation from incidents
- [ ] Build bidirectional sync for Jira status updates
- [ ] Implement CSV parser and time-series data import
- [ ] Create JSON import with schema validation
- [ ] Build integration marketplace UI
- [ ] Develop setup wizards for each integration
- [ ] Add integration status monitoring and error logging
- [ ] Create integration templates (Datadog dashboards, Jira workflows)
- [ ] Write integration documentation and troubleshooting guides
- [ ] Test all integrations end-to-end
- [ ] Conduct security review of OAuth flows and token storage
- [ ] Collect integration usage analytics for prioritization

## Dependencies

- Story 6-1: Public API (used by Grafana plugin)
- Story 4-3: Incident Management (Jira integration)
- Story 2-1: Event Ingestion (CSV/JSON import target)

## Estimation

**Story Points**: 13

**Breakdown**:
- Datadog export integration: 3 points
- Grafana plugin development: 4 points
- Jira integration with OAuth: 3 points
- CSV/JSON import and marketplace: 3 points
