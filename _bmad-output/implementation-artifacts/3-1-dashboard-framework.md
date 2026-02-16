---
status: done
completedAt: "2026-02-06"
epic: "Epic 3: Dashboards & Visualizations"
storyPoints: 8
---

# Story 3-1: Dashboard Layout Framework

## Description

Build the foundational dashboard layout system providing a responsive 12-column grid that users can customize through drag-and-drop widget placement and resizing. The framework uses modern React libraries (dnd-kit for drag-and-drop, react-grid-layout for responsive grids) to deliver a professional dashboard editing experience comparable to enterprise tools like Datadog and Grafana.

The layout system persists each user's dashboard configuration in the database, allowing customization without affecting other team members. Users can create multiple dashboards per workspace, each with its own widget arrangement. The system provides default dashboard templates for common use cases (application monitoring, user analytics, error tracking) to enable quick setup for new users.

Performance is critical given the real-time nature of analytics dashboards. The grid system uses CSS Grid for native browser performance, lazy-loading for off-screen widgets, and virtualization for dashboards with many widgets. Layout changes are debounced and persisted in batches to reduce database writes during editing sessions.

## Acceptance Criteria

- AC-1: Responsive 12-column CSS Grid layout adapts to screen sizes (desktop, tablet, mobile)
- AC-2: Drag-and-drop widget repositioning using dnd-kit with snap-to-grid behavior
- AC-3: Widget resize handles (corner and edge) with minimum size constraints (2x2 grid units)
- AC-4: Layout persistence API saves dashboard configuration per user with versioning
- AC-5: Edit mode toggle clearly indicates when layout is editable vs view-only
- AC-6: Default dashboard templates for common use cases (5+ templates)
- AC-7: Dashboard cloning creates copy with same widget layout
- AC-8: Layout changes auto-save with debouncing (2-second delay)
- AC-9: Undo/redo support for layout changes during edit session
- AC-10: Mobile-responsive behavior stacks widgets vertically on small screens

## Technical Notes

**Technology Stack**:
- @dnd-kit/core for drag-and-drop interactions
- react-grid-layout for responsive grid positioning
- CSS Grid for native browser layout performance
- Zustand for client-side layout state management
- Optimistic updates with rollback on API failure

**Layout Data Model**:
```typescript
interface DashboardLayout {
  id: string;
  tenant_id: string;
  user_id: string;
  name: string;
  widgets: WidgetConfig[];
  created_at: string;
  updated_at: string;
}

interface WidgetConfig {
  id: string;
  type: 'line_chart' | 'bar_chart' | 'metric_card' | 'table';
  x: number;  // Grid column (0-11)
  y: number;  // Grid row
  w: number;  // Width in grid units (1-12)
  h: number;  // Height in grid units (min 2)
  config: Record<string, unknown>;  // Widget-specific settings
}
```

**Default Templates**:
1. Application Monitoring: Error rate, response time, throughput, database queries
2. User Analytics: Active users, new signups, session duration, page views
3. Error Tracking: Error count by type, error rate over time, recent errors table
4. Performance Dashboard: p50/p95/p99 latency, throughput, cache hit rate
5. Business Metrics: Conversion rate, revenue, user engagement, feature adoption

**Responsive Breakpoints**:
- Desktop: > 1280px (12 columns, full drag-and-drop)
- Tablet: 768-1280px (6 columns, simplified editing)
- Mobile: < 768px (1 column, view-only, no drag-and-drop)

**Edge Cases**:
- Concurrent editing by multiple users (last write wins, show warning)
- Widget deleted during resize (cancel resize, remove from layout)
- Network error during save (show error toast, allow retry)
- Invalid grid position after screen resize (auto-adjust to valid position)
- Maximum widgets per dashboard (limit to 50, show warning at 40)

## Tasks

- [x] Set up react-grid-layout with responsive breakpoints
- [x] Implement CSS Grid base layout with 12 columns
- [x] Integrate dnd-kit for drag-and-drop with collision detection
- [x] Build widget resize functionality with min/max constraints
- [x] Create edit mode toggle with visual indicators
- [x] Implement layout persistence API (GET, PUT /dashboards/:id/layout)
- [x] Build auto-save with debouncing (2-second delay)
- [x] Add undo/redo state management with Zustand
- [x] Create 5 default dashboard templates
- [x] Implement dashboard cloning feature
- [x] Add mobile-responsive stacking behavior
- [x] Build layout validation and error handling
- [x] Create dashboard selector and navigation
- [x] Write layout persistence integration tests
- [x] Document dashboard editing user guide

## Dependencies

- Story 1-1: Platform Initialization (Next.js, database)
- Story 1-3: Multi-Tenancy (tenant-aware dashboards)

## Estimation

**Story Points**: 8

**Breakdown**:
- Grid layout and responsive design: 2 points
- Drag-and-drop implementation: 2 points
- Widget resizing: 1 point
- Persistence and state management: 2 points
- Templates and testing: 1 point
