# Roadmap: Paper Polish Workflow

## Milestones

- ✅ **v1.0 Paper Polish Workflow** — Phases 1-10 (shipped 2026-03-13)
- 🚧 **v2.0 Repo-to-Paper & Bilingual Enhancement** — Phases 11-18 (in progress)

## Phases

<details>
<summary>✅ v1.0 Paper Polish Workflow (Phases 1-10) — SHIPPED 2026-03-13</summary>

- [x] Phase 1: Reference Libraries (2/2 plans) — completed 2026-03-11
- [x] Phase 2: Skill Conventions (1/1 plan) — completed 2026-03-11
- [x] Phase 3: Translation Skill (1/1 plan) — completed 2026-03-12
- [x] Phase 4: Polish Skill (1/1 plan) — completed 2026-03-12
- [x] Phase 5: De-AI Skill (1/1 plan) — completed 2026-03-12
- [x] Phase 6: Reviewer Simulation Skill (1/1 plan) — completed 2026-03-12
- [x] Phase 7: Abstract and Experiment Skills (2/2 plans) — completed 2026-03-12
- [x] Phase 8: Figure/Table and Logic Skills (2/2 plans) — completed 2026-03-12
- [x] Phase 9: Literature and Support Skills (3/3 plans) — completed 2026-03-12
- [x] Phase 10: Documentation (1/1 plan) — completed 2026-03-12

Full details: `.planning/milestones/v1.0-ROADMAP.md`

</details>

### 🚧 v2.0 Repo-to-Paper & Bilingual Enhancement (In Progress)

**Milestone Goal:** Enable automatic paper draft generation from experiment repositories with top-down structure-first workflow, standardize bilingual paragraph-by-paragraph comparison across Skills, and fix interactive question patterns.

- [x] **Phase 11: Convention & Tech Debt** - Resolve v1.0 debt and update conventions before any new authoring (completed 2026-03-17)
- [x] **Phase 12: AskUserQuestion Fix** - Fix P0 bug: paper-polish-workflow uses non-existent tool names (completed 2026-03-17)
- [x] **Phase 13: Bilingual Pattern Standardization** - Create shared bilingual output specification and reference file (completed 2026-03-17)
- [x] **Phase 14: Repo-to-Paper Core Structure** - Repo scanning, outline generation (H1/H2/H3), and user checkpoints (completed 2026-03-18)
- [x] **Phase 15: Literature Integration** - Semantic Scholar batch search at H2 stage with ref file output (completed 2026-03-18)
- [ ] **Phase 16: Body Generation & Bilingual Output** - Section body text with evidence annotations and bilingual comparison
- [ ] **Phase 17: Existing Skills Bilingual Update** - Add bilingual output mode to 7 existing Skills
- [ ] **Phase 18: Workflow Memory** - Skill records user workflow sequences and offers recommendations

## Phase Details

### Phase 11: Convention & Tech Debt
**Goal**: All v1.0 tech debt resolved and conventions updated so new authoring starts from a clean, consistent foundation
**Depends on**: Nothing (first v2.0 phase)
**Requirements**: DEBT-01, DEBT-02, DEBT-03, UXFIX-02
**Success Criteria** (what must be TRUE):
  1. `skill-conventions.md` documents the `required: []` escape hatch pattern with rationale and examples (logic-skill, visualization-skill referenced)
  2. `literature-skill` frontmatter uses capability category "External MCP" instead of vendor-specific "Semantic Scholar MCP"
  3. `cover-letter-skill` lists CEUS journal template in `required` instead of `leaf_hints`
  4. `skill-conventions.md` includes AskUserQuestion enforcement rule with invocation examples, per-Skill audit approach (not blanket mandate), and direct-mode exemption
**Plans**: 1 plan

Plans:
- [ ] 11-01-PLAN.md — Update conventions (escape hatch, AskUserQuestion enforcement, bilingual eligibility), sync skeleton, verify DEBT-02/DEBT-03

### Phase 12: AskUserQuestion Fix
**Goal**: paper-polish-workflow interactive questions work correctly using real Claude Code tool names
**Depends on**: Phase 11 (conventions must be updated first so fix aligns with new AskUserQuestion guidance)
**Requirements**: UXFIX-01
**Success Criteria** (what must be TRUE):
  1. `paper-polish-workflow/SKILL.md` references `AskUserQuestion` (not `mcp_question`) for all structured questions
  2. All tool references in `paper-polish-workflow/SKILL.md` use correct Claude Code tool names (`Read`, `Write`, `Edit`, not `mcp_read`/`mcp_write`)
  3. User invoking paper-polish-workflow receives structured AskUserQuestion prompts (with options) instead of plain dialogue text
**Plans**: 1 plan

Plans:
- [ ] 12-01-PLAN.md — Rewrite SKILL.md with correct tool names, skeleton-compliant structure, and 4-step workflow

### Phase 13: Bilingual Pattern Standardization
**Goal**: A single authoritative bilingual output specification exists that all Skills can reference for consistent paragraph-by-paragraph comparison
**Depends on**: Phase 11 (conventions updated with bilingual scope categories)
**Requirements**: BILN-02, BILN-03
**Success Criteria** (what must be TRUE):
  1. `references/bilingual-output.md` exists defining two format variants: LaTeX comment (for .tex output) and markdown blockquote (for .md output), with examples
  2. The bilingual reference file documents when to use each variant (output format determines variant, not Skill identity)
  3. Existing translation-skill bilingual pattern is verified as consistent with the new specification (no format divergence)
**Plans**: 1 plan

Plans:
- [ ] 13-01-PLAN.md — Create bilingual output specification with format variants, opt-out mechanism, and Phase 17 migration checklist

### Phase 14: Repo-to-Paper Core Structure
**Goal**: User can point to an experiment repo and get a complete H1/H2/H3 outline with approval checkpoints at each level
**Depends on**: Phase 13 (bilingual pattern must exist before repo-to-paper Skill references it)
**Requirements**: REPO-01, REPO-02, REPO-03, REPO-05, REPO-08
**Success Criteria** (what must be TRUE):
  1. User provides a repo path and the Skill automatically scans it, identifying README, configs, result files, and code with a structured summary
  2. User reviews and approves H1 (section) headings before the Skill proceeds to H2 generation
  3. User reviews and approves H2 (subsection) headings with detailed outlines before the Skill proceeds to H3 generation
  4. User reviews and approves H3 headings before the Skill proceeds to body generation
  5. `references/repo-patterns.md` exists as a maintainable reference file containing scan heuristics (file type patterns, section mapping rules) separate from Skill logic
**Plans**: 1 plan

Plans:
- [ ] 14-01-PLAN.md — Create repo-patterns.md reference file and repo-to-paper-skill SKILL.md with 4-step checkpoint workflow

### Phase 15: Literature Integration
**Goal**: By H2 completion, all section references are collected via Semantic Scholar and saved as ref files with metadata and abstracts
**Depends on**: Phase 14 (H2 section titles must exist before literature queries can be derived)
**Requirements**: REPO-04
**Success Criteria** (what must be TRUE):
  1. At H2 checkpoint completion, the Skill derives search queries from section titles and collects references via Semantic Scholar MCP
  2. Ref files are saved to `.paper-refs/` directory with title, abstract summary, BibTeX, and relevance note per reference
  3. When Semantic Scholar MCP is unavailable, the Skill inserts `[CITATION NEEDED]` placeholders and continues without blocking
**Plans**: 1 plan

Plans:
- [ ] 15-01-PLAN.md — Add Step 2.5 Literature Collection to repo-to-paper-skill with MCP batch search, ref file output, and graceful fallback

### Phase 16: Body Generation & Bilingual Output
**Goal**: User gets complete section body text with evidence annotations, journal formatting, and optional bilingual comparison
**Depends on**: Phase 15 (literature refs must be available for body generation), Phase 13 (bilingual pattern)
**Requirements**: REPO-06, REPO-07, BILN-01
**Success Criteria** (what must be TRUE):
  1. Generated body text includes `[SOURCE: file:line]` annotations on claims derived from repo data
  2. Output uses CEUS journal template formatting (section structure, heading conventions, citation style)
  3. Bilingual output mode produces paragraph-by-paragraph English + Chinese comparison following the standardized bilingual format from `references/bilingual-output.md`
  4. Quantitative claims without supporting repo data use explicit placeholders (`[RESULTS NEEDED]`, `[EXACT VALUE: metric]`) instead of hallucinated numbers
**Plans**: TBD

Plans:
- [ ] 16-01: TBD

### Phase 17: Existing Skills Bilingual Update
**Goal**: 7 existing Skills support bilingual paragraph-by-paragraph comparison output as an opt-in mode
**Depends on**: Phase 13 (bilingual pattern specification must be finalized and validated)
**Requirements**: BILN-04
**Success Criteria** (what must be TRUE):
  1. translation-skill, polish-skill, de-ai-skill, reviewer-skill, abstract-skill, experiment-skill, and logic-skill each support an opt-in bilingual output mode
  2. All 7 updated Skills use the format variants from `references/bilingual-output.md` (LaTeX comment for .tex, markdown blockquote for .md)
  3. Bilingual mode is off by default; existing behavior is unchanged when bilingual is not requested
**Plans**: TBD

Plans:
- [ ] 17-01: TBD

### Phase 18: Workflow Memory
**Goal**: Skills learn from user behavior and surface relevant workflow recommendations on future invocations
**Depends on**: Phase 12 (AskUserQuestion fix, since recommendations are presented via structured questions)
**Requirements**: UXFIX-03
**Success Criteria** (what must be TRUE):
  1. Skill execution records the user's workflow sequence (which Skills invoked in what order) to a project-level config file
  2. On next invocation, if a recognized workflow pattern is detected, the Skill offers a recommendation (e.g., "You usually run de-ai after polishing -- proceed?")
  3. User can dismiss or accept workflow recommendations without disrupting normal Skill operation
**Plans**: TBD

Plans:
- [ ] 18-01: TBD

## Progress

**Execution Order:**
Phases execute in numeric order: 11 -> 12 -> 13 -> 14 -> 15 -> 16 -> 17 -> 18

| Phase | Milestone | Plans Complete | Status | Completed |
|-------|-----------|----------------|--------|-----------|
| 1. Reference Libraries | v1.0 | 2/2 | Complete | 2026-03-11 |
| 2. Skill Conventions | v1.0 | 1/1 | Complete | 2026-03-11 |
| 3. Translation Skill | v1.0 | 1/1 | Complete | 2026-03-12 |
| 4. Polish Skill | v1.0 | 1/1 | Complete | 2026-03-12 |
| 5. De-AI Skill | v1.0 | 1/1 | Complete | 2026-03-12 |
| 6. Reviewer Simulation Skill | v1.0 | 1/1 | Complete | 2026-03-12 |
| 7. Abstract & Experiment Skills | v1.0 | 2/2 | Complete | 2026-03-12 |
| 8. Figure/Table & Logic Skills | v1.0 | 2/2 | Complete | 2026-03-12 |
| 9. Literature & Support Skills | v1.0 | 3/3 | Complete | 2026-03-12 |
| 10. Documentation | v1.0 | 1/1 | Complete | 2026-03-12 |
| 11. Convention & Tech Debt | 1/1 | Complete    | 2026-03-17 | - |
| 12. AskUserQuestion Fix | 1/1 | Complete    | 2026-03-17 | - |
| 13. Bilingual Pattern Standardization | 1/1 | Complete    | 2026-03-17 | - |
| 14. Repo-to-Paper Core Structure | 1/1 | Complete    | 2026-03-18 | - |
| 15. Literature Integration | 1/1 | Complete    | 2026-03-18 | - |
| 16. Body Generation & Bilingual Output | v2.0 | 0/TBD | Not started | - |
| 17. Existing Skills Bilingual Update | v2.0 | 0/TBD | Not started | - |
| 18. Workflow Memory | v2.0 | 0/TBD | Not started | - |
