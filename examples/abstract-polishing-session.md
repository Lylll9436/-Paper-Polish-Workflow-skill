# Example Session: Polishing an Abstract

This document shows a complete session of polishing an abstract for a CEUS journal submission.

## Context

- **Paper**: Dual-layer GNN framework for urban perception prediction
- **Target Journal**: CEUS (Computers, Environment and Urban Systems)
- **Section**: Abstract

---

## Phase 0: Pre-Polish Baseline

**AI**: 
```
Before we start polishing, I need some information:
1. What is the target journal? Any special requirements?
2. Where are the example/reference papers?
3. Which section should we start with?
```

**User**: CEUS journal, examples in docs/example, start with abstract

**AI**: 
```
## Pre-Polish Checklist
- Target: CEUS (≤8,000 words total, ≤250 words abstract)
- Example papers: docs/example/*.pdf
- Section: Abstract
- Current abstract word count: ~280 words (needs trimming)
```

---

## Step 1: Confirm Macro Structure

**AI**:
```markdown
## Abstract Structure Analysis

| Component | Function | Suggested Sentences |
|-----------|----------|---------------------|
| Background | Research importance | 1-2 |
| Gap | Existing limitations | 1-2 |
| Method | Proposed approach | 3-4 |
| Results | Key findings | 1-2 |
| Contribution | Significance | 1 |

Please confirm this structure, or describe adjustments needed.
```

**User**: The structure looks good. Confirm.

---

## Step 2: Confirm Per-Sentence Logic

**AI**:
```markdown
## Per-Sentence Logic Mapping

| Sentence | Logic Function | Key Points |
|----------|----------------|------------|
| S1 | Background | Urban perception importance for planning/policy |
| S2 | Data | SVI as critical data source |
| S3 | Gap | Pixel-level features → ignore entities/relations → black box → can't support decisions |
| S4 | Method Overview | Dual-layer framework: micro semantics + macro context |
| S5 | Method Micro | LLM extracts scene graphs (entities, relations, attributes) |
| S6 | Method Micro | GNN encodes graphs, fuses with text summaries |
| S7 | Method Macro | Dual-edge urban graph (topology + POI similarity) |
| S8 | Method Macro | GAT propagates features for urban context |
| S9 | Results | 8 cities, 1.8M comparisons, AUC [TBD], outperforms baselines |
| S10 | Contribution | Accuracy + interpretability + transferable pre-trained GNN |

Please confirm each sentence's logic.
```

**User**: 
- S1: Confirm
- S2: Confirm  
- S3: Modify - emphasize that entities/relations are central to human perception
- S4-S8: Confirm
- S9: Confirm
- S10: Confirm

**AI**: Updated S3 logic to: "Pixel-level features → ignore entities/relations (which are central to human perception) → black box → can't support decisions"

---

## Step 3: Confirm Specific Expressions

**AI** uses `mcp_question`:

```
┌─────────────────────────────────────────────────────────────┐
│ S1 Expression                                               │
│ ─────────────────────────────────────────────────────────── │
│ S1: Urban perception importance for planning/policy         │
│                                                             │
│ ○ A: Human perception of urban environments has gained      │
│      increasing importance in urban planning, wellbeing     │
│      research, and policy-making.                           │
│                                                             │
│ ○ B: How people perceive urban environments has become      │
│      increasingly influential in urban planning and         │
│      public health.                                         │
│                                                             │
│ ○ C: Understanding how people perceive street environments  │
│      is essential for urban planning and policy decisions.  │
│                                                             │
│ ○ Type your own answer                                      │
└─────────────────────────────────────────────────────────────┘
```

**User selects**: A

```
┌─────────────────────────────────────────────────────────────┐
│ S2 Expression                                               │
│ ─────────────────────────────────────────────────────────── │
│ S2: SVI as critical data source                             │
│                                                             │
│ ○ A: Street view imagery (SVI) has emerged as a critical    │
│      data source enabling city-scale perception analysis.   │
│                                                             │
│ ○ B: The widespread availability of street view imagery     │
│      has enabled large-scale, automated perception          │
│      assessment across cities.                              │
│                                                             │
│ ○ C: Street view imagery provides the visual foundation     │
│      for scalable urban perception research, capturing      │
│      the pedestrian-level perspective.                      │
│                                                             │
│ ○ Type your own answer                                      │
└─────────────────────────────────────────────────────────────┘
```

**User selects**: C

*(Process continues for all sentences...)*

---

## Step 4: Reference Paper Consultation

**User**: "Is 'urban planning, wellbeing research, and policy-making' the right terminology?"

**AI** analyzes example papers:

```markdown
## Reference Paper Patterns

| Paper | Opening Phrase | Pattern |
|-------|---------------|---------|
| Paper 1 | "environmental perception and socioeconomic predictions" | task-focused |
| Paper 2 | "urban planning and public health" | domain-focused |
| Paper 3 | "public health and urban livability" | outcome-focused |
| Paper 4 | "urban safety detection" | specific task |

## Suggested Alternatives

| Option | Expression |
|--------|------------|
| A | urban planning, wellbeing research, and policy-making |
| B | urban planning, public health, and socioeconomic analysis |
| C | urban livability, public health, and planning interventions |
| D | urban planning, wellbeing research, and policy-making (keep original) |
```

**User selects**: D (keep original)

---

## Step 5: Repetition & Coherence Check

**AI**:
```markdown
## Repetition Check

| Location | Issue | Suggestion |
|----------|-------|------------|
| S4 vs S10 | Both mention "pre-trained GNN" | Remove from S4, keep in S10 |

## Coherence Check

| Location | Issue | Suggestion |
|----------|-------|------------|
| S3→S4 | Abrupt transition from gap to method | Add transition word |
```

**AI** uses `mcp_question` for transition:

```
┌─────────────────────────────────────────────────────────────┐
│ Transition Word                                             │
│ ─────────────────────────────────────────────────────────── │
│ S3→S4 transition: From 'problem' to 'solution'              │
│                                                             │
│ ○ A: "To address these limitations, this study..."          │
│ ○ B: "In response, this study..."                           │
│ ○ C: "Building on recent advances, we..."                   │
│ ○ D: Keep as is (no transition)                             │
└─────────────────────────────────────────────────────────────┘
```

**User selects**: A

---

## Final Version

**AI**:
```markdown
## Final Abstract

Human perception of urban environments has gained increasing importance 
in urban planning, wellbeing research, and policy-making. Street view 
imagery provides the visual foundation for scalable urban perception 
research, capturing the pedestrian-level perspective. However, existing 
methods rely on pixel-level feature extraction, neglecting the semantic 
entities and relations that are central to human perception, and operate 
as black boxes that cannot inform real-world planning decisions. To 
address these limitations, this study introduces a dual-layer framework 
combining micro-level scene semantics with macro-level urban context. 
The micro-level module employs large language models (LLMs) to extract 
structured scene graphs comprising entities, relations, and attributes 
from each street view image. These graphs are encoded using graph neural 
networks and fused with textual scene summaries. At the macro level, we 
construct an urban graph with two types of edges: physical street 
connectivity and functional similarity based on point-of-interest 
distributions. Features are then propagated through graph attention 
networks to capture urban context. We evaluate the framework on 8 global 
cities with 1.8 million LLM-annotated pairwise comparisons, achieving an 
AUC of [TBD] and outperforming [TBD] baselines. The framework improves 
prediction accuracy while providing interpretable outputs that support 
urban decision-making, and delivers a pre-trained GNN transferable to 
various downstream tasks.

**Word Count**: ~230 words ✅ (within 250 limit)

Confirm to write to file?
```

**User**: Yes

**AI**: ✅ Written to `docs/paper/sections/00_abstract_polished.md`

---

## Summary

| Step | Action | User Input |
|------|--------|------------|
| Phase 0 | Collect requirements | CEUS, docs/example |
| Step 1 | Confirm structure | Confirmed |
| Step 2 | Confirm sentence logic | Modified S3 |
| Step 3 | Select expressions | A, C, B, B, ... |
| Step 4 | Reference consultation | Keep original |
| Step 5 | Coherence check | Added transition |
| Write | Final confirmation | Yes |

**Total conversation turns**: ~15
**Time saved**: Systematic approach avoids back-and-forth revisions
