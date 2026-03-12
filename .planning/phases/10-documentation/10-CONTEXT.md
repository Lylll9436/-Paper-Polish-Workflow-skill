# Phase 10: Documentation - Context

**Gathered:** 2026-03-12
**Status:** Ready for planning

<domain>
## Phase Boundary

Create a complete bilingual README.md at the repository root that enables any user to install and start using the 11-Skill paper-polish workflow suite. Covers: installation, Skill inventory table, quick-start workflows, and a contribution (issue feedback) guide. New Skills and other capabilities are out of scope for this phase.

</domain>

<decisions>
## Implementation Decisions

### Bilingual Structure
- Two parallel halves: complete English version in the upper half, complete Chinese version in the lower half
- English first, Chinese second
- Each half is fully self-contained and symmetric — same sections, same structure
- Section headings in each half use their own language (English headings in upper half, Chinese headings in lower half)
- Skill inventory table, installation steps, and quick-start workflow examples are fully duplicated in both halves

### README File Structure
- Single file: `README.md` at repository root — GitHub auto-renders it
- Contribution guide is a small embedded section at the end of each half (not a separate CONTRIBUTING.md)
- Contribution guide content: issue feedback guidance only (bug reports, feature requests) — not a full PR/Skill-authoring guide
- Page header: project name + one-sentence description + GitHub badges (license, etc.)

### Skill Inventory Table
- 3 columns: Skill name | Trigger examples (Chinese + English) | One-line description
- Skills grouped by function:
  - **Writing workflow group**: translation-skill, polish-skill, de-ai-skill, reviewer-simulation-skill
  - **Support tools group**: abstract-skill, cover-letter-skill, experiment-skill, caption-skill, logic-skill, literature-skill, visualization-skill
- literature-skill row gets a ⚠️ annotation: `⚠️ Requires Semantic Scholar MCP`
- MCP setup details covered in the installation section, not in the table

### Installation Approach
- One-liner instruction: give the repository GitHub URL to Claude Code and ask it to install the Skills
- No manual copy steps — Claude Code handles cloning and copying SKILL.md files to `.claude/skills/`
- Installation section briefly explains what Claude Code does: copies each Skill directory to `.claude/skills/`
- MCP setup for literature-skill: one short paragraph with step-by-step Claude Code settings instructions

### Quick Start Workflows
- Three independent scenario examples (each with step-by-step trigger commands + one-sentence explanation per step):
  1. **Paper submission chain**: translate → polish → de-ai → reviewer (core 4-Skill chain)
  2. **Figure/table assistance**: caption-skill + visualization-skill (support tools for experimental figures)
  3. **Literature search**: literature-skill → generate BibTeX → abstract-skill (research support chain)
- Each step shown as: trigger keyword + dash + what it does in one sentence
- No full example prompts or sample outputs — commands + descriptions only
- No "prerequisites" block before quick-start; installation section covers setup

### Claude's Discretion
- Exact badge selection and badge styling
- Spacing and visual breaks between sections
- Whether to add a table of contents at the top
- Exact wording of one-line Skill descriptions (can draw from SKILL.md descriptions)
- Order of sections within each half (installation → inventory → quick start → contribution)

</decisions>

<specifics>
## Specific Ideas

- Installation UX: "Paste the GitHub repo URL to Claude Code and say 'please install these skills'" — frictionless, no manual steps
- Skill table trigger examples should show both Chinese and English triggers side by side (e.g., "polish-skill / 润色这段英文") to immediately convey bilingual capability
- The three quick-start chains mirror the three use-case groups identified during project design: writing, figure/data, research support

</specifics>

<code_context>
## Existing Code Insights

### Reusable Assets
- `.claude/skills/*/SKILL.md` — each file's `description` field and `triggers.examples` field can be directly lifted for the inventory table rows and trigger examples
- `references/skill-conventions.md` — defines the skill skeleton; can reference it in contribution guide if needed
- `references/skill-skeleton.md` — available as a starting point reference

### Established Patterns
- All Skills follow bilingual trigger convention (Chinese + English examples in frontmatter) — inventory table can directly reuse these
- literature-skill is the only Skill with an external dependency (Semantic Scholar MCP) — all others are pure markdown with no setup

### Integration Points
- README lives at repo root — no integration with `.claude/skills/` structure needed
- Installation instructions reference `.claude/skills/` as the target directory

</code_context>

<deferred>
## Deferred Ideas

None — discussion stayed within phase scope.

</deferred>

---

*Phase: 10-documentation*
*Context gathered: 2026-03-12*
