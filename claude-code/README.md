# Claude Code Memory Manual

A practical manual for giving Anthropic's Claude Code agent persistent context that actually works.

## What this manual covers

- How Claude Code's two memory layers (`CLAUDE.md` files you write plus auto-memory Claude Code writes for itself) fit together
- The file hierarchy across global, project, and local scopes, and how path-scoped rules work
- Why the 200-line soft constraint exists and how adherence degrades before truncation
- Design patterns for separating factual context from behavioral governance
- Compression strategies for governance documents that grow too large
- Maintenance cadence and how CLAUDE.md behaves under `/compact`
- Verification commands to confirm your setup is working

## Who this is for

Operators using Claude Code who want the agent to start sessions with durable, well-scoped context, without reducing instruction adherence by overloading the memory layer.

## Shared principle

Memory files are high-priority system instructions, not documentation. Every line you add competes with Claude Code's internal ~50-instruction system prompt for attention.

## Read the manual

[Claude_Code_Memory_Manual.md](./Claude_Code_Memory_Manual.md)

## Related

- [Codex AGENTS.md Manual](../codex/) — the same problem space for OpenAI's Codex
- [Official Claude Code documentation](https://docs.anthropic.com/en/docs/claude-code/overview)

## Version

v1.0, April 2026. Generated with Claude Opus 4.6 (Anthropic).
