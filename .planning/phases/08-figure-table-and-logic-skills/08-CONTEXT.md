# Phase 8: Figure/Table and Logic Skills - Context

**Gathered:** 2026-03-12
**Status:** Ready for planning

<domain>
## Phase Boundary

Build two new Skills following Phase 2 conventions:
1. **Caption Skill** (SECT-02): Generate or optimize figure/table captions with geography-specific awareness (study area, CRS, legend details, data source). Output written to .tex file.
2. **Logic Verification Skill** (SECT-04): Trace argument chains across full paper, identify gaps/contradictions/unsupported claims/terminology inconsistencies. Output written to Markdown file.

This phase does not implement automated fixes — Caption Skill produces captions, Logic Skill identifies problems only.

</domain>

<decisions>
## Implementation Decisions

### Caption Skill — execution paths
- Support BOTH Generate and Optimize paths (dual-path, same as Abstract Skill pattern)
- Generate path: user provides figure/table content description → new caption produced
- Optimize path: user provides existing weak caption + content description → improved caption produced
- Path selected by trigger inference or Ask Strategy pre-question (if ambiguous)

### Caption Skill — Figure vs Table distinction
- Distinguish processing logic for figures and tables:
  - **Figure captions**: describe visual layers, spatial/geometric properties, legend summary, data source
  - **Table captions**: describe row/column semantics, statistical context, units, data source
- User specifies type (Figure or Table) during Ask Strategy; not inferred silently

### Caption Skill — output
- Output written directly to the user's .tex file (not conversation-only)
- User must provide the target .tex file path
- Caption appended/inserted at the appropriate LaTeX `\caption{}` command location
- Consistent with Translation Skill's file-write pattern

### Caption Skill — input modes
- Two input modes: file (CSV, TXT, data table) and pasted text description
- User can paste raw data or describe the figure/table in natural language
- Both modes accepted; Skill infers content from either

### Caption Skill — geography metadata
- Ask Strategy phase proactively asks for key geography metadata:
  - Figure type (map / chart / diagram / photo?)
  - Study area / city name (if spatial)
  - Data source brief (e.g., OpenStreetMap, Landsat-8, Census 2020)
  - CRS only if user knows it and it's relevant to the figure type
- Caption should include: study area / city name AND data source brief when provided
- CRS notation, legend items, scale bar: ask proactively but only include if user provides them
- Missing metadata: skip gracefully — generate from what user provides, do NOT use [MISSING: ...] placeholders or refuse

### Caption Skill — journal adaptation
- When target journal specified: load journal template (e.g., references/journals/ceus.md) and adapt caption length and style
- Missing journal template → refuse execution (consistent with Translation/Polish/De-AI/Reviewer Skills)

### Logic Verification Skill — verification scope
- Verify ALL FOUR issue types:
  1. **Cross-section argument chain**: Does Introduction claim get supported by Methods? Do Results support Discussion conclusions?
  2. **Unsupported claims**: Statements that assert but provide no evidence or citation
  3. **Terminology inconsistency**: Same concept named differently across sections (e.g., "Model A" / "the proposed model" / "our approach")
  4. **Number/conclusion contradictions**: Numbers in Introduction differ from Results, or Conclusion contradicts Discussion

### Logic Verification Skill — input
- Full paper required — no partial-section input accepted
- Partial sections (just Introduction, just Methods) → refuse with explanation: "Cross-section verification requires the full paper"
- Supports both file path and pasted text input (consistent with other Skills)

### Logic Verification Skill — output behavior
- Identify problems + directional suggestions only — no rewritten text
- Each issue: problem description + why it's a problem + one-sentence directional suggestion
- Section reference format: section title in brackets, e.g., `[Introduction]`, `[Methods, 第3段]`
- Does NOT produce full rewrites (that's Polish Skill's domain)

### Logic Verification Skill — report structure (two-part)
- **Part 1 — Argument Chain View**: Maps each section's primary claim in sequence (Introduction → Methods → Results → Discussion → Conclusion). Shows how claims connect or fail to connect.
- **Part 2 — Categorized Issue List**: Issues grouped by type (Argument Chain Gap / Unsupported Claim / Terminology Inconsistency / Number Contradiction), each with section reference and directional suggestion

### Logic Verification Skill — bilingual output
- English issue descriptions with inline Chinese blockquote translation after each issue
- Same pattern as Reviewer Simulation Skill
- Argument Chain View: English only (structural, not language-sensitive)

### Logic Verification Skill — output file
- Written to Markdown file: `{input_filename_without_ext}_logic.md`
- For pasted text input: presented in conversation (no file to write to)
- Consistent with Reviewer Simulation's `*_review.md` naming convention

### Shared decisions (carry-forward from prior phases)
- ~300 line hard budget per SKILL.md
- Bilingual Chinese/English triggers in frontmatter `examples` field
- Direct mode as default
- Anti-AI patterns NOT loaded by default (review/analysis Skills, not writing Skills)
- Both Skills follow skill-conventions.md frontmatter contract exactly

</decisions>

<specifics>
## Specific Ideas

- Caption Skill's geography-aware Ask Strategy mirrors the Reviewer Simulation's structured pre-questions — proactive but not exhaustive
- Logic Verification's argument chain view could be formatted as a simple table (Section | Primary Claim | Connects To) — compact and scannable
- The "skip gracefully" rule for missing Caption metadata avoids blocking users who know their content but not the CRS — practical for real usage
- Logic Skill's terminology inconsistency check should handle both explicit renames (different words) and implicit ones (abbreviation introduced in Methods but not in Introduction)

</specifics>

<code_context>
## Existing Code Insights

### Reusable Assets
- `.claude/skills/abstract-skill/SKILL.md`: dual-path pattern (generate/restructure) — Caption Skill should follow this structure for generate/optimize paths
- `.claude/skills/reviewer-simulation-skill/SKILL.md`: bilingual report format, file output pattern, section-level references — Logic Skill should mirror this
- `.claude/skills/de-ai-skill/SKILL.md`: two-phase detect-then-act with user confirmation — Logic Skill may borrow phase separation pattern
- `references/expression-patterns/geography-domain.md`: geography-specific expression patterns — Caption Skill should load this leaf for spatial figure captions
- `references/journals/ceus.md`: journal caption length and style conventions — Caption Skill loads when journal specified
- `references/skill-conventions.md` + `references/skill-skeleton.md`: authoring contract for both new Skills

### Established Patterns
- Dual-path Skills: Abstract Skill (Phase 7) — confirmed pattern for generate/optimize
- Two-part report: Reviewer Simulation (Phase 6) — confirmed pattern for Logic Skill
- File output with naming convention: Reviewer Simulation `*_review.md` → Logic Skill uses `*_logic.md`
- Journal missing → refuse: Translation/Polish/De-AI/Reviewer — Caption Skill follows same rule
- Ask Strategy with proactive geography questions: Translation Skill asks about target journal — Caption Skill extends this for geography metadata

### Integration Points
- Caption Skill: `.claude/skills/caption-skill/SKILL.md`
  - Loads `references/expression-patterns/geography-domain.md` for spatial figure captions
  - Loads `references/journals/ceus.md` when CEUS specified
  - Writes output to user-specified `.tex` file
- Logic Skill: `.claude/skills/logic-skill/SKILL.md`
  - No expression pattern leaf loads needed (analysis task, not writing task)
  - Writes output to `*_logic.md` file alongside input file

</code_context>

<deferred>
## Deferred Ideas

None — discussion stayed within Phase 8 boundary.

</deferred>

---

*Phase: 08-figure-table-and-logic-skills*
*Context gathered: 2026-03-12*
