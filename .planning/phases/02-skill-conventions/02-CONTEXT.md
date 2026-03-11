# Phase 2: Skill Conventions - Context

**Gathered:** 2026-03-11
**Status:** Ready for planning

<domain>
## Phase Boundary

Define the shared conventions that all downstream Claude Code Skills in this project must follow. This phase locks the Skill template contract, frontmatter requirements, reference-loading rules, interaction modes, fallback behavior, and the shape of an example skeleton that later phases can copy. It does not redesign any individual feature Skill in full.

</domain>

<decisions>
## Implementation Decisions

### Frontmatter contract
- Use a strict frontmatter contract for future Skills.
- Every Skill should include at least: `name`, `description`, `triggers`, `tools`, `references`, `input_modes`, and `output_contract`.
- `triggers` should be expressed as trigger intent plus example phrases, not as a flat keyword list only.
- `tools` should list the tools the Skill is allowed to depend on by default; detailed usage rules belong in the body.
- `input_modes` and `output_contract` should appear as short enums or summaries in frontmatter, with the operational detail explained in the body.

### Reference and progressive-disclosure rules
- If a Skill clearly knows which leaf reference module it needs, it may load that leaf file directly.
- If the Skill is not sure which leaf file fits, it should start from the stable overview file and choose from there.
- The `references` field should list stable entrypoint files plus optional hints about relevant leaf modules.
- `SKILL.md` may keep a moderate amount of inline guidance, but long reference tables and reusable content should live under `references/`.
- If the referenced resource does not match the current task, the default behavior is to stop and ask the user for clarification rather than silently guessing a fallback path.

### Interaction policy
- The default posture of future Skills should be ask first, then act.
- Any branch that materially changes output direction should be clarified before the main workflow proceeds.
- When structured interaction tools are unavailable, the default fallback is 1 to 3 plain-text questions covering only the highest-impact gaps.
- Skill conventions should explicitly support the modes `interactive`, `guided`, `direct`, and `batch`.

### Skill template skeleton
- Use a semi-templated Skill body: a required shared backbone plus room for skill-type-specific additions.
- The required section types should include: `Purpose`, `Trigger`, `Modes`, `References`, `Ask Strategy`, `Workflow`, `Output Contract`, `Edge Cases`, and `Fallbacks`.
- Workflow style may vary by Skill type: direct Skills can stay short, while guided Skills can use fuller step-by-step flow.
- Keep at least one minimal example strongly recommended in each Skill, especially for invocation shape or expected output shape.

### Claude's Discretion
- Exact YAML field ordering and naming style within the strict frontmatter contract.
- The exact line-budget rule wording for `SKILL.md` and how strongly the example skeleton enforces it.
- Whether a conventions document and example skeleton live in one file set or in separate but linked files.

</decisions>

<specifics>
## Specific Ideas

- The current legacy shell should be treated as a migration source, not as the final convention model.
- The old `paper-polish-workflow/SKILL.md` already shows why frontmatter must carry more contract information than `name` and `description` alone.
- The old shell's heavy reliance on `mcp_question` makes fallback rules a first-class convention topic rather than a per-Skill afterthought.
- The existing legacy shell is about 259 lines, which suggests a disciplined line budget is feasible if reusable reference content remains external.

</specifics>

<code_context>
## Existing Code Insights

### Reusable Assets
- `paper-polish-workflow/SKILL.md`: a concrete legacy shell that reveals current anti-patterns and migration needs, including thin frontmatter and tool coupling.
- `references/expression-patterns.md` plus `references/expression-patterns/`: now provide a stable shared reference contract that future Skill conventions should explicitly support.
- `references/anti-ai-patterns.md` plus `references/anti-ai-patterns/`: provide a second modular reference pattern that conventions can treat as canonical.
- `references/journals/ceus.md`: already acts as a stable journal contract and should be the model for future journal references in conventions.

### Established Patterns
- Shared references now follow a stable entrypoint plus optional leaf-module structure.
- The repo remains markdown-only with no build step, so conventions must stay human-readable and directly loadable.
- Public docs (`README`, `CONTRIBUTING`) already describe the modular reference layout, so Skill conventions should align with that published contract.

### Integration Points
- Phase 3 and onward will rely on the conventions defined here when creating independent Skills.
- Phase 4 and Phase 5 are especially sensitive to mode and fallback conventions because they need both guided and direct behavior.
- No repo-local `.claude/skills/` library of new Skills exists yet, so this phase defines the pattern before later phases populate it.

</code_context>

<deferred>
## Deferred Ideas

None - discussion stayed within the Phase 2 boundary.

</deferred>

---

*Phase: 02-skill-conventions*
*Context gathered: 2026-03-11*
