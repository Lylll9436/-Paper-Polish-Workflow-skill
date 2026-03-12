# Roadmap: Paper Polish Workflow

## Overview

This roadmap delivers a complete academic writing AI tool suite as Claude Code Skills. The journey starts with shared reference materials and Skill conventions (the foundation everything else depends on), moves through core writing capabilities (translation, polishing, de-AI, review), then builds section-specific and support Skills, and finishes with project documentation. Each phase delivers independently usable Skills that produce submission-ready output.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [x] **Phase 1: Reference Libraries** - Build shared expression patterns, anti-AI patterns, and CEUS journal template
- [x] **Phase 2: Skill Conventions** - Establish Skill template pattern, YAML frontmatter format, and progressive disclosure rules
- [ ] **Phase 3: Translation Skill** - Chinese-to-English academic translation with LaTeX output
- [ ] **Phase 4: Polish Skill** - Redesigned English polishing with flexible guided and quick-fix modes
- [x] **Phase 5: De-AI Skill** - AI detection pattern rewriting to reduce AI traces while maintaining quality
- [ ] **Phase 6: Reviewer Simulation Skill** - Paper review from reviewer perspective with actionable feedback
- [x] **Phase 7: Abstract and Experiment Skills** - Abstract generation and experiment analysis/discussion generation (completed 2026-03-12)
- [ ] **Phase 8: Figure/Table and Logic Skills** - Caption generation and cross-section logic verification
- [ ] **Phase 9: Literature and Support Skills** - Semantic Scholar literature search, cover letter, and visualization recommendation
- [ ] **Phase 10: Documentation** - Project README with installation, Skill inventory, quick start, and contribution guide

## Phase Details

### Phase 1: Reference Libraries
**Goal**: Shared reference materials exist and are structured for on-demand loading by all downstream Skills
**Depends on**: Nothing (first phase)
**Requirements**: FNDN-01, FNDN-02, FNDN-03
**Success Criteria** (what must be TRUE):
  1. Expression patterns file covers sentence starters, transitions, hedging, results reporting, and conclusions with bilingual (Chinese/English) examples
  2. CEUS journal template file exists with consistent section headers, formatting rules, and style guidelines that any Skill can load via Read tool
  3. Anti-AI patterns file contains a high-frequency AI vocabulary blacklist with human-sounding alternatives organized by category
  4. All reference files are under `references/` directory and loadable independently (no file exceeds context-friendly size)
**Plans**: 2 planned

Plans:
- [x] 01-01: Restructure expression references for on-demand loading
- [x] 01-02: Build CEUS and anti-AI reference contracts

### Phase 2: Skill Conventions
**Goal**: A documented Skill template pattern exists that all subsequent Skills follow, ensuring consistency across the entire suite
**Depends on**: Phase 1
**Requirements**: FNDN-04
**Success Criteria** (what must be TRUE):
  1. Skill conventions document defines YAML frontmatter format with required fields (name, triggers, tools, references)
  2. Conventions specify the ~300 line limit and progressive disclosure pattern (lean SKILL.md that loads references on-demand)
  3. Conventions include tool usage rules (which tools are available, when to use AskUserQuestion vs. automatic flow)
  4. A concrete example Skill skeleton exists that future phases can copy as starting point
**Plans**: 1 planned

Plans:
- [x] 02-01: Define canonical skill conventions and skeleton

### Phase 3: Translation Skill
**Goal**: Users can translate Chinese academic drafts into polished English academic text ready for journal submission
**Depends on**: Phase 2
**Requirements**: CORE-01
**Success Criteria** (what must be TRUE):
  1. User can invoke the Translation Skill with a Chinese draft (file path or pasted text) and receive English academic output in LaTeX format
  2. Translation preserves technical terminology accurately and uses expression patterns from the shared reference library
  3. Output follows CEUS journal style when the user specifies CEUS as target journal
  4. Skill follows the conventions established in Phase 2 (YAML frontmatter, under 300 lines, progressive disclosure)
**Plans**: 1 planned

Plans:
- [ ] 03-01-PLAN.md — Author and validate the Translation Skill SKILL.md

### Phase 4: Polish Skill
**Goal**: Users can polish English academic text through a flexible workflow that adapts to their needs (quick fix or guided multi-pass)
**Depends on**: Phase 2
**Requirements**: CORE-02
**Success Criteria** (what must be TRUE):
  1. User can invoke quick-fix mode for direct polishing of a specific passage without multi-step workflow
  2. User can invoke guided mode that progresses through structure, logic, and expression passes with feedback at each stage
  3. Polish Skill loads expression patterns on-demand and applies journal-specific style rules when a target journal is specified
  4. The redesigned flow replaces the existing rigid 6-step SKILL.md entirely
  5. Skill handles both full-paper and section-level polishing gracefully
**Plans**: 1 plan

Plans:
- [ ] 04-01-PLAN.md — Author and validate the Polish Skill SKILL.md

### Phase 5: De-AI Skill
**Goal**: Users can detect and rewrite AI-generated patterns in their text to reduce detection scores while preserving academic quality
**Depends on**: Phase 1 (anti-AI patterns), Phase 2 (Skill conventions)
**Requirements**: CORE-03
**Success Criteria** (what must be TRUE):
  1. Skill identifies specific AI-generated patterns in input text (highlighting which phrases/structures trigger detection)
  2. Skill rewrites flagged passages using alternatives from the anti-AI patterns reference library
  3. Rewritten text maintains academic quality and meaning (not just synonym substitution)
  4. User can review before/after comparison of each rewritten passage
**Plans**: 1 plan

Plans:
- [x] 05-01-PLAN.md -- Author and validate the De-AI Skill SKILL.md

### Phase 6: Reviewer Simulation Skill
**Goal**: Users can get a simulated peer review of their paper that identifies weaknesses and provides actionable improvement suggestions
**Depends on**: Phase 2
**Requirements**: CORE-04
**Success Criteria** (what must be TRUE):
  1. Skill reads a full paper (or sections) and produces structured reviewer feedback covering novelty, methodology, writing quality, and presentation
  2. Each identified weakness comes with a specific, actionable improvement suggestion (not just "this needs improvement")
  3. Review output follows conventions of real journal reviews (major concerns, minor concerns, questions for authors)
  4. Skill can target review criteria to a specific journal when specified (e.g., CEUS relevance criteria)
**Plans**: 1 plan

Plans:
- [ ] 06-01-PLAN.md -- Author and validate the Reviewer Simulation Skill SKILL.md

### Phase 7: Abstract and Experiment Skills
**Goal**: Users can generate submission-ready abstracts and transform experiment results into discussion paragraphs
**Depends on**: Phase 2
**Requirements**: SECT-01, SECT-03
**Success Criteria** (what must be TRUE):
  1. Abstract Skill generates abstracts following the 5-sentence formula (contribution, difficulty, method, evidence, key result)
  2. Abstract Skill can optimize an existing abstract by restructuring it to the formula while preserving content
  3. Experiment analysis Skill accepts result data (tables, statistics) and identifies significant patterns and trends
  4. Experiment analysis Skill generates discussion paragraphs that connect findings to research questions and existing literature
**Plans**: 2 plans

Plans:
- [ ] 07-01-PLAN.md — Author and validate the Abstract Skill SKILL.md (SECT-01)
- [ ] 07-02-PLAN.md — Author and validate the Experiment Skill SKILL.md (SECT-03)

### Phase 8: Figure/Table and Logic Skills
**Goal**: Users can generate optimized captions for figures/tables and verify logical consistency across paper sections
**Depends on**: Phase 2
**Requirements**: SECT-02, SECT-04
**Success Criteria** (what must be TRUE):
  1. Caption Skill generates or optimizes figure/table captions with geography-specific awareness (study area descriptions, CRS notation, legend details)
  2. Captions follow target journal formatting requirements when a journal template is specified
  3. Logic verification Skill traces argument chains across sections and identifies gaps, contradictions, or unsupported claims
  4. Logic verification output provides specific section/paragraph references for each identified issue
**Plans**: 1 plan

Plans:
- [ ] 08-01: TBD
- [ ] 08-02: TBD

### Phase 9: Literature and Support Skills
**Goal**: Users can search literature via Semantic Scholar, generate cover letters, and get visualization recommendations
**Depends on**: Phase 2
**Requirements**: SUPP-01, SUPP-02, SUPP-03
**Success Criteria** (what must be TRUE):
  1. Literature Skill uses Semantic Scholar MCP to search papers by topic, verify citation accuracy, and generate BibTeX entries
  2. Literature Skill handles MCP unavailability gracefully (clear error message, fallback guidance)
  3. Cover letter Skill generates submission-ready letters based on paper content and target journal requirements
  4. Visualization Skill recommends chart types for experimental data with geography-specific options (choropleth maps, spatial scatter plots, hot spot maps)
**Plans**: 1 plan

Plans:
- [ ] 09-01: TBD
- [ ] 09-02: TBD
- [ ] 09-03: TBD

### Phase 10: Documentation
**Goal**: The project has a complete bilingual README that enables any user to install and start using the Skill suite
**Depends on**: Phase 9 (all Skills complete)
**Requirements**: DOCS-01
**Success Criteria** (what must be TRUE):
  1. README contains clear installation instructions (clone repo, copy to .claude/skills/)
  2. README includes a complete Skill inventory table with name, trigger command, and one-line description for each Skill
  3. README provides a quick start guide showing a real workflow (e.g., translate -> polish -> de-AI -> review)
  4. README is bilingual (English and Chinese sections)
**Plans**: 1 plan

Plans:
- [ ] 10-01: TBD

## Progress

**Execution Order:**
Phases execute in numeric order: 1 -> 2 -> 3 -> 4 -> 5 -> 6 -> 7 -> 8 -> 9 -> 10

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Reference Libraries | 2/2 | Complete | 2026-03-11 |
| 2. Skill Conventions | 1/1 | Complete | 2026-03-11 |
| 3. Translation Skill | 0/1 | Not started | - |
| 4. Polish Skill | 0/1 | Not started | - |
| 5. De-AI Skill | 1/1 | Complete | 2026-03-12 |
| 6. Reviewer Simulation Skill | 0/1 | Not started | - |
| 7. Abstract and Experiment Skills | 2/2 | Complete   | 2026-03-12 |
| 8. Figure/Table and Logic Skills | 0/2 | Not started | - |
| 9. Literature and Support Skills | 0/3 | Not started | - |
| 10. Documentation | 0/1 | Not started | - |
