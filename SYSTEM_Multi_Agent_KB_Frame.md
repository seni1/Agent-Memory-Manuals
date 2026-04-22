# System Multi-Agent KB Frame

## Purpose

This repository should be read as a knowledgebase governance and memory administration system designed for coding-agent operation.

That means the top-level story is not "Codex in a repo" or "Claude in a repo." Those are adapter surfaces. The system itself is the governed knowledge layer that multiple agents can operate against safely.

This document captures that framing explicitly so downstream repos can adapt it after review rather than improvising it piecemeal.

## Verdict

If the real objective is agent-driven knowledgebase and memory administration, then a Codex manual that reads as standalone is strategically off-frame.

That framing is not necessarily wrong, but it is incomplete in a way that can become misleading. It suggests "Codex operations in a repo" when the deeper reality is "multi-agent governance of a knowledge system, with Claude first and Codex added as a second operational surface."

## Evidence

- The real work in these systems is usually canon control, archive boundaries, README navigation, handoff logic, deprecation control, and memory and instruction surfaces across agents.
- The strongest parts of Codex operating guidance are often not uniquely Codex ideas. They are shared governance ideas implemented through Codex primitives.
- Once those shared governance ideas are mistaken for a Codex-specific worldview, the repo starts reading tool-first instead of system-first.

## Risks of a Standalone Agent Frame

### 1. Hidden architecture risk

Future readers may think the repo is primarily about one agent's operations when it is really about governing a KB so multiple coding agents can act on it safely.

### 2. Duplication risk

Once each agent gets a standalone manual, shared governance starts getting restated in parallel, and drift becomes likely.

### 3. Design-narrowing risk

The question shifts from "what must be true for any agent to work safely here?" to "how do I make a given agent behave?" That is a smaller and weaker design frame.

## Recommended Abstraction Boundary

The repo should read as:

- a knowledgebase governance and memory administration system
- designed for coding-agent operation
- with a shared governance substrate
- and agent-specific operational adapters for Claude Code, Codex, and later agents

This is the right boundary because it keeps agent-specific operating manuals in adapter position rather than letting them define the whole system.

## Recommended Three-Layer Model

### 1. System layer

Defines the repo's actual purpose, canon rules, archive logic, navigation doctrine, handoff model, and memory philosophy.

### 2. Shared agent layer

Defines cross-agent rules that should hold regardless of vendor or execution harness.

Examples:

- small trusted canon
- explicit status labels
- verification over self-report
- no hidden authority
- hard archive boundaries
- human bottleneck awareness
- update downstream workflows when canon changes

### 3. Agent adapter layer

Defines how a specific agent plugs into the system.

Examples:

- Claude-specific operating manual
- Codex-specific operating manual
- tool-specific verification notes
- surface-specific instruction patterns

This keeps any one manual from pretending to be the system itself.

## Concrete Document Architecture

Recommended structure for this repo:

1. System documents
2. Shared governance documents
3. Agent adapter documents
4. Review and critique documents
5. Examples or downstream implementation patterns

Interpretation:

- system docs define the worldview
- shared docs define cross-agent governance
- agent docs explain how a given platform fits into that worldview
- review docs capture critique without becoming canon
- downstream repos adapt the framework after review rather than inventing it piecemeal

## Naming Scheme

### System docs

Prefix: `SYSTEM_`

Examples:

- `SYSTEM_Multi_Agent_KB_Frame.md`
- `SYSTEM_KB_Memory_Policy.md`
- `SYSTEM_Document_Architecture.md`

### Shared governance docs

Prefix: `AGENT_SHARED_`

Examples:

- `AGENT_SHARED_Governance_Principles.md`
- `AGENT_SHARED_Memory_Administration.md`
- `AGENT_SHARED_Verification_Discipline.md`

### Agent adapter docs

Prefix: `AGENT_<PLATFORM>_`

Examples:

- `AGENT_CLAUDE_Operating_Manual.md`
- `AGENT_CODEX_Operating_Manual.md`

### Review docs

Prefix: `REVIEW_`

Examples:

- `REVIEW_CODEX_Operating_Manual.md`
- `REVIEW_Memory_Policy.md`

### Experimental or idea docs

Prefix: `IDEA_` or place under an explicit draft/examples zone.

These should not sit beside system canon without status marking.

## Working Rule

Adapt the frame in this manuals repo first. Do not push it into downstream operational repos until the framing has been reviewed and accepted.

That keeps conceptual architecture work separate from live repo governance work.

## Bottom Line

The Codex manual should not stand as the repo's implied worldview. It should sit beneath a higher-order agent-governance frame for the knowledgebase.

Otherwise the repo will slowly optimize for tool-specific instruction manuals instead of the actual thing being built: a governed operational memory system for multiple coding agents.
