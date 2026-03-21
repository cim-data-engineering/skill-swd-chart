# SWD Charts — Specialty Chart Reference

Implementation patterns for waterfall charts and heatmaps.

---

## 1. Waterfall Chart

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
- Label each bar with its delta value (e.g., "+30", "-12")

---

## 2. Heatmap

A table with color saturation encoding relative magnitude. Build with HTML/CSS in the Visualizer.

**Implementation:**
- Render as an HTML table
- Use varying saturation of ACCENT_BLUE: higher value = more saturated
- Always include the numeric values inside each cell
- Always include a legend showing the LOW-HIGH color scale
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
