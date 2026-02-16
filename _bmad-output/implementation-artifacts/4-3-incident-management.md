---
status: backlog
epic: "Epic 4: Alerting & Incident Management"
storyPoints: 8
---

# Story 4-3: Incident Management

## Description

Build a comprehensive incident management system that transforms firing alerts into trackable incidents with structured workflows, collaboration tools, and post-mortem documentation. The system enables teams to efficiently respond to production issues through clear ownership, timeline tracking, and standardized resolution processes. This moves beyond simple alerting to full incident lifecycle management.

Incidents are automatically created from critical and warning level alerts, or manually created by users for proactive issue tracking. Each incident has a state (open, acknowledged, investigating, resolved), assigned owner, severity level, and detailed timeline capturing all activities (alerts fired, comments added, status changes, resolution steps). The timeline provides complete incident context for debugging and post-mortem analysis.

Post-mortem documentation is critical for organizational learning from incidents. The system provides structured templates guiding teams through root cause analysis, impact assessment, timeline reconstruction, and action item definition. Post-mortems are linked to incidents and searchable across the organization, enabling teams to learn from past issues and prevent recurrence.

## Acceptance Criteria

- AC-1: Automatic incident creation from firing critical/warning alerts with alert context
- AC-2: Manual incident creation for proactive issue tracking
- AC-3: Incident states: open → acknowledged → investigating → resolved with state transitions
- AC-4: Incident assignment to users with notification and reassignment capabilities
- AC-5: Timeline view showing all events: alert fires, comments, status changes, resolution
- AC-6: Commenting system for collaboration with @mentions and notifications
- AC-7: Acknowledge workflow with acknowledgment timestamp and acknowledger tracking
- AC-8: Resolution workflow with resolution notes and closing timestamp
- AC-9: Post-mortem template with sections: summary, timeline, root cause, action items, lessons learned
- AC-10: Incident metrics dashboard: MTTR (mean time to resolution), incident count by severity, open incidents

## Technical Notes

**Incident Schema**:
```typescript
interface Incident {
  id: string;
  tenant_id: string;
  title: string;
  description: string;
  severity: 'critical' | 'high' | 'medium' | 'low';
  state: 'open' | 'acknowledged' | 'investigating' | 'resolved';
  assigned_to?: string;  // User ID
  created_by: string;
  created_at: string;
  acknowledged_at?: string;
  resolved_at?: string;
  mttr?: number;  // Seconds from creation to resolution
  alert_id?: string;  // Linked alert if incident auto-created
  post_mortem_id?: string;
}

interface IncidentEvent {
  id: string;
  incident_id: string;
  type: 'alert_fired' | 'comment' | 'state_change' | 'assignment' | 'resolution';
  actor_id: string;  // User who performed action
  data: Record<string, unknown>;  // Event-specific data
  created_at: string;
}

interface PostMortem {
  id: string;
  incident_id: string;
  summary: string;
  timeline: string;  // Markdown
  root_cause: string;
  impact: string;
  action_items: ActionItem[];
  lessons_learned: string;
  created_by: string;
  created_at: string;
}
```

**Incident Lifecycle**:
1. **Creation**: Alert fires (critical/warning) → Auto-create incident → Notify assigned on-call
2. **Acknowledgment**: On-call engineer acknowledges → State: acknowledged → Stop escalation
3. **Investigation**: Engineer debugs → Add comments/findings → State: investigating
4. **Resolution**: Issue fixed → Mark resolved with notes → Calculate MTTR → Close incident
5. **Post-Mortem**: Write post-mortem (required for critical incidents) → Share with team

**Timeline View**:
```
[10:32] 🚨 Alert Fired: Error rate above 5%
[10:33] 👤 Alice acknowledged incident
[10:35] 💬 Alice: "Investigating database connection pool exhaustion"
[10:38] 💬 Bob: "Deploying hotfix to increase pool size"
[10:42] ✅ Bob resolved incident: "Hotfix deployed, error rate back to normal"
[10:45] 📝 Post-mortem created by Alice
```

**MTTR Calculation**:
```typescript
function calculateMTTR(incident: Incident): number {
  if (!incident.resolved_at) return 0;
  const createdMs = new Date(incident.created_at).getTime();
  const resolvedMs = new Date(incident.resolved_at).getTime();
  return (resolvedMs - createdMs) / 1000;  // Seconds
}
```

**Post-Mortem Template**:
```markdown
# Incident Post-Mortem: [Title]

## Summary
Brief description of what happened and impact.

## Timeline
Detailed timeline of events leading to, during, and after the incident.

## Root Cause
What was the underlying cause? Why did it happen?

## Impact
- Users affected: X
- Duration: Y minutes
- Services impacted: Z

## Action Items
- [ ] Fix root cause (Assigned: Alice, Due: 2026-02-20)
- [ ] Add monitoring to prevent recurrence (Assigned: Bob, Due: 2026-02-25)

## Lessons Learned
What did we learn? What should we change?
```

**Edge Cases**:
- Multiple alerts fire same issue (group into single incident with parent-child relationship)
- Incident resolved but alert re-fires (reopen incident, preserve history)
- User assigned to incident leaves company (reassign to manager or default on-call)
- Post-mortem required but not completed (reminder notifications after 48 hours)
- Incident with no activity for 7 days (auto-comment prompting status update)

## Tasks

- [ ] Design incident and post-mortem database schemas
- [ ] Build incident creation from alerts (automatic trigger)
- [ ] Create manual incident creation form
- [ ] Implement incident state machine with validations
- [ ] Build incident assignment and reassignment logic
- [ ] Create timeline view with all event types
- [ ] Implement commenting system with @mentions
- [ ] Build acknowledge workflow with timestamp tracking
- [ ] Create resolution workflow with notes and MTTR calculation
- [ ] Design post-mortem template and editor interface
- [ ] Implement action item tracking linked to post-mortems
- [ ] Build incident metrics dashboard (MTTR, count by severity)
- [ ] Create incident search and filtering UI
- [ ] Add reminder notifications for incomplete post-mortems
- [ ] Write incident management user guide

## Dependencies

- Story 4-1: Alert Rules Engine (alerts to create incidents)
- Story 4-2: Notification Channels (incident notifications)

## Estimation

**Story Points**: 8

**Breakdown**:
- Incident creation and states: 2 points
- Timeline and comments: 2 points
- Workflow (acknowledge/resolve): 2 points
- Post-mortem system: 2 points
