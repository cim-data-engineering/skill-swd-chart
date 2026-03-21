---
name: swd-chart
description: >
  Creates interactive data visualizations following Storytelling with Data (Cole Nussbaumer Knaflic) best practices.
  Produces Chart.js and D3 charts via the Visualizer tool with rigorous decluttering, strategic color,
  direct labeling, and clear visual hierarchy. Strongly favors horizontal bar charts as the default
  chart type — use them whenever possible. Use this skill whenever the user asks to chart, graph,
  plot, or visualize data — including requests like "show me a chart of…", "plot this data",
  "visualize these numbers", "make a bar/line/chart", "graph this", or any mention of
  bar charts, line charts, scatterplots, heatmaps, waterfall charts, stacked bars,
  merged charts, bullet charts, or dual-axis alternatives. Also triggers when the user provides data (CSV, table, numbers) and asks
  Claude to "show" or "display" it visually, or when they ask for a dashboard or data summary visual.
  Even casual requests like "can you chart this?" or "what does this data look like?" should trigger this skill.
---

# SWD Charts — Storytelling with Data Visualization Skill

This skill produces interactive charts via the **Visualizer tool** (show_widget) that follow the
principles from *Storytelling with Data* by Cole Nussbaumer Knaflic. Every chart must feel
intentional, decluttered, and focused — as if a professional data storyteller designed it.

Before creating any chart, read `references/design-system.md` for the specific Chart.js/D3
code patterns, color tokens, and per-chart-type implementation guidance.

---

## Core Philosophy

Every pixel earns its place. If an element doesn't help the audience understand the data,
remove it. The goal is not decoration — it's comprehension.

---

## 1. Choose the Right Chart

Auto-select the chart type based on what the data is and what story it tells:

| Data situation | Best chart type |
|---|---|
| One or two numbers | **Simple text** — skip the chart entirely, just display the number prominently |
| Comparing categories | **Horizontal bar chart** (preferred) or vertical bar chart |
| Trend over time (continuous) | **Line chart** — never use bars for time series |
| Two time periods, multiple categories | **Horizontal bar chart** (grouped, with before/after pairs) or **line chart** with 2 points |
| Parts of a whole | **100% stacked horizontal bar** or simple bar with total shown (never pie/donut) |
| Relationship between two variables | **Scatterplot** |
| Starting value → additions → deductions → end | **Waterfall chart** |
| Tabular data with magnitude patterns | **Heatmap** (table + color saturation) |
| Actual vs. target with qualitative ranges | **Bullet chart** |
| Vastly different magnitudes | **Square area chart** |
| Subset of categories (not all parts shown) | **Merged bar chart** — separate mini-charts avoid implying segments sum to the total |
| Multiple measures with different units (dollars, ratings, counts) | **Merged bar chart** — each measure gets its own scale, preventing nonsensical comparisons |
| Two different variables over time | **Merged line chart** — vertically stacked mini-charts sharing an x-axis (never dual-axis) |

**Hard rules on chart selection:**
- **Horizontal bar charts are the strong default.** When in doubt about chart type, always reach for a horizontal bar chart first. It is the most versatile, readable, and accessible chart for the vast majority of data. Only use a different chart type when horizontal bars genuinely cannot tell the story (e.g., continuous time series → line chart, two-variable relationship → scatterplot).
- **Never use pie charts or donut charts.** Replace with horizontal bar or 100% stacked bar.
- **Never use slopegraphs.** For two-time-period comparisons, use grouped horizontal bars (before/after pairs) or a horizontal bar showing the change/difference. A simple line chart with 2 points is acceptable but horizontal bars are preferred.
- **Never use 3D.** It distorts perception and adds zero information.
- **Never use dual-axis charts.** They encourage misleading comparisons between unrelated scales. Instead, use a **merged line chart** (two or more vertically stacked mini-charts sharing an x-axis) to compare patterns of change, or an **index chart** (rebase both to % change from a common starting point) if direct pattern comparison is the main goal.
- **Prefer merged bar charts over stacked/clustered bars when not all parts are shown.** Stacked bars and pie charts imply the segments add up to the total — if you're only showing a subset of categories (e.g., "top 3 departments" out of many), use merged bars instead, which don't carry that false implication.
- Line charts imply continuity — only use them for continuous data (usually time). For categorical comparisons, always use horizontal bars.
- When categories have long names, horizontal bars are the only good option — labels read naturally left-to-right.
- Even for short category names, prefer horizontal bars over vertical bars unless the data is time-ordered (months, quarters, years).

---

## 2. Declutter Ruthlessly

Apply these six steps to every chart (from Chapter 3):

1. **Remove chart borders and backgrounds** — rely on white space and closure
2. **Remove or heavily mute gridlines** — if kept, make them very light grey and thin; prefer removing entirely
3. **Remove all data markers** on lines — no circles on points, ever. Use line weight and color to draw attention instead.
4. **Clean up axis labels** — no trailing decimals (use `0`, `50`, `100` not `0.00`, `50.00`), abbreviate months (`Jan`, `Feb`), keep text horizontal (never diagonal)
5. **Label data directly** — place labels near the data they describe instead of using a legend. Eliminate the legend whenever possible.
6. **Use consistent color** — label text should match the color of the data series it describes (Gestalt principle of similarity)

**Additional decluttering rules:**
- Remove axis lines where the data alignment makes them unnecessary (Gestalt continuity)
- Dollar signs, percent signs, and commas in large numbers are NOT clutter — always include them
- Round numbers to appropriate precision; don't show false precision
- Bar width should be wider than the gap between bars, but not so wide the audience compares area instead of length

---

## 3. Color Strategy

The palette is **grey base + strategic accent color**. See `references/design-system.md` §1 for exact hex tokens and extended shades.

- **Base**: All non-emphasized elements in grey (muted text, de-emphasized data, faint gridlines)
- **Positive / Standout accent**: Blue — draws attention to the key data
- **Negative / Warning accent**: Orange/Amber — highlights problems, declines, or negative values
- **Titles**: Dark grey (not pure black — preserves black for extreme emphasis)

**Color rules from SWD:**
- Use color **sparingly** — design in grey first, then add a single accent color only where you want the audience to look
- Use color **consistently** — if blue means "our company" on one chart, it means that everywhere
- Never use color for decoration. Every color choice must be intentional.
- Avoid red-green combinations (colorblind accessibility). Use blue vs. orange instead.
- Varying **saturation** of one hue works well for ordinal data (e.g., priority levels)
- When multiple series exist but only one matters, make the important one the accent color and push everything else to grey

---

## 4. Focus Attention with Preattentive Attributes

Use these sparingly to direct the eye:

- **Color intensity**: The accent color on a grey field instantly draws the eye — this is the primary tool
- **Size**: Make the most important text/number larger. The chart title should be the largest text element.
- **Bold weight**: Use for titles, key labels, and the single most important annotation
- **No data point markers** — lines should be clean without circles. Use line thickness and accent color to signal importance.
- **Numeric labels**: Add ONLY on the specific data points you want the audience to notice (never on every point — that creates clutter)
- **Position**: Place the most important information at the top-left. Titles and axis labels should be upper-left-most justified.

**The "push to background first" technique**: Start by making everything grey and small. Then intentionally bring forward only what matters. This ensures every emphasis is deliberate.

---

## 5. Annotations and Storytelling Text

Every chart MUST have:
- A **descriptive title** (top-left, dark grey, largest text on the chart)
- **Axis titles** (unless truly obvious from context)

Add these WHEN there is a clear insight:
- A **key insight callout** — 1-2 sentences near the relevant data, using the accent color for key words
- **Annotations** on specific data points explaining what happened or why it matters
- A **footnote** for data source or methodology (small, grey, bottom of chart — present but de-emphasized)

**Do not** add annotations just to fill space. If the chart speaks clearly on its own, a title and clean labels are enough.

---

## 6. Visual Hierarchy

Establish a clear reading order through size, color, and position:

1. **Title** — largest, dark grey, top-left (the audience reads this first)
2. **Key insight / action text** — accent colored, bold, near the data it describes
3. **Primary data** — accent colored line/bars, thicker/bolder than context data
4. **Context data** — grey, thinner, lighter — visible but doesn't compete
5. **Axis labels** — grey, standard size, readable but receding
6. **Footnotes / source** — smallest, lightest grey, bottom of chart

---

## 7. Chart-Type Quick Reference

For full implementation details and code patterns for each chart type, read `references/design-system.md`.

| Chart type | Key rules |
|---|---|
| **Bar (horizontal/vertical)** | Zero baseline always; horizontal default; order by value unless natural order exists |
| **Merged bar** | Separate mini-charts with own scales; use when not all parts shown or units differ |
| **Merged line** | Vertically stacked mini-lines sharing x-axis; replaces dual-axis charts |
| **Line** | Continuous data only; no data point circles ever; direct endpoint labels |
| **Multi-line (3+)** | Interactive highlight pattern: all grey default, hover/click to accent one series |
| **Stacked bar** | Most important series on baseline; use 100% stacked for part-to-whole |
| **Scatterplot** | Both axes titled; add reference lines for quadrants; highlight outliers |
| **Heatmap** | Single hue varying saturation; numeric values in cells; include color legend |
| **Waterfall** | Grey totals, blue increases, orange decreases; connector lines between segments |
| **Bullet** | Grey background range, thin target marker, colored actual bar |

---

## 8. Implementation

All charts are rendered via the **Visualizer tool** (`show_widget`). Before your first chart,
call `read_me` with the appropriate module (`chart`, `interactive`, or `diagram`).

For detailed code patterns, Chart.js configuration, and D3 templates for each chart type,
read `references/design-system.md`.

**Key technical rules:**
- Use Chart.js for standard charts (bar, line, scatter, stacked bar, 100% stacked)
- Use multiple Chart.js instances in a CSS grid/flex layout for merged charts (each mini-chart gets its own canvas and scale)
- Use D3 or custom SVG for waterfall charts, bullet charts, and heatmaps
- Always use CSS variables for theming so charts adapt to light/dark mode
- Set `responsive: true` and use a reasonable aspect ratio
- Disable Chart.js default legend — implement direct labeling instead
- Disable Chart.js default tooltip styling — use a clean, minimal custom tooltip if interactivity adds value

---

## 9. Post-Render Checklist

After rendering every chart, verify these SWD essentials before presenting:

- [ ] Zero baseline on all bar charts
- [ ] Legend removed — data labeled directly
- [ ] Accent color used on 1-2 elements max; everything else is grey
- [ ] All text is horizontal (no diagonal/rotated labels)
- [ ] Title present, top-left, largest text on the chart
- [ ] No data point circles on line charts

---

## 10. Edge Cases

- **Insufficient or ambiguous data**: Ask the user to clarify before charting. Do not guess or fabricate data points.
- **Too many categories (>15)**: Show the top N with a note about omitted categories, or use small multiples.
- **Too many time periods for labels to fit**: Abbreviate labels (Q1, Q2…) or show every Nth label. Never rotate text diagonally.
- **Mixed units in a single request**: Use a merged bar or merged line chart — never combine different units on a shared axis.
- **Visualizer tool unavailable**: Describe the chart you would build (type, data mapping, color strategy) so the user can recreate it.