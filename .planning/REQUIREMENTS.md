# Requirements: Paper Polish Workflow

**Defined:** 2026-03-10
**Core Value:** Every Skill must produce output that is directly usable in a real paper submission

## v1 Requirements

Requirements for initial release. Each maps to roadmap phases.

### Foundation (FNDN)

- [ ] **FNDN-01**: Shared expression patterns library covers sentence starters, transitions, hedging, results, and conclusions with bilingual examples
- [ ] **FNDN-02**: Journal template system with CEUS as first template, following a strict contract (consistent section headers across all journal files)
- [ ] **FNDN-03**: Anti-AI patterns library with high-frequency AI vocabulary blacklist and human-sounding alternatives
- [ ] **FNDN-04**: Skill template conventions document defining YAML frontmatter format, tool usage rules, ~300 line limit, and progressive disclosure pattern

### Core Skills (CORE)

- [ ] **CORE-01**: Chinese-to-English translation Skill accepts Chinese draft and outputs polished English academic text in LaTeX format
- [ ] **CORE-02**: English polishing Skill with flexible flow supporting both quick-fix mode (direct polish) and guided mode (structure → logic → expression)
- [ ] **CORE-03**: De-AI Skill detects AI-generated patterns and rewrites text to reduce detection scores while maintaining academic quality
- [ ] **CORE-04**: Reviewer simulation Skill reviews paper from reviewer perspective, identifies weaknesses, and provides actionable improvement suggestions

### Section Skills (SECT)

- [ ] **SECT-01**: Abstract generation/optimization Skill produces abstracts following the 5-sentence formula (contribution, difficulty, method, evidence, result)
- [ ] **SECT-02**: Figure and table caption Skill generates or optimizes captions with geography-specific awareness (study area, CRS, legend description)
- [ ] **SECT-03**: Experiment analysis Skill helps analyze results, identify patterns, and generate discussion paragraphs
- [ ] **SECT-04**: Logic verification Skill checks argument chains, identifies gaps, inconsistencies, and unsupported claims across sections

### Support Skills (SUPP)

- [ ] **SUPP-01**: Cover letter Skill generates submission cover letters based on paper content and target journal requirements
- [ ] **SUPP-02**: Literature search Skill uses Semantic Scholar MCP to find relevant papers, verify citations, and generate BibTeX entries
- [ ] **SUPP-03**: Visualization recommendation Skill suggests appropriate chart types for experimental results with geography-specific options (choropleth, spatial plots)

### Documentation (DOCS)

- [ ] **DOCS-01**: Project README with installation instructions, Skill inventory, quick start guide, and contribution guidelines (bilingual)

## v2 Requirements

Deferred to future release. Tracked but not in current roadmap.

### Additional Journals

- **JRNL-01**: IJGIS (International Journal of Geographical Information Science) template
- **JRNL-02**: Cities journal template
- **JRNL-03**: CS/AI conference templates (NeurIPS, ICML, ICLR format)

### Extended Features

- **EXTD-01**: English-to-Chinese translation Skill (reverse direction)
- **EXTD-02**: Chinese paper polishing Skill (中文论文润色)
- **EXTD-03**: Text compression Skill (缩写，reduce word count)
- **EXTD-04**: Text expansion Skill (扩写，expand content)
- **EXTD-05**: Paper architecture diagram generation guidance
- **EXTD-06**: Rebuttal writing Skill (based on reviewer comments)
- **EXTD-07**: Cross-section consistency checker (numbers, terminology alignment)

## Out of Scope

| Feature | Reason |
|---------|--------|
| Paper generation from scratch | Assists human writers, does not replace them — ethical boundary |
| Grammar-only checking | Grammarly/LanguageTool do this better; our Skills are higher-level |
| Image/figure generation | Requires image generation models; out of scope for text Skills |
| Plagiarism detection | Requires specialized databases (Turnitin, iThenticate); cannot replicate |
| Web UI or standalone app | This is a Claude Code Skill project, not a SaaS |
| OpenCode/Cursor support | Focus on Claude Code; other platforms are v2+ consideration |
| Real-time collaboration | Single-user workflow |

## Traceability

Which phases cover which requirements. Updated during roadmap creation.

| Requirement | Phase | Status |
|-------------|-------|--------|
| FNDN-01 | Phase 1: Reference Libraries | Pending |
| FNDN-02 | Phase 1: Reference Libraries | Pending |
| FNDN-03 | Phase 1: Reference Libraries | Pending |
| FNDN-04 | Phase 2: Skill Conventions | Pending |
| CORE-01 | Phase 3: Translation Skill | Pending |
| CORE-02 | Phase 4: Polish Skill | Pending |
| CORE-03 | Phase 5: De-AI Skill | Pending |
| CORE-04 | Phase 6: Reviewer Simulation Skill | Pending |
| SECT-01 | Phase 7: Abstract and Experiment Skills | Pending |
| SECT-02 | Phase 8: Figure/Table and Logic Skills | Pending |
| SECT-03 | Phase 7: Abstract and Experiment Skills | Pending |
| SECT-04 | Phase 8: Figure/Table and Logic Skills | Pending |
| SUPP-01 | Phase 9: Literature and Support Skills | Pending |
| SUPP-02 | Phase 9: Literature and Support Skills | Pending |
| SUPP-03 | Phase 9: Literature and Support Skills | Pending |
| DOCS-01 | Phase 10: Documentation | Pending |

**Coverage:**
- v1 requirements: 16 total
- Mapped to phases: 16
- Unmapped: 0

---
*Requirements defined: 2026-03-10*
*Last updated: 2026-03-10 after roadmap creation*
