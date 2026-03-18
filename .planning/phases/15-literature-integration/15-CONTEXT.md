# Phase 15: Literature Integration - Context

**Gathered:** 2026-03-18
**Status:** Ready for planning

<domain>
## Phase Boundary

At H2 outline completion, automatically derive search queries from section titles and collect references via Semantic Scholar MCP. References are saved as per-section ref files with metadata and abstracts to `.paper-refs/` in the target repo. This phase modifies the existing repo-to-paper-skill to add a literature collection step. Body text generation is Phase 16; the existing standalone literature-skill is unaffected.

</domain>

<decisions>
## Implementation Decisions

### Integration Approach
- **Modify repo-to-paper-skill** to add a new Step 2.5: Literature Collection, inserted between H2 confirmation (Step 3) and H3 generation (Step 4)
- Step numbering uses decimal (2.5) to avoid renumbering existing steps
- **Line budget**: Allow exceeding the 300-line Skill convention. Phase 14 already flagged this risk; functional completeness takes priority
- **MCP fallback**: If Semantic Scholar MCP is unavailable at pre-flight, skip the entire Step 2.5. Insert `[CITATION NEEDED]` placeholders in the outline where references would appear, and continue to H3 generation without blocking

### Query Derivation
- **H1 + H2 combined query strategy**: Use H1 section title as broad topic context, combined with H2 subsection title and description to form targeted search queries
- Extract core terms from English titles/descriptions only — ignore Chinese translations in bilingual mode
- No user keyword supplementation needed; H2 titles and descriptions (already user-confirmed) contain sufficient domain vocabulary
- Query length target: 2-5 key terms per search

### Reference Volume
- **5-10 refs per H2 subsection**, driven by claim density — the number of distinct arguments/observations a section needs to make determines how many supporting references are collected
- Not a fixed count; Claude assesses how many viewpoints the section needs to support and adjusts accordingly
- For a typical 6-section paper with ~15 H2 subsections, expect 30-60 total references

### User Review
- **Automatic collection with progress display**: Each H2 completion shows a concise progress line: `✓ 1.1 Research Background: 8 refs found`
- Zero results: `⚠ 1.1 Research Background: 0 refs found` + mark as `[CITATION NEEDED]`, continue to next H2
- After all H2 sections processed, display a **summary table** showing all collected refs grouped by section (title, year, citations, associated H2)
- User confirms the summary to proceed to H3 generation. No interactive deletion/addition at this stage — user can manually edit `.paper-refs/` files later

### Ref File Organization
- **Per-section files** in `{repo_path}/.paper-refs/` directory: one file per H1 section (e.g., `introduction.md`, `methods.md`, `results.md`)
- Each file contains refs grouped by H2 subsection
- **Markdown card format** per ref entry:
  ```
  ### Author et al. (Year)
  **Title:** Full paper title
  **Authors:** Author1; Author2
  **Year:** YYYY | **Citations:** N
  **Relevance:** [H2 subsection label]
  One-sentence explanation of why this paper is relevant to this subsection.

  > Abstract summary (1-2 sentences from MCP data)

  ```bibtex
  @article{citationkey, ...}
  ```
  ```
- **Duplicate handling**: Allow same paper to appear in multiple section files. Each section file is self-contained. BibTeX deduplication deferred to Phase 16
- **Directory location**: `{repo_path}/.paper-refs/` — placed in the target experiment repo root, alongside the repo's own files

### Claude's Discretion
- Exact query formulation algorithm (how to combine H1 + H2 terms)
- Relevance filtering threshold (which papers from search results to keep vs discard)
- Abstract summary length (1-2 sentences extracted from MCP abstract data)
- How to handle MCP rate limiting if encountered during batch queries
- BibTeX citation key generation format (reuse literature-skill's `firstAuthorYYYYkeyword` pattern)

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Skill being modified
- `.claude/skills/repo-to-paper-skill/SKILL.md` — Current 4-step workflow; Step 2.5 will be inserted after Step 3 (H2 confirmation)

### MCP patterns to reuse
- `.claude/skills/literature-skill/SKILL.md` — Semantic Scholar MCP pre-flight check, search call patterns, abstract fetching, BibTeX generation format

### Convention files
- `references/skill-conventions.md` — Skill structure rules, line budget guidance, AskUserQuestion enforcement
- `references/skill-skeleton.md` — Skeleton template for Skill modifications

### Bilingual spec
- `references/bilingual-output.md` — Bilingual output format; ref file content is English-only but progress/summary display follows bilingual mode if active

### Requirements
- `.planning/REQUIREMENTS.md` — REPO-04 (literature integration at H2 stage)

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- `literature-skill/SKILL.md` Step 1: MCP pre-flight check pattern (`papers-search-basic` with `{"query": "test", "limit": 1}`) — reuse directly
- `literature-skill/SKILL.md` Step 3: Search call and abstract fetching pattern — adapt for batch usage
- `literature-skill/SKILL.md` Step 5: BibTeX generation from MCP data — reuse citation key format and field handling

### Established Patterns
- Reference files in `references/` are single flat Markdown files. `.paper-refs/` files follow the same Markdown convention
- MCP unavailability → refuse immediately (literature-skill pattern). For repo-to-paper, adapted to: skip step + placeholders instead of full refusal
- repo-to-paper-skill uses plain text display + user confirmation for outline review — same pattern for literature summary

### Integration Points
- repo-to-paper-skill Step 3 (H2 confirmation) → NEW Step 2.5 (Literature Collection) → Step 4 (H3 generation)
- `.paper-refs/` directory created at `{repo_path}/.paper-refs/` during Step 2.5
- Phase 16 (body generation) will read `.paper-refs/` section files to insert citations into body text

</code_context>

<specifics>
## Specific Ideas

- Reference count should be claim-driven: "每个章节需要提出多少观点，就需要多少论据来证明说的是对的" — the number of supporting references matches the number of arguments/claims the section makes
- STATE.md risk noted: "Semantic Scholar batch query performance at 20-30 calls untested" — plan should account for potential rate limiting

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope.

</deferred>

---

*Phase: 15-literature-integration*
*Context gathered: 2026-03-18*
