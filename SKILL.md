---
name: swd-chart
description: >
  Creates interactive data visualizations following Storytelling with Data (Cole Nussbaumer Knaflic) best practices.
  Produces Chart.js and D3 charts via the Visualizer tool with rigorous decluttering, strategic color,
  direct labeling, and clear visual hierarchy. Strongly favors horizontal bar charts as the default
  chart type — use them whenever possible. This skill should ONLY be invoked when the user explicitly
  calls it via the /swd-chart command. Do NOT auto-trigger on general requests like "show me",
  "display this data", or when the user simply provides data.
---

# SWD Charts — Storytelling with Data Visualization Skill

This skill produces interactive charts via the **Visualizer tool** (show_widget) that follow the
principles from *Storytelling with Data* by Cole Nussbaumer Knaflic. Every chart must feel
intentional, decluttered, and focused — as if a professional data storyteller designed it.

Invoked via `/swd-chart`. Provide your data inline or describe the dataset.

Before creating any chart, read `references/core-config.md` for color tokens, Chart.js base
config, and the chart selection decision tree. Then read the chart-type-specific reference:
- `references/bar-charts.md` — when building horizontal, vertical, or stacked bar charts
- `references/line-charts.md` — when building line charts or merged line charts
- `references/specialty-charts.md` — when building waterfall charts or heatmaps

---

## Core Philosophy

Every pixel earns its place. If an element doesn't help the audience understand the data,
remove it. The goal is not decoration — it's comprehension.

---

## 1. Story First — Identify the Message Before Charting

**Do not jump straight to rendering.** Before choosing a chart type, analyse the data and
infer the possible stories it could tell. Then present options to the user for confirmation.

**Workflow:**

1. **Analyse the data** — look for rankings, trends, outliers, gaps, concentrations, changes over time, comparisons to targets, or composition patterns.
2. **Infer 2-3 candidate stories** — frame each as a clear, one-sentence message the chart would deliver. For example:
   - "East region leads all others in revenue by a wide margin"
   - "Customer satisfaction has declined steadily over the last 4 quarters"
   - "The top 3 departments account for 72% of total spend"
   - "Revenue and headcount are growing but satisfaction is flat"
3. **Recommend a chart type for each story** — pair each candidate message with the best visualization and briefly explain why it fits.
4. **Ask the user to pick or refine** — present the options and wait for confirmation before rendering. If the data clearly tells only one story, present that single recommendation and ask the user to confirm.

**Example output to the user:**

> Based on your data, here are the stories I see:
>
> 1. **"East region dominates revenue"** — Horizontal bar chart, sorted by value, East highlighted in blue. Best if the point is the ranking.
> 2. **"Revenue is growing but satisfaction is flat"** — Two separate line charts (merged line), one for each metric. Best if the point is the diverging trends.
> 3. **"Top 3 regions account for 68% of total revenue"** — 100% stacked bar with top 3 highlighted. Best if the point is concentration.
>
> Which story do you want to tell, or would you like a different angle?

---

## 2. Choose the Right Chart

Select the chart type based on the confirmed story and the shape of the data:

| Data situation | Best chart type |
|---|---|
| One or two numbers | **Simple text** — skip the chart entirely, just display the number prominently |
| Comparing categories | **Horizontal bar chart** (preferred) or vertical bar chart |
| Trend over time (continuous) | **Line chart** — never use bars for time series |
| Two time periods, multiple categories | **Horizontal bar chart** (grouped, with before/after pairs) or **line chart** with 2 points |
| Parts of a whole | **100% stacked horizontal bar** or simple bar with total shown (never pie/donut) |
| Starting value → additions → deductions → end | **Waterfall chart** |
| Tabular data with magnitude patterns | **Heatmap** (table + color saturation) |
| Vastly different magnitudes | **Square area chart** |
| Two different variables over time | **Merged line chart** — vertically stacked mini-charts sharing an x-axis (never dual-axis) |

**Hard rules on chart selection:**
- **Horizontal bar charts are the strong default.** When in doubt about chart type, always reach for a horizontal bar chart first. It is the most versatile, readable, and accessible chart for the vast majority of data. Only use a different chart type when horizontal bars genuinely cannot tell the story (e.g., continuous time series → line chart).
- **Never use pie charts or donut charts.** Replace with horizontal bar or 100% stacked bar.
- **Never use slopegraphs.** For two-time-period comparisons, use grouped horizontal bars (before/after pairs) or a horizontal bar showing the change/difference. A simple line chart with 2 points is acceptable but horizontal bars are preferred.
- **Never use 3D.** It distorts perception and adds zero information.
- **Never use dual-axis charts.** They encourage misleading comparisons between unrelated scales. Instead, use a **merged line chart** (two or more vertically stacked mini-charts sharing an x-axis) to compare patterns of change, or an **index chart** (rebase both to % change from a common starting point) if direct pattern comparison is the main goal.
- Line charts imply continuity — only use them for continuous data (usually time). For categorical comparisons, always use horizontal bars.
- When categories have long names, horizontal bars are the only good option — labels read naturally left-to-right.
- Even for short category names, prefer horizontal bars over vertical bars unless the data is time-ordered (months, quarters, years).

---

## 3. Managing Many Categories

**Never plot more than ~10-12 categories in a single chart.** More than that creates clutter and dilutes the message. When the data exceeds this threshold, recommend one of these tactics based on the confirmed story:

| Tactic | When to use |
|---|---|
| **Top N** | The story is about leaders or best performers. Show only the top N, add a footnote: "Showing top 10 of 45 regions" |
| **Bottom N** | The story is about underperformers or problem areas. Show only the bottom N. |
| **Top vs Bottom side-by-side** | The story is about the gap between best and worst. Render two charts next to each other: "Top 10 performers" and "Bottom 10 performers" |
| **Highlight + grey** | The story is about a few specific categories in context. Show all categories but accent only the key ones in blue, push the rest to grey |
| **"Other" bucket** | The story is about concentration (e.g., "top 3 account for 70%"). Aggregate the remaining categories into a single "Other" bar |

**Rules:**
- Always recommend the tactic that best supports the confirmed story — don't ask the user to pick from a menu of tactics unless the best choice is genuinely ambiguous
- When using Top/Bottom N, always note how many total categories exist so the audience has context
- When using side-by-side charts, keep the same scale on both so bars are directly comparable
- The "Other" bucket goes at the bottom of a sorted bar chart, never in the middle

---

## 4. Declutter Ruthlessly

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

## 5. Color Strategy

The palette is **grey base + strategic accent color**. See `references/core-config.md` §1 for exact hex tokens and extended shades.

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

## 6. Focus Attention with Preattentive Attributes

Use these sparingly to direct the eye:

- **Color intensity**: The accent color on a grey field instantly draws the eye — this is the primary tool
- **Size**: Make the most important text/number larger. The chart title should be the largest text element.
- **Bold weight**: Use for titles, key labels, and the single most important annotation
- **Line emphasis**: Use line thickness and accent color to signal importance (data markers are removed per §4)
- **Numeric labels**: Add ONLY on the specific data points you want the audience to notice (never on every point — that creates clutter)
- **Position**: Place the most important information at the top-left. Titles and axis labels should be upper-left-most justified.

**The "push to background first" technique**: Start by making everything grey and small. Then intentionally bring forward only what matters. This ensures every emphasis is deliberate.

---

## 7. Annotations and Storytelling Text

Every chart MUST have:
- A **descriptive title** (top-left, dark grey, largest text on the chart)
- **Axis titles** (unless truly obvious from context)

Add these WHEN there is a clear insight:
- A **key insight callout** — 1-2 sentences near the relevant data, using the accent color for key words
- **Annotations** on specific data points explaining what happened or why it matters
- A **footnote** for data source or methodology (small, grey, bottom of chart — present but de-emphasized)

**Do not** add annotations just to fill space. If the chart speaks clearly on its own, a title and clean labels are enough.

---

## 8. Visual Hierarchy

Establish a clear reading order through size, color, and position:

1. **Title** — largest, dark grey, top-left (the audience reads this first)
2. **Key insight / action text** — accent colored, bold, near the data it describes
3. **Primary data** — accent colored line/bars, thicker/bolder than context data
4. **Context data** — grey, thinner, lighter — visible but doesn't compete
5. **Axis labels** — grey, standard size, readable but receding
6. **Footnotes / source** — smallest, lightest grey, bottom of chart

---

## 9. Implementation

All charts are rendered via the **Visualizer tool** (`show_widget`). Before your first chart,
call `read_me` with the appropriate module (`chart`, `interactive`, or `diagram`).

For detailed code patterns and Chart.js configuration, read the relevant chart-type reference
file (`references/bar-charts.md`, `references/line-charts.md`, or `references/specialty-charts.md`).

**Key technical rules:**
- Use Chart.js for standard charts (bar, line, stacked bar, 100% stacked)
- Use multiple Chart.js instances in a CSS grid/flex layout for merged line charts (each mini-chart gets its own canvas and scale)
- Use D3 or custom SVG for waterfall charts and heatmaps
- Always use CSS variables for theming so charts adapt to light/dark mode
- Set `responsive: true` and use a reasonable aspect ratio
- Disable Chart.js default legend — implement direct labeling instead
- Disable Chart.js default tooltip styling — use a clean, minimal custom tooltip if interactivity adds value

---

## 10. Post-Render Checklist

After rendering every chart, verify these SWD essentials before presenting:

- [ ] Zero baseline on all bar charts
- [ ] Legend removed — data labeled directly
- [ ] Accent color used on 1-2 elements max; everything else is grey
- [ ] All text is horizontal (no diagonal/rotated labels)
- [ ] Title present, top-left, largest text on the chart
- [ ] No data point circles on line charts

---

## 11. Edge Cases

- **Insufficient or ambiguous data**: Ask the user to clarify before charting. Do not guess or fabricate data points.
- **Too many categories**: See §3 Managing Many Categories — never plot more than ~10-12 in a single chart.
- **Too many time periods for labels to fit**: Abbreviate labels (Q1, Q2…) or show every Nth label. Never rotate text diagonally.
- **Mixed units in a single request**: Use a merged line chart or separate charts — never combine different units on a shared axis.
- **Visualizer tool unavailable**: Describe the chart you would build (type, data mapping, color strategy) so the user can recreate it.