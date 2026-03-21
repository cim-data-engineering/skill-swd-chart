---
name: swd-charts
description: >
  Creates interactive data visualizations following Storytelling with Data (Cole Nussbaumer Knaflic) best practices.
  Produces Chart.js and D3 charts via the Visualizer tool with rigorous decluttering, strategic color,
  direct labeling, and clear visual hierarchy. Strongly favors horizontal bar charts as the default
  chart type — use them whenever possible. Use this skill whenever the user asks to chart, graph,
  plot, or visualize data — including requests like "show me a chart of…", "plot this data",
  "visualize these numbers", "make a bar/line/chart", "graph this", or any mention of
  bar charts, line charts, scatterplots, heatmaps, waterfall charts, stacked bars,
  or bullet charts. Also triggers when the user provides data (CSV, table, numbers) and asks
  Claude to "show" or "display" it visually, or when they ask for a dashboard or data summary visual.
  Even casual requests like "can you chart this?" or "what does this data look like?" should trigger this skill.
---

# SWD Charts — Storytelling with Data Visualization Skill

This skill produces interactive charts via the **Visualizer tool** (show_widget) that follow the
principles from *Storytelling with Data* by Cole Nussbaumer Knaflic. Every chart must feel
intentional, decluttered, and focused — as if a professional data storyteller designed it.

Before creating any chart, read `references/chart-patterns.md` for the specific Chart.js/D3
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

**Hard rules on chart selection:**
- **Horizontal bar charts are the strong default.** When in doubt about chart type, always reach for a horizontal bar chart first. It is the most versatile, readable, and accessible chart for the vast majority of data. Only use a different chart type when horizontal bars genuinely cannot tell the story (e.g., continuous time series → line chart, two-variable relationship → scatterplot).
- **Never use pie charts or donut charts.** Replace with horizontal bar or 100% stacked bar.
- **Never use slopegraphs.** For two-time-period comparisons, use grouped horizontal bars (before/after pairs) or a horizontal bar showing the change/difference. A simple line chart with 2 points is acceptable but horizontal bars are preferred.
- **Never use 3D.** It distorts perception and adds zero information.
- **Avoid secondary y-axes.** Instead, either label data directly or split into two vertically stacked charts sharing an x-axis.
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

The palette is **grey base + strategic accent color**:

- **Base**: All non-emphasized elements in grey (`#9B9B9B` for text/labels, `#D4D4D4` for muted data, `#E8E8E8` for very light backgrounds)
- **Positive / Standout accent**: Blue `#2563EB` — used to draw attention to the key data
- **Negative / Warning accent**: Orange/Amber `#D97706` — used when highlighting problems, declines, or negative values
- **Graph title**: Dark grey `#4A4A4A` (not pure black — preserves black for extreme emphasis)
- **Axis labels and ticks**: Medium grey `#9B9B9B`
- **Gridlines** (if kept): Very light `#ECECEC`, thin (1px)

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

## 7. Specific Chart-Type Guidance

### Bar Charts (the default chart type)
- **Always use a zero baseline.** Non-zero baselines on bar charts are misleading and destroy credibility.
- **Default to horizontal bars.** Horizontal bars are the best chart for most data — labels read naturally, the audience encounters category names before the data (left-to-right reading), and they handle long labels gracefully. Only use vertical bars when the x-axis represents time (months, quarters, years) and there are many time periods.
- Order bars intentionally: by value (largest to smallest) unless there's a natural category order (e.g., age groups).
- For stacked bars, only the bottom series (touching the baseline) allows easy comparison. Keep the most important series there.
- For two-time-period comparisons (where someone might think of a slopegraph), use a **grouped horizontal bar** with before/after pairs, or a **horizontal bar showing the change amount** (blue for increase, orange for decrease).

### Line Charts
- Use for continuous data only (usually time series).
- Consistent time intervals on the x-axis — never mix decades with years.
- Directly label each line at its endpoint (right side) instead of using a legend.
- Use a thicker line for the primary series and thinner grey lines for context.
- **No data point circles** — ever. Not on hover, not on endpoints, nowhere. Lines only.
- Show range/confidence with a light shaded area behind the line if relevant.
- Distinguish forecast from actual: solid line for actual, dashed/thinner for forecast, with light background shading on the forecast region.

### Multi-Line Charts (3+ series) — the Spaghetti Graph Solution
The interactive highlight pattern is the default for 3+ series:
- **Default state**: all lines in light grey, direct endpoint labels in matching grey
- **Hover on chart**: nearest line dataset highlights (bold accent color + thicker), others fade. Headline updates with that series' story.
- **Click button**: locks a series. Hover is ignored while locked. Click again to unlock.
- Buttons render as small pills above the chart. Active button gets accent-tinted background.
- Accent color: blue for positive/growing trends, orange for declining trends.
- When cursor leaves chart area and nothing is locked, reset to neutral grey state.
- For 2 lines only, skip interactivity — just color one blue and one grey.
- For >6 lines, use small multiples (one mini-chart per series) instead.

### Scatterplots
- Label axes clearly — both dimensions need titles.
- Use reference lines (like averages) to create quadrants that aid interpretation.
- Highlight above/below-average points with accent color.

### Heatmaps
- Always include a color legend showing the low-to-high scale.
- Use a single hue with varying saturation (not a rainbow).
- Keep the numeric values visible inside cells.

### Waterfall Charts
- Color-code: grey for start/end totals, blue for increases, orange for decreases.
- Show connector lines between segments.
- Label each segment with its value.

---

## 8. Implementation

All charts are rendered via the **Visualizer tool** (`show_widget`). Before your first chart,
call `read_me` with the appropriate module (`chart`, `interactive`, or `diagram`).

For detailed code patterns, Chart.js configuration, and D3 templates for each chart type,
read `references/chart-patterns.md`.

**Key technical rules:**
- Use Chart.js for standard charts (bar, line, scatter, stacked bar, 100% stacked)
- Use D3 or custom SVG for waterfall charts, bullet charts, and heatmaps
- Always use CSS variables for theming so charts adapt to light/dark mode
- Set `responsive: true` and use a reasonable aspect ratio
- Disable Chart.js default legend — implement direct labeling instead
- Disable Chart.js default tooltip styling — use a clean, minimal custom tooltip if interactivity adds value