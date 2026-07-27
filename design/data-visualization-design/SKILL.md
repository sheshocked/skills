---
name: data-visualization-design
description: 
category: design
tags: [data-visualization-design]
---

## When to Use
Presenting data as charts, graphs, maps, or dashboards. When users need to identify trends, compare values, understand distributions, or make data-driven decisions.

## Core Concepts
- **Chart type selection**: Bar = comparison, line = trend over time, pie = part-of-whole (≤5 slices), scatter = correlation
- **Data-ink ratio** (Tufte): Maximize ink that represents data, minimize decorative elements
- **Pre-attentive attributes**: Length, position, angle, color — processed by brain before conscious thought
- **Lie factor**: Visual representation should scale proportionally to data values
- **Cleveland's hierarchy**: Position on common scale > position on non-aligned scales > length > angle > area
- **Color encoding**: Sequential (ordered data), diverging (data with midpoint), categorical (distinct groups)
- **Annotation**: Label key data points directly instead of relying on legends

## Workflow
1. Define the question the visualization must answer
2. Choose the simplest chart type that answers the question
3. Sort data meaningfully (by value, not alphabetically) unless time-series
4. Label directly: axis labels, value labels on key points, title states the insight
5. Use color sparingly — highlight one data series, gray out the rest
6. Add annotations for anomalies, milestones, or key takeaways
7. Test: can a non-expert understand the chart in < 5 seconds?
8. Build responsive: charts must work at mobile widths (consider horizontal bars instead of vertical)

## Key Patterns
- **Horizontal bar chart** for category comparison:
  ```
  Sort descending by value.
  Label bars directly (no legend).
  Use one accent color for the top/benchmark, gray for the rest.
  ```
- **Sparkline for context**: Inline mini line charts in tables to show trend without full chart
- **Annotated timeline**: Line chart with callout boxes at key events ("Product launch", "Price increase")
- **Progress donut for single KPI**:
  ```
  Arc from 0° to (percentage × 360°).
  Center text: the value (e.g., "73%").
  Caption: the metric name.
  ```
- **Responsive chart pattern**: At < 640px, switch to horizontal bars (labels fit better)

## Pitfalls
- 3D charts — distortion makes comparison impossible; always use 2D
- Dual y-axes — confusing and often misleading; use separate charts instead
- Pie charts with 8+ slices — impossible to compare; use a horizontal bar chart
- Truncated y-axis — exaggerates differences; start at zero for bar charts
- Color-coding without a legend — or with a legend far from the chart
- Dashboard overload — too many charts per screen; prioritize 3-5 key metrics

## Verification
- The "5-second test": show chart for 5 seconds, ask "what's the main takeaway?" — users must answer correctly
- Check data-ink ratio: remove gridlines, borders, decorative elements — only data ink remains
- Verify all axes start at zero for bar charts
- Test with color-blindness simulation — data must be distinguishable without color
- Responsive check: chart readable at 320px width