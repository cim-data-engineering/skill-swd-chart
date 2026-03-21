# SWD Charts — Line Chart Reference

Implementation patterns for standard line charts (single, multi-series, interactive)
and merged line charts (the dual-axis alternative).

## Table of Contents
1. [Line Chart](#1-line-chart)
2. [Multi-Series Interactive Pattern](#2-multi-series-interactive-pattern)
3. [Showing a Range](#3-showing-a-range)
4. [Data Markers](#4-data-markers)
5. [Merged Line Chart (dual-axis alternative)](#5-merged-line-chart)

---

## 1. Line Chart

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

**Direct endpoint labels** — always label each line at its last data point using the
afterDatasetsDraw plugin. The focused line label should be font-weight 600, others 400.
Both use the line's current borderColor so they fade/highlight together.

**Important: consistent time intervals**
- The x-axis MUST use consistent intervals. Never mix decades and years, or months and quarters.

For **2 lines only**, skip the interactive pattern — just show one in ACCENT_BLUE and one in
BASE_MID with direct endpoint labels. The comparison is simple enough without interaction.

For **>6 lines**, consider small multiples instead (one mini-chart per series sharing the
same x-axis and y-scale) rather than the interactive approach.

---

## 2. Multi-Series Interactive Pattern

For 3+ lines, ALWAYS use the interactive highlight pattern. The default state shows all lines
in BASE_LIGHT grey. Two interaction modes work together:

1. **Hover on chart lines** — nearest dataset highlights (bold + accent color), others fade to grey.
   The headline above the chart updates with that series' story. When the cursor leaves the
   chart area, everything resets to the neutral grey state.
2. **Click on buttons** — locks a series highlight. While locked, chart hover is disabled.
   Click the same button again to unlock and return to hover mode.

There are NO data point circles/markers — just clean lines that change weight and color.

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

**Headline** — an action title (14px, bold) above the chart that updates dynamically.
Default state: neutral instruction text in BASE_DARK. Highlighted state: the specific
series' story in the accent color.

**Buttons** — render as small pill buttons above the chart. Default: grey border, grey text.
Active/locked: accent-tinted background, accent text, accent border, font-weight 600.

---

## 3. Showing a Range

- Use `fill: 'origin'` or a custom band between min/max lines
- Fill with a very light blue (`rgba(37, 99, 235, 0.08)`)
- Show the average/primary line in solid ACCENT_BLUE on top

---

## 4. Data Markers

- Default: NO markers anywhere — no circles on points, not on hover, not on endpoints
- The only exception: when annotating a specific single data point with a callout (e.g.,
  a spike), draw a small marker ONLY on that one point via a custom plugin, not via
  Chart.js pointRadius

---

## 5. Merged Line Chart

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

<script src="https://cdn.jsdelivr.net/npm/chart.js@4/dist/chart.umd.min.js"></script>
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
- Use 150-180px height per mini-chart so 2-3 fit comfortably
- Apply all standard SWD line rules: no data markers, direct endpoint labels, accent color for emphasis
- Use different accent colors (blue for one variable, orange for the other) to distinguish the two series
- Add a unit label above each mini-chart (e.g., "Revenue ($K)", "Satisfaction (1-10)")
- For 3+ variables: stack three mini-charts; beyond that, consider whether the audience really needs all variables together
