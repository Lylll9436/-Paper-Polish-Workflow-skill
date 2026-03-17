---
phase: 12
slug: askuserquestion-fix
status: draft
shadcn_initialized: false
preset: none
created: 2026-03-17
---

# Phase 12 — UI Design Contract

> Interaction contract for a non-frontend phase. Phase 12 rewrites a Markdown Skill file (`paper-polish-workflow/SKILL.md`) that defines CLI-based interaction patterns via Claude Code's AskUserQuestion tool. There is no browser-rendered UI, no CSS, and no component library. The design contract focuses on copywriting and interaction patterns embedded in the Skill file.

---

## Design System

| Property | Value |
|----------|-------|
| Tool | none |
| Preset | not applicable |
| Component library | none — Claude Code Skills are Markdown files, not rendered UI |
| Icon library | not applicable |
| Font | not applicable — output is terminal text controlled by Claude Code |

**Rationale:** This project is a Claude Code Skill suite. Skills are `.md` files loaded by Claude Code at runtime. The "interface" is the Claude Code CLI terminal, which the Skill author does not control visually. All visual rendering (font, color, spacing) is determined by the user's terminal and Claude Code client.

---

## Spacing Scale

Not applicable. Phase 12 produces no rendered visual output. The Skill file is a Markdown document; its internal formatting follows `skill-conventions.md` line budget and section structure, not a pixel-based spacing scale.

---

## Typography

Not applicable. The output medium is Claude Code's terminal. The Skill author controls content and structure of text, not its rendered font, size, or weight.

---

## Color

Not applicable. No visual rendering is under the Skill author's control. Terminal output styling is handled by the Claude Code client.

---

## Copywriting Contract

This is the primary design contract for Phase 12. The rewritten `paper-polish-workflow/SKILL.md` defines user-facing interaction copy through AskUserQuestion calls and workflow prompts. All copy below must appear in the final Skill file.

### AskUserQuestion Interaction Copy

The Skill file must contain exactly one concise AskUserQuestion pseudocode example using generic placeholders. The canonical format is:

```
AskUserQuestion({
  question: "Which expression do you prefer for [sentence function]?",
  options: [
    { label: "[Expression A]", description: "[full sentence with expression A]" },
    { label: "[Expression B]", description: "[full sentence with expression B]" },
    { label: "[Expression C]", description: "[full sentence with expression C]" }
  ]
})
```

**Rules for AskUserQuestion copy in the Skill file:**
- Use generic placeholders (`[Expression A]`, `[Expression B]`), never hardcoded academic content
- One example only — do not repeat the pattern in multiple workflow steps
- No `questions` array wrapping — single question per call
- No `multiple: false` field — not part of convention format
- Question text must be a specific question about a specific decision, not a vague prompt

### Workflow Step Labels

The 4 workflow steps must use these exact headings:

| Step | Heading | Source |
|------|---------|--------|
| 1 | `### Step 1: Collect Context` | CONTEXT.md decision |
| 2 | `### Step 2: Structure & Logic Confirmation` | CONTEXT.md decision |
| 3 | `### Step 3: Expression Polish & Consistency` | CONTEXT.md decision |
| 4 | `### Step 4: Output` | CONTEXT.md decision |

### Mode Labels

| Mode | Label | Default | Description Copy |
|------|-------|---------|-----------------|
| interactive | `interactive` | Yes | Full 4-step flow with user confirmation at each decision point |
| guided | `guided` | No | Multi-pass with confirmation at key checkpoints only |
| direct | `direct` | No | Single-pass polish using defaults; skip AskUserQuestion |
| batch | `batch` | No | Same operation applied across multiple sections sequentially |

### Fallback Copy

The Skill file must include a `## Fallbacks` section with this exact table:

| Scenario | Fallback Copy |
|----------|---------------|
| Structured Interaction unavailable | "Ask 1-3 plain-text questions covering highest-impact gaps; do not block workflow" |
| Reference file missing | "Log the missing file, proceed with reduced capability, warn the user" |
| Target journal not specified | "Ask once; if declined, use general academic style" |
| PDF reference paper unreadable | "Ask user to paste relevant excerpts instead" |

### Trigger Copy (Frontmatter)

Bilingual trigger examples that must appear in `triggers.examples`:

| Language | Trigger Phrases |
|----------|----------------|
| English | "Polish my paper section by section", "Help me revise my introduction step by step", "Guide me through polishing this draft" |
| Chinese | "润色论文", "精修论文", "帮我逐步润色这篇论文" |

### Purpose Copy

The `## Purpose` section must communicate these three ideas in 2-3 sentences:
1. Top-down systematic workflow (structure before expression)
2. User confirmation at each decision point
3. Reference-driven expression selection from academic patterns

### Edge Case Copy

The `## Edge Cases` table must address at minimum these scenarios:

| Scenario | Handling Description |
|----------|---------------------|
| Unprofessional word flagged | Present 2-3 alternatives via AskUserQuestion; accept if user insists |
| Section too long for single pass | Split into paragraph-level sub-passes; maintain cross-paragraph coherence |
| No journal specified | Default to general academic style; note in output that no journal-specific formatting was applied |
| Mixed language input | Detect dominant language; ask user to confirm target language before polishing |
| Reference paper provided as PDF | Use Read tool to load PDF; extract relevant style patterns for expression matching |

---

## Registry Safety

| Registry | Blocks Used | Safety Gate |
|----------|-------------|-------------|
| not applicable | none | not applicable — no component registry; project is Markdown files only |

---

## Phase-Specific Interaction Contract

Since standard visual design dimensions do not apply, this section defines the interaction contract that downstream consumers (planner, executor, auditor) must enforce.

### Structural Contract

| Property | Constraint | Verification |
|----------|-----------|--------------|
| Total file length | 300 lines maximum | `wc -l paper-polish-workflow/SKILL.md` <= 300 |
| Workflow steps | Exactly 4 `### Step` headings | Count `### Step` headers = 4 |
| Tool names | Zero occurrences of `mcp_` prefix | `grep -c 'mcp_' paper-polish-workflow/SKILL.md` = 0 |
| AskUserQuestion | At least 1 occurrence | `grep -c 'AskUserQuestion' paper-polish-workflow/SKILL.md` >= 1 |
| Correct tool names | Read, Write, Edit all present | `grep -cE '\bRead\b\|\bWrite\b\|\bEdit\b' paper-polish-workflow/SKILL.md` >= 1 |
| Frontmatter | Valid YAML with `---` delimiters | `head -50` contains required fields |
| Section order | Matches skill-skeleton.md exactly | Manual: Purpose, Trigger, Modes, References, Ask Strategy, Workflow, Output Contract, Edge Cases, Fallbacks |
| Bilingual triggers | Both English and Chinese in frontmatter | `grep -c '润色' paper-polish-workflow/SKILL.md` >= 1 |

### Sub-Step Preservation Contract

Every current workflow capability must map to the new 4-step structure. The executor must verify these sub-operations are present somewhere in the rewritten file:

| Sub-Operation | Required In Step | Source |
|---------------|-----------------|--------|
| Load references (expression-patterns, journal) | Step 1 | Current Phase 0 |
| Identify target journal | Step 1 | Current Phase 0 |
| Read input file | Step 1 | Current Phase 0 |
| Extract key numbers/claims | Step 1 | Current Phase 0 |
| Present structure table | Step 2 | Current Step 1 |
| Confirm logic chain | Step 2 | Current Step 2 |
| Expression options via AskUserQuestion | Step 3 | Current Step 3 |
| Reference paper consultation (on demand) | Step 3 | Current Step 4 |
| Journal style check | Step 3 | Current Step 4.5 |
| Repetition and coherence pass | Step 3 | Current Step 5 |
| Cross-section consistency | Step 3 | Current Step 5.5 |
| Highlights generation (if journal requires) | Step 4 | Current Step 5.7 |
| Read-aloud suggestion | Step 4 | Current Step 6 |
| Write to file | Step 4 | Current Step 6 |
| Report word count | Step 4 | Current Step 6 |

---

## Checker Sign-Off

- [ ] Dimension 1 Copywriting: PASS — AskUserQuestion example, trigger copy, fallback copy, mode labels, edge case copy all present and prescriptive
- [ ] Dimension 2 Visuals: NOT APPLICABLE — non-frontend phase, Markdown Skill file only
- [ ] Dimension 3 Color: NOT APPLICABLE — no rendered visual output
- [ ] Dimension 4 Typography: NOT APPLICABLE — terminal output not under Skill author control
- [ ] Dimension 5 Spacing: NOT APPLICABLE — Markdown file, no pixel-based layout
- [ ] Dimension 6 Registry Safety: NOT APPLICABLE — no component registry

**Approval:** pending

---

*Phase: 12-askuserquestion-fix*
*UI-SPEC created: 2026-03-17*
*Source: CONTEXT.md decisions + RESEARCH.md code examples + skill-conventions.md patterns*
