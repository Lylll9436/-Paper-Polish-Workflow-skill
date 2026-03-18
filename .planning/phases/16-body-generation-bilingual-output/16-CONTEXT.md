# Phase 16: Body Generation & Bilingual Output - Context

**Gathered:** 2026-03-18
**Status:** Ready for planning

<domain>
## Phase Boundary

User gets complete section body text generated from the confirmed H3 outline, with `[SOURCE: file:line]` annotations for repo-derived claims, CEUS journal formatting, and paragraph-by-paragraph bilingual comparison. This phase adds Step 5 to the existing `repo-to-paper-skill`. Scope: body text generation only — outline generation is Phase 14, literature collection is Phase 15, existing Skills bilingual update is Phase 17.

</domain>

<decisions>
## Implementation Decisions

### Session Continuity
- **Step 5 auto-continues** in the same session after H3 confirmation — no separate invocation needed
- When `.paper-refs/` is missing (Step 2.5 was skipped / MCP unavailable): **continue generating**, all citation positions use `[CITATION NEEDED]` placeholders — do not block
- Step 5 uses the **H3 outline already in memory** — does not re-read `paper_outline.md`
- **Before generating**, Step 5 displays all H1 sections via `AskUserQuestion` (multiSelect) for user to select which sections to generate in this session

### Output Format & File Organization
- **LaTeX (.tex)** output format — CEUS-ready for direct submission
- **Per-H1 files**: one `.tex` file per H1 section (e.g., `introduction.tex`, `methods.tex`, `results.tex`)
- **Directory**: `{repo_path}/.paper-output/` — alongside `.paper-refs/` in the experiment repo root
- Bilingual uses **LaTeX comment format** (`% --- Paragraph N ---` + `%` Chinese lines before English) per `references/bilingual-output.md`
- After all selected sections complete, auto-generate `references.bib` in `.paper-output/`

### Generation Pacing & Confirmation Granularity
- **Section-by-section**: generate one H1 section, then wait for confirmation before the next
- **Confirmation method**: `AskUserQuestion` with options (Confirm / Modify / Skip) — distinct from outline review which uses plain text (Phase 14 decision for outline; body sections are more complex)
- **Modification loop**: user describes changes → regenerate entire section → re-display via AskUserQuestion → loop until confirmed
- After all selected sections confirmed, generate `references.bib` and display completion summary

### Citation Integration
- **`\cite{key}` inline embedding**: all citations written as `\cite{citationkey}` directly in the paragraph
- **Strict source boundary**: `\cite{}` keys MUST only reference papers present in `.paper-refs/` files — no citations from Claude's background knowledge
- Content that needs a citation but has no supporting `.paper-refs/` entry → use `[CITATION NEEDED]` placeholder
- **Auto-generate `references.bib`**: read all `.paper-refs/*.md` files, extract all BibTeX blocks, deduplicate by citation key, write to `.paper-output/references.bib`

### Anti-Hallucination Rules
- **Repo-derived claims** (specific numbers, model names, dataset sizes, configurations): MUST include `[SOURCE: file:line]` annotation — never stated without repo evidence
- **Quantitative results** without supporting repo data: use `[RESULTS NEEDED]` placeholder (do not invent numbers)
- **Specific metric values** where the general metric is known but value is not: use `[EXACT VALUE: metric]`
- **Background/context prose** (general domain knowledge, problem framing): may be written as normal academic text — no placeholder required
- This follows REQUIREMENTS.md REPO-06 / REPO-07 success criteria exactly

### Claude's Discretion
- Exact paragraph structure and length within each section
- How to sequence H2 subsection content within the H1 section .tex file
- CEUS writing style enforcement (past tense for methods/results, cautious verbs, no "novel"/"groundbreaking")
- Whether to include `\section{}`, `\subsection{}`, `\subsubsection{}` LaTeX heading commands in output files

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Skill being modified
- `.claude/skills/repo-to-paper-skill/SKILL.md` — Current 4-step workflow (Steps 1-4 + Step 2.5); Step 5 will be appended after Step 4 (H3 confirmation)

### Bilingual spec
- `references/bilingual-output.md` — Bilingual format specification; for LaTeX output use `% --- Paragraph N ---` + `%` comment lines for Chinese. Paragraph markers REQUIRED for paragraph-level files.

### Journal template
- `references/journals/ceus.md` — CEUS formatting contract: section structure, writing style (past tense for methods/results, cautious verbs, avoid "novel"/"groundbreaking"), 8,000 word limit

### Convention files
- `references/skill-conventions.md` — Skill structure rules, ~300-line budget (body generation may need flexibility), AskUserQuestion enforcement
- `references/skill-skeleton.md` — Skeleton template for Skill modifications

### MCP / citation patterns (for references.bib generation)
- `.claude/skills/literature-skill/SKILL.md` — BibTeX citation key format (`firstAuthorLastnameLowercaseYYYYfirstKeyword`), field handling rules, anti-hallucination citation rule

### Prior phase context (upstream dependencies)
- `.planning/phases/14-repo-to-paper-core-structure/14-CONTEXT.md` — Outline confirmation interaction design (plain text + "ok" for outline; Step 5 uses AskUserQuestion per Phase 16 decision)
- `.planning/phases/15-literature-integration/15-CONTEXT.md` — `.paper-refs/` file format (Markdown cards with BibTeX blocks), Step 2.5 MCP fallback behavior

### Requirements
- `.planning/REQUIREMENTS.md` — REPO-06 (`[SOURCE: file:line]` annotations), REPO-07 (CEUS formatting), BILN-01 (bilingual paragraph-by-paragraph output)

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- `repo-to-paper-skill/SKILL.md` Steps 1-4 + Step 2.5 — Full existing workflow; Step 5 appends after Step 4. Modify `output_contract` to add `body_text` output type.
- `literature-skill/SKILL.md` BibTeX generation — Citation key format and field-handling rules reuse directly for `references.bib` generation
- `.paper-refs/{section}.md` files — Source for both `\cite{key}` keys and BibTeX entries in `references.bib`

### Established Patterns
- Outline review uses **plain text + user reply** (Phase 14 decision). Body section review uses **AskUserQuestion** (Phase 16 decision — explicitly different)
- Anti-hallucination: `[CITATION NEEDED]`, `[RESULTS NEEDED]`, `[EXACT VALUE: metric]` placeholder vocabulary already established in ROADMAP.md success criteria
- Bilingual LaTeX format: `% --- Paragraph N ---` line, then `%` Chinese comment lines, then English text (per `references/bilingual-output.md`)
- MCP unavailability → graceful degradation with placeholders (established in Step 2.5)
- Line budget ~300 lines (may need flexibility — Phase 14/15 already flagged this risk)

### Integration Points
- Step 5 appended to `repo-to-paper-skill/SKILL.md` after Step 4
- Reads `.paper-refs/` for citation keys and BibTeX
- Writes `.paper-output/{section}.tex` files and `.paper-output/references.bib`
- `output_contract` in SKILL.md frontmatter gains `body_text` type
- Phase 17 (bilingual update for existing Skills) is independent — does not modify repo-to-paper-skill

</code_context>

<specifics>
## Specific Ideas

No specific requirements — decisions above are sufficient for implementation.

</specifics>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope.

</deferred>

---

*Phase: 16-body-generation-bilingual-output*
*Context gathered: 2026-03-18*
