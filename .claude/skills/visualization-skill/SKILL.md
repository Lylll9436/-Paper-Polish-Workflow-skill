---
name: visualization-skill
description: >-
  Recommend appropriate chart types for experimental data with rationale and tool hints.
  Geography-aware: choropleth, spatial scatter, kernel density when spatial data detected.
  为实验数据推荐合适的图表类型，支持地理空间数据可视化建议。
triggers:
  primary_intent: recommend visualization types for experimental data
  examples:
    - "What chart should I use for this data?"
    - "帮我选择合适的可视化方式"
    - "Recommend a visualization for my spatial analysis results"
    - "我应该用什么图展示这个结果"
    - "Which plot type fits my regression results?"
    - "帮我推荐一个展示时空数据的图表"
tools:
  - Structured Interaction
references:
  required: []
  leaf_hints: []
input_modes:
  - pasted_text
output_contract:
  - recommendation_list
---

## Purpose

This Skill accepts a plain-language description of experimental data (data type, key variables, sample size) and a research question (what the figure must communicate), then recommends 2–3 chart types ordered by fit. Each recommendation includes the chart type name, a 1–2 sentence rationale referencing the user's specific data and question, and Python/R library hints (names only, no code blocks). The Skill is geography-aware: if the data description contains spatial signals (coordinates, regions, administrative boundaries, GIS data), it proactively includes choropleth maps, spatial scatter plots, or kernel density maps as candidates alongside general types. When no spatial signals are present, only general chart types are recommended — geography charts are never forced onto non-spatial data. The Skill accepts text descriptions only; it does not read or process actual data files.

## Trigger

**Activates when the user asks to:**
- Choose a chart type or recommend a visualization for their data
- Decide how to display experimental results
- 帮我选择可视化方式、我应该用什么图、帮我推荐图表类型

**Example invocations:**
- "What chart should I use for this data?" / "帮我选择合适的可视化方式"
- "Recommend a visualization for my spatial analysis results" / "帮我推荐一个展示时空数据的图表"
- "Which plot type fits my regression results?" / "我应该用什么图展示这个结果"

## Modes

| Mode | Default | Behavior |
|------|---------|----------|
| `direct` | Yes | Single-pass recommendation after Ask Strategy; 2–3 cards delivered in one response |
| `batch` | | Not supported — each recommendation requires specific data description and research question |

**Default mode:** `direct`. User provides data description and research question; Skill delivers recommendation cards in one response.

## References

### Required (always loaded)

None. No reference files loaded. Recommendations derived from the data description and research question provided by the user.

### Leaf Hints

None.

## Ask Strategy

**Two inputs required before generating recommendations:**

1. **Data description:** What is the data? (data type, key variables, spatial or non-spatial, sample size if relevant) — ask if not included in the trigger.
2. **Research question for this figure:** What should the chart communicate? (e.g., distribution, comparison, trend, spatial pattern, correlation) — ask if not included in the trigger.

**Rules:**
- If both inputs are present in the trigger, proceed without asking.
- If only one input is provided, ask for the missing one before proceeding.
- Do not generate recommendations without both inputs.
- Use Structured Interaction when available; fall back to two plain-text questions otherwise.

## Workflow

### Step 1 — Collect Context

- Extract data description and research question from the trigger if present.
- Ask for any missing inputs per Ask Strategy above.

### Step 2 — Detect Spatial Signals and Select Chart Candidates

- Scan the data description for spatial signal keywords: "coordinates", "latitude", "longitude", "spatial", "region", "district", "administrative", "polygon", "GIS", "map", "geospatial", "study area", "choropleth", "raster", "urban" (see Edge Cases for "urban" boundary handling).
- **Spatial signals present:** include geography chart types as candidates (choropleth map, spatial scatter plot, kernel density map, hexbin map) alongside general types (bar chart, line chart, scatter plot, box plot, violin plot, heatmap).
- **No spatial signals:** use only general chart types — never recommend choropleth or spatial scatter for non-spatial data.
- Select 2–3 chart types ordered by fit. Best fit first.
- Use the research question to break ties: "distribution" → histogram/box; "comparison" → bar; "trend" → line; "correlation" → scatter; "spatial pattern" → choropleth/spatial scatter.

### Step 3 — Generate Recommendation Cards

- Write 2–3 cards using the locked format below.
- The reason must reference the user's specific data and question — not generic descriptions.
- Tool hints: when spatial data detected, include geopandas/contextily/tmap; when non-spatial, include matplotlib/seaborn/ggplot2.
- No code blocks — library and function names only (e.g., geopandas.plot(), seaborn.boxplot()).

**Locked card format:**

```
**Recommendation N: [Chart Type Name]**
- Why it fits: [1–2 sentences directly referencing the user's data type and research question]
- Tool hints: Python: [library.function()]; R: [package + geom/function]
```

## Output Contract

| Output | Format | Condition |
|--------|--------|-----------|
| `recommendation_list` | 2–3 numbered recommendation cards displayed in conversation | Always — no file write |

**Spatial data example (3 cards):**

**Recommendation 1: Choropleth Map**
- Why it fits: Your district-level green space coverage data maps directly to administrative polygon units — choropleth encodes the continuous coverage variable as color fill across spatial units.
- Tool hints: Python: geopandas.plot(), matplotlib; R: ggplot2 + sf + tmap

**Recommendation 2: Spatial Scatter Plot**
- Why it fits: If point-level measurements exist within districts, scatter placement shows within-district variation and spatial clustering.
- Tool hints: Python: geopandas.plot(), contextily for basemap; R: ggplot2 + geom_sf()

**Recommendation 3: Kernel Density Map**
- Why it fits: When point density matters more than precise values, kernel density smoothing reveals spatial hotspots and avoids overplotting.
- Tool hints: Python: scipy.stats.gaussian_kde, seaborn.kdeplot(); R: ggplot2 + stat_density_2d()

**Non-spatial data example (2 cards):**

**Recommendation 1: Grouped Bar Chart**
- Why it fits: Three neighborhoods with satisfaction score means are directly comparable side-by-side; grouped bars make between-group differences immediately readable.
- Tool hints: Python: seaborn.barplot(), matplotlib; R: ggplot2 + geom_bar()

**Recommendation 2: Violin Plot**
- Why it fits: Satisfaction score distributions (1–5 scale, 500 respondents) benefit from showing full shape rather than just means — violin reveals whether distributions are bimodal or skewed.
- Tool hints: Python: seaborn.violinplot(); R: ggplot2 + geom_violin()

## Edge Cases

| Situation | Handling |
|-----------|----------|
| No spatial signals in data description but user explicitly requests a map | Explain why a map may not be appropriate for non-spatial data; offer to include spatial chart types if user confirms data has a spatial dimension |
| Data description mentions "urban" or "city" without explicit spatial coordinates | Treat as boundary case: ask "Does your data include spatial coordinates or administrative unit boundaries?" before including spatial chart types |
| Only one variable mentioned in data description | Acknowledge limited dimensionality; recommend univariate charts (histogram, bar chart) suited to the constraint |
| User provides a file path instead of a data description | Ask user to describe the data instead: "Please describe your data (type, variables, sample size) — I cannot read data files directly" |
| User asks for "best" chart without any context | Run Ask Strategy; do not guess from insufficient information |
| Research question is ambiguous (e.g., "show the results") | Ask for clarification: "What aspect of the results should the figure communicate — distribution, comparison, trend, or spatial pattern?" |

## Fallbacks

| Scenario | Fallback |
|----------|----------|
| Structured Interaction unavailable | Ask data description and research question as two sequential plain-text questions; proceed with same recommendation logic |
| Data description too vague to determine spatial vs. non-spatial | Default to non-spatial chart types; note: "If your data has a spatial dimension, include that in the description for geography-aware recommendations" |
| Research question not answerable by a single chart type | Acknowledge and recommend the chart type that covers the primary question; note the secondary question as a candidate for a separate figure |

## Examples

**Example 1 — Spatial data:**

User: "I have district-level urban green space coverage data for 50 districts in Shenzhen. Research question: how does green space vary spatially across districts?"

Spatial signals detected: "district", "Shenzhen" (urban + administrative unit context confirmed). Three recommendations generated using geopandas (Python) and tmap (R) tool hints. Cards: Choropleth Map (best fit for polygon-unit continuous variable), Spatial Scatter Plot (if point data exists), Kernel Density Map (if density pattern is the focus).

**Example 2 — Non-spatial data:**

User: "I have survey responses from 500 residents on satisfaction scores (1–5) for three neighborhoods. Research question: how do satisfaction scores compare across neighborhoods?"

No spatial signals detected. Two recommendations generated using seaborn and ggplot2 tool hints. Cards: Grouped Bar Chart (comparison across three groups), Violin Plot (shows full score distribution shape). No geography chart types included.

---

*Skill: visualization-skill*
*Conventions: references/skill-conventions.md*
