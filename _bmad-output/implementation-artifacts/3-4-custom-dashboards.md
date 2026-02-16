---
status: backlog
epic: "Epic 3: Dashboards & Visualizations"
storyPoints: 8
---

# Story 3-4: Custom Dashboard Builder

## Description

Empower users to create custom dashboards tailored to their specific monitoring needs through a visual builder interface. The builder provides drag-and-drop widget creation, point-and-click metric selection, and a configuration panel for customizing chart appearance and data sources. This self-serve capability eliminates dependence on engineering teams for dashboard creation, enabling product managers and analysts to build their own analytics views.

The dashboard builder follows a progressive disclosure pattern: start simple with pre-configured widgets, then reveal advanced options for power users. The metric selector provides autocomplete search across all available metrics with helpful descriptions and example values. Widget configuration panels expose settings for time ranges, aggregation functions, grouping dimensions, and visual styling (colors, chart types, axis ranges).

Dashboard sharing enables collaboration through team-wide visibility controls. Users can create private dashboards for personal analysis, share with specific team members for collaboration, or publish to the entire workspace for organization-wide visibility. Cloning existing dashboards accelerates creation of similar views, while saving dashboards as templates enables reuse across projects and teams.

## Acceptance Criteria

- AC-1: Visual dashboard builder interface with "Add Widget" button and type picker modal
- AC-2: Data source selector with search and autocomplete across metrics (events, aggregates, custom)
- AC-3: Metric selector with dimension selection (e.g., "Count events by type")
- AC-4: Widget configuration panel with time range, aggregation, filters, and grouping
- AC-5: Visual styling options: chart type, color palette, axis labels, legend position
- AC-6: Dashboard sharing controls: private, shared with users, public to workspace
- AC-7: Dashboard cloning creates editable copy with same widgets and configuration
- AC-8: Save dashboard as template for reuse across projects
- AC-9: Template library showing organization templates and public examples
- AC-10: Preview mode shows how dashboard will appear to viewers before saving

## Technical Notes

**Builder UI Components**:
- Widget type picker: Modal with visual cards for each chart type
- Metric selector: Searchable dropdown with categorized metrics (Events, Sessions, Errors, Custom)
- Configuration panel: Sidebar with collapsible sections (Data, Time Range, Visualization, Advanced)
- Preview pane: Live chart preview updating as configuration changes
- Save controls: Name, description, sharing settings, and save/publish buttons

**Metric Selection Flow**:
1. User clicks "Add Widget" → Widget type picker modal
2. Select widget type (Line Chart) → Metric selector appears
3. Search for metric (e.g., "error") → Autocomplete shows "Error Count", "Error Rate"
4. Select metric → Dimension selector shows grouping options (type, browser, country)
5. Configure aggregation (sum, count, avg, p95, etc.)
6. Preview updates in real-time → Save to add widget to dashboard

**Dashboard Sharing Model**:
```typescript
interface DashboardSharing {
  visibility: 'private' | 'shared' | 'workspace';
  shared_with_users?: string[];  // User IDs when visibility=shared
  allow_comments: boolean;
  allow_cloning: boolean;
  public_link?: string;  // Optional public read-only link
}
```

**Template System**:
- Organization templates: Created by admins, available to all workspace members
- Personal templates: User's own dashboards saved as templates
- Public gallery: CloudMetrics-provided templates for common use cases
- Template metadata: Name, description, tags, screenshot, creator, usage count

**Edge Cases**:
- Metric deleted while widget configured (show error, prompt to select different metric)
- User loses access to dashboard during editing (save fails, show permission error)
- Template uses custom metrics not available in workspace (prompt to create or skip)
- Dashboard with 50+ widgets (warn about performance impact)
- Sharing with user who later leaves workspace (automatically revoke access)

## Tasks

- [ ] Design and build dashboard builder UI layout
- [ ] Create widget type picker modal with visual cards
- [ ] Build metric selector with search and autocomplete
- [ ] Implement dimension selector for grouping and filtering
- [ ] Create widget configuration panel with collapsible sections
- [ ] Build live preview pane with real-time configuration updates
- [ ] Implement dashboard sharing controls and permissions
- [ ] Create dashboard cloning functionality
- [ ] Build template save and publish system
- [ ] Implement template library with organization and public templates
- [ ] Add dashboard preview mode for viewers
- [ ] Create validation for required fields (name, at least one widget)
- [ ] Write user guide documentation for dashboard builder
- [ ] Build onboarding tour highlighting builder features
- [ ] Test dashboard builder with non-technical users

## Dependencies

- Story 3-1: Dashboard Framework (layout system)
- Story 3-2: Chart Widgets (widget types to add)
- Story 2-3: Data Storage Layer (available metrics)

## Estimation

**Story Points**: 8

**Breakdown**:
- Builder UI and widget picker: 2 points
- Metric and dimension selectors: 2 points
- Configuration panels: 2 points
- Sharing and templates: 2 points
