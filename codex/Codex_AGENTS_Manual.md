# Codex AGENTS.md Manual
**How to design persistent Codex instructions that remain usable under real operating conditions**

*Version 1.3 - April 20, 2026*
*Author: Seni Kamara*
*Distribution edition, grounded in OpenAI Codex documentation and local CLI behavior*

---

## Purpose

`AGENTS.md` is a control layer, not a knowledge base.

That is the thesis of this manual.

Codex does not begin with your project context loaded into working memory. It has to discover guidance from the repo, the runtime, and the current session. If you use `AGENTS.md` to carry everything, it turns into a second wiki and starts failing at the job it is supposed to do.

This manual is for people who want Codex to start work with durable context without turning the instruction layer into a second documentation system.

If you get the boundaries wrong, Codex will still work, but it will work inconsistently. The failure mode is not total breakdown. It is quiet drift: wrong file choices, stale assumptions, hidden overrides, and overconfident synthesis.

This manual separates:

- **Verified platform behavior**: grounded in OpenAI Codex documentation and local CLI inspection
- **Recommended operating practice**: judgment about how to use the platform well

Do not confuse the two.

---

## What AGENTS.md Is

### Verified platform behavior

Codex reads `AGENTS.md` files before starting work and builds an instruction chain from them. This is a checked-in, user-controlled persistence layer for project guidance.

In many Codex environments, other instruction and context surfaces also exist:

- built-in system behavior
- developer and app-level instructions
- skills, tools, and runtime context
- optional memories
- the live user prompt

So `AGENTS.md` is not the whole control surface.

### Recommended operating practice

Treat `AGENTS.md` as a **control layer**, not a knowledge base.

In practice, it is the most important durable repository-layer instruction surface you explicitly author.

Use it for:

- durable working rules
- repo-specific conventions
- active priorities that affect execution
- terminology Codex is likely to misuse
- key commands Codex should run or avoid

Do not use it for:

- long-form project explanation
- broad historical context
- directory navigation
- canonical domain truth
- archive inventories

Those belong elsewhere.

---

## What Belongs Where

### Recommended operating practice

Use each layer for a different job:

| Layer | Job | Typical content |
|---|---|---|
| `AGENTS.md` | Behavioral control | How Codex should work here |
| `README.md` | Navigation | What this directory is, where to start, what not to touch |
| Canonical docs | Domain truth | Definitions, methodology, policy, approved source text |
| `codex/rules/*.rules` | Execution policy | Which out-of-sandbox commands can run |
| Memories | Local recall | Helpful carry-forward context, never load-bearing |

If you put navigation or factual authority into `AGENTS.md`, you burn instruction budget on content that should be discoverable. If you put behavioral rules only in README files, you make them easier to miss and easier to dilute.

Operationally, the best pattern is:

1. keep `AGENTS.md` short and load-bearing
2. keep READMEs explicit and navigational
3. keep canon small and authoritative
4. keep archive boundaries hard

---

## How Codex Discovers AGENTS.md

### Verified platform behavior

According to the OpenAI Codex AGENTS guide:

1. **Global scope**
   Codex checks your Codex home directory, usually `~/.codex`. If `AGENTS.override.md` exists, Codex uses that. Otherwise it uses `AGENTS.md`. It uses the first non-empty file at this level.
2. **Project scope**
   Starting at the project root, typically the Git root, Codex walks down to the current working directory. In each directory it checks:
   - `AGENTS.override.md`
   - `AGENTS.md`
   - fallback filenames from `project_doc_fallback_filenames`
   Codex includes at most one file per directory.
3. **Merge order**
   Codex concatenates files from root to current directory, with files nearer the current directory appearing later in the combined prompt.
4. **Size cap**
   Codex stops adding files once the combined size hits `project_doc_max_bytes`, which defaults to 32 KiB.
5. **Empty files**
   Empty files are skipped.

### Recommended operating practice

This has three consequences:

1. A nested instruction file is not stronger because it is "more specific" in the abstract. It is stronger because it appears later in the assembled prompt.
2. A hidden `AGENTS.override.md` is a governance risk, because it can quietly short-circuit the expected file at that directory level.
3. A large earlier file can push later files out of the prompt entirely.

---

## AGENTS.override.md

### Verified platform behavior

`AGENTS.override.md` takes precedence over `AGENTS.md` at the same directory level. At the global level, if `~/.codex/AGENTS.override.md` exists, Codex reads it instead of `~/.codex/AGENTS.md`. In project scope, Codex checks the override first in each directory.

### Recommended operating practice

Use overrides sparingly.

Good uses:

- temporary client-specific behavior
- a subproject that genuinely conflicts with parent instructions
- a short-lived special working mode you will remove afterward

Bad uses:

- "just in case" experiments
- shadow governance nobody remembers
- replacing a parent file because it became bloated

An override is not just a stronger instruction file. It is a bypass. That makes it useful, but also dangerous.

If you use one, document it in the nearest checked-in README or repo-level AGENTS file so another operator is not debugging invisible prompt state.

---

## Size and Attention Budget

### Verified platform behavior

The OpenAI docs explicitly document a combined size cap controlled by `project_doc_max_bytes`, defaulting to 32 KiB.

### Recommended operating practice

The hard cap is real, and the main hazard is silent truncation.

If a large global file or repo file grows enough, a later nested file can simply stop being included. That means a local override or subproject instruction layer can disappear without an obvious warning while the rest of the chain still looks healthy.

Treat this as a production risk, not a formatting nuisance.

The softer problem arrives earlier: once instructions become verbose, repetitive, or multi-purpose, adherence gets worse even before truncation.

That does **not** mean you should invent a fake universal line limit. Heuristics like "20 to 40 lines" or "under 200 lines" are not platform law.

Use these as operator heuristics instead:

- Global `AGENTS.md`: only stable personal or team defaults
- Repo `AGENTS.md`: only what would cause concrete execution mistakes if missing
- Nested `AGENTS.md`: only local divergences
- Overrides: smaller than the file they replace

If a file keeps growing, the first question is not "Can I raise the cap?" It is "What content belongs in README or canon instead?"

Compression beats expansion almost every time.

---

## Memories

### Verified platform behavior

OpenAI's Codex memories docs state:

- memories are off by default
- memories were not available at launch in the EEA, UK, or Switzerland
- memories update in the background, not immediately at thread end
- memory files live under `~/.codex/memories/`
- `/memories` controls thread-level memory behavior

### Recommended operating practice

Treat memories as **local recall**, not governance.

Good uses:

- recurring preferences
- stable workflows
- personal defaults you do not want to restate every time

Bad uses:

- required repo policy
- team rules
- anything another operator must be able to inspect in the repository

If a future session must know it, check it into the repo or place it in the global `AGENTS.md`. Do not rely on memory generation timing.

---

## AGENTS.md vs Rules

### Verified platform behavior

OpenAI documents `codex/rules/*.rules` as a separate system for controlling which commands Codex can run outside the sandbox. Rules are written in Starlark with `prefix_rule()`. The rules docs explicitly mark the feature as experimental.

### Recommended operating practice

Do not use `AGENTS.md` to do the job of rules.

`AGENTS.md` can express preferences like:

- prefer `rg` over `grep`
- ask before installing dependencies
- avoid destructive commands unless explicitly requested

But those are still instructions in natural language. If the real requirement is enforcement of out-of-sandbox command approval behavior, use `rules`.

Likewise, do not put coding conventions into `.rules` files. They are the wrong mechanism.

Think of the split this way:

- `AGENTS.md` tells Codex how to behave in work
- `rules` tells Codex what it may execute outside the sandbox

---

## The Right Design Pattern

### Recommended operating practice

For most repos, the winning pattern is:

1. **Global baseline**
   Personal or team defaults in `~/.codex/AGENTS.md`
2. **Repo instruction layer**
   Repo-specific working rules in `<repo>/AGENTS.md`
3. **Nested instructions only where needed**
   Add subdirectory `AGENTS.md` files only when local conventions genuinely diverge
4. **README layer**
   Every human-facing directory should explain purpose, authority, and start points
5. **Canonical layer**
   Small set of authoritative docs for actual domain truth
6. **Archive layer**
   Old or deprecated material behind a clear boundary

The instruction chain should be small. The repo itself can still be richly documented through README and canonical docs.

---

## What to Put in AGENTS.md

### Recommended operating practice

Put in:

- exact build, test, lint, and review commands
- stack-specific conventions Codex is likely to miss
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
- motivational prose
- rules you do not actually enforce

Useful litmus test:

> If this line disappeared, would Codex be materially more likely to make a concrete mistake in the next session?

If the answer is no, remove it from `AGENTS.md`.

---

## Failure Modes

### Recommended operating practice

These are the ones that matter in practice:

### 1. The AGENTS wiki failure

You keep adding context until `AGENTS.md` becomes a repository-in-miniature. The file gets harder to audit, weaker as instruction, and more likely to crowd out nested guidance.

### 2. The hidden override failure

An `AGENTS.override.md` survives from a prior engagement and silently short-circuits normal behavior.

### 3. The stale canon failure

`AGENTS.md` says how to act, but the canonical documents no longer reflect the current truth. Codex obeys instructions against outdated authority.

### 4. The README gap failure

The instruction layer is clean, but the repo has poor navigation. Codex then compensates by guessing which files matter.

### 5. The memory substitution failure

An operator starts trusting memories to carry required policy across threads. Another operator cannot inspect or share that context.

### 6. The false verification failure

Codex correctly summarizes a rule when asked, but still does not operate cleanly because another layer, missing file, or hidden override is shaping behavior.

---

## Multi-Operator Repos

### Recommended operating practice

When more than one person works with Codex against the same repo, the governance surface changes. Three invisible layers start to matter.

**Global `AGENTS.md` is personal state.** Every operator has their own `~/.codex/AGENTS.md`, and you cannot see what another operator has in theirs. Their global file may contradict, soften, or strengthen your repo guidance in ways you never observe. The checked-in repo `AGENTS.md` is the only layer the team can actually review together. Treat it as the team contract. Do not put team-required rules only in your global file.

**Overrides are local hazards in a shared repo.** If a teammate creates `AGENTS.override.md` in a subdirectory and commits it, it silently replaces the `AGENTS.md` another teammate might have expected to inherit. Hidden overrides are the single most common cause of "Codex behaves differently on my machine." Make override audits part of code review, not just incident response.

**Memories do not transfer between operators.** Memory files live under each operator's `~/.codex/memories/`. They are private. Anything captured there exists only for that user. This is not a limitation to work around: it is the right design for a personal recall layer. But it means team policy can never ride on memories. If the team must know it, check it in.

### Team hygiene checklist

Before treating a repo as operator-ready for shared Codex use:

1. **Audit for committed overrides.**
   Run `find . -name AGENTS.override.md` at the repo root. Each result should have a documented reason in the nearest README or in the repo-level `AGENTS.md`.
2. **Check `codex/rules/` in.**
   The rules layer is team-shared execution policy. Commit it to the repo so every operator gets the same command-approval behavior. Do not leave critical rules in personal `~/.codex/rules/`.
3. **Document fallback filename choices.**
   If the repo uses non-default instruction filenames via `project_doc_fallback_filenames`, document the choice in the repo README so new operators know why their Codex ignores files they expected to matter.
4. **Treat the checked-in chain as the reviewable artifact.**
   Everything else is operator-local. Code review, incident response, and onboarding all run against the checked-in layer. If a behavior is not reproducible from the checked-in files alone, it is not team policy.
5. **Name the archive boundary explicitly.**
   Multi-operator repos accumulate deprecated directories faster than single-operator ones. Make archive paths discoverable (e.g., `archive/` or `deprecated/`) and name them in the repo `AGENTS.md` so Codex does not promote old material back to the active surface under one operator but not another.

The short version: anything that is not in the repo cannot be team policy. Every operator-local layer is an invisibility risk.

---

## Verification Workflow

Asking Codex to summarize its instructions is only a smoke test. It is useful, but it is not strong verification.

Use this order instead.

### 1. Inspect the actual prompt surface when possible

```bash
codex debug prompt-input
```

This is the best direct check when available, because it shows the model-visible prompt input list rather than asking the model to describe itself.

On this manual's validation environment, this command is present in `codex-cli 0.122.0-alpha.1`. Treat it as version-dependent and confirm availability on your own install.

### 2. Run a smoke test

```bash
codex --ask-for-approval never "Summarize the current instructions."
```

This tells you whether Codex has at least absorbed the expected guidance. It does not prove the chain is complete or uncontested.

### 3. Check instruction discovery settings

Look at:

- `project_doc_fallback_filenames`
- `project_doc_max_bytes`
- `project_root_markers`
- `CODEX_HOME`

These govern what Codex can discover and how much it reads.

### 4. Check for hidden overrides

Search for:

```bash
find . -name AGENTS.md -o -name AGENTS.override.md
```

Then inspect whether any override is replacing a file you expected to load.

### 5. Test rules separately

```bash
codex execpolicy check --pretty --rules ~/.codex/rules/default.rules -- gh pr view 123
```

Do not use AGENTS behavior as a proxy for rule behavior. They are separate systems.

### 6. Inspect session logs if needed

If behavior still looks wrong, use Codex logs and startup diagnostics to see what loaded.

---

## Practical Configuration Knobs

### Verified platform behavior

The Codex config reference documents these keys:

- `project_doc_fallback_filenames`
- `project_doc_max_bytes`
- `project_root_markers`
- `[features].memories`
- `memories.generate_memories`
- `memories.use_memories`

### Recommended operating practice

Do not raise `project_doc_max_bytes` as your first move.

Raise it only when:

- your instruction chain is already clean
- your repo genuinely needs extra scoped guidance
- you have checked for duplication across levels

Otherwise you are using configuration to mask poor instruction design.

---

## Example Layout

```text
~/.codex/
├── AGENTS.md
├── AGENTS.override.md                # temporary only
└── rules/
    └── default.rules

<repo>/
├── AGENTS.md
├── README.md
├── canonical_index.md                # optional but useful in governed repos
├── codex/
│   └── rules/
│       └── team.rules
├── docs/
│   └── methodology/
│       └── README.md
└── services/
    └── payments/
        ├── AGENTS.md
        └── README.md
```

Interpretation:

- `AGENTS.md` controls execution behavior
- `README.md` handles navigation
- canonical docs hold actual source truth
- `rules` handles out-of-sandbox execution policy

---

## Recommended Authoring Standard

Use this checklist when editing any `AGENTS.md`:

1. Is the line durable across many sessions?
2. Would its absence cause a real execution mistake?
3. Is this instruction better than putting the same content in README?
4. Is it local to this directory, or duplicated from a parent?
5. Is it short enough to survive alongside other instructions?

If you cannot justify a line on those terms, it probably does not belong.

---

## Minimal Templates

### Global `~/.codex/AGENTS.md`

```markdown
# Working Agreements

- Be direct about uncertainty and tradeoffs.
- Ask before adding new dependencies.
- Prefer `rg` for search and run tests after code changes.
- Flag destructive actions before executing them.
```

### Repo `<repo>/AGENTS.md`

```markdown
# Repo Guidance

- Use `pnpm test` and `pnpm lint` after TypeScript edits.
- Keep API schema changes synchronized with `docs/api/README.md`.
- Treat `docs/canon/` as authoritative for product terminology.
- Update migration notes when changing deployment behavior.
```

### Nested `<repo>/subdir/AGENTS.md`

```markdown
# Payments Subproject

- Do not change currency rounding behavior without explicit confirmation.
- Run `pnpm test payments` after edits in this directory.
```

If your templates are much larger than this by default, you are probably front-loading too much context.

---

## Codex in CI and Automation

### Verified platform behavior

Codex has a dedicated non-interactive mode: `codex exec` (alias `codex e`). It runs a single prompt to completion and exits. The instruction discovery chain is the same as interactive mode: Codex rebuilds the `AGENTS.md` chain on every run. There is no separate CI-specific discovery path.

Key flags and environment variables for automation:

- `--cd <path>` sets the workspace root before execution. This determines project-scope instruction loading.
- `CODEX_HOME` relocates the Codex home directory for the run. This determines global-scope instruction loading.
- `--ask-for-approval never` disables interactive approval prompts. Required for non-interactive runs.
- `--sandbox <policy>` selects the sandbox level (`read-only`, `workspace-write`, `danger-full-access`). `read-only` is the default for `codex exec`.
- `--full-auto` is a preset: workspace-write sandbox plus on-request approvals.
- `--json` streams JSON Lines events to stdout for machine parsing.
- `-o <file>` / `--output-last-message` writes the final assistant message to a file.
- `--skip-rollout` disables session persistence under `~/.codex/sessions/`.
- `CODEX_API_KEY` provides credentials for `codex exec` runs without reusing saved CLI authentication.

### Recommended operating practice

CI runs do not need new instruction mechanics, but they do need different operator hygiene. The environment is stateless, the identity is shared, and the failure modes are different.

**Isolate the global layer.** Personal `~/.codex/AGENTS.md` files are not CI policy. If your CI runs use a developer's home directory by accident, you inherit whatever personal working agreements that developer wrote. Point CI at a dedicated global layer:

```bash
CODEX_HOME=$(pwd)/.codex-ci codex exec --ask-for-approval never "<prompt>"
```

Commit `.codex-ci/AGENTS.md` alongside the rest of the CI config. This way the CI global layer is reviewable with the rest of the pipeline.

**Be explicit about the workspace root.** If your CI runner invokes Codex from a directory that is not the project root, project-scope discovery will start from that directory and may miss higher-level `AGENTS.md` files. Set `--cd` explicitly rather than relying on the invoking shell's working directory.

**Never rely on memories in CI.** Memories are a per-user local-recall layer. They do not exist in a fresh CI runner. If your AGENTS.md behavior depends on context that was only ever captured in a developer's personal memories, the CI run will diverge silently from local runs.

**Keep secrets out of `AGENTS.md`.** Everything in the instruction chain is visible to the model and will appear in traces and logs. API keys, credentials, and secret URLs belong in environment variables, not in checked-in or CI-side instruction files.

**Verify the chain explicitly at pipeline setup.** Before trusting CI behavior, run once with a smoke-test prompt and capture the output:

```bash
codex exec --ask-for-approval never --output-last-message /tmp/check.txt \
  "List the instruction files you loaded and summarize their top three rules."
```

If the file list or the rule summary is not what you expected, the discovery chain is wrong. Fix it before running real tasks in CI.

**Treat the sandbox policy as part of the instruction contract.** `--sandbox read-only` and `--sandbox workspace-write` produce materially different agent behavior for the same `AGENTS.md` content. Document the sandbox level you are running under in the CI config, and do not silently escalate it to `danger-full-access` without an explicit justification.

---

## Migrating from Claude Code

If you work across Claude Code and Codex, the design principles overlap but the primitives differ. Use this table as a practical bridge, not as a promise of one-to-one semantics.

| Claude Code | Codex equivalent | Operational note |
|---|---|---|
| `~/.claude/CLAUDE.md` | `~/.codex/AGENTS.md` | Closest global analogue |
| `<project>/CLAUDE.md` | `<project>/AGENTS.md` | Closest repo analogue |
| Path-scoped rules markdown | Nested `AGENTS.md` plus README/canon split | Codex does not use Claude's same path-scoped markdown instruction model |
| `CLAUDE.local.md` | `AGENTS.override.md` | Similar local-override role, but treat Codex overrides as sharper tools |
| `/memory` style recall use | `/memories` plus `~/.codex/memories/` | Codex memories are optional and should not carry load-bearing governance |
| Inline imported context patterns | README plus canonical docs | Codex works better when navigation and authority live outside `AGENTS.md` |
| Large instruction stack in one doc | Small `AGENTS.md` plus strong repo surfaces | Codex rewards tighter instruction layers and clearer repo boundaries |

If you maintain both systems, keep the high-level governance principles aligned while letting the repo architecture fit each agent's actual primitives.

---

## Operational Bottom Line

The best way to make Codex reliable is not to stuff more guidance into `AGENTS.md`.

It is to give Codex a repo with clean boundaries:

- a small instruction layer
- a strong README layer
- a small canonical layer
- a hard archive boundary
- rules for execution policy
- memories treated as optional recall only

That is what makes a Codex environment operational rather than merely verbose.

---

## Appendix A: Advanced Templates

These are not minimal examples. They are operational starting points for governed repos with canonical layers, multiple operators, and explicit archive boundaries.

### Advanced global `~/.codex/AGENTS.md`

```markdown
# Working Agreements

- Be direct about uncertainty, tradeoffs, and weak evidence.
- Distinguish evidence, inference, and speculation when making claims.
- Ask before adding dependencies, changing external interfaces, or taking destructive actions.
- Prefer `rg` for search and exact project commands for test and lint steps.
- When canon changes, update the live workflow that depends on it.
- For messy repos or merged AI output, use audit -> implement -> clean.
```

### Advanced repo `<repo>/AGENTS.md`

```markdown
# Repo Guidance

- Treat `canonical_index.md` as the authority map for live source documents.
- Use directory `README.md` files for navigation, current state, and handoff entry points.
- Keep `AGENTS.md` focused on execution behavior; do not duplicate canon or archive inventories here.
- Preserve the archive boundary. Do not restore archived files to the active surface without explicit instruction.
- If methodology or terminology changes, update the affected canonical files and any live downstream workflow docs in the same pass.
- Before closing a task, verify that references, README pointers, and deprecation markers still resolve.
```

### Advanced subproject `<repo>/subdir/AGENTS.md`

Example: a published API package in a governed monorepo.

```markdown
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

- OpenAI Codex docs for `AGENTS.md`
- OpenAI Codex docs for memories
- OpenAI Codex docs for rules
- OpenAI Codex docs for non-interactive mode
- OpenAI Codex config reference
- OpenAI Codex CLI slash commands docs
- local CLI inspection (`codex-cli 0.122.0-alpha.1` at time of writing; behavior may vary across versions)

Official references:

- [Custom instructions with AGENTS.md](https://developers.openai.com/codex/guides/agents-md)
- [Memories](https://developers.openai.com/codex/memories)
- [Rules](https://developers.openai.com/codex/rules)
- [Non-interactive Mode](https://developers.openai.com/codex/noninteractive)
- [Configuration Reference](https://developers.openai.com/codex/config-reference)
- [Slash commands in Codex CLI](https://developers.openai.com/codex/cli/slash-commands)

---

> **AI Disclosure:** This manual was developed with AI assistance (Codex CLI) and then reviewed and revised against official OpenAI Codex documentation and local CLI behavior. AI-assisted drafting can introduce mistakes. Verify product-specific details against the current OpenAI Codex docs and your installed CLI version before treating any behavior as stable.
