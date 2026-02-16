---
status: backlog
epic: "Epic 5: Team Collaboration & Workspace Management"
storyPoints: 5
---

# Story 5-1: Team Roles & Invitations

## Description

Implement role-based access control (RBAC) for team collaboration with four distinct roles: Owner, Admin, Editor, and Viewer. Each role has specific capabilities defining what actions users can perform within the workspace. The invitation system enables secure onboarding of team members through email invitations with accept/decline workflows, expiring invitation tokens, and automatic role assignment upon acceptance.

The role hierarchy follows the principle of least privilege: Viewers can only read dashboards and alerts, Editors can create and modify dashboards, Admins can manage team members and workspace settings, and Owners have full control including billing and workspace deletion. This graduated permission model supports organizations of all sizes from small teams to large enterprises with complex access requirements.

Bulk invitation via CSV upload accelerates team onboarding for growing organizations. The system validates email addresses, prevents duplicate invitations, and sends personalized invitation emails to each recipient. Invitation tracking shows pending, accepted, and expired invitations with the ability to resend or revoke invitations before acceptance.

## Acceptance Criteria

- AC-1: Role capability matrix defines permissions for Owner, Admin, Editor, Viewer across all resources
- AC-2: Email invitation system sends personalized invitations with accept/decline links
- AC-3: Invitation tokens expire after 7 days with ability to resend expired invitations
- AC-4: Team member list shows all members with roles, invitation status, and last active date
- AC-5: Role management allows Owners/Admins to change member roles with audit logging
- AC-6: Bulk invite via CSV upload validates emails and sends batch invitations
- AC-7: Invitation acceptance creates user account (if new) and links to workspace with assigned role
- AC-8: Invitation decline logs refusal and notifies inviter
- AC-9: Member removal requires confirmation and optionally transfers ownership of resources
- AC-10: Last Owner cannot leave workspace (must transfer ownership or delete workspace)

## Technical Notes

**Role Capability Matrix**:
| Capability | Viewer | Editor | Admin | Owner |
|-----------|--------|--------|-------|-------|
| View dashboards | ✅ | ✅ | ✅ | ✅ |
| Create dashboards | ❌ | ✅ | ✅ | ✅ |
| Edit dashboards | ❌ | Own only | ✅ | ✅ |
| Delete dashboards | ❌ | Own only | ✅ | ✅ |
| View alerts | ✅ | ✅ | ✅ | ✅ |
| Create alerts | ❌ | ✅ | ✅ | ✅ |
| Manage team | ❌ | ❌ | ✅ | ✅ |
| Workspace settings | ❌ | ❌ | ✅ | ✅ |
| Billing management | ❌ | ❌ | ❌ | ✅ |
| Delete workspace | ❌ | ❌ | ❌ | ✅ |

**Invitation Flow**:
1. Admin clicks "Invite Member" → Enters email and selects role
2. System generates secure invitation token (UUID v4, 7-day expiry)
3. Invitation email sent with personalized message and accept/decline links
4. Recipient clicks accept → Redirected to CloudMetrics signup/login
5. Upon authentication, user linked to workspace with assigned role
6. Inviter notified of acceptance

**Invitation Schema**:
```typescript
interface Invitation {
  id: string;
  tenant_id: string;
  email: string;
  role: 'owner' | 'admin' | 'editor' | 'viewer';
  token: string;  // Secure UUID v4
  invited_by: string;  // User ID
  invited_at: string;
  expires_at: string;  // 7 days from invited_at
  status: 'pending' | 'accepted' | 'declined' | 'expired';
  accepted_at?: string;
}
```

**Bulk Invite CSV Format**:
```csv
email,role,custom_message
alice@example.com,admin,Welcome to the ops team!
bob@example.com,editor,
carol@example.com,viewer,
```

**Edge Cases**:
- User already member of workspace (show error, don't send duplicate invitation)
- Invitation to existing CloudMetrics user (skip signup, just link to workspace)
- User accepts multiple invitations before logging in (link all workspaces upon login)
- Admin removes themselves (allow if other Admins exist, prevent if last Admin)
- Owner transfers ownership (require new Owner acceptance, then demote to Admin)
- Expired invitation accepted (show error, offer to request new invitation)

## Tasks

- [ ] Design role capability matrix and document permissions
- [ ] Create invitation database schema with token generation
- [ ] Build invitation email templates (invite, acceptance, decline)
- [ ] Implement email invitation sending with SendGrid
- [ ] Create invitation accept/decline endpoints
- [ ] Build team member management UI with role display
- [ ] Implement role assignment and change functionality
- [ ] Add invitation status tracking (pending, accepted, declined)
- [ ] Create bulk invite CSV upload and validation
- [ ] Implement member removal with resource transfer
- [ ] Add Last Owner protection (cannot remove only Owner)
- [ ] Build invitation expiry and resend logic
- [ ] Write RBAC middleware for permission enforcement
- [ ] Create team management integration tests
- [ ] Document role permissions and invitation workflows

## Dependencies

- Story 1-2: Authentication System (user accounts)
- Story 1-3: Multi-Tenancy (workspace membership)

## Estimation

**Story Points**: 5

**Breakdown**:
- Role matrix and schema: 1 point
- Invitation system: 2 points
- Team management UI: 1 point
- Bulk invite and testing: 1 point
