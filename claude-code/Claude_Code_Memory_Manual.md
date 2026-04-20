# The Claude Code Memory Manual
**How to give your coding agent persistent context that actually works**

*Version 1.0 — April 2026*
*Author: Seni Kamara | Generated with Claude Opus 4.6 (Anthropic)*

---

## Why This Matters

Claude Code is stateless. Every session starts from zero. The agent knows nothing about your codebase, your preferences, your conventions, or your project's history until you tell it. The only mechanism that persists across sessions is the memory system: a set of markdown files that CC reads automatically at launch.

Most users either ignore memory entirely (and repeat themselves every session) or overload it with everything they can think of (and watch CC ignore most of it). Both are failure modes.

This manual covers the architecture, the constraints, and the design patterns that make CC memory effective.

---

## Part 1: How the Memory System Works

CC has two complementary memory layers. Both load at the start of every session.

### CLAUDE.md Files (You Write These)

Markdown files containing persistent instructions, project context, and behavioral rules. You write them, CC reads them. They are the only content guaranteed to load every time.

### Auto Memory (CC Writes These)

Notes CC saves for itself as it works: build commands, debugging insights, architecture patterns, code style observations. On by default since v2.1.59. Stored at `~/.claude/projects/<project>/memory/MEMORY.md` with optional topic files alongside it.

You can browse, edit, or delete auto memory files. They are plain markdown. Run `/memory` inside any session to manage them.

---

## Part 2: The File Hierarchy

CC discovers memory files by traversing from your current directory up to the filesystem root. Files loaded later have higher effective priority (the model attends more to content that appears later in context).

Loading order:

```
~/.claude/CLAUDE.md                     Global: applies to every project
~/.claude/rules/*.md                    Global rules (supports path-scoping)
<project>/CLAUDE.md                     Project: applies to this repo only
<project>/.claude/CLAUDE.md             Project (alternative location)
<project>/.claude/rules/*.md            Project rules (supports path-scoping)
<project>/CLAUDE.local.md              Local: personal, gitignored by default
```

All levels **combine** (they do not replace each other). On conflict, the more specific file wins.

### Path-Scoped Rules

Files in any `rules/` directory can use YAML frontmatter to restrict which file paths they apply to:

```yaml
---
paths:
  - "src/api/**"
  - "*.graphql"
---
# API conventions
All API handlers must validate input using the shared validate() helper.
```

This rule only loads when CC is working on files matching those patterns.

---

## Part 3: The Sizing Constraint (Critical)

This is the most important section in this manual.

CC's system prompt already contains approximately 50 internal instructions. Research on instruction-following in large language models shows a consistent pattern: **as instruction count increases, adherence quality decreases uniformly.** CC does not simply ignore newer instructions. It begins to follow all of them less reliably.

The practical implications:

- **Files over 200 lines consume excessive context and reduce adherence.** This is from Anthropic's own documentation.
- Every instruction you add competes with CC's internal system prompt for the model's attention.
- Saying the same thing five different ways does not reinforce it. It dilutes everything.

### What This Means for Your CLAUDE.md

Your memory files are not documentation. They are not README files. They are not knowledge bases. They are **high-priority system instructions competing for finite attention.**

Write them the way you would write a pager alert: only what matters, only what would cause errors if missing, as concise as possible.

---

## Part 4: Design Patterns

### Pattern 1: Two-Document Architecture

Separate **factual context** from **behavioral governance**.

| Document | Purpose | Changes When |
|---|---|---|
| `CLAUDE.md` | What you're working on: projects, tech stack, active priorities, key terminology, tooling | Projects shift, priorities change, new tools adopted |
| Behavioral rules file | How the agent should work: evaluation standards, output quality rules, communication style | Working style evolves (rarely) |

Why separate them: factual context changes frequently. Behavioral governance is stable. Coupling them means every project update risks accidentally modifying your working-style rules, and vice versa.

Place the behavioral file in `~/.claude/rules/` so it loads as a distinct memory entry rather than one monolithic block.

### Pattern 2: Global Baseline + Project Overrides

Install your universal context globally. Add repo-specific context at the project level.

```
~/.claude/
├── CLAUDE.md                    # Your identity, skills, cross-project context
└── rules/
    └── working-style.md         # How you want the agent to behave

~/Projects/webapp/
├── CLAUDE.md                    # This project's stack, conventions, architecture

~/Projects/data-pipeline/
├── CLAUDE.md                    # This project's stack, conventions, architecture
```

**Project CLAUDE.md files should be 20-40 lines.** They add repo-specific context only. They never restate global content. If a project file starts growing past 40 lines, you are probably duplicating your global baseline.

### Pattern 3: Rule Splitting for Large Governance Files

If your behavioral governance exceeds 200 lines, split it into separate rule files by domain:

```
~/.claude/rules/
├── evaluation-protocol.md       # How to evaluate and critique work
├── output-standards.md          # Quality requirements for deliverables
├── communication-style.md       # Tone, format, terminology rules
└── repo-governance.md           # SSOT, naming, compliance rules
```

Each file loads as a separate memory entry. CC may attend to them better than one massive file. Use this pattern when your governance needs are complex but you want to stay within the per-file size constraint.

### Pattern 4: @Imports for On-Demand Detail

CLAUDE.md files can import other files using `@` notation:

```markdown
# CLAUDE.md
@./docs/architecture.md
@~/shared/style-guide.md
```

Imported files are processed recursively and inserted before the file that references them. External imports (outside the project directory) require explicit approval on first load.

Use imports to keep your CLAUDE.md lean while making deeper documentation accessible when CC needs it.

---

## Part 5: What Belongs in Memory (and What Does Not)

### Include (would cause errors or wasted time if missing):
- Build, test, and lint commands (exact invocations, not just tool names)
- Architecture decisions that affect how code should be written
- Coding conventions specific to your project (naming patterns, file structure)
- Key terminology where precision matters (e.g., domain-specific terms that CC might confuse)
- Active priorities and current project state
- Environment requirements (required env vars, expected services)
- Common pitfalls CC should avoid

### Exclude (noise that dilutes attention):
- Things CC already knows (standard language syntax, common library APIs)
- Obvious reminders ("write clean code," "add comments")
- Long explanations that could be discovered by reading the codebase
- Historical context that does not affect current work
- Aspirational guidelines you do not actually enforce

### The Litmus Test

Before adding any instruction to CLAUDE.md, ask: **Would CC make a concrete mistake in the next session if this line were missing?** If the answer is no, it does not belong in memory.

---

## Part 6: Compression Strategies

When memory files grow too large, the solution is not to delete content. It is to compress.

### Technique 1: Eliminate Rhetorical Repetition

Most governance documents say the same core principle multiple times in different frames. "Be critical, not compliant" might appear as a principle, a rule, an anti-pattern list, a positive restatement, and a summary. For human readers, repetition aids retention. For CC, repetition burns attention budget and reduces adherence to every instruction uniformly.

State each principle once. State it precisely. Move on.

### Technique 2: Collapse Lists into Dense Prose

```markdown
# Before (7 lines)
The agent must:
- identify the weakest link first
- say plainly what is not working
- separate "promising" from "defensible"
- challenge assumptions
- surface trade-offs
- test whether the argument survives pressure
- resist momentum when it carries error

# After (2 lines)
The agent must identify the weakest link first, say plainly what is not working,
separate "promising" from "defensible," and resist momentum when it carries error.
```

Both convey the same information. The second uses 70% fewer lines.

### Technique 3: Remove Sections Duplicated Across Documents

If your global CLAUDE.md already defines your tech stack, your project CLAUDE.md should not redefine it. If your behavioral rules file already covers communication style, your CLAUDE.md should not include a communication section. Audit for cross-document duplication ruthlessly.

### Technique 4: Cut the Meta-Commentary

Instructions like "This section is essential" or "Agents and collaborators should treat the following as load-bearing" are meta-commentary about the document, not instructions for the agent. CC does not need to be told that your instructions are important. It reads everything at the same priority level. Remove framing language and keep directives.

### Realistic Compression Ratios

Well-written but verbose governance documents typically compress 60-75% without content loss. The intellectual payload of a 600-line document usually fits in 150-180 lines. The remainder is rhetorical reinforcement, examples of what was already stated, and meta-commentary.

---

## Part 7: Maintenance

### Update Cadence

- **Global CLAUDE.md:** When your cross-project context changes (new tools, shifted priorities, role changes). Typically monthly.
- **Project CLAUDE.md:** When repo-specific context shifts (new workstream, changed architecture, different conventions). As needed.
- **Behavioral rules:** When your working-style governance evolves. Rarely.
- **Auto memory:** Run `/memory` periodically. Delete stale entries CC accumulated. Auto memory files that grow large become noise.

### Preventing Bloat

Memory files drift toward bloat because every frustration becomes a new instruction. "CC keeps using tabs instead of spaces" becomes a line in CLAUDE.md. Over months, dozens of these accumulate.

Quarterly, audit your memory files against the litmus test: would CC make a concrete mistake without this line? Remove everything that fails.

### Compaction Behavior

CLAUDE.md fully survives `/compact`. After compaction, CC re-reads your CLAUDE.md from disk and re-injects it fresh. If an instruction disappeared after compaction, it was given only in conversation, not in CLAUDE.md. Add it to memory to make it persist.

---

## Part 8: Verification

After setting up or modifying your memory files:

```bash
# Start a CC session in your project
cd ~/your-project
claude

# View all loaded memory files
/memory

# Confirm your files appear in the list
# Then test with a domain-specific question:
# "What are the conventions for this project?"
# "How should you approach code review in this repo?"
```

If CC's answer reflects your memory files, the setup is working. If it gives generic responses, check that your files are in the correct locations and under 200 lines each.

---

## Part 9: Quick Reference

### File Locations
```
~/.claude/CLAUDE.md                  # Global context (all projects)
~/.claude/rules/*.md                 # Global behavioral rules
<project>/CLAUDE.md                  # Project context (this repo)
<project>/.claude/rules/*.md         # Project rules (path-scoped)
<project>/CLAUDE.local.md            # Personal overrides (gitignored)
```

### Commands
```
/memory                              # View and edit loaded memory files
/init                                # Auto-generate CLAUDE.md for current project
/compact                             # Compress context (re-reads CLAUDE.md from disk)
```

### Size Constraints
- **Per file:** Under 200 lines for reliable adherence
- **Total loaded:** Minimize. Every line competes with CC's ~50 internal instructions
- **Project CLAUDE.md:** 20-40 lines (repo-specific only)
- **Auto memory index:** Keep MEMORY.md concise. Move detail into topic files

### Design Principles
1. Memory files are system instructions, not documentation
2. Separate factual context from behavioral governance
3. Global baseline + project overrides
4. State each principle once, precisely
5. If CC would not make a mistake without it, remove it

---

*Built from direct experience configuring Claude Code memory across research, commercial, and multi-agent workflows. Constraints and architecture validated against Anthropic's official documentation as of April 2026.*

---

> **AI Disclosure:** This manual was generated by Claude Opus 4.6 (Anthropic) in collaboration with the author. Claude is AI and can make mistakes. Please double-check responses. Verify all technical details against [Anthropic's official Claude Code documentation](https://docs.anthropic.com/en/docs/claude-code/overview) for the most current information.
