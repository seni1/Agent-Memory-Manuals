# Claude Code Memory Manual

A practical manual for designing persistent Claude Code instructions that stay usable under real operating conditions.

## What this manual covers

- How Claude Code's memory layers (`CLAUDE.md` hierarchy plus auto memory) fit together across global, project, and local scopes
- How path-scoped rules with YAML frontmatter load alongside `CLAUDE.md`
- Why attention-based adherence decay is the real constraint, not a hard byte cap
- The boundary between `CLAUDE.md`, rules files, READMEs, canonical docs, auto memory, and `settings.json`
- Seven named failure modes operators run into in practice
- A verification workflow that goes beyond asking Claude Code to summarize its own instructions
- Considerations for multi-operator repos and Claude Code in CI or non-interactive automation
- Minimal and advanced authoring templates for global, project, and path-scoped files
- A migration table for operators coming from Codex

## Who this is for

Operators using Claude Code who want the agent to start sessions with durable, well-scoped context, without reducing instruction adherence by overloading the memory layer.

## Shared principle

`CLAUDE.md` is a control layer, not a knowledge base. Keep it short, load-bearing, and focused on execution behavior. Put navigation in READMEs, factual authority in canonical docs, path-scoped governance in rules files, and runtime configuration in `settings.json`.

## Read the manual

[Claude_Code_Memory_Manual.md](./Claude_Code_Memory_Manual.md)

## Related

- [Codex AGENTS.md Manual](../codex/) — the same problem space for OpenAI's Codex
- [Official Claude Code documentation](https://docs.anthropic.com/en/docs/claude-code/overview)

## Version

v2.0, April 20, 2026. Check the manual header for current validation basis.

Prior versions are archived under [`archive/`](./archive/) for historical traceability. Do not cite or circulate archived versions; use the current manual as authority.
