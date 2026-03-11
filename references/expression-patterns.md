# Academic Expression Patterns

A modular reference library for academic writing in Claude Code Skills.

This overview file stays at a stable path so downstream Skills can always load it first, then choose a narrower module when only one writing scenario is needed.

---

## How to Use This Library

- Read this file when you need a quick orientation or want to choose the right module.
- Load one leaf file when the task is scoped to a specific rhetorical move.
- Prefer leaf files for long paper workflows to keep context usage low.
- Keep `references/expression-patterns.md` as the default entrypoint in Skills and documentation.

## Load on Demand

| Need | Read This File | Why |
|------|----------------|-----|
| Introduce topic, state gap, frame contribution | `references/expression-patterns/introduction-and-gap.md` | Covers opening moves, literature gaps, and transition into contribution |
| Describe data, study area, methods, or pipeline | `references/expression-patterns/methods-and-data.md` | Keeps methodological description precise and journal-ready |
| Report findings and interpret them | `references/expression-patterns/results-and-discussion.md` | Helps with evidence reporting and disciplined discussion language |
| Conclude claims, contributions, and future work | `references/expression-patterns/conclusions-and-claims.md` | Supports calibrated claims and closing paragraphs |
| Add geography / urban-science phrasing | `references/expression-patterns/geography-domain.md` | Supplies domain-specific phrasing for space, place, policy, and planning |

## Module Map

- `references/expression-patterns/introduction-and-gap.md`
- `references/expression-patterns/methods-and-data.md`
- `references/expression-patterns/results-and-discussion.md`
- `references/expression-patterns/conclusions-and-claims.md`
- `references/expression-patterns/geography-domain.md`

## Stable Heading Contract

All leaf modules should keep these headings so future Skills can target them predictably:

- `## Recommended Expressions`
- `## Avoid Expressions`
- `## Usage Scenarios`
- `## Bilingual Example Patterns`

## Quick Picks

| Writing move | Quick expression | 中文提示 |
|--------------|------------------|----------|
| Emphasize importance | `[Topic] has become increasingly important in ...` | 用于引出研究背景，不要一上来就夸张声称“革命性” |
| Mark a gap | `However, existing studies remain limited in ...` | 用于指出不足，语气保持克制 |
| Describe evidence | `The results show that ...` | 先说结果，再补统计量或比较对象 |
| Calibrate a claim | `These findings suggest that ...` | 用于避免过度结论化表达 |

## Maintenance Rules

- Add new patterns to the most specific leaf file possible.
- Keep overview content short; detailed tables belong in leaf modules.
- If a leaf file becomes too broad, split it without changing this overview path.

---

*Entry point: references/expression-patterns.md*
*Updated for modular loading: 2026-03-11*
