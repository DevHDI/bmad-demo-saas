---
status: in-progress
startedAt: "2026-02-07"
epic: "Epic 3: Dashboards & Visualizations"
storyPoints: 8
---

# Story 3-2: Chart & Visualization Widgets

## Description

Develop a comprehensive library of data visualization widgets using Recharts, providing the building blocks for analytics dashboards. The widget library includes line charts for time-series trends, bar charts for categorical comparisons, pie/donut charts for composition analysis, area charts for cumulative metrics, and metric cards for KPI displays with sparklines and trend indicators.

Each widget is designed following UX best practices from the design specifications: dark mode by default, colorblind-safe palette, clear data-ink ratio (avoiding chartjunk), and responsive behavior across screen sizes. Charts support interactive features including hover tooltips with crosshairs, click-to-zoom, legend filtering, and data export to PNG/CSV for sharing.

The widget system is theme-aware, automatically adapting colors and styles to match the CloudMetrics design system (JetBrains Mono for data, Inter for labels, blue/violet/cyan color series). All charts respect accessibility standards with ARIA labels, keyboard navigation, and screen reader support for data tables behind visualizations.

## Acceptance Criteria

- AC-1: Line chart widget with multi-series support (up to 10 series), legend, and hover tooltips
- AC-2: Bar chart with vertical/horizontal orientation, grouped/stacked modes, and value labels
- AC-3: Pie/donut chart with interactive segments, percentage labels, and max 8 slices (rest grouped as "Other")
- AC-4: Area chart with gradient fills, stacking support, and baseline zero
- AC-5: Metric card showing large numeric value, trend indicator (arrow + percentage), and sparkline (last 24 points)
- AC-6: All charts support dark/light mode with theme-appropriate colors
- AC-7: Chart export functionality (PNG screenshot, CSV data download)
- AC-8: Responsive charts adapt to container size with debounced redraw
- AC-9: Loading states with skeleton screens during data fetch
- AC-10: Empty states with helpful messaging when no data available

## Technical Notes

**Recharts Configuration**:
```typescript
import { LineChart, Line, XAxis, YAxis, CartesianGrid, Tooltip, Legend, ResponsiveContainer } from 'recharts';

// Theme-aware color palette
const chartColors = {
  series: ['#3B82F6', '#8B5CF6', '#06B6D4', '#F59E0B', '#22C55E', '#EC4899'],
  grid: '#334155',
  text: '#94A3B8',
  background: '#1E293B'
};
```

**Widget Types**:
1. **LineChart**: Time-series data with date x-axis, numeric y-axis, multiple series
2. **BarChart**: Categorical x-axis, numeric y-axis, grouped or stacked bars
3. **PieChart**: Categorical data with percentages, donut variant with center metric
4. **AreaChart**: Similar to line chart but with filled gradient area below line
5. **MetricCard**: Single KPI value with optional sparkline and comparison to previous period

**Chart Interactions**:
- Hover tooltips: Crosshair with formatted values for all series
- Legend click: Toggle series visibility
- Zoom: Click-and-drag on chart area to zoom into time range
- Export: Button in widget header for PNG/CSV download

**Responsive Behavior**:
- Charts use ResponsiveContainer to fill available space
- Font sizes scale down on mobile (12px → 10px)
- Legends move below chart on narrow screens
- Touch-friendly hit targets for mobile interactions

**Edge Cases**:
- No data available (show empty state with illustration and message)
- Single data point (show point marker, no line)
- All values zero (adjust y-axis to show scale, not flat line)
- Extremely large/small numbers (use SI suffixes: K, M, B)
- Chart container resize during render (debounce redraw by 150ms)

## Tasks

- [x] Set up Recharts library with theme provider integration
- [x] Create base ChartWidget wrapper component with header and menu
- [x] Build LineChart widget with multi-series support
- [x] Implement hover tooltips with crosshair and formatted values
- [x] Build BarChart widget with grouped and stacked modes
- [x] Create PieChart/DonutChart with interactive segments
- [ ] Build AreaChart widget with gradient fills
- [ ] Create MetricCard component with sparkline integration
- [ ] Implement chart export to PNG using html-to-image library
- [ ] Add CSV data export functionality
- [ ] Create loading skeleton states for all widgets
- [ ] Build empty state components with helpful messaging
- [ ] Implement responsive behavior for mobile screens
- [ ] Add accessibility features (ARIA labels, keyboard nav)
- [ ] Write Storybook stories for all widget variants
- [ ] Create widget configuration panel for editing

## Dependencies

- Story 3-1: Dashboard Framework (widget container system)
- Story 2-3: Data Storage Layer (query engine for chart data)

## Estimation

**Story Points**: 8

**Breakdown**:
- LineChart and BarChart: 2 points
- PieChart and AreaChart: 2 points
- MetricCard with sparkline: 2 points
- Export, loading states, accessibility: 2 points
