# SWD Charts — Design System Reference

This file contains the complete implementation details for building SWD-compliant charts
via the Visualizer tool. Read this before creating any chart.

## Table of Contents
1. [Color Palette & Tokens](#color-palette)
2. [Typography](#typography)
3. [Chart.js Global Configuration](#chartjs-global-config)
4. [Chart Selection Decision Tree](#chart-selection)
5. [Chart Type: Horizontal Bar](#horizontal-bar)
6. [Chart Type: Vertical Bar (Column)](#vertical-bar)
7. [Chart Type: Stacked Bar](#stacked-bar)
8. [Chart Type: Merged Bar Chart](#merged-bar)
9. [Chart Type: Merged Line Chart](#merged-line)
10. [Chart Type: Waterfall](#waterfall)
11. [Chart Type: Line Chart](#line-chart)
12. [Two-Period Comparison (replaces Slopegraph)](#two-period-comparison)
13. [Chart Type: Scatterplot](#scatterplot)
14. [Chart Type: Heatmap](#heatmap)
15. [Chart Type: Bullet Chart](#bullet-chart)
16. [Annotation Patterns](#annotations)
17. [Anti-Patterns to Avoid](#anti-patterns)

---

## 1. Color Palette & Tokens <a name="color-palette"></a>

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

## 2. Typography <a name="typography"></a>

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

## 3. Chart.js Global Configuration <a name="chartjs-global-config"></a>

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

## 4. Chart Selection Decision Tree <a name="chart-selection"></a>

Ask yourself these questions about the data:

```
Q: How many numbers are we showing?
├─ Just 1-2 numbers → Use SIMPLE TEXT (big number + context sentence), not a chart
│
Q: Are there multiple measures with DIFFERENT UNITS (dollars, ratings, counts)?
├─ Yes, categorical data → MERGED BAR CHART (each measure on its own scale)
├─ Yes, over time → MERGED LINE CHART (stacked vertically, shared x-axis)
│  └─ If comparing relative patterns of change is the main point → consider INDEX CHART
│
Q: Showing breakdown of totals but NOT ALL PARTS are included?
├─ Yes → MERGED BAR CHART (avoids implying segments sum to the total)
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
│  ├─ Building to a total (start + adds - subtracts = end) → WATERFALL
│  │
│  └─ Actual vs target → BULLET CHART
│
├─ Two variables, looking for relationship → SCATTERPLOT
│
└─ Table of values where magnitude matters → HEATMAP
```

---

## 5. Horizontal Bar Chart <a name="horizontal-bar"></a>

The go-to chart for categorical data. Category names read naturally left-to-right.

```javascript
// Horizontal bar — SWD style
type: 'bar',
indexAxis: 'y',   // Makes it horizontal
options: {
  ...SWD.baseConfig,
  indexAxis: 'y',
  scales: {
    x: {
      ...SWD.baseConfig.scales.x,
      grid: { display: true, color: '#E5E7EB', lineWidth: 0.5 },
      ticks: { ...SWD.baseConfig.scales.x.ticks },
      beginAtZero: true,
    },
    y: {
      ...SWD.baseConfig.scales.y,
      grid: { display: false },
      ticks: {
        ...SWD.baseConfig.scales.y.ticks,
        font: { size: 12 },   // Slightly larger for category names
        crossAlign: 'far',     // Left-align category labels
      },
    },
  },
},
data: {
  labels: categories,
  datasets: [{
    data: values,
    backgroundColor: values.map(v =>
      highlightIndices.includes(values.indexOf(v)) ? SWD.colors.blue : SWD.colors.light
    ),
    barPercentage: 0.75,
    categoryPercentage: 0.85,
  }],
}
```

**SWD-specific rules for bars:**
- Order categories by value (largest at top) unless a natural order exists (e.g., age ranges)
- If highlighting one category, make it ACCENT_BLUE and everything else BASE_LIGHT
- Add data labels at the end of bars when specific values matter
- For value labels on bars: place inside the bar (right-aligned) if bar is long enough, otherwise outside

---

## 6. Vertical Bar Chart (Column) <a name="vertical-bar"></a>

Use when the x-axis represents time periods (months, quarters, years) or when horizontal
doesn't work for the layout.

```javascript
type: 'bar',
// Same base config as horizontal but with indexAxis defaulting to 'x'
// Key difference: x-axis gets no gridlines, y-axis gets faint gridlines
```

**Rules:**
- MUST have zero baseline (beginAtZero: true)
- Abbreviate month names (Jan, Feb…) to keep labels horizontal — never rotate labels diagonally
- If labels must be long, switch to horizontal bar instead
- Multi-series: limit to 2-3 series max; more than that gets hard to compare

---

## 7. Stacked Bar Chart <a name="stacked-bar"></a>

Use for part-to-whole comparisons or showing composition over time.

```javascript
// 100% stacked horizontal — SWD style (e.g., survey Likert data)
options: {
  ...SWD.baseConfig,
  indexAxis: 'y',
  scales: {
    x: {
      stacked: true,
      max: 100,
      ticks: { callback: v => v + '%' },
    },
    y: { stacked: true },
  },
},
datasets: [
  { label: 'Strongly Disagree', backgroundColor: '#D97706', data: [...] },
  { label: 'Disagree', backgroundColor: '#FCD34D', data: [...] },
  { label: 'Neutral', backgroundColor: '#D1D5DB', data: [...] },
  { label: 'Agree', backgroundColor: '#93C5FD', data: [...] },
  { label: 'Strongly Agree', backgroundColor: '#2563EB', data: [...] },
]
```

**Rules:**
- Only the bottom series (touching the baseline) is easy to compare across categories
- For 100% stacked, consider showing the most important segment touching a baseline
- Include "Total %" labels or absolute totals as a supplement when helpful
- Use varying saturation of ONE color for ordinal scales (e.g., low→high, disagree→agree)
- Add a small inline legend above the chart (styled minimally) since direct labeling inside thin segments is impractical

---

## 8. Merged Bar Chart <a name="merged-bar"></a>

Separate mini-bar-charts arranged in a grid, each with its own scale. From *Practical Charts*
(Desbarats) — use instead of stacked/clustered bars when not all parts are shown or when
measures have different units.

**When to use:**
- Showing a **subset** of parts (e.g., "top 3 departments" out of many) — stacked bars misleadingly imply segments sum to the total
- Showing **different units of measurement** for the same groups (e.g., revenue in $, satisfaction rating 1-10, headcount) — a shared scale creates nonsensical comparisons
- Comparing specific parts **across different totals** when precise comparison matters more than seeing the total

**Implementation — multiple Chart.js canvases in a CSS grid:**

```html
<div style="font-family: system-ui, -apple-system, sans-serif; padding: 16px; max-width: 700px;">

  <!-- Title -->
  <div style="font-size:14px; font-weight:700; color:#4B5563; margin-bottom:2px;">
    [Insight headline]
  </div>
  <div style="font-size:13px; color:#9CA3AF; margin-bottom:12px;">
    [Descriptive subtitle]
  </div>

  <!-- Grid of mini-charts: one column per measure -->
  <div style="display:grid; grid-template-columns:120px repeat(N, 1fr); gap:4px 12px; align-items:center;">

    <!-- Header row: empty cell for labels, then measure names -->
    <div></div>
    <div style="font-size:11px; font-weight:600; color:#9CA3AF; text-align:center;">Measure A ($)</div>
    <div style="font-size:11px; font-weight:600; color:#9CA3AF; text-align:center;">Measure B (rating)</div>
    <!-- ...one per measure -->

    <!-- Data rows: one per group/category -->
    <!-- Row 1 -->
    <div style="font-size:12px; color:#4B5563; text-align:right; padding-right:8px;">Category 1</div>
    <div style="position:relative; height:32px;"><canvas id="chart-0-0"></canvas></div>
    <div style="position:relative; height:32px;"><canvas id="chart-0-1"></canvas></div>

    <!-- Row 2, Row 3, etc. -->
  </div>

  <div style="font-size:10px; color:#9CA3AF; margin-top:8px;">Data source: [source]</div>
</div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.js"></script>
<script>
  // For each mini-chart cell: create a tiny horizontal bar chart
  // Each measure column shares a common max scale (computed across all groups for that measure)
  // This ensures bars within each column are comparable

  const SWD = { blue: '#2563EB', light: '#D1D5DB', dark: '#4B5563', mid: '#9CA3AF', faint: '#E5E7EB' };

  function createMiniBar(canvasId, value, maxVal, color) {
    new Chart(document.getElementById(canvasId), {
      type: 'bar',
      data: {
        labels: [''],
        datasets: [{ data: [value], backgroundColor: color, barPercentage: 0.7, categoryPercentage: 1.0 }]
      },
      options: {
        indexAxis: 'y',
        responsive: true,
        maintainAspectRatio: false,
        plugins: { legend: { display: false }, tooltip: { enabled: false } },
        scales: {
          x: { display: false, min: 0, max: maxVal },
          y: { display: false }
        },
        // Add value label via plugin
        animation: { duration: 0 }
      },
      plugins: [{
        afterDraw(chart) {
          const ctx = chart.ctx;
          const meta = chart.getDatasetMeta(0).data[0];
          ctx.fillStyle = SWD.dark;
          ctx.font = '600 11px system-ui';
          ctx.textAlign = 'left';
          ctx.textBaseline = 'middle';
          ctx.fillText(value.toLocaleString(), meta.x + 4, meta.y);
        }
      }]
    });
  }

  // Example usage:
  // const measures = [{name:'Revenue ($K)', values:[120, 95, 140], max:140},
  //                   {name:'Satisfaction', values:[8.2, 7.5, 9.1], max:10}];
  // const groups = ['East', 'Central', 'West'];
  // groups.forEach((g, gi) => {
  //   measures.forEach((m, mi) => {
  //     createMiniBar(`chart-${gi}-${mi}`, m.values[gi], m.max, SWD.blue);
  //   });
  // });
</script>
```

**Rules:**
- Each measure column must have its own scale (computed from the max value in that column)
- Category labels appear once on the left — never repeat them
- Add a measure title (with unit) above each column
- Zero baseline on every mini-bar (enforced by `min: 0`)
- Value labels appear at the end of each bar for precision
- Use ACCENT_BLUE for all bars, or highlight specific groups with ACCENT_BLUE and push others to BASE_LIGHT
- For 2 measures: two columns. For 3+: consider whether the grid becomes too wide; if so, stack measure groups vertically instead

---

## 9. Merged Line Chart (dual-axis alternative) <a name="merged-line"></a>

Instead of dual-axis charts, stack two or more mini-line-charts vertically sharing the same
x-axis. The audience can compare patterns of change (spikes, dips, trends) without being
misled by unrelated scales.

**When to use:**
- Comparing two variables with **different units** over time (e.g., revenue in $ and customer satisfaction 1-10)
- Any situation where a dual-axis chart is tempting — use this instead
- When the audience needs to see actual values (not just relative change — for relative change, consider index charts)

**Implementation — vertically stacked Chart.js canvases:**

```html
<div style="font-family: system-ui, -apple-system, sans-serif; padding: 16px; max-width: 700px;">

  <!-- Title -->
  <div style="font-size:14px; font-weight:700; color:#4B5563; margin-bottom:2px;">
    [Insight headline]
  </div>
  <div style="font-size:13px; color:#9CA3AF; margin-bottom:12px;">
    [Descriptive subtitle]
  </div>

  <!-- Stacked mini-line-charts -->
  <div style="display:flex; flex-direction:column; gap:4px;">

    <!-- Upper chart: hides x-axis labels -->
    <div>
      <div style="font-size:11px; font-weight:600; color:#9CA3AF; margin-bottom:2px;">Revenue ($K)</div>
      <div style="position:relative; height:160px;">
        <canvas id="line-top"></canvas>
      </div>
    </div>

    <!-- Bottom chart: shows x-axis labels -->
    <div>
      <div style="font-size:11px; font-weight:600; color:#9CA3AF; margin-bottom:2px;">Satisfaction (1-10)</div>
      <div style="position:relative; height:160px;">
        <canvas id="line-bottom"></canvas>
      </div>
    </div>

  </div>

  <div style="font-size:10px; color:#9CA3AF; margin-top:8px;">Data source: [source]</div>
</div>

<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.js"></script>
<script>
  const SWD = { blue: '#2563EB', orange: '#D97706', dark: '#4B5563', mid: '#9CA3AF', faint: '#E5E7EB' };
  const labels = ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun'];

  function createMergedLine(canvasId, data, color, yLabel, showXAxis) {
    new Chart(document.getElementById(canvasId), {
      type: 'line',
      data: {
        labels: labels,
        datasets: [{
          data: data,
          borderColor: color,
          borderWidth: 2.5,
          pointRadius: 0,
          pointHoverRadius: 0,
          pointHitRadius: 14,
          tension: 0,
          fill: false,
        }]
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
        plugins: { legend: { display: false }, tooltip: { enabled: true } },
        scales: {
          x: {
            border: { display: false },
            grid: { display: false },
            ticks: {
              display: showXAxis,   // Only bottom chart shows x-axis labels
              color: SWD.mid,
              font: { size: 11 },
            },
          },
          y: {
            border: { display: false },
            grid: { display: true, color: SWD.faint, lineWidth: 0.5 },
            ticks: { color: SWD.mid, font: { size: 11 }, padding: 8 },
          },
        },
        layout: { padding: { top: 4, right: 16, bottom: 0, left: 8 } },
        elements: { point: { radius: 0, hoverRadius: 0 } },
      },
      // Direct endpoint label plugin
      plugins: [{
        afterDraw(chart) {
          const ds = chart.getDatasetMeta(0).data;
          const last = ds[ds.length - 1];
          const ctx = chart.ctx;
          ctx.fillStyle = color;
          ctx.font = '600 11px system-ui';
          ctx.textAlign = 'left';
          ctx.textBaseline = 'middle';
          ctx.fillText(chart.data.datasets[0].data[ds.length - 1].toLocaleString(), last.x + 6, last.y);
        }
      }]
    });
  }

  // Top chart (upper variable) — hide x-axis
  // createMergedLine('line-top', [120, 135, 110, 150, 142, 165], SWD.blue, 'Revenue ($K)', false);
  // Bottom chart (lower variable) — show x-axis
  // createMergedLine('line-bottom', [7.2, 7.5, 6.8, 8.0, 7.9, 8.3], SWD.orange, 'Satisfaction', true);
</script>
```

**Rules:**
- Each mini-chart has its own y-axis with its own unit — never share a y-axis between different units
- The shared x-axis (time) displays labels only on the **bottom** chart; upper charts hide them
- Charts must share consistent width so x-axis points align vertically — this is what allows pattern comparison
- Use 150–180px height per mini-chart so 2–3 fit comfortably
- Apply all standard SWD line rules: no data markers, direct endpoint labels, accent color for emphasis
- Use different accent colors (blue for one variable, orange for the other) to distinguish the two series
- Add a unit label above each mini-chart (e.g., "Revenue ($K)", "Satisfaction (1-10)")
- For 3+ variables: stack three mini-charts; beyond that, consider whether the audience really needs all variables together

---

## 10. Waterfall Chart <a name="waterfall"></a>

Shows a starting value, increases, decreases, and resulting end value. Build with D3 or
custom Chart.js (floating bars).

**Implementation approach with Chart.js floating bars:**
```javascript
// Each bar is a [start, end] tuple
// Increases: blue fill, positioned above the running total
// Decreases: orange fill, positioned below
// Totals (first and last): darker blue, start from zero
datasets: [{
  data: bars.map(b => [b.start, b.end]),
  backgroundColor: bars.map(b =>
    b.isTotal ? '#1D4ED8' : b.isIncrease ? '#2563EB' : '#D97706'
  ),
}]
```

**Rules:**
- Always show the starting total and ending total as full bars from zero
- Increases go up (blue), decreases hang down (orange)
- Connect bars with thin grey connector lines to show the running total
- Label each bar with its delta value (e.g., "+30", "−12")

---

## 11. Line Chart <a name="line-chart"></a>

The default for time-series data.

```javascript
type: 'line',
options: {
  ...SWD.baseConfig,
  // For line charts, x-axis can omit gridlines, y-axis keeps faint ones
},
datasets: [{
  label: 'Primary series',
  data: values,
  borderColor: SWD.colors.blue,
  borderWidth: 2.5,
  pointRadius: 0,           // No markers by default
  pointHoverRadius: 5,
}]
```

**Multi-series rules (the spaghetti graph problem):**

For 3+ lines, ALWAYS use the interactive highlight pattern. The default state shows all lines
in BASE_LIGHT grey. Two interaction modes work together:

1. **Hover on chart lines** — nearest dataset highlights (bold + accent color), others fade to grey.
   The headline above the chart updates with that series' story. When the cursor leaves the
   chart area, everything resets to the neutral grey state.
2. **Click on buttons** — locks a series highlight. While locked, chart hover is disabled.
   Click the same button again to unlock and return to hover mode.

There are NO data point circles/markers — just clean lines that change weight and color.

**Implementation pattern:**
```javascript
// Chart config for interactive multi-line
interaction: { mode: 'dataset', intersect: false },  // Hover snaps to nearest line
animation: { duration: 150 },                         // Fast transitions feel responsive

// All lines start grey
datasets: series.map(s => ({
  label: s.name,
  data: s.data,
  borderColor: '#D1D5DB',       // BASE_LIGHT
  borderWidth: 1.5,
  pointRadius: 0,               // Never show circles
  pointHoverRadius: 0,          // Never show circles on hover either
  pointHitRadius: 14,           // But keep a generous hit area for hover detection
  tension: 0,
  fill: false,
}));

// onHover callback — skip if a button is locked
onHover: function(evt, els) {
  if (locked !== -1) return;
  if (els.length > 0) applyHighlight(els[0].datasetIndex);
  else applyHighlight(-1);
}

// Mouse leave on canvas resets (only if not locked)
canvas.addEventListener('mouseleave', function() {
  if (locked === -1) applyHighlight(-1);
});

// Button click toggles lock
button.addEventListener('click', function() {
  if (locked === i) { locked = -1; applyHighlight(-1); }
  else { locked = i; applyHighlight(i); }
});

// Highlight function — accent color depends on trend direction
function applyHighlight(idx) {
  var accent = (data[idx].endVal < data[idx].startVal) ? ORANGE : BLUE;
  datasets.forEach(function(d, i) {
    if (i === idx) { d.borderColor = accent; d.borderWidth = 3; }
    else           { d.borderColor = '#D1D5DB'; d.borderWidth = 1; }
  });
  // Update headline text and color
  // Update button active state styling
  chart.update();
}
```

**Direct endpoint labels** — always label each line at its last data point using the
afterDatasetsDraw plugin. The focused line label should be font-weight 600, others 400.
Both use the line's current borderColor so they fade/highlight together.

**Headline** — an action title (14px, bold) above the chart that updates dynamically.
Default state: neutral instruction text in BASE_DARK. Highlighted state: the specific
series' story in the accent color.

**Buttons** — render as small pill buttons above the chart. Default: grey border, grey text.
Active/locked: accent-tinted background, accent text, accent border, font-weight 600.

For **2 lines only**, skip the interactive pattern — just show one in ACCENT_BLUE and one in
BASE_MID with direct endpoint labels. The comparison is simple enough without interaction.

For **>6 lines**, consider small multiples instead (one mini-chart per series sharing the
same x-axis and y-scale) rather than the interactive approach.

**Showing a range:**
- Use `fill: 'origin'` or a custom band between min/max lines
- Fill with a very light blue (`rgba(37, 99, 235, 0.08)`)
- Show the average/primary line in solid ACCENT_BLUE on top

**Data markers:**
- Default: NO markers anywhere — no circles on points, not on hover, not on endpoints
- The only exception: when annotating a specific single data point with a callout (e.g.,
  a spike), draw a small marker ONLY on that one point via a custom plugin, not via
  Chart.js pointRadius

**Important: consistent time intervals**
- The x-axis MUST use consistent intervals. Never mix decades and years, or months and quarters.

---

## 12. Slopegraph — DEPRECATED <a name="slopegraph"></a>

**Do not use slopegraphs.** For comparing two time points or two conditions across multiple categories, use one of these instead:

1. **Grouped horizontal bar chart** (preferred) — show before/after as paired bars per category, using ACCENT_BLUE for the later period and BASE_LIGHT for the earlier. Order categories by the magnitude of change.
2. **Horizontal bar chart showing change** — compute the difference and show a single bar per category representing the change amount. Use ACCENT_BLUE for increases and ACCENT_ORANGE for decreases.
3. **Simple 2-point line chart** — if the time dimension matters more than the category comparison, a line chart with just 2 x-axis points works, but horizontal bars are usually clearer.

---

## 13. Scatterplot <a name="scatterplot"></a>

Shows relationship between two continuous variables.

```javascript
type: 'scatter',
datasets: [{
  data: points.map(p => ({ x: p.x, y: p.y })),
  backgroundColor: SWD.colors.blue + '99',  // Semi-transparent
  borderColor: SWD.colors.blue,
  borderWidth: 1,
  pointRadius: 5,
}]
```

**Rules:**
- Both axes need clear titles (what each dimension represents)
- Add reference lines (averages, thresholds) as thin dashed grey lines
- Highlight outliers or key points with ACCENT_BLUE/ORANGE and annotations
- Consider adding quadrant labels when the axes create meaningful regions

---

## 14. Heatmap <a name="heatmap"></a>

A table with color saturation encoding relative magnitude. Build with HTML/CSS in the Visualizer.

**Implementation:**
- Render as an HTML table
- Use varying saturation of ACCENT_BLUE: higher value = more saturated
- Always include the numeric values inside each cell
- Always include a legend showing the LOW–HIGH color scale
- Light borders or white space between cells (never heavy borders)

**Color scale:**
```javascript
// Map value to blue saturation
const minVal = Math.min(...allValues);
const maxVal = Math.max(...allValues);
const ratio = (val - minVal) / (maxVal - minVal);
// Background: interpolate from '#F3F4F6' (near white) to '#2563EB' (full blue)
// Text: use white text when ratio > 0.5, dark text otherwise
```

---

## 15. Bullet Chart <a name="bullet-chart"></a>

Shows actual performance against a target. Build with D3/SVG.

**Structure:**
- Grey background bar shows the range/scale
- Thin line marker shows the target
- Colored bar (narrower) shows the actual value
- ACCENT_BLUE if actual meets/exceeds target, ACCENT_ORANGE if below

---

## 16. Annotation Patterns <a name="annotations"></a>

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

## 17. Anti-Patterns to Avoid <a name="anti-patterns"></a>

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

## HTML Wrapper Template

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

<script src="https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.1/chart.umd.js"></script>
<script>
  // Chart.js code here using SWD config patterns
</script>
```

For D3/SVG charts (waterfall, heatmaps), replace the canvas with an inline `<svg>` element
and use the same HTML wrapper structure for titles and footnotes.