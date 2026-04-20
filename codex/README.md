# Codex AGENTS.md Manual

A practical manual for designing persistent Codex instructions that stay usable under real operating conditions.

## What this manual covers

- How Codex discovers and assembles `AGENTS.md` files across global and project scopes
- Where the 32 KiB combined size cap becomes a silent-truncation risk
- How `AGENTS.override.md` differs from a regular instruction file and when to use it
- The boundary between `AGENTS.md`, `README.md`, canonical documentation, `rules`, and memories
- A verification workflow that goes beyond asking Codex to summarize its own instructions
- Six named failure modes operators run into in practice
- Considerations for multi-operator repos and Codex in CI or non-interactive automation
- Minimal and advanced authoring templates for global, repo, and subproject scope
- A migration table for users coming from Claude Code

## Who this is for

Operators using Codex CLI, Codex IDE, or Codex Cloud who want Codex to start work with durable context without turning the instruction layer into a second documentation system.

The design principles translate across coding agents that read `AGENTS.md`, but the mechanics documented here are Codex-specific. The `AGENTS.md` format itself is an open convention stewarded by the Agentic AI Foundation under the Linux Foundation and read by multiple coding agents, though behavior varies by implementation.

## Shared principle

`AGENTS.md` is a control layer, not a knowledge base. Keep it short, load-bearing, and focused on execution behavior. Put navigation in READMEs, factual authority in canonical docs, and command-execution policy in `rules`.

## Read the manual

[Codex_AGENTS_Manual.md](./Codex_AGENTS_Manual.md)

## Related

- [Claude Code Memory Manual](../claude-code/) — the same problem space for Anthropic's Claude Code
- [Official OpenAI Codex documentation](https://developers.openai.com/codex)
- [AGENTS.md open format](https://agents.md)

## Version

v1.3, April 2026. Validated against `codex-cli 0.122.0-alpha.1`. Check the manual header for current validation basis.
