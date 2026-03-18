# Phase 14: Repo-to-Paper Core Structure - Context

**Gathered:** 2026-03-18
**Status:** Ready for planning

<domain>
## Phase Boundary

User can point to an experiment repo and get a complete H1/H2/H3 outline with approval checkpoints at each level. This phase creates the repo-to-paper Skill and `references/repo-patterns.md` reference file. Body text generation is Phase 16; literature integration is Phase 15.

</domain>

<decisions>
## Implementation Decisions

### Repo Scan Behavior
- **Scan method**: Directory structure inference based on common Python ML project patterns defined in `references/repo-patterns.md`
- **Scan depth**: Only top 2 levels of directories (root + first-level subdirectories). User can manually specify deeper paths if needed
- **Result presentation**: Categorized summary table — files grouped by category (documentation, config, results, code, figures, dependencies), each listing key files and brief content description
- **Missing files**: Warn + continue. Mark missing items in scan summary (e.g., "⚠ No README.md found") but proceed with outline generation using available information. Missing sections get placeholder markers

### Checkpoint Interaction Design
- **Interaction method**: Plain text display + confirmation. Output the outline as formatted text; user replies "ok" to proceed or describes modifications needed. AskUserQuestion is NOT used for outline review (too complex for option boxes)
- **Modification loop**: On user feedback, directly modify the outline and re-display for confirmation. Loop until user is satisfied — no iteration limit
- **Generation cascade**: Layer-by-layer full generation. All H1 first → user confirms → all H2 for all sections → user confirms → all H3 → user confirms. User sees the full picture at each layer before judging
- **Detail level**: Title + one-sentence description at each level. Concise but sufficient for judging structure

### Outline Generation Logic
- **H1 source**: Journal template (if specified) provides standard section structure, adjusted based on repo content. Default to generic IMRaD (Introduction, Methods, Results and Discussion, Conclusion) when no journal specified
- **H2/H3 source**: Read actual file contents (README, config files, result files) to generate more accurate sub-headings. H1 only needs directory structure
- **Mapping rules**: Defined in `references/repo-patterns.md` — file categories map to paper sections (documentation → Introduction, config → Methodology, results → Results, etc.)
- **Source annotation**: H2/H3 entries annotated with inference source — e.g., `← from: config.yaml, results/table1.csv` — so user understands the derivation

### repo-patterns.md Design
- **Content**: Two parts — (1) file type identification patterns (glob → category → priority table), (2) category to paper section mapping rules table
- **Format**: Markdown tables for both parts, with Notes columns for rationale
- **Scope**: Python ML projects only (v2.0). Non-Python support deferred per ADVR-01
- **Size**: ~100 lines, single flat file. Lean reference for runtime loading, similar in scope to `references/bilingual-output.md`

### Claude's Discretion
- Exact file patterns in repo-patterns.md (which globs to include)
- How to handle ambiguous file categories (e.g., a Jupyter notebook that contains both code and results)
- Skill frontmatter field values (triggers, tools, references)
- Whether to include a "Scan Summary" confirmation step before proceeding to H1 generation
- Workflow step naming and sub-step distribution within ~300-line Skill budget

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Convention files
- `references/skill-conventions.md` — Authoritative Skill structure rules, ~300-line budget, AskUserQuestion enforcement, bilingual eligibility, mode taxonomy
- `references/skill-skeleton.md` — Copyable skeleton template for new Skill creation

### Journal template
- `references/journals/ceus.md` — CEUS journal contract; repo-to-paper Skill must be able to load journal templates for H1 structure

### Bilingual spec
- `references/bilingual-output.md` — Bilingual output format specification (Phase 13). Repo-to-paper Skill is bilingual-eligible (produces academic text)

### Existing multi-phase Skill (pattern reference)
- `.claude/skills/experiment-skill/SKILL.md` — Two-phase workflow with user confirmation checkpoint between phases. Pattern reference for checkpoint design

### Requirements
- `.planning/REQUIREMENTS.md` — REPO-01 (scan), REPO-02 (H1), REPO-03 (H2), REPO-05 (H3), REPO-08 (repo-patterns.md)

### State context
- `.planning/STATE.md` — Known risks: Skill line budget may need flexibility (~320 lines estimated), repo scan calibrated for Python ML only

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- `references/bilingual-output.md` — Single flat reference file pattern (~100 lines). Direct model for `repo-patterns.md` file structure
- `experiment-skill/SKILL.md` — Two-phase workflow with user confirmation between phases. Pattern for multi-checkpoint Skill design
- `references/journals/ceus.md` — Journal template format. Repo-to-paper needs to load this for H1 structure when CEUS is specified

### Established Patterns
- Reference files in `references/` are single flat files or entrypoint + leaf. `repo-patterns.md` should be a single flat file (no leaf hierarchy needed)
- Skills use `Read` tool to load reference files at runtime. Keep `repo-patterns.md` concise for context efficiency
- Skill line budget: ~300 lines. STATE.md notes this may need flexibility for repo-to-paper (~320 estimated)
- All Skills follow `skill-skeleton.md` structure with YAML frontmatter

### Integration Points
- Repo-to-paper Skill will be created at `.claude/skills/repo-to-paper-skill/SKILL.md`
- `references/repo-patterns.md` will be loaded as a required reference by the Skill
- Phase 15 (literature integration) depends on H2 outline being generated here
- Phase 16 (body generation) depends on H3 outline being generated here

</code_context>

<specifics>
## Specific Ideas

No specific requirements — decisions above are clear enough for standard implementation.

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope.

</deferred>

---

*Phase: 14-repo-to-paper-core-structure*
*Context gathered: 2026-03-18*
