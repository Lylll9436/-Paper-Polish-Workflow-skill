# Skill: Academic Paper Polish Workflow

A systematic, top-down workflow for polishing academic papers. Works from structure → logic → expression, with user confirmation at each step.

## Key Features

- **Top-down approach**: Structure → Sentence Logic → Expression
- **User-led**: Confirm at each step before proceeding
- **Option-based**: Provide 2-4 expression options using `mcp_question` for efficient selection
- **Reference-driven**: Consult example papers for professional phrasing

---

## Trigger Conditions

Use this skill when user requests:
- "润色论文" / "精修论文"
- "polish paper" / "revise paper section"
- "help me improve my paper writing"

---

## Core Principles

1. **Top-down (从大到小)**: Structure → Logic → Expression (never jump to wording before confirming logic)
2. **User-led (用户主导)**: Every step requires user confirmation before proceeding
3. **Option-based (选项制)**: Use `mcp_question` tool to present expression choices efficiently
4. **Reference-driven (范文驱动)**: When user questions professionalism, consult example papers
5. **Sentence-by-sentence (逐句确认)**: Don't present large blocks of changes; confirm incrementally

---

## Complete Workflow

### Phase 0: Pre-Polish Baseline (预检基线)

Collect the following before starting:

```markdown
## Pre-Polish Checklist
- [ ] Target journal and requirements (word limits, abstract limits, special formats)
- [ ] Current section word count vs target
- [ ] Key numbers extraction (city count, data scale, performance metrics)
- [ ] Example papers location (if available)
- [ ] Any previous analysis reports to reference
```

**Ask user**:
1. What is the target journal? Any special requirements?
2. Where are the example/reference papers?
3. Which section to start with?

---

### Step 1: Confirm Macro Structure (确认宏观结构)

**Purpose**: Confirm the logical composition of the section

**Actions**:
1. Read current section content
2. Analyze and list logical structure (e.g., Abstract = Background + Gap + Method + Results + Contribution)
3. Present structure table for user confirmation

**Output Format**:
```markdown
## [Section] Structure Analysis

| Component | Function | Suggested Proportion |
|-----------|----------|---------------------|
| Background | Importance of research area | 1-2 sentences |
| Gap | Limitations of existing methods | 1-2 sentences |
| Method | Overview of proposed approach | 3-4 sentences |
| Results | Key findings | 1-2 sentences |
| Contribution | Research significance | 1 sentence |

**Please confirm this structure, or describe adjustments needed.**
```

**Checkpoint**: User confirms structure before proceeding to Step 2

---

### Step 2: Confirm Per-Sentence Logic (确认每句话的逻辑)

**Purpose**: Break down structure into specific logic points for each sentence

**Actions**:
1. Split current content into sentences
2. Assign logical function to each sentence
3. Present logic chain table

**Output Format**:
```markdown
## Per-Sentence Logic Mapping

| Sentence | Logic Function | Key Points |
|----------|----------------|------------|
| S1 | Background | Urban perception importance for planning |
| S2 | Data | SVI as critical data source |
| S3 | Gap | Pixel-level → ignores entity relations → black box |
| ... | ... | ... |

**Please confirm each sentence's logic:**
- S1: ✅ Confirm / ❌ Modify (please describe)
- S2: ✅ Confirm / ❌ Modify
- ...
```

**Checkpoint**: User confirms all sentence logic before proceeding to Step 3

---

### Step 3: Confirm Specific Expressions - Using mcp_question (确认具体表达)

**Purpose**: Based on confirmed logic, provide expression options for each sentence

**IMPORTANT: Use `mcp_question` tool for efficient multi-selection**

**Actions**:
1. Process 3-5 sentences at a time (batch for efficiency)
2. Provide 2-4 expression options per sentence
3. Call `mcp_question` with all options

**Implementation Example**:

```javascript
mcp_question({
  questions: [
    {
      question: "S1: Urban perception importance for planning decisions",
      header: "S1 Expression",
      options: [
        { 
          label: "A", 
          description: "Human perception of urban environments has gained increasing importance in urban planning, wellbeing research, and policy-making." 
        },
        { 
          label: "B", 
          description: "How people perceive urban environments has become increasingly influential in urban planning and public health." 
        },
        { 
          label: "C", 
          description: "Understanding how people perceive street environments is essential for urban planning and policy decisions." 
        }
      ],
      multiple: false
    },
    {
      question: "S2: Traditional methods are limited",
      header: "S2 Expression",
      options: [
        { 
          label: "A", 
          description: "Traditional methods such as field surveys and questionnaires are costly, time-consuming, and limited in spatial coverage." 
        },
        { 
          label: "B", 
          description: "Conventional approaches relying on surveys face constraints in cost, time, and scalability." 
        },
        { 
          label: "C", 
          description: "Measuring perception through traditional surveys is resource-intensive and difficult to scale." 
        }
      ],
      multiple: false
    }
  ]
})
```

**Benefits of mcp_question**:
- User can answer multiple sentences in one interaction
- Cleaner UI than text-based selection
- Reduces back-and-forth conversation turns
- User can type custom answers (custom option enabled by default)

**Fallback**: If mcp_question is unavailable, use text-based table format:
```markdown
| Option | Expression |
|--------|------------|
| A | ... |
| B | ... |
| C | ... |

Select A / B / C or provide your version?
```

---

### Step 4: Reference Paper Consultation (范文参考)

**Trigger**: When user questions whether an expression is professional

**Actions**:
1. Read example papers (usually PDFs, use `mcp_look_at` to analyze)
2. Extract relevant expression patterns from similar sections
3. Provide new options based on reference patterns

**Output Format**:
```markdown
## Reference Paper Patterns

| Paper | Opening Sentence | Pattern |
|-------|-----------------|---------|
| Paper 1 | "Street view imagery has become..." | [Data source] has become important for [field] |
| Paper 2 | "Measuring urban safety perception is..." | [Task] is important and complex |

## New Options Based on References

| Option | Expression | Reference Pattern |
|--------|------------|-------------------|
| A | ... | Paper 1 pattern |
| B | ... | Paper 2 pattern |
```

---

### Step 4.5: Journal Style Check (期刊风格检查)

**For CEUS journal specifically**:

```markdown
## CEUS Style Checklist

- [ ] Geospatial perspective emphasized? (CEUS requirement)
- [ ] Policy/planning implications in Discussion?
- [ ] Sustainability angle considered?
- [ ] Avoid words like "novel", "groundbreaking"
- [ ] Verb tense: Methods=past, Results=past, Discussion=present
```

---

### Step 5: Repetition & Coherence Check (重复与连贯性检查)

**Actions**:
1. Check for repeated expressions between sentences/paragraphs
2. Check if transition words are needed
3. Present issues and suggest fixes

**Use mcp_question for transition word selection**:

```javascript
mcp_question({
  questions: [
    {
      question: "S3→S4 transition: From 'problem' to 'solution'",
      header: "Transition Word",
      options: [
        { label: "A: Add 'To address these limitations'", description: "To address these limitations, this study introduces..." },
        { label: "B: Add 'In response'", description: "In response, this study introduces..." },
        { label: "C: Add 'Building on these opportunities'", description: "Building on these opportunities, we introduce..." },
        { label: "D: Keep as is", description: "No transition word needed" }
      ],
      multiple: false
    }
  ]
})
```

**Output Format**:
```markdown
## Repetition Check

| Location | Repeated Content | Suggestion |
|----------|-----------------|------------|
| S4 vs S10 | Both mention "pre-trained GNN" | Remove from S4, keep in S10 |

## Coherence Check

| Location | Issue | Suggestion |
|----------|-------|------------|
| S3→S4 | Abrupt transition | Add "To address these limitations, " |
```

---

### Step 5.5: Cross-Section Consistency (跨章节一致性)

**Check items**:
- Numbers consistent across sections (e.g., city count, data scale)
- Terminology consistent (e.g., SVI vs street view imagery)
- Claims in Abstract match Conclusion

---

### Step 5.7: Generate Highlights - CEUS Required (生成Highlights)

**CEUS requirement**: 3-5 highlights, each ≤85 characters

**Output Format**:
```markdown
## CEUS Highlights

| # | Highlight | Characters | Status |
|---|-----------|------------|--------|
| 1 | Dual-layer framework integrates scene semantics with urban context | 67 | ✅ |
| 2 | LLM-powered scene graphs provide interpretable semantic features | 64 | ✅ |
| 3 | Pre-trained GNN transfers to downstream urban analytics tasks | 60 | ✅ |
| 4 | Experiments on 8 cities with 1.8 million pairwise comparisons | 58 | ✅ |
| 5 | Framework outperforms CNN and Transformer baselines significantly | 66 | ✅ |

Rules:
- Each highlight ≤85 characters
- No abbreviations (unless universally known)
- No citations
```

---

### Step 6: Read-Aloud Final Check (朗读最终检查)

**Suggest to user**:
1. Read the final version aloud
2. Mark awkward phrasing
3. Check sentence length (if hard to read without pause, consider splitting)

---

### Write to File (写入文件)

**Actions**:
1. Compile all confirmed content
2. Present final version for user confirmation
3. Write to `*_polished.md` file after confirmation

**Output Format**:
```markdown
## Final Version

> [Complete polished content]

**Word Count**: XXX words

**Confirm to write to file?**
```

---

## Journal-Specific Requirements

### CEUS (Computers, Environment and Urban Systems)

| Requirement | Specification |
|-------------|---------------|
| Total word limit | ≤8,000 words |
| Abstract | ≤250 words |
| Highlights | 3-5 items, each ≤85 characters, **required** |
| Graphical Abstract | Recommended (≥531×1328 pixels) |
| Data Availability | Statement required |
| Review type | Double-blind (anonymized manuscript required) |

### Word Budget Reference (8000 total)

| Section | Suggested Words |
|---------|-----------------|
| Abstract | 200-250 |
| Introduction | 700-900 |
| Related Work | 800-1,000 |
| Methodology | 2,000-2,500 |
| Experiments | 1,500-2,000 |
| Discussion | 500-700 |
| Conclusion | 200-400 |

---

## Common Issue Handling

### User says "this word isn't professional"
→ Use `mcp_look_at` to analyze example papers
→ Extract patterns from similar sections
→ Provide reference-based options via `mcp_question`

### User says "this sentence is abrupt"
→ Provide transition word options via `mcp_question`
→ Common transitions: "To address...", "In response...", "However...", "Moreover..."

### User says "there's repetition"
→ Identify repeated content
→ Suggest which to remove with reasoning

### User wants to modify logic structure
→ Return to Step 1 or Step 2 to re-confirm

### User wants to expand/reduce word count
→ Identify which paragraphs to expand/compress
→ Get user confirmation on approach before proceeding

---

## Tools Used

| Tool | Purpose |
|------|---------|
| `mcp_read` | Read current paper sections |
| `mcp_look_at` | Analyze example paper PDFs |
| `mcp_question` | **Primary tool for expression selection** - present options efficiently |
| `mcp_write` | Write polished content to file |
| `mcp_edit` | Make targeted edits to existing files |

---

## Example Conversation Flow

```
User: Help me polish the abstract

AI: [Phase 0] 
    - What is the target journal?
    - Where are the example papers?
    
User: CEUS, examples in docs/example

AI: [Step 1] 
    Present abstract structure analysis table
    "Please confirm structure..."

User: Confirmed

AI: [Step 2]
    Present per-sentence logic table
    "Please confirm each sentence's logic..."

User: S1 confirmed, S2 should emphasize...

AI: [Step 3]
    Call mcp_question with expression options for S1-S4
    
User: (Selects via mcp_question UI or types custom answer)

AI: Record selections, continue with S5-S8...

AI: [Step 4] (if needed)
    User questions professionalism of a term
    Use mcp_look_at to analyze example papers
    Present reference-based new options

AI: [Step 5]
    Check repetition and coherence
    Use mcp_question for transition word selection

AI: [Write to File]
    Present final version
    "Confirm to write to file?"
    
User: Yes

AI: Write to *_polished.md
```
