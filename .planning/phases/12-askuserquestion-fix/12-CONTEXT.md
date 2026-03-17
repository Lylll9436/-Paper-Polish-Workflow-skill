# Phase 12: AskUserQuestion Fix - Context

**Gathered:** 2026-03-17
**Status:** Ready for planning

<domain>
## Phase Boundary

Fix `paper-polish-workflow/SKILL.md` so all tool references use correct Claude Code tool names and the file structure matches `skill-conventions.md` / `skill-skeleton.md`. This is a P0 bug fix + structural alignment — no new features or workflow capabilities are added.

</domain>

<decisions>
## Implementation Decisions

### Restructuring Scope
- **Full restructure**: Fix all tool names AND restructure to match skill-skeleton.md structure
- Add standard frontmatter (triggers, tools, references, input_modes, output_contract)
- Add missing convention sections: Modes, Ask Strategy, Edge Cases, Fallbacks
- Compress from 347 lines to within 300-line budget
- One-shot delivery — no phased approach

### Workflow Step Consolidation
- Merge current 6 steps into 4 steps:
  1. Collect Context (pre-polish baseline + reference loading)
  2. Structure & Logic Confirmation (macro structure + per-sentence logic)
  3. Expression Polish & Consistency (expression selection via AskUserQuestion + repetition/coherence check + cross-section consistency)
  4. Output (final review + write to file)
- Step 4.5 (journal style check) and Step 5.7 (highlights) fold into relevant merged steps

### Tool Name Corrections
- `mcp_question` → `AskUserQuestion` (all occurrences, 8+)
- `mcp_read` → `Read`
- `mcp_write` → `Write`
- `mcp_edit` → `Edit`
- `mcp_look_at` → `Read` (Claude Code Read tool handles PDFs)

### Example Code Format
- Use pseudocode format matching skill-conventions.md style: `AskUserQuestion({ question:..., options:[...] })`
- Use generic placeholders (`[Expression A]`, `[Expression B]`) instead of hardcoded content
- One concise example in the workflow is sufficient

### PDF Tool Handling
- Replace `mcp_look_at` with `Read` — Claude Code's Read tool supports PDF files
- Keep reference paper consultation as a workflow feature, just with correct tool name

### Fallback Behavior
- Follow skill-conventions.md Fallback Rules pattern
- Structured interaction unavailable → fall back to 1-3 plain-text questions, don't block workflow
- Remove the old "If mcp_question is unavailable, use text-based table format" pattern

### Claude's Discretion
- Exact wording of the 4 merged workflow steps
- How to distribute sub-steps (journal check, highlights, cross-section) across the 4 main steps
- Frontmatter field values (triggers examples, tools list, references paths)
- Whether to keep the "Common Issue Handling" table or fold it into Edge Cases

</decisions>

<canonical_refs>
## Canonical References

**Downstream agents MUST read these before planning or implementing.**

### Convention files
- `references/skill-conventions.md` — Authoritative rules for Skill structure, AskUserQuestion enforcement (line 190+), bilingual eligibility, line budget
- `references/skill-skeleton.md` — Copyable skeleton template that the restructured file must match

### File being modified
- `paper-polish-workflow/SKILL.md` — The target file (current state: 347 lines, wrong tool names, non-standard structure)

### Journal references
- `references/journals/ceus.md` — CEUS journal contract, referenced by the Skill's workflow

</canonical_refs>

<code_context>
## Existing Code Insights

### Reusable Assets
- `references/skill-skeleton.md` — Direct template for restructuring; copy structure, fill with paper-polish content
- `references/skill-conventions.md` lines 190-224 — AskUserQuestion enforcement section with good/bad example format

### Established Patterns
- All other Skills in `.claude/skills/` follow the skeleton structure — paper-polish-workflow should match
- Phase 11 established AskUserQuestion enforcement as per-Skill audit with direct/batch mode exemptions

### Integration Points
- `paper-polish-workflow/SKILL.md` is loaded by Claude Code when user triggers the skill
- References `references/journals/ceus.md` at runtime for journal-specific formatting
- References `references/expression-patterns.md` for academic expression patterns

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

*Phase: 12-askuserquestion-fix*
*Context gathered: 2026-03-17*
