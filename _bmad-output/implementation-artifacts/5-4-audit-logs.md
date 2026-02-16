---
status: backlog
epic: "Epic 5: Team Collaboration & Workspace Management"
storyPoints: 5
---

# Story 5-4: Audit Logs & Activity Tracking

## Description

Implement comprehensive audit logging capturing all user actions for security monitoring, compliance requirements, and debugging. The audit system records who did what, when, and from where for all write operations (create, update, delete) across all resources including dashboards, alerts, team members, workspace settings, and billing changes. This immutable log provides accountability and forensic capabilities for security incidents.

The audit log viewer provides searchable and filterable access to all audit events with advanced query capabilities. Admins can filter by user, action type, resource type, date range, and IP address to investigate specific activities or generate compliance reports. The system highlights suspicious patterns like bulk deletions, permission changes outside business hours, or access from unusual locations.

Export functionality enables compliance reporting and integration with external SIEM (Security Information and Event Management) systems. Logs can be exported to CSV for spreadsheet analysis or JSON for automated processing. Retention policies ensure logs are preserved according to regulatory requirements (90 days minimum, up to 7 years for enterprise compliance).

## Acceptance Criteria

- AC-1: Audit logging middleware captures all write operations (create, update, delete) automatically
- AC-2: Audit log schema includes: timestamp, user, action, resource type, resource ID, IP address, user agent, changes (before/after)
- AC-3: Audit log viewer with table showing recent events (newest first)
- AC-4: Search functionality with full-text search across all log fields
- AC-5: Multi-dimensional filtering: user, action type, resource type, date range, IP address
- AC-6: Diff view showing before/after state changes for update operations
- AC-7: CSV export for compliance reporting with customizable columns
- AC-8: JSON export for SIEM integration and automated analysis
- AC-9: Retention policy configuration per workspace (90 days, 1 year, 7 years)
- AC-10: Audit log immutability (cannot be modified or deleted by any user)

## Technical Notes

**Audit Log Schema**:
```typescript
interface AuditLog {
  id: string;
  tenant_id: string;
  timestamp: string;
  user_id: string;
  user_email: string;  // Denormalized for deleted users
  action: 'create' | 'update' | 'delete';
  resource_type: 'dashboard' | 'alert' | 'team_member' | 'workspace_settings' | 'billing';
  resource_id: string;
  resource_name: string;  // Denormalized for deleted resources
  ip_address: string;
  user_agent: string;
  changes?: {
    before: Record<string, unknown>;
    after: Record<string, unknown>;
  };
  metadata?: Record<string, unknown>;  // Additional context
}
```

**Audit Middleware**:
```typescript
export function auditMiddleware(req, res, next) {
  // Capture original methods
  const originalSend = res.send;

  res.send = function(data) {
    // On successful write operation
    if (req.method !== 'GET' && res.statusCode < 400) {
      auditLog.create({
        tenant_id: req.tenant_id,
        user_id: req.user.id,
        action: getAction(req.method),
        resource_type: getResourceType(req.path),
        resource_id: getResourceId(req.params),
        ip_address: req.ip,
        user_agent: req.get('user-agent'),
        changes: extractChanges(req, data)
      });
    }

    return originalSend.call(this, data);
  };

  next();
}
```

**Audit Log Viewer Filters**:
- User: Dropdown with autocomplete of all workspace users
- Action: Checkbox group (Create, Update, Delete)
- Resource: Dropdown (Dashboard, Alert, Team Member, etc.)
- Date range: Date picker with presets (Today, Last 7 days, Last 30 days, Custom)
- IP address: Text input with CIDR support
- Search: Full-text search across all text fields

**Compliance Export Formats**:
```csv
// CSV format
Timestamp,User,Action,Resource Type,Resource Name,IP Address
2026-02-16T10:32:15Z,alice@example.com,update,dashboard,Error Dashboard,192.168.1.100
```

```json
// JSON format
{
  "logs": [
    {
      "timestamp": "2026-02-16T10:32:15Z",
      "user": "alice@example.com",
      "action": "update",
      "resource_type": "dashboard",
      "resource_id": "dash-123",
      "resource_name": "Error Dashboard",
      "ip_address": "192.168.1.100",
      "changes": {
        "before": { "name": "Errors" },
        "after": { "name": "Error Dashboard" }
      }
    }
  ],
  "total": 1,
  "exported_at": "2026-02-16T11:00:00Z"
}
```

**Retention Policies**:
- Free tier: 90 days (minimum compliance)
- Pro tier: 1 year
- Enterprise tier: 7 years (configurable)
- Automated archival: Logs older than retention period moved to S3 cold storage

**Edge Cases**:
- User deleted after audit log created (preserve user_email, show as "Deleted User")
- Resource deleted after audit log (preserve resource_name for context)
- Bulk operations (e.g., delete 50 dashboards) (create 50 individual audit entries)
- System operations without user (attribute to "System" user)
- Audit log export for 1M+ records (paginate, stream export, warn about size)

## Tasks

- [ ] Design audit log database schema with proper indexing
- [ ] Implement audit logging middleware for all API routes
- [ ] Build action and resource type mapping utilities
- [ ] Create audit log viewer UI with table and pagination
- [ ] Implement full-text search across audit logs
- [ ] Build multi-dimensional filtering UI
- [ ] Create diff view for showing before/after changes
- [ ] Implement CSV export with customizable columns
- [ ] Add JSON export for SIEM integration
- [ ] Build retention policy configuration and archival job
- [ ] Add audit log immutability enforcement (read-only database user)
- [ ] Create suspicious activity highlighting (bulk deletes, odd-hours access)
- [ ] Write audit log integration tests
- [ ] Document audit logging for compliance certifications
- [ ] Conduct audit log security review

## Dependencies

- Story 1-3: Multi-Tenancy (tenant-aware logging)
- Story 5-1: Team Roles (user permissions for viewing logs)

## Estimation

**Story Points**: 5

**Breakdown**:
- Audit middleware and schema: 1 point
- Audit log viewer UI: 2 points
- Search, filtering, and export: 2 points
