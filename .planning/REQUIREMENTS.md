# Requirements: Paper Polish Workflow

**Defined:** 2026-03-17
**Core Value:** Every Skill must produce output that is directly usable in a real paper submission

## v2.0 Requirements

Requirements for v2.0 milestone: Repo-to-Paper & Bilingual Enhancement.

### Repo-to-Paper

- [x] **REPO-01**: User can point Skill to an experiment repo and get automatic full scan identifying valuable files (README, configs, results, code)
- [x] **REPO-02**: User can generate H1 (section) headings from repo analysis and review/approve before proceeding
- [x] **REPO-03**: User can generate H2 (subsection) headings with detailed outlines and review/approve before proceeding
- [ ] **REPO-04**: At H2 completion, all section references are collected via Semantic Scholar with metadata + abstracts and saved to ref files
- [x] **REPO-05**: User can generate H3 headings and review/approve before proceeding
- [ ] **REPO-06**: User can generate body text for each section with `[SOURCE: file:line]` annotations for claims derived from repo data
- [ ] **REPO-07**: Generated output uses journal template (CEUS) formatting
- [x] **REPO-08**: Repo scan heuristics are extracted to a reference file `references/repo-patterns.md` for maintainability

### Bilingual

- [ ] **BILN-01**: Repo-to-paper output includes paragraph-by-paragraph English + Chinese comparison
- [x] **BILN-02**: Bilingual output format standardized: LaTeX comment for .tex output, markdown blockquote for .md output
- [x] **BILN-03**: Shared bilingual reference file `references/bilingual-output.md` created as pattern documentation
- [ ] **BILN-04**: 7 existing Skills updated to support bilingual output (translation, polish, de-ai, reviewer, abstract, experiment, logic)

### UX Fix

- [x] **UXFIX-01**: paper-polish-workflow uses AskUserQuestion for all structured questions instead of plain dialogue
- [x] **UXFIX-02**: skill-conventions.md updated with AskUserQuestion enforcement rule and invocation examples
- [ ] **UXFIX-03**: Skill automatically records user's frequent workflow sequences, saves as project-level config, and offers as recommendations on next invocation

### Tech Debt

- [x] **DEBT-01**: skill-conventions.md documents escape hatch for Skills with `required: []`
- [x] **DEBT-02**: literature-skill uses capability category "External MCP" instead of vendor-specific "Semantic Scholar MCP"
- [x] **DEBT-03**: cover-letter-skill moves CEUS from leaf_hints to required

## Future Requirements

Deferred beyond v2.0.

### Additional Journal Templates

- **TMPL-01**: IJGIS journal template support
- **TMPL-02**: Cities journal template support

### Platform Expansion

- **PLAT-01**: OpenCode/Cursor compatibility layer

### Advanced Repo Analysis

- **ADVR-01**: Non-Python repo support (R, MATLAB, mixed-language)
- **ADVR-02**: Multi-repo comparison and synthesis

## Out of Scope

| Feature | Reason |
|---------|--------|
| Automated experiment execution | We read results, not run experiments (Orchestra AI-Research-SKILLs territory) |
| Full bilingual on caption/cover-letter/visualization/literature Skills | No user value: captions are always English, cover letters are journal-specific, viz is recommendation-only, literature is BibTeX |
| Real-time collaboration | Single-user workflow unchanged from v1.0 |
| Web UI | Claude Code Skill project, not SaaS |

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| REPO-01 | Phase 14 | Complete |
| REPO-02 | Phase 14 | Complete |
| REPO-03 | Phase 14 | Complete |
| REPO-04 | Phase 15 | Pending |
| REPO-05 | Phase 14 | Complete |
| REPO-06 | Phase 16 | Pending |
| REPO-07 | Phase 16 | Pending |
| REPO-08 | Phase 14 | Complete |
| BILN-01 | Phase 16 | Pending |
| BILN-02 | Phase 13 | Complete |
| BILN-03 | Phase 13 | Complete |
| BILN-04 | Phase 17 | Pending |
| UXFIX-01 | Phase 12 | Complete |
| UXFIX-02 | Phase 11 | Complete |
| UXFIX-03 | Phase 18 | Pending |
| DEBT-01 | Phase 11 | Complete |
| DEBT-02 | Phase 11 | Complete |
| DEBT-03 | Phase 11 | Complete |

**Coverage:**
- v2.0 requirements: 18 total
- Mapped to phases: 18
- Unmapped: 0

---
*Requirements defined: 2026-03-17*
*Last updated: 2026-03-17 — traceability updated after roadmap creation*
