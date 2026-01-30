# Paper Polish Workflow

A systematic, AI-assisted workflow for polishing academic papers. Designed for use with Claude/OpenCode AI assistants.

## 🎯 What is this?

An AI skill that guides you through paper polishing **step by step**:

```
Structure → Sentence Logic → Expression
```

Instead of making arbitrary changes, the AI helps you:
1. ✅ Confirm the overall structure of each section
2. ✅ Confirm the logic of each sentence  
3. ✅ Select from multiple expression options
4. ✅ Check consistency and coherence

## ✨ Features

- **Top-down approach**: Never jumps to wording before confirming logic
- **User-led**: Every step requires your confirmation
- **Option-based**: Uses interactive selection for efficient choices
- **Reference-driven**: Consults example papers for professional phrasing
- **Journal-aware**: Includes CEUS-specific requirements (extensible to other journals)

## 📦 Installation

### For OpenCode Users

```bash
# Copy to your project's skills directory
mkdir -p .opencode/skills
cp skill/paper-polish-workflow.md .opencode/skills/
```

### For Claude Code Users

Copy `skill/paper-polish-workflow.md` to your project and reference it in your configuration.

## 🚀 Quick Start

Simply ask the AI to polish your paper:

```
User: Help me polish the abstract

AI: [Phase 0] 
    - What is the target journal?
    - Where are the example papers?
    
User: CEUS, examples in docs/example

AI: [Step 1] Presents structure for confirmation...
AI: [Step 2] Presents per-sentence logic...
AI: [Step 3] Presents expression options...
AI: [Step 5] Checks coherence, suggests transitions...
AI: Writes polished version to file
```

## 📋 Workflow Steps

| Step | What Happens |
|------|--------------|
| **Phase 0** | AI asks about target journal, word limits, example papers |
| **Step 1** | AI presents section structure for confirmation |
| **Step 2** | AI presents per-sentence logic for confirmation |
| **Step 3** | AI presents expression options (multiple choice) |
| **Step 4** | AI consults example papers if needed |
| **Step 5** | AI checks repetition and coherence |
| **Write** | AI presents final version and writes to file |

## 🎓 Supported Journals

### CEUS (Computers, Environment and Urban Systems)

Built-in support for:
- Word limits (≤8,000 total, ≤250 abstract)
- Highlights generation (3-5 items, ≤85 chars each)
- Style checklist (geospatial perspective, policy implications)
- Section word budget recommendations

### Adding Other Journals

Edit `paper-polish-workflow.md` to add requirements for other journals.

## 💡 Key Concepts

### Top-Down Polishing

```
┌─────────────────────────────────────┐
│  1. STRUCTURE                       │
│     What sections/components?       │
│     ↓                               │
│  2. LOGIC                           │
│     What should each sentence say?  │
│     ↓                               │
│  3. EXPRESSION                      │
│     How to say it professionally?   │
└─────────────────────────────────────┘
```

### Interactive Option Selection

The workflow uses `mcp_question` for efficient selection:

```
┌─────────────────────────────────────────────┐
│ S1 Expression                               │
│ ─────────────────────────────────────────── │
│ S1: Urban perception importance             │
│                                             │
│ ○ A: Human perception of urban...           │
│ ○ B: How people perceive urban...           │
│ ○ C: Understanding how people...            │
│ ○ Type your own answer                      │
└─────────────────────────────────────────────┘
```

## 📁 Project Structure

```
paper-polish-workflow/
├── README.md                 # This file
├── LICENSE                   # MIT License
├── skill/
│   └── paper-polish-workflow.md   # The skill definition
└── examples/
    └── abstract-polishing-session.md  # Example session
```

## 🔧 Requirements

- OpenCode or Claude Code with tool access
- Tools needed:
  - `mcp_question` - for option selection
  - `mcp_read`, `mcp_write` - for file operations
  - `mcp_look_at` - for PDF analysis (optional)

## 📖 Example Session

See [examples/abstract-polishing-session.md](examples/abstract-polishing-session.md) for a complete walkthrough.

## 🤝 Contributing

Contributions welcome! Please submit issues or pull requests for:
- Additional journal templates
- Improved expression patterns  
- Workflow optimizations
- Bug fixes

## 📄 License

MIT License - Feel free to modify and share.

## 🙏 Acknowledgments

Developed through iterative refinement while polishing a dual-layer urban perception paper for CEUS journal submission.
