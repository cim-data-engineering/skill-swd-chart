# SWD Charts — Core Configuration

Shared color tokens, typography, Chart.js base config, and cross-cutting patterns.
Read this before creating any chart.

## Table of Contents
1. [Color Palette & Tokens](#1-color-palette--tokens)
2. [Typography](#2-typography)
3. [Chart.js Global Configuration](#3-chartjs-global-configuration)
4. [Chart Selection Decision Tree](#4-chart-selection-decision-tree)
5. [Annotation Patterns](#5-annotation-patterns)
6. [Anti-Patterns to Avoid](#6-anti-patterns-to-avoid)
7. [HTML Wrapper Template](#7-html-wrapper-template)

---

## 1. Color Palette & Tokens

### Primary palette
```
ACCENT_BLUE    = '#2563EB'   // Primary highlight — positive, standout, key data
ACCENT_ORANGE  = '#D97706'   // Negative highlight — warning, decline, concern
BASE_DARK      = '#4B5563'   // Chart titles, primary axis labels
BASE_MID       = '#9CA3AF'   // Secondary text, axis lines, tick labels
BASE_LIGHT     = '#D1D5DB'   // De-emphasized data series, secondary lines
BASE_FAINT     = '#E5E7EB'   // Gridlines (when kept), subtle borders
BG_TRANSPARENT = 'transparent' // Chart and plot area backgrounds — always transparent
WHITE          = '#FFFFFF'   // Text on colored bars, tooltip backgrounds
```

### Extended accent shades (for multi-series when necessary)
```
BLUE_LIGHT     = '#93C5FD'   // Secondary blue series (de-emphasized)
BLUE_DARK      = '#1D4ED8'   // Tertiary blue (darker, for waterfall totals)
ORANGE_LIGHT   = '#FCD34D'   // Secondary orange (lighter warning)
GREY_SERIES    = ['#6B7280', '#9CA3AF', '#D1D5DB']  // Competitor/context series
```

### When to use which color
- **One data series, no special story:** Use ACCENT_BLUE for all bars/lines
- **One series, highlight subset:** ACCENT_BLUE for highlighted bars, BASE_LIGHT for the rest
- **Positive vs negative:** ACCENT_BLUE for positive values, ACCENT_ORANGE for negative
- **Primary vs context:** ACCENT_BLUE for "our data," GREY_SERIES for competitors/benchmarks
- **Never use more than 3 distinct colors** in a single chart (excluding grey)

---

## 2. Typography

Use the CSS variable `var(--vz-font-family)` from the Visualizer. Fallback: `system-ui, -apple-system, sans-serif`.

```
Title:        16px, font-weight 700, color BASE_DARK
Subtitle:     13px, font-weight 400, color BASE_MID
Axis title:   12px, font-weight 600, color BASE_MID
Axis labels:  11px, font-weight 400, color BASE_MID
Data labels:  11px, font-weight 600, color BASE_DARK (or WHITE on colored fills)
Annotations:  11px, font-weight 400, color BASE_DARK
Footnote:     10px, font-weight 400, color BASE_MID
```

---

## 3. Chart.js Global Configuration

Apply this base config to every Chart.js chart. It implements the SWD decluttering principles.

```javascript
// SWD Base Configuration — apply to all Chart.js charts
const SWD = {
  colors: {
    blue: '#2563EB',
    orange: '#D97706',
    dark: '#4B5563',
    mid: '#9CA3AF',
    light: '#D1D5DB',
    faint: '#E5E7EB',
    white: '#FFFFFF',
  },

  baseConfig: {
    responsive: true,
    maintainAspectRatio: false,
    animation: { duration: 600, easing: 'easeOutQuart' },

    layout: {
      padding: { top: 8, right: 16, bottom: 8, left: 8 }
    },

    plugins: {
      legend: { display: false },        // Never show legend — label directly
      title: { display: false },          // We render titles in HTML above the canvas
      tooltip: {
        backgroundColor: '#FFFFFF',
        titleColor: '#4B5563',
        bodyColor: '#4B5563',
        borderColor: '#E5E7EB',
        borderWidth: 1,
        cornerRadius: 4,
        padding: 10,
        titleFont: { weight: '600', size: 12 },
        bodyFont: { size: 11 },
        displayColors: false,             // Remove colored squares in tooltip
      },
    },

    scales: {
      x: {
        border: { display: false },       // Remove axis line
        grid: { display: false },          // Remove gridlines
        ticks: {
          color: '#9CA3AF',
          font: { size: 11 },
          padding: 4,
        },
      },
      y: {
        border: { display: false },
        grid: {
          display: true,                   // Light horizontal gridlines for value reference
          color: '#E5E7EB',
          lineWidth: 0.5,
        },
        ticks: {
          color: '#9CA3AF',
          font: { size: 11 },
          padding: 8,
        },
      },
    },

    elements: {
      bar: {
        borderWidth: 0,
        borderRadius: 2,
      },
      line: {
        borderWidth: 2.5,
        tension: 0,                        // No curve — straight segments (honest data)
        fill: false,
      },
      point: {
        radius: 0,                         // Never show data markers
        hoverRadius: 0,                    // Never show circles on hover
        hitRadius: 14,                     // But keep generous hit area for interaction
      },
    },
  },
};
```

### Bar width rule
Bars should be wider than the gaps between them. In Chart.js, set `barPercentage: 0.75` and
`categoryPercentage: 0.85` as a starting point. Adjust if bars look too thin or too thick.

### Zero baseline rule
Bar charts MUST always include zero. Never set `beginAtZero: false` on a bar chart axis.
Line charts may use a non-zero baseline if the range is narrow relative to values, but be cautious.

---

## 4. Chart Selection Decision Tree

Ask yourself these questions about the data:

```
Q: How many numbers are we showing?
├─ Just 1-2 numbers → Use SIMPLE TEXT (big number + context sentence), not a chart
│
Q: Are there multiple measures with DIFFERENT UNITS over time?
├─ Yes → MERGED LINE CHART (stacked vertically, shared x-axis)
│  └─ If comparing relative patterns of change is the main point → consider INDEX CHART
│
Q: Is the data categorical or continuous?
├─ Continuous (time series, sequential) → LINE CHART
│  ├─ Single metric over time → single line
│  ├─ 2-4 metrics over time → multi-line (label directly, avoid >4 lines)
│  ├─ >4 lines → highlight one at a time or use small multiples
│  ├─ Showing a range/confidence → line with shaded band
│  └─ Only 2 time points → HORIZONTAL BAR (grouped, before/after pairs) or simple 2-point line
│
├─ Categorical (groups, segments)
│  ├─ Comparing values across categories → HORIZONTAL BAR (preferred) or VERTICAL BAR
│  │  ├─ Long category names → definitely horizontal bar
│  │  ├─ Ordered by value (largest first) unless natural order exists
│  │  └─ Time-based categories (quarters, months) → vertical bar
│  │
│  ├─ Part-to-whole
│  │  ├─ Simple proportion → 100% STACKED HORIZONTAL BAR
│  │  ├─ Survey Likert scale → 100% STACKED HORIZONTAL BAR (diverging from center)
│  │  └─ Just a few segments → simple text with percentages
│  │
│  └─ Building to a total (start + adds - subtracts = end) → WATERFALL
│
└─ Table of values where magnitude matters → HEATMAP
```

---

## 5. Annotation Patterns

### When to annotate
- There is a clear, specific insight (a spike, a crossover, a gap, an inflection point)
- The data tells a story that benefits from a brief callout
- External context explains a data anomaly

### When NOT to annotate
- The user is exploring data without a pre-determined narrative
- The chart is straightforward and speaks for itself
- You would be stating the obvious ("Revenue went up")

### Annotation style
```html
<!-- Action title (above chart, when there's a clear takeaway) -->
<div style="font-size:14px; font-weight:700; color:#4B5563; margin-bottom:2px;">
  Revenue grew 23% YoY, driven by APAC expansion
</div>
<div style="font-size:12px; color:#9CA3AF; margin-bottom:12px;">
  Quarterly revenue by region, FY2023–FY2025
</div>

<!-- Inline annotation (on the chart itself, near the data point) -->
<!-- Use a small text element positioned near the relevant data with a thin leader line if needed -->
```

### Annotation content rules
- Brief phrases, not full sentences (e.g., "2 employees quit in May" not "Two employees quit during the month of May which caused...")
- Position near the data they describe (proximity principle)
- Same font as the rest of the chart, but can be slightly smaller
- Use BASE_DARK color, or match the accent color of the data being annotated

---

## 6. Anti-Patterns to Avoid

These are violations of SWD principles. Never do these:

| Anti-Pattern | Why It's Bad | What to Do Instead |
|---|---|---|
| Pie chart / donut chart | Eyes can't compare angles or arc lengths accurately | Horizontal bar chart |
| 3D effects | Distorts values, adds meaningless visual noise | Always use 2D |
| Rainbow color palette | Too many colors = nothing stands out | Grey base + 1-2 accent colors |
| Legend box off to the side | Forces back-and-forth eye movement | Label data series directly |
| Chart border / background fill | Unnecessary visual clutter | Transparent background, no border |
| Heavy gridlines | Compete with data for attention | Remove or make very faint |
| Data markers on every point | Adds cognitive load without insight | Remove, or use only on key points |
| Diagonal / rotated text | 52% slower to read than horizontal | Abbreviate labels or switch to horizontal bar |
| Dual-axis chart / Secondary y-axis | Confusing — audiences misread which scale applies; encourages nonsensical comparisons between unrelated units | Use merged line charts (stacked vertically, shared x-axis) or index charts (rebase to % change) |
| Non-zero baseline on bars | Visually exaggerates differences, misleading | Always start bar charts at zero |
| Trailing decimals (50.00) | Look more complicated than necessary | Write 50 |
| Excessive data labels | Creates clutter | Label only the points that matter |
| Center-aligned title | Doesn't create clean alignment lines | Left-align all text |
| Stretching chart to fill space | Wastes white space that could add clarity | Size chart appropriately to content |

---

## 7. HTML Wrapper Template

Every chart rendered via show_widget should use this structure:

```html
<div style="font-family: system-ui, -apple-system, sans-serif; padding: 16px; max-width: 700px;">

  <!-- Action title (only when there's a clear insight) -->
  <div style="font-size:14px; font-weight:700; color:#4B5563; margin-bottom:2px;">
    [Insight headline if applicable]
  </div>

  <!-- Chart title (always present) -->
  <div style="font-size:13px; color:#9CA3AF; margin-bottom:12px;">
    [Descriptive chart title] · [unit or time range]
  </div>

  <!-- Inline legend (only for multi-series when direct labeling isn't feasible) -->
  <!-- Use small colored dots + text, not a box -->

  <!-- Chart canvas -->
  <div style="position:relative; height:350px;">
    <canvas id="chart"></canvas>
  </div>

  <!-- Footnote (optional, for data source or methodology note) -->
  <div style="font-size:10px; color:#9CA3AF; margin-top:8px;">
    Data source: [source]
  </div>
</div>

<script src="https://cdn.jsdelivr.net/npm/chart.js@4/dist/chart.umd.min.js"></script>
<script>
  // Chart.js code here using SWD config patterns
</script>
```

For D3/SVG charts (waterfall, heatmaps), replace the canvas with an inline `<svg>` element
and use the same HTML wrapper structure for titles and footnotes.
