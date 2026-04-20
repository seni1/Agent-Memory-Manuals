# Claude Code Memory Manual
**How to design persistent Claude Code instructions that remain usable under real operating conditions**

*Version 2.0 - April 20, 2026*
*Author: Seni Kamara*
*Distribution edition, grounded in Anthropic Claude Code documentation and local CLI behavior*

---

## Purpose

`CLAUDE.md` is a control layer, not a knowledge base.

That is the thesis of this manual.

Claude Code is stateless. Every session starts from zero. The agent knows nothing about your codebase, your conventions, or your project's history until you tell it. The memory system - a set of markdown files Claude Code reads at launch - is the only mechanism that persists across sessions. If you use it to carry everything, it turns into a second wiki and starts failing at the job it is supposed to do.

This manual is for people who want Claude Code to start work with durable context without turning the instruction layer into a second documentation system.

If you get the boundaries wrong, Claude Code will still work, but it will work inconsistently. The failure mode is not total breakdown. It is quiet drift: diluted adherence, stale assumptions, duplicated governance, and overloaded prompts that gradually lose precision.

This manual separates:

- **Verified platform behavior**: grounded in Anthropic Claude Code documentation and local CLI inspection
- **Recommended operating practice**: judgment about how to use the platform well

Do not confuse the two.

---

## What CLAUDE.md Is

### Verified platform behavior

Claude Code reads `CLAUDE.md` files at the start of every session and combines them into the system prompt before work begins. This is a checked-in or operator-controlled persistence layer for project guidance.

Claude Code also has a second, complementary memory layer it writes for itself: **auto memory**, stored under `~/.claude/projects/<project>/memory/` and managed through the `/memory` slash command. Auto memory is enabled by default in current Claude Code versions.

Other instruction and context surfaces also exist:

- built-in system behavior (Claude Code's internal system prompt)
- settings in `settings.json` (permissions, MCP servers, hooks, memory configuration)
- skills and plugins
- the live user prompt and session context

So `CLAUDE.md` is not the whole control surface.

### Recommended operating practice

Treat `CLAUDE.md` as a **control layer**, not a knowledge base.

In practice, it is the most important durable repository-layer instruction surface you explicitly author.

Use it for:

- durable working rules that affect execution
- repo-specific conventions Claude Code is likely to miss
- active priorities and current project state
- terminology Claude Code is likely to confuse
- exact build, test, lint, and review commands
- approval boundaries and dangerous actions to avoid

Do not use it for:

- long-form project explanation
- broad historical context
- directory inventories
- canonical domain truth
- archive manifests

Those belong elsewhere.

---

## What Belongs Where

### Recommended operating practice

Use each layer for a different job:

| Layer | Job | Typical content |
|---|---|---|
| `CLAUDE.md` | Behavioral control | How Claude Code should work here |
| `~/.claude/rules/*.md` | Cross-cutting or path-scoped governance | Working style, evaluation rules, scoped conventions |
| `README.md` | Navigation | What this directory is, where to start, what not to touch |
| Canonical docs | Domain truth | Definitions, methodology, policy, approved source text |
| Auto memory | Local recall | Build commands, debugging notes, operator preferences |
| `settings.json` | Runtime configuration | Permissions, MCP servers, hooks |

If you put navigation or factual authority into `CLAUDE.md`, you burn instruction budget on content that should be discoverable. If you put behavioral rules only in README files, you make them easier to miss and easier to dilute.

Operationally, the best pattern is:

1. keep `CLAUDE.md` short and load-bearing
2. keep READMEs explicit and navigational
3. keep canon small and authoritative
4. keep archive boundaries hard
5. keep rules files focused by domain or path

---

## How Claude Code Discovers CLAUDE.md

### Verified platform behavior

Claude Code discovers memory files by traversing from the current working directory upward to the filesystem root, and by reading from the global configuration directory.

The loading surfaces are:

1. **Global scope** - `~/.claude/CLAUDE.md` and any files in `~/.claude/rules/`
2. **Project scope** - starting at the project root, Claude Code looks for:
   - `<project>/CLAUDE.md`
   - `<project>/.claude/CLAUDE.md` (alternative project location)
   - `<project>/.claude/rules/*.md` (project-scoped rules)
3. **Local scope** - `<project>/CLAUDE.local.md`, which is personal and typically gitignored

All files combine into the system prompt. They do not replace each other. Files loaded later have higher effective priority because language models attend more strongly to content appearing later in context.

Files in any `rules/` directory can use YAML frontmatter to restrict the file paths they apply to:

```yaml
---
paths:
  - "src/api/**"
  - "*.graphql"
---
```

Rules with path scoping load only when Claude Code is working on matching files.

### Recommended operating practice

This has three consequences:

1. A nested or project instruction file is not stronger because it is "more specific" in the abstract. It is stronger because it appears later in the assembled prompt.
2. A `CLAUDE.local.md` can silently override behavior you expect from the checked-in `CLAUDE.md`. Treat it as shadow governance if it is not documented.
3. A large global file can compress the attention budget available for project-level guidance, even though nothing has been truncated.

---

## CLAUDE.local.md

### Verified platform behavior

`CLAUDE.local.md` lives at the project root. It loads after `CLAUDE.md` in the same directory, so its content appears later in the prompt and has higher effective priority. It is gitignored by default templates, which means it does not travel with the repo.

### Recommended operating practice

Use it sparingly.

Good uses:

- personal experimental overrides while you test a change
- local-only debugging preferences
- operator-specific secrets handling rules
- a short-lived "I am focused on X right now" context that you will remove afterward

Bad uses:

- team-required rules that other operators will never see
- shadow governance that contradicts the checked-in file
- replacing a parent `CLAUDE.md` because it became bloated

A local file is not just a stronger instruction file. It is a bypass that is invisible to teammates. That makes it useful, but also dangerous.

If you rely on one, document its existence in the repo `CLAUDE.md` or the project README so another operator is not debugging invisible prompt state.

---

## Size and Attention Budget

### Verified platform behavior

Anthropic's Claude Code documentation notes that memory files contribute to the system prompt and recommends keeping individual files concise. The internal system prompt already contains a substantial number of core instructions before any of your memory files load.

Claude Code does not publish a precise byte cap for combined memory files in the same way Codex exposes `project_doc_max_bytes`. The constraint is attention-based rather than a hard truncation threshold: as total instruction count grows, per-instruction adherence degrades well before anything is dropped.

### Recommended operating practice

The main hazard is not truncation. It is silent adherence decay.

Research on instruction-following in large language models shows a consistent pattern: as instruction count increases, adherence to any single instruction decreases uniformly. Claude Code does not ignore newer instructions first. It starts following all of them less reliably.

That does not mean you should adopt fake universal line limits. Heuristics like "under 200 lines" or "20 to 40 lines per project file" are useful defaults, not platform law.

Use these as operator heuristics instead:

- Global `CLAUDE.md`: only stable personal or team defaults
- Project `CLAUDE.md`: only what would cause concrete execution mistakes if missing
- Rules files: split by domain when one file grows past a comfortable review length
- `CLAUDE.local.md`: smaller than the file it effectively layers over

If a file keeps growing, the first question is not "Can I push past the soft limit?" It is "What content belongs in README, in canon, or in auto memory instead?"

Compression beats expansion almost every time.

---

## Auto Memory

### Verified platform behavior

Claude Code writes notes for itself during sessions: build commands, debugging insights, architectural observations, code style patterns. These are stored in plain markdown under `~/.claude/projects/<project-path-slug>/memory/` with a `MEMORY.md` index and optional topic files alongside it.

Key behaviors:

- auto memory is on by default in current Claude Code versions
- it persists across sessions for the same project
- operators can browse, edit, or delete auto memory files directly on disk
- the `/memory` slash command opens an in-session interface for managing them
- auto memory is local to the user's machine and is not shared across operators
- auto memory configuration lives in `settings.json` under memory-related keys

### Recommended operating practice

Treat auto memory as **local recall**, not governance.

Good uses:

- recurring build or debug commands you want Claude Code to remember
- stable personal workflows
- operator preferences you do not want to restate every time
- architectural patterns Claude Code inferred correctly and should keep using

Bad uses:

- required repo policy
- team rules
- anything another operator must be able to inspect in the repository

If a future session must know it, check it into the repo `CLAUDE.md` or the global `CLAUDE.md`. Do not rely on auto memory to carry required behavior, because the next operator will not have the same auto memory files.

Auto memory also needs maintenance. Run `/memory` periodically to audit and prune. Files that grow large become noise, and noise dilutes the rest of the prompt.

---

## CLAUDE.md vs Rules

### Verified platform behavior

`~/.claude/rules/*.md` and `<project>/.claude/rules/*.md` are separate files loaded alongside the main `CLAUDE.md`. Each rule file can carry YAML frontmatter that restricts when it loads based on file path patterns.

Rules are still natural-language markdown. They are not a separate execution policy system in the way Codex `rules` are Starlark programs that gate command approval.

### Recommended operating practice

Do not use `CLAUDE.md` to do the job of rules files, and do not use rules files to do the job of `CLAUDE.md`.

Use `CLAUDE.md` for:

- cross-cutting execution behavior that applies to the whole project
- stack, commands, and conventions that affect every edit
- handoff-sensitive rules that every session must honor

Use `rules/` files for:

- path-scoped conventions (API routes, migrations, a specific package)
- large governance blocks that would otherwise bloat `CLAUDE.md`
- domain-specific rules that only apply to a subset of work

Think of the split this way:

- `CLAUDE.md` says how Claude Code should behave everywhere in this project
- `rules/*.md` says how Claude Code should behave when working on specific paths or domains

Splitting rules out of `CLAUDE.md` also means each rule file loads as its own memory entry, which is usually easier to attend to than one long monolithic file.

---

## The Right Design Pattern

### Recommended operating practice

For most repos, the winning pattern is:

1. **Global baseline**
   Personal defaults in `~/.claude/CLAUDE.md` and stable working-style governance in `~/.claude/rules/`
2. **Repo instruction layer**
   Repo-specific working rules in `<project>/CLAUDE.md`
3. **Path-scoped rules only where needed**
   Add `.claude/rules/*.md` files only when local conventions genuinely diverge
4. **README layer**
   Every human-facing directory should explain purpose, authority, and start points
5. **Canonical layer**
   Small set of authoritative docs for actual domain truth
6. **Archive layer**
   Old or deprecated material behind a clear boundary

The instruction chain should be small. The repo itself can still be richly documented through READMEs and canonical docs.

The two-document architecture is especially useful at the global scope:

| Document | Purpose | Changes when |
|---|---|---|
| `~/.claude/CLAUDE.md` | What you are working on: projects, tech stack, active priorities, terminology | Projects shift, priorities change, new tools adopted |
| `~/.claude/rules/working-style.md` | How the agent should work: evaluation standards, output quality, communication style | Working style evolves (rarely) |

Why separate them: factual context changes frequently. Behavioral governance is stable. Coupling them means every project update risks accidentally modifying your working-style rules, and vice versa.

---

## What to Put in CLAUDE.md

### Recommended operating practice

Put in:

- exact build, test, lint, and review commands
- stack-specific conventions Claude Code is likely to miss
- naming or file-structure rules that affect edits
- critical domain terms likely to be confused
- active workstream constraints
- approval boundaries and dangerous actions to avoid
- handoff-sensitive rules, such as "update canon if changing methodology"

Keep out:

- broad architecture essays
- duplicate content already present in canonical docs
- directory inventories
- old migration notes
- motivational prose ("write clean code")
- rules you do not actually enforce
- things Claude Code already knows (standard language syntax, common library APIs)

Useful litmus test:

> If this line disappeared, would Claude Code be materially more likely to make a concrete mistake in the next session?

If the answer is no, remove it from `CLAUDE.md`.

---

## Failure Modes

### Recommended operating practice

These are the ones that matter in practice:

### 1. The CLAUDE.md wiki failure

You keep adding context until `CLAUDE.md` becomes a repository-in-miniature. The file gets harder to audit, weaker as instruction, and more likely to crowd out path-scoped rules and project-specific guidance.

### 2. The local-override drift failure

A `CLAUDE.local.md` survives from a prior task and silently shifts behavior for one operator while the team-level file stays clean. Other operators cannot reproduce the behavior and cannot see the source.

### 3. The stale canon failure

`CLAUDE.md` says how to act, but the canonical documents it points to no longer reflect the current truth. Claude Code obeys instructions against outdated authority.

### 4. The README gap failure

The instruction layer is clean, but the repo has poor navigation. Claude Code then compensates by guessing which files matter, which reintroduces exactly the drift `CLAUDE.md` was supposed to prevent.

### 5. The auto-memory substitution failure

An operator starts relying on auto memory to carry required policy across sessions. Another operator joins the project and cannot see or share that context, because auto memory is local.

### 6. The rhetorical-repetition failure

The same principle gets restated three different ways across `CLAUDE.md`, a rules file, and a README, in the belief that repetition reinforces it. For Claude Code, repetition burns attention budget and reduces adherence to every instruction uniformly. State principles once, precisely.

### 7. The false verification failure

Claude Code correctly summarizes a rule when asked, but still does not operate cleanly because another layer, a missing file, or a stale local override is shaping behavior. A verbal summary is a smoke test, not a verification.

---

## Multi-Operator Repos

### Recommended operating practice

When more than one person works with Claude Code against the same repo, the governance surface changes. Three invisible layers start to matter.

**Global `CLAUDE.md` is personal state.** Every operator has their own `~/.claude/CLAUDE.md`, and you cannot see what another operator has in theirs. Their global file may contradict, soften, or strengthen your repo guidance in ways you never observe. The checked-in repo `CLAUDE.md` is the only layer the team can review together. Treat it as the team contract. Do not put team-required rules only in your personal global file.

**`CLAUDE.local.md` files are local hazards in a shared repo.** Even though they are gitignored by default, an operator can accidentally check one in, or create one whose behavior other operators cannot reproduce. A teammate debugging "Claude Code is acting strangely for me" will not think to look for a file that does not exist in the repository. Make local-override audits part of onboarding and post-mortems.

**Auto memory does not transfer between operators.** Memory files live under each operator's `~/.claude/projects/`. They are private. Anything captured there exists only for that user. This is not a limitation to work around: it is the right design for a local-recall layer. But it means team policy can never ride on auto memory. If the team must know it, check it in.

### Team hygiene checklist

Before treating a repo as operator-ready for shared Claude Code use:

1. **Audit for committed or orphaned local files.**
   Run `find . -name CLAUDE.local.md` at the repo root. Each result should have a documented reason in the nearest README or in the repo `CLAUDE.md`.
2. **Commit shared rules to the repo.**
   Rules that apply to the whole team belong in `<project>/.claude/rules/`, not in each operator's `~/.claude/rules/`. Otherwise you have no shared execution behavior, only shared vibes.
3. **Document settings.json expectations.**
   If the repo requires specific permissions, MCP servers, or hook configuration, document it in the project README and, where possible, check in a sample `settings.json` so new operators can align.
4. **Treat the checked-in chain as the reviewable artifact.**
   Everything else is operator-local. Code review, incident response, and onboarding all run against the checked-in layer. If a behavior is not reproducible from the checked-in files alone, it is not team policy.
5. **Name the archive boundary explicitly.**
   Multi-operator repos accumulate deprecated directories faster than single-operator ones. Make archive paths discoverable (e.g., `archive/` or `deprecated/`) and name them in the repo `CLAUDE.md` so Claude Code does not promote old material back to the active surface under one operator but not another.

The short version: anything that is not in the repo cannot be team policy. Every operator-local layer is an invisibility risk.

---

## Verification Workflow

Asking Claude Code to summarize its instructions is only a smoke test. It is useful, but it is not strong verification.

Use this order instead.

### 1. Inspect the loaded memory surface

Inside a Claude Code session in the project:

```
/memory
```

This shows the memory files Claude Code has loaded, including global, project, and rules files. If a file you expect to load is not listed, the discovery chain is wrong before you even look at content.

### 2. Run a smoke test

Ask Claude Code to describe the currently loaded governance:

```
Summarize the loaded memory files and list your top rules for this project.
```

This tells you whether Claude Code has at least absorbed the expected guidance. It does not prove the chain is complete or uncontested.

### 3. Check discovery settings

Look at `settings.json` (both `~/.claude/settings.json` and `<project>/.claude/settings.json` if present). Pay attention to memory-related keys, permissions, and any MCP server or plugin configuration that could inject additional context.

### 4. Check for local overrides

Search for any `CLAUDE.local.md` files:

```bash
find . -name "CLAUDE.local.md"
```

Then inspect whether a local file is silently reshaping behavior that should be coming from the checked-in `CLAUDE.md`.

### 5. Test path-scoped rules separately

If you have rules with YAML path scoping, verify they actually fire on the paths you expect. Open a file that should trigger the rule and ask Claude Code what guidance applies:

```
What rules currently apply to this file?
```

If the scoped rule is not listed, either the frontmatter is wrong or the path pattern does not match.

### 6. Test behavior against a concrete task

Run a domain-specific prompt that would only answer correctly if the memory files loaded:

```
What are the conventions for this project?
How should you approach code review in this repo?
What is the exact test command I use?
```

A generic answer means the memory files are either not loading, not adhering, or crowded out by competing instructions.

---

## Practical Configuration Knobs

### Verified platform behavior

Claude Code reads configuration from `settings.json` files at two scopes:

- `~/.claude/settings.json` - user-level defaults
- `<project>/.claude/settings.json` - project-level overrides

Keys operators most often touch include:

- `permissions` - allow/deny lists for tool use
- `mcpServers` - Model Context Protocol server connections
- `hooks` - lifecycle hooks for automated behavior
- memory-related keys controlling auto memory behavior
- keyboard bindings in `~/.claude/keybindings.json`

Slash commands relevant to memory management:

- `/memory` - browse and edit loaded memory files
- `/init` - auto-generate a starter `CLAUDE.md` for the current project
- `/compact` - compress context (re-reads `CLAUDE.md` from disk afterward)
- `/context` - inspect current context usage

### Recommended operating practice

Do not treat configuration as a substitute for good instruction design.

Permissions, hooks, and MCP servers shape what Claude Code can do. They do not replace the need for clear instructions about what it should do. Tightening permissions without tightening `CLAUDE.md` just makes the agent fail more quietly.

Before adjusting a configuration knob, ask: is my current instruction layer already clean? If not, the configuration change is probably papering over an instruction problem.

---

## Example Layout

```text
~/.claude/
├── CLAUDE.md
├── rules/
│   ├── working-style.md
│   └── evaluation-protocol.md
├── settings.json
└── projects/
    └── <project-slug>/
        └── memory/
            └── MEMORY.md

<repo>/
├── CLAUDE.md
├── CLAUDE.local.md                 # optional, gitignored
├── README.md
├── canonical_index.md              # optional but useful in governed repos
├── .claude/
│   ├── rules/
│   │   └── api-conventions.md      # path-scoped via frontmatter
│   └── settings.json               # project-specific permissions/servers
├── docs/
│   └── methodology/
│       └── README.md
└── services/
    └── payments/
        └── README.md
```

Interpretation:

- `CLAUDE.md` controls execution behavior
- `rules/*.md` carries cross-cutting or path-scoped governance
- `README.md` handles navigation
- canonical docs hold actual source truth
- `settings.json` controls runtime configuration, not behavior

---

## Recommended Authoring Standard

Use this checklist when editing any `CLAUDE.md` or rules file:

1. Is the line durable across many sessions?
2. Would its absence cause a real execution mistake?
3. Is this instruction better than putting the same content in README?
4. Is it global, or should it be scoped to a project or path?
5. Is it short enough to survive alongside other instructions?

If you cannot justify a line on those terms, it probably does not belong.

---

## Compression Strategies

When memory files grow too large, the solution is rarely to delete content outright. It is to compress.

### Technique 1: Eliminate rhetorical repetition

Most governance documents say the same core principle multiple times in different frames. "Be critical, not compliant" might appear as a principle, a rule, an anti-pattern list, a positive restatement, and a summary. For human readers, repetition aids retention. For Claude Code, repetition burns attention budget and reduces adherence to every instruction uniformly.

State each principle once. State it precisely. Move on.

### Technique 2: Collapse lists into dense prose

```markdown
Before (7 lines):
The agent must:
- identify the weakest link first
- say plainly what is not working
- separate "promising" from "defensible"
- challenge assumptions
- surface trade-offs
- test whether the argument survives pressure
- resist momentum when it carries error

After (2 lines):
The agent must identify the weakest link first, say plainly what is not working,
separate "promising" from "defensible," and resist momentum when it carries error.
```

Both convey the same information. The second uses 70% fewer lines.

### Technique 3: Remove content duplicated across documents

If your global `CLAUDE.md` already defines your tech stack, your project `CLAUDE.md` should not redefine it. If your behavioral rules file already covers communication style, neither `CLAUDE.md` should include a communication section. Audit for cross-document duplication ruthlessly.

### Technique 4: Cut meta-commentary

Instructions like "This section is essential" or "Agents should treat the following as load-bearing" are meta-commentary about the document, not instructions for the agent. Claude Code does not need to be told that your instructions are important. It reads everything at the same priority level. Remove framing language and keep directives.

### Realistic compression ratios

Well-written but verbose governance documents typically compress 60-75% without content loss. The intellectual payload of a 600-line document usually fits in 150-180 lines. The remainder is rhetorical reinforcement, examples of what was already stated, and meta-commentary.

---

## Maintenance

### Update cadence

- **Global `CLAUDE.md`**: when your cross-project context changes (new tools, shifted priorities, role changes). Typically monthly.
- **Project `CLAUDE.md`**: when repo-specific context shifts (new workstream, changed architecture, different conventions). As needed.
- **Rules files**: when your working-style or domain governance evolves. Rarely.
- **Auto memory**: run `/memory` periodically. Prune stale entries. Auto memory files that grow large become noise.

### Preventing bloat

Memory files drift toward bloat because every frustration becomes a new instruction. "Claude Code keeps using tabs instead of spaces" becomes a line in `CLAUDE.md`. Over months, dozens of these accumulate.

Quarterly, audit your memory files against the litmus test: would Claude Code make a concrete mistake without this line? Remove everything that fails.

### Compaction behavior

`CLAUDE.md` fully survives `/compact`. After compaction, Claude Code re-reads `CLAUDE.md` from disk and re-injects it fresh. If an instruction disappeared after compaction, it was given only in conversation, not in `CLAUDE.md`. Add it to memory if you want it to persist.

---

## Minimal Templates

### Global `~/.claude/CLAUDE.md`

```markdown
# Working Agreements

- Be direct about uncertainty and tradeoffs.
- Prefer `rg` for search.
- Ask before installing new dependencies or taking destructive actions.
- Run tests after code changes unless I say otherwise.
- Flag assumptions explicitly before acting on them.
```

### Project `<project>/CLAUDE.md`

```markdown
# Repo Guidance

- Use `pnpm test` and `pnpm lint` after TypeScript edits.
- Keep API schema changes synchronized with `docs/api/README.md`.
- Treat `docs/canon/` as authoritative for product terminology.
- Update migration notes when changing deployment behavior.
```

### Path-scoped rule `<project>/.claude/rules/api-conventions.md`

```markdown
---
paths:
  - "src/api/**"
  - "*.graphql"
---

# API conventions

- All API handlers must validate input using the shared `validate()` helper.
- Response envelopes follow `{ data, error, meta }`.
- Do not bypass the rate-limit middleware without explicit justification.
```

If your templates are much larger than this by default, you are probably front-loading too much context.

---

## Claude Code in CI and Automation

### Verified platform behavior

Claude Code has a non-interactive mode invoked with the `-p` or `--print` flag:

```bash
claude -p "prompt"
```

This runs a single prompt to completion and exits. The memory discovery chain is the same as interactive mode: Claude Code rebuilds the `CLAUDE.md` chain on every run from wherever it is invoked.

Relevant flags and environment for automation:

- `--output-format json` streams structured output for machine parsing
- `--dangerously-skip-permissions` disables interactive approval prompts (use with care)
- `--permission-mode <mode>` selects permission behavior
- Claude Code reads the invoking shell's current working directory to determine project scope

### Recommended operating practice

CI runs do not need new instruction mechanics, but they do need different operator hygiene.

**Isolate the global layer.** Personal `~/.claude/CLAUDE.md` files are not CI policy. If your CI runs use a developer's home directory by accident, you inherit whatever personal working agreements that developer wrote. Point CI at a dedicated global layer by setting the home directory explicitly or by using a container with a controlled `~/.claude/`.

**Be explicit about the workspace root.** If your CI runner invokes Claude Code from a directory that is not the project root, project-scope discovery will start from that directory and may miss higher-level memory files. Set the working directory explicitly in the CI config, not implicitly through shell state.

**Never rely on auto memory in CI.** Auto memory is a per-user local-recall layer. It does not exist in a fresh CI runner. If your `CLAUDE.md` behavior depends on context that was only ever captured in a developer's personal auto memory, the CI run will diverge silently from local runs.

**Keep secrets out of `CLAUDE.md`.** Everything in the instruction chain is visible to the model and will appear in traces and logs. API keys, credentials, and secret URLs belong in environment variables, not in checked-in instruction files.

**Verify the chain explicitly at pipeline setup.** Before trusting CI behavior, run once with a smoke-test prompt:

```bash
claude -p --output-format json \
  "List the memory files you loaded and summarize their top three rules."
```

If the file list or the rule summary is not what you expected, the discovery chain is wrong. Fix it before running real tasks in CI.

**Treat the permission mode as part of the instruction contract.** Different permission modes produce materially different agent behavior for the same `CLAUDE.md` content. Document the permission mode you are running under in the CI config, and do not silently escalate it without an explicit justification.

---

## Migrating from Codex

If you work across Codex and Claude Code, the design principles overlap but the primitives differ. Use this table as a practical bridge, not as a promise of one-to-one semantics.

| Codex | Claude Code equivalent | Operational note |
|---|---|---|
| `~/.codex/AGENTS.md` | `~/.claude/CLAUDE.md` | Closest global analogue |
| `<project>/AGENTS.md` | `<project>/CLAUDE.md` | Closest repo analogue |
| `AGENTS.override.md` | `CLAUDE.local.md` | Similar local-override role; both carry the same invisibility risk |
| Nested `AGENTS.md` per directory | `.claude/rules/*.md` with path scoping | Claude Code uses explicit path patterns in frontmatter instead of directory walking |
| Codex memories (`~/.codex/memories/`) | Auto memory (`~/.claude/projects/.../memory/`) | Both are local-recall only; do not treat as team policy in either system |
| `codex/rules/*.rules` (Starlark) | Permissions in `settings.json` | Claude Code controls command approval through permissions, not a separate policy language |
| `project_doc_max_bytes` (hard cap) | Attention-based soft constraint | Claude Code does not publish an equivalent byte cap; the failure mode is adherence decay, not truncation |

If you maintain both systems, keep the high-level governance principles aligned while letting the repo architecture fit each agent's actual primitives.

---

## Operational Bottom Line

The best way to make Claude Code reliable is not to stuff more guidance into `CLAUDE.md`.

It is to give Claude Code a repo with clean boundaries:

- a small instruction layer
- a strong README layer
- a small canonical layer
- a hard archive boundary
- path-scoped rules only where local conventions diverge
- auto memory treated as optional recall only

That is what makes a Claude Code environment operational rather than merely verbose.

---

## Appendix A: Advanced Templates

These are not minimal examples. They are operational starting points for governed repos with canonical layers, multiple operators, and explicit archive boundaries.

### Advanced global `~/.claude/CLAUDE.md`

```markdown
# Working Agreements

- Be direct about uncertainty, tradeoffs, and weak evidence.
- Distinguish evidence, inference, and speculation when making claims.
- Ask before adding dependencies, changing external interfaces, or taking destructive actions.
- Prefer `rg` for search and exact project commands for test and lint steps.
- When canon changes, update the live workflow that depends on it.
- For messy repos or merged AI output, use audit -> implement -> clean.
```

### Advanced project `<project>/CLAUDE.md`

```markdown
# Repo Guidance

- Treat `canonical_index.md` as the authority map for live source documents.
- Use directory `README.md` files for navigation, current state, and handoff entry points.
- Keep `CLAUDE.md` focused on execution behavior; do not duplicate canon or archive inventories here.
- Preserve the archive boundary. Do not restore archived files to the active surface without explicit instruction.
- If methodology or terminology changes, update the affected canonical files and any live downstream workflow docs in the same pass.
- Before closing a task, verify that references, README pointers, and deprecation markers still resolve.
```

### Advanced path-scoped rule

Example: a published API package in a governed monorepo.

```markdown
---
paths:
  - "services/api/**"
  - "docs/api/**"
---

# Published API Package

- Treat `docs/api/spec.yaml` as the canonical contract. Update the spec before changing request or response shapes.
- When altering the public surface, update generated clients, examples, and migration notes in the same change set.
- Do not reintroduce endpoints marked under `archive/deprecated/` without explicit instruction.
- Run `pnpm test api` and regenerate clients after schema changes.
- Before closing a task, confirm that downstream docs and client samples still resolve against the updated spec.
```

The pattern illustrated here (canonical source, synchronized downstream consumers, hard archive boundary, explicit verification step) generalizes to other governed subprojects: SDK packages, data schema definitions, public contract directories, or any subdirectory where local changes have broader dependency consequences.

Use the minimal templates when you are starting from nothing. Use the advanced templates when the repo already has governance, canon, and multi-operator handoff requirements.

---

## Sources and Validation

This manual was grounded against:

- Anthropic Claude Code documentation for memory, settings, and slash commands
- local CLI inspection of current Claude Code builds at time of writing
- direct operator experience configuring Claude Code memory across research, commercial, and multi-agent workflows

Behavior may vary across Claude Code versions. Treat any version-specific claim here as time-stamped and verify against the current official documentation and your installed CLI before relying on it.

Official references:

- [Claude Code overview](https://docs.anthropic.com/en/docs/claude-code/overview)
- [Claude Code memory](https://docs.anthropic.com/en/docs/claude-code/memory)
- [Claude Code settings](https://docs.anthropic.com/en/docs/claude-code/settings)
- [Claude Code slash commands](https://docs.anthropic.com/en/docs/claude-code/slash-commands)
- [Claude Code headless mode](https://docs.anthropic.com/en/docs/claude-code/headless)

---

> **AI Disclosure:** This manual was developed with AI assistance (Claude Code) and then reviewed and revised against official Anthropic Claude Code documentation and local CLI behavior. AI-assisted drafting can introduce mistakes. Verify product-specific details against the current Claude Code docs and your installed CLI version before treating any behavior as stable.
