---
name: dashboard-design
description: 
category: design
tags: [dashboard-design]
---

## When to Use
Building data-rich interfaces for monitoring, analytics, admin panels, or operations dashboards. When users need to scan, filter, and act on multiple data streams simultaneously.

## Core Concepts
- **Dashboard purpose**: Three types — monitoring (status), analytical (exploration), operational (action)
- **Information density**: Dashboards require higher density than marketing pages — balance with whitespace
- **Scanning patterns**: F-pattern (list-heavy), Z-pattern (card-heavy), quadrant scan (balanced grids)
- **KPI cards**: Hero metrics at top — large numbers with trend indicators (↑↓ and % change)
- **Widget composition**: Chart, table, list, stat card — each widget serves a specific purpose
- **Filtering strategy**: Global filters (affect all widgets) vs. widget-specific filters
- **Real-time updates**: Live data with clear update indicators (last updated timestamp, pulse animation)

## Workflow
1. Define user role: what decisions does this person make daily? What data do they need?
2. Prioritize metrics: top 3-5 KPIs go in hero cards at the top
3. Design layout grid: 4-column (desktop) / 2-column (tablet) / 1-column (mobile)
4. Build widget types: stat card, line chart, bar chart, table, map, activity feed
5. Implement global filters: date range, category, team, status
6. Add drill-down: clicking a chart element filters the whole dashboard or navigates to detail
7. Design for scan-ability: 3-second scan should reveal overall status
8. Handle loading/skeleton states for async data

## Key Patterns
- **KPI hero card**:
  ```
  Metric name: "Monthly Active Users"
  Value: "12,847"
  Trend: "↑ 8.2% from last month" (green)
  Sparkline: inline mini chart showing 12-month trend
  ```
- **Dashboard grid layout**:
  ```
  Row 1: [KPI] [KPI] [KPI] [KPI]              ← 4 stat cards
  Row 2: [Line chart: Revenue] [Pie: By plan]   ← 2:1 split
  Row 3: [Table: Recent activity] (full width)   ← scrollable table
  Row 4: [Bar: By region] [List: Top items]      ← 1:1 split
  ```
- **Filter bar pattern**:
  ```
  [Date range: Last 30 days ▼] [Status: All ▼] [Team: All ▼] [Clear filters]
  ```
- **Table with inline actions**: Row hover reveals "Edit", "Delete", "View" — keeps table clean
- **Empty dashboard state**: "Connect your first data source to see metrics here" + setup CTA
- **Responsive dashboard**: Cards stack to single column on mobile, charts switch to horizontal bars

## Pitfalls
- Too many metrics on screen — if everything is important, nothing is; prioritize ruthlessly
- Charts without clear labels — every axis must have a label and unit (%, $, count)
- No time range selector — users always want to change the date window
- Tables without search/filter/sort — data tables are useless without these
- Loading spinners replacing entire dashboard — use skeleton cards, load widgets independently
- Assuming desktop-only — many users check dashboards on mobile; test the mobile layout
- No empty/loading/error states — data sources fail; handle gracefully

## Verification
- 3-second test: can user determine overall system health in 3 seconds?
- Filter test: apply date range filter — all widgets update, URL reflects filter state
- Mobile test: dashboard is usable on phone — key metrics visible, charts readable
- Performance: dashboard loads in < 3s with all widgets (lazy-load below-fold)
- Drill-down test: click any chart element — user can navigate to underlying data