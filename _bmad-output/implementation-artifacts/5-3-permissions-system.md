---
status: backlog
epic: "Epic 5: Team Collaboration & Workspace Management"
storyPoints: 8
---

# Story 5-3: Fine-Grained Permissions System

## Description

Extend role-based access control with fine-grained resource-level permissions allowing users to control access to individual dashboards, alerts, and data sources beyond workspace-wide roles. While workspace roles (Owner, Admin, Editor, Viewer) provide baseline permissions, resource-level permissions enable scenarios like "this dashboard is private to me" or "this alert is shared with the ops team only."

The permission model uses inheritance: users start with capabilities granted by their workspace role, and resource-level permissions can further restrict or expand access. For example, an Editor can create dashboards that only they can modify, or an Admin can grant a Viewer permission to edit a specific dashboard. This flexibility supports both open collaboration and sensitive data protection within the same workspace.

Permission enforcement occurs at both API and UI levels. The API middleware checks permissions before allowing any resource access or modification, returning 403 Forbidden for unauthorized attempts. The UI conditionally renders actions based on permissions, hiding edit/delete buttons from users without appropriate access. All permission checks are logged in the audit trail for security monitoring and compliance.

## Acceptance Criteria

- AC-1: Resource-level permission model for dashboards, alerts, and data sources
- AC-2: Permission types: owner, editor, viewer with resource-specific semantics
- AC-3: Permission inheritance from workspace roles with override capability
- AC-4: Permission management UI on each resource's settings page
- AC-5: Share modal for adding users/teams to resource with permission selection
- AC-6: API middleware enforces permissions on all resource endpoints (GET, PUT, DELETE)
- AC-7: UI conditionally renders actions (edit, delete, share) based on user permissions
- AC-8: Permission audit logging captures all access control changes
- AC-9: Bulk permission operations (share with team, revoke all access)
- AC-10: Permission conflicts resolved with most restrictive policy wins

## Technical Notes

**Permission Model**:
```typescript
interface ResourcePermission {
  id: string;
  resource_type: 'dashboard' | 'alert' | 'data_source';
  resource_id: string;
  tenant_id: string;
  grantee_type: 'user' | 'team';
  grantee_id: string;
  permission: 'owner' | 'editor' | 'viewer';
  granted_by: string;  // User ID who granted permission
  granted_at: string;
}
```

**Permission Inheritance**:
1. Start with workspace role capabilities (baseline)
2. Apply resource-level permissions (more specific)
3. Most restrictive policy wins on conflict
4. Example: Workspace Admin + Dashboard Viewer = Viewer on that dashboard

**Permission Semantics by Resource**:
| Resource | Owner | Editor | Viewer |
|----------|-------|--------|--------|
| Dashboard | Full control, delete, share | Modify layout/widgets | Read-only view |
| Alert | Edit, delete, mute, share | Edit conditions only | View status |
| Data Source | Edit, delete, share | Query access | No access |

**Permission Check Middleware**:
```typescript
async function checkPermission(
  userId: string,
  resourceType: string,
  resourceId: string,
  requiredPermission: 'view' | 'edit' | 'delete'
): Promise<boolean> {
  // 1. Get user's workspace role
  const workspaceRole = await getWorkspaceRole(userId);

  // 2. Get resource-level permissions
  const resourcePerms = await getResourcePermissions(resourceId, userId);

  // 3. Combine and evaluate
  return evaluatePermissions(workspaceRole, resourcePerms, requiredPermission);
}
```

**Share Modal UI**:
- Search bar to find users/teams to share with
- Permission dropdown (Viewer, Editor, Owner)
- Current shares list showing who has access
- Revoke button for each share
- "Copy link" for public sharing (if enabled)

**Edge Cases**:
- Last Owner removes themselves (prevent, require designating new Owner)
- Permission granted to user outside workspace (validate user is member)
- Resource deleted with existing permissions (cascade delete permissions)
- Permission granted to deactivated user (permissions persist, enforce on activation)
- Circular permission dependencies (not possible with current model)

## Tasks

- [ ] Design resource permission database schema
- [ ] Implement permission inheritance evaluation logic
- [ ] Build permission check middleware for API routes
- [ ] Create permission management UI for dashboard settings
- [ ] Implement share modal with user/team search
- [ ] Add permission checks to alert and data source endpoints
- [ ] Build bulk permission operations (share with team)
- [ ] Implement UI conditional rendering based on permissions
- [ ] Add permission audit logging
- [ ] Create permission conflict resolution logic
- [ ] Write permission evaluation integration tests
- [ ] Build permission debugging tool for admins
- [ ] Test permission enforcement across all resource types
- [ ] Document permission model and best practices
- [ ] Conduct security review of permission implementation

## Dependencies

- Story 5-1: Team Roles (workspace roles to inherit from)
- Story 3-1: Dashboard Framework (dashboard permissions)
- Story 4-1: Alert Rules Engine (alert permissions)

## Estimation

**Story Points**: 8

**Breakdown**:
- Permission model and schema: 2 points
- API middleware enforcement: 2 points
- Permission management UI: 2 points
- Inheritance and testing: 2 points
