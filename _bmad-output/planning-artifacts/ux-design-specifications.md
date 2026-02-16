---
status: done
completedAt: "2026-01-18"
---
# CloudMetrics - UX Design Specifications

## 1. Design Principles

### 1.1 Core Principles

1. **Data Clarity**: Every visualization should communicate one clear insight. Avoid chartjunk, maximize data-ink ratio (Tufte).
2. **Progressive Complexity**: Simple by default, powerful when needed. The default dashboard works out of the box; power users can customize everything.
3. **Real-time Confidence**: Users must always know if data is live or stale. Clear indicators for connection status, last-updated timestamps, and loading states.
4. **Dark-Mode First**: Analytics dashboards are often monitored on wall displays or during incidents. Dark mode reduces eye strain and looks professional.

### 1.2 Design System

**Typography**:
- Headings: JetBrains Mono (500/700 weight) - technical, precise feel
- Body: Inter (400/500 weight) - highly legible at small sizes
- Data: JetBrains Mono (400 weight) - monospace for numbers and metrics
- Scale: 11px / 12px / 14px / 16px / 20px / 24px / 32px

**Color Palette (Dark Mode)**:
- Background: #0F172A (slate-900)
- Surface: #1E293B (slate-800)
- Elevated: #334155 (slate-700)
- Primary: #3B82F6 (blue-500) - Interactive elements
- Success: #22C55E (green-500) - Healthy, increasing
- Warning: #F59E0B (amber-500) - Degraded, caution
- Error: #EF4444 (red-500) - Critical, decreasing
- Info: #06B6D4 (cyan-500) - Informational
- Text Primary: #F8FAFC (slate-50)
- Text Secondary: #94A3B8 (slate-400)

**Chart Colors** (sequential, colorblind-safe):
- Series 1: #3B82F6 (blue)
- Series 2: #8B5CF6 (violet)
- Series 3: #06B6D4 (cyan)
- Series 4: #F59E0B (amber)
- Series 5: #22C55E (green)
- Series 6: #EC4899 (pink)

**Spacing**: 4px base unit

**Border Radius**: 6px (small), 8px (medium), 12px (large)

## 2. Page Specifications

### 2.1 Main Dashboard View

**Layout**: Full-width with collapsible sidebar (240px)

**Header Bar**:
- Logo + workspace name
- Global time range selector (Last 15m / 1h / 6h / 24h / 7d / 30d / Custom)
- Auto-refresh toggle with interval selector (5s / 15s / 30s / 1m / Off)
- Connection status indicator (green dot = live, yellow = reconnecting, red = disconnected)
- User avatar + dropdown

**Dashboard Grid**:
- CSS Grid, 12 columns
- Widgets snap to grid on drag
- Resize handles on bottom-right corner
- Widget header: title + time range override + menu (edit, duplicate, delete)
- Widget loading: skeleton with shimmer effect

### 2.2 Widget Types

**Metric Card**:
- Large number display (48px JetBrains Mono)
- Trend indicator (arrow up/down + percentage)
- Sparkline chart (last 24 data points)
- Comparison text ("vs. last period")

**Line Chart**:
- Multi-series with legend
- Hover tooltip with crosshair
- Zoom via click-and-drag
- Y-axis auto-scaling
- Grid lines at 25% intervals

**Bar Chart**:
- Vertical or horizontal orientation
- Grouped or stacked mode
- Value labels on hover
- Sortable by value

**Pie/Donut Chart**:
- Interactive segments (hover to expand)
- Center metric (donut only)
- Legend with percentages
- Maximum 8 segments (group rest as "Other")

**Data Table**:
- Sortable columns
- Pagination (10/25/50 rows)
- Search/filter row
- Export button (CSV)

### 2.3 Alert Management

**Alert List**:
- Table view with columns: Status, Rule Name, Severity, Last Triggered, Actions
- Status badges: Active (green), Firing (red pulsing), Muted (gray)
- Inline enable/disable toggle
- Severity color coding: Critical (red), Warning (amber), Info (blue)

**Alert Rule Editor**:
- Step 1: Select metric and aggregation
- Step 2: Define condition (threshold, anomaly, composite)
- Step 3: Configure notification channels
- Step 4: Set evaluation interval and mute schedule
- Preview: Show how rule would have triggered on historical data

### 2.4 Settings Pages

**Workspace Settings**:
- General (name, logo, timezone)
- Team members (invite, role management)
- Billing (current plan, usage, invoices)
- API keys (create, revoke, permissions)
- Integrations (connected services)

## 3. Interaction Patterns

### 3.1 Real-time Indicators
- **Live dot**: Pulsing green dot in header when SSE connected
- **Stale badge**: "Data may be stale" yellow banner if SSE disconnects > 30s
- **Update flash**: Brief highlight animation when metric values change
- **Reconnecting**: Subtle toast "Reconnecting..." with spinner

### 3.2 Dashboard Editing
- **Edit mode**: Toggle via pencil icon, shows grid lines and resize handles
- **Add widget**: "+" button opens widget type picker modal
- **Move widget**: Drag from header bar, ghost element shows drop position
- **Resize widget**: Drag corner handle, minimum 2x2 grid units
- **Save/Cancel**: Floating action bar at bottom during edit mode

### 3.3 Time Range Selection
- **Presets**: Quick buttons for common ranges
- **Custom**: Calendar picker for start/end date
- **Zoom**: Click-and-drag on any chart to zoom into time range
- **Compare**: Toggle "Compare to previous period" for overlay

## 4. Responsive Strategy

CloudMetrics prioritizes desktop experience (analytics dashboards are primarily desktop tools), with tablet support for monitoring scenarios:

| Breakpoint | Width | Behavior |
|------------|-------|----------|
| Desktop | > 1280px | Full dashboard with sidebar |
| Compact | 1024-1280px | Collapsed sidebar, widgets reflow |
| Tablet | 768-1024px | Single-column widget stack |
| Mobile | < 768px | View-only mode, simplified charts |

## 5. Accessibility

- WCAG 2.1 AA compliance
- Chart data available as accessible tables
- Color-blind safe palette (tested with Coblis simulator)
- High contrast mode support
- Keyboard navigation for all dashboard operations
- Screen reader announcements for metric changes
