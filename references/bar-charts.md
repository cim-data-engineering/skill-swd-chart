# SWD Charts — Bar Chart Reference

Implementation patterns for horizontal, vertical, and stacked bar charts,
plus the two-period comparison pattern that replaces slopegraphs.

## Table of Contents
1. [Horizontal Bar Chart](#1-horizontal-bar-chart)
2. [Vertical Bar Chart (Column)](#2-vertical-bar-chart-column)
3. [Stacked Bar Chart](#3-stacked-bar-chart)
4. [Two-Period Comparison (replaces Slopegraph)](#4-two-period-comparison)

---

## 1. Horizontal Bar Chart

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

## 2. Vertical Bar Chart (Column)

Use when the x-axis represents time periods (months, quarters, years) or when horizontal
doesn't work for the layout.

```javascript
type: 'bar',
// Same base config as horizontal but with indexAxis defaulting to 'x'
// Key difference: x-axis gets no gridlines, y-axis gets faint gridlines
```

**Rules:**
- MUST have zero baseline (beginAtZero: true)
- Abbreviate month names (Jan, Feb...) to keep labels horizontal — never rotate labels diagonally
- If labels must be long, switch to horizontal bar instead
- Multi-series: limit to 2-3 series max; more than that gets hard to compare

---

## 3. Stacked Bar Chart

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
- Use varying saturation of ONE color for ordinal scales (e.g., low->high, disagree->agree)
- Add a small inline legend above the chart (styled minimally) since direct labeling inside thin segments is impractical

---

## 4. Two-Period Comparison

**Do not use slopegraphs.** For comparing two time points or two conditions across multiple categories, use one of these instead:

1. **Grouped horizontal bar chart** (preferred) — show before/after as paired bars per category, using ACCENT_BLUE for the later period and BASE_LIGHT for the earlier. Order categories by the magnitude of change.
2. **Horizontal bar chart showing change** — compute the difference and show a single bar per category representing the change amount. Use ACCENT_BLUE for increases and ACCENT_ORANGE for decreases.
3. **Simple 2-point line chart** — if the time dimension matters more than the category comparison, a line chart with just 2 x-axis points works, but horizontal bars are usually clearer.
