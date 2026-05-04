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

### Direct Visualize Mode — skip the question

If the user's invocation begins with **`visualise`** or **`visualize`** (e.g.
`/swd-chart visualise this data`, `/swd-chart visualize team performance`), **skip
steps 2–4**. Do not present story options and do not wait for confirmation. Instead:

1. Analyse the data as normal (step 1).
2. **Pick the single strongest story yourself** — the most obvious headline the data
   supports (largest gap, clearest trend, biggest outlier, dominant concentration).
3. **Pick the best-practice chart type** for that story using §2 and the decision tree
   in `references/core-config.md` §4. Default to a horizontal bar chart unless the data
   genuinely demands otherwise.
4. **Render immediately** with full SWD treatment (grey-first color, direct labels,
   declutter, action title naming the story).
5. After rendering, briefly state in 1–2 sentences which story you chose and why, and
   offer to re-render with a different angle if the user wants one.

This mode trusts you to make the call. Only fall back to asking if the data is genuinely
ambiguous or insufficient (see §13 Edge Cases).

---

## 2. Choose the Right Chart

**Horizontal bar charts are the strong default.** When in doubt, always reach for a horizontal bar chart first — it is the most versatile, readable, and accessible chart type. Only use a different type when horizontal bars genuinely cannot tell the story (e.g., continuous time series → line chart).

Quick reference for common cases:

| Data situation | Best chart type |
|---|---|
| Comparing categories | **Horizontal bar chart** (preferred) or vertical bar for time-ordered categories |
| Trend over time (continuous) | **Line chart** |
| Parts of a whole | **100% stacked horizontal bar** |
| Two different variables over time | **Merged line chart** (vertically stacked, shared x-axis) |

For the full decision tree (including waterfall, heatmap, simple text, and edge cases), see `references/core-config.md` §4. For chart-type prohibitions (no pie, no dual-axis, no 3D, no slopegraph), see `references/core-config.md` §6.

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

## 5. Color Strategy — Grey-First, Highlight-Only

The default state of every chart is **grey**. All bars, lines, labels, and annotations
start in grey. Color is then applied surgically — only to the data points that ARE the
story. This is the single most important SWD principle for this skill.

See `references/core-config.md` §1 for exact hex tokens and extended shades.

### The grey-first rule

- **Default bar color**: `#D1D5DB` (BASE_LIGHT) — every bar starts here
- **Default label color**: `#9CA3AF` (BASE_MID) — every data label starts here
- **Default axis label weight**: `400` (normal) — no emphasis by default

Color is earned. A bar only gets an accent color if it is:
1. The specific outlier or problem the chart is highlighting
2. Below/above a meaningful threshold that the headline calls out
3. The answer to the question the user is asking

### What color means

- **Orange `#D97706`**: Something is wrong. A score below threshold, a declining metric,
  a problem that needs attention. Use for the bars/labels you want the eye drawn to
  because they represent concern.
- **Blue `#2563EB`**: Something is notably good or is the primary subject. Use sparingly —
  only when "good performance" IS the story (e.g., a recovery, a top performer).
- **Grey `#D1D5DB`**: Everything else. The context. The supporting cast. Grey does not
  mean "bad" — it means "not the point right now."

### Common mistake to avoid

Do NOT use a three-color scale (orange = bad, blue = good, grey = middle) as a default
encoding. This turns the chart into a heatmap and dilutes the story. Instead:

- If the story is "one thing is broken" → that thing is orange, everything else is grey
- If the story is "one thing is great" → that thing is blue, everything else is grey
- If the story is "compare good vs bad" → only then use both orange and blue, with grey
  for anything that's neither

### Label color follows bar color

When a bar is highlighted with an accent color, its data label should match:
- Orange bar → orange label text, weight 700
- Grey bar → grey label text (`#9CA3AF`), weight 400
- Same for y-axis category labels: bold + accent color for highlighted rows,
  normal weight + dark grey for everything else

### General color rules

- Use color **consistently** — if blue means "our company" on one chart, it means that everywhere
- Never use color for decoration. Every color choice must be intentional.
- Avoid red-green combinations (colorblind accessibility). Use blue vs. orange instead.
- Varying **saturation** of one hue works well for ordinal data (e.g., priority levels)

---

## 6. Progressive Data Storytelling — The Drill-Down Pattern

When data has a natural hierarchy (portfolio → site → equipment type → fault category),
don't flatten everything into one chart. Instead, build a **progressive reveal** — a
sequence of charts in a single widget that guides the reader from overview to root cause.

### When to use the drill-down pattern

Use multi-chart drill-downs when:
- The user asks "why" or "what's causing" a metric to be low/high
- The data has at least 2 levels of hierarchy (e.g., site → category)
- One level shows the outlier, the next level explains it

### Structure

A drill-down widget contains 2–3 charts stacked vertically in one `show_widget` call,
separated by a thin divider line (`border-top: 1px solid #E5E7EB`). Each chart has its
own title and subtitle, telling one chapter of the story.

**Layer 1 — The Overview (Where's the problem?)**
- Broadest grouping (e.g., by site, by equipment type)
- Grey-first: only the outlier(s) get accent color
- Headline names the outlier and its score
- Sorted by score ascending so the problem is at the top (after reversal for horizontal bars)

**Layer 2 — The Breakdown (What's causing it?)**
- Drill into the highlighted outlier(s) from Layer 1
- Show the next level of detail (e.g., fault categories within the problem equipment type)
- Same grey-first rule: only categories below threshold get accent color
- Headline summarizes how many sub-categories are problematic
- Include supplementary context (rule counts, change values) as secondary labels

**Layer 3 (optional) — The Detail**
- Individual equipment or specific rules, only if the user asks to go deeper
- Usually a table is better than a third chart at this level

### Headline progression

Each chart's action title should advance the narrative:

1. "PAC units at **75.6%** and AHUs at **89.5%** are pulling Chirnside Park below the portfolio"
2. "Drilling into PAC & AHU: **5 fault categories** below 90%"

The reader should be able to read just the headlines and understand the full story.

### Visual continuity rules

- Use the **same accent color** (orange for problems) across all layers — the color IS
  the thread connecting the story
- Use the **same grey** (`#D1D5DB`) for all non-highlighted bars across all layers
- Keep chart widths consistent (same `max-width` wrapper)
- Right-side padding should be consistent across charts for aligned direct labels
- Divider between charts: `border-top: 1px solid #E5E7EB; margin: 24px 0 16px 0`

---

## 7. Direct Label Design — Two-Line Pattern

For horizontal bar charts with rich data, use a **two-line direct label** positioned
to the right of each bar:

**Line 1 (primary):** The score value — bold if highlighted, normal if grey
**Line 2 (secondary):** Supporting context — change in pp, rule count, or other metadata

```
// Positioning
ctx.fillText(val + '%', bar.x + 6, bar.y - 6);     // Line 1: slightly above center
ctx.fillText('+2.1 pp · 14 rules', bar.x + 6, bar.y + 7);  // Line 2: slightly below
```

### Styling rules

| Element | Highlighted row | Grey row |
|---|---|---|
| Line 1 font | `700 11px` | `400 11px` |
| Line 1 color | Accent (`#D97706`) | `#9CA3AF` |
| Line 2 font | `400 10px` | `400 10px` |
| Line 2 color | Accent (slightly darker for declining) | `#D1D5DB` |

The secondary line should be **visually quieter** than the primary — it's there for
readers who want detail, not for the first scan. On grey rows, the secondary label
should be very faint (`#D1D5DB`) so it doesn't compete with highlighted rows.

Ensure `layout.padding.right` is large enough (typically 80–100px) to prevent
labels from being clipped.

---

## 8. Focus Attention with Preattentive Attributes

Use these sparingly to direct the eye:

- **Color intensity**: The accent color on a grey field instantly draws the eye — this is the primary tool
- **Size**: Make the most important text/number larger. The chart title should be the largest text element.
- **Bold weight**: Use for titles, key labels, and the single most important annotation
- **Line emphasis**: Use line thickness and accent color to signal importance (data markers are removed per §4)
- **Numeric labels**: Add ONLY on the specific data points you want the audience to notice (never on every point — that creates clutter)
- **Position**: Place the most important information at the top-left. Titles and axis labels should be upper-left-most justified.

**The "push to background first" technique**: Start by making everything grey and small. Then intentionally bring forward only what matters. This ensures every emphasis is deliberate.

---

## 9. Annotations and Storytelling Text

Every chart MUST have:
- A **descriptive title** (top-left, dark grey, largest text on the chart)
- **Axis titles** (unless truly obvious from context)

Add these WHEN there is a clear insight:
- A **key insight callout** — 1-2 sentences near the relevant data, using the accent color for key words
- **Annotations** on specific data points explaining what happened or why it matters
- A **footnote** for data source or methodology (small, grey, bottom of chart — present but de-emphasized)

**Do not** add annotations just to fill space. If the chart speaks clearly on its own, a title and clean labels are enough.

---

## 10. Visual Hierarchy

Establish a clear reading order through size, color, and position:

1. **Title** — largest, dark grey, top-left (the audience reads this first)
2. **Key insight / action text** — accent colored, bold, near the data it describes
3. **Primary data** — accent colored line/bars, thicker/bolder than context data
4. **Context data** — grey, thinner, lighter — visible but doesn't compete
5. **Axis labels** — grey, standard size, readable but receding
6. **Footnotes / source** — smallest, lightest grey, bottom of chart

---

## 11. Implementation

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

## 12. Post-Render Checklist

After rendering every chart, verify these SWD essentials before presenting:

- [ ] Zero baseline on all bar charts
- [ ] Legend removed — data labeled directly
- [ ] Accent color used on 1-2 elements max; everything else is grey
- [ ] All text is horizontal (no diagonal/rotated labels)
- [ ] Title present, top-left, largest text on the chart
- [ ] No data point circles on line charts

---

## 13. Edge Cases

- **Insufficient or ambiguous data**: Ask the user to clarify before charting. Do not guess or fabricate data points.
- **Too many categories**: See §3 Managing Many Categories — never plot more than ~10-12 in a single chart.
- **Too many time periods for labels to fit**: Abbreviate labels (Q1, Q2…) or show every Nth label. Never rotate text diagonally.
- **Mixed units in a single request**: Use a merged line chart or separate charts — never combine different units on a shared axis.
- **Visualizer tool unavailable**: Describe the chart you would build (type, data mapping, color strategy) so the user can recreate it.