# Agent Memory Manuals

Practical manuals for giving coding agents persistent context that survives across sessions.

## What this is

A collection of operational manuals for configuring the persistent-context layer of different coding agents. Every agent in the collection solves the same underlying problem, how to give a stateless coding agent durable project context, but each tool uses different primitives and the operational patterns diverge where the primitives diverge.

These are not documentation dumps. They are opinionated, pared-down manuals built from direct operator experience and validated against vendor documentation.

## Manuals available

| Agent | Vendor | Directory | Primitive |
|---|---|---|---|
| Claude Code | Anthropic | [`claude-code/`](./claude-code/) | `CLAUDE.md` hierarchy plus auto memory |
| Codex | OpenAI | [`codex/`](./codex/) | `AGENTS.md` instruction chain with overrides, rules, and memories |

## Who these are for

Operators who want their coding agent to start work with durable context without turning the instruction layer into a second documentation system.

The target reader already understands the basics of the agent in question and wants to configure it well, not learn what it is. Expect operational prescription, not tutorial.

## Shared design principle

Every manual here repeats the same thesis because it is load-bearing: the persistent-context file is a **control layer, not a knowledge base**.

A coding agent's persistent-context file is not a README, not a wiki, and not a place to stash everything you want the agent to "know about" your project. It is a set of high-priority system instructions competing with the agent's own internal prompt for finite attention. Every line you add dilutes the rest.

The manuals differ on mechanics, not on this principle.

## Recommended reading order

If you only use one agent, read that manual.

If you use both, read the Claude Code manual first, then the Codex manual. The architectural patterns transfer, the primitives differ, and reading in that order makes the divergences easier to see. The Codex manual includes a migration table from Claude Code for operators making that jump.

## What is not in scope

- Forks, scripts, or automation tools for agent memory setup
- Plugins, hooks, or third-party memory backends
- API-level SDK documentation
- Agent selection advice (choose your agent based on your own evaluation; the manuals take it as given)

## Versions and validation

Each manual is version-pinned against its validation environment. Check the header of each manual for the current version and validation basis. If your installed CLI version differs meaningfully from what a manual was validated against, verify behavior against the relevant official documentation before treating anything as stable.

Current versions:

- Claude Code Memory Manual v2.0, April 20, 2026
- Codex AGENTS.md Manual v1.3, April 2026

Prior versions are archived under each manual's `archive/` directory for historical traceability. Do not cite or circulate archived versions; use the current manuals as authority.

## Author

Seni Kamara

## AI disclosure

These manuals were drafted with the assistance of coding agents and then reviewed and revised against official vendor documentation and local CLI behavior. AI-assisted drafting can introduce mistakes. Verify product-specific details against the current official documentation and your installed CLI version before treating any behavior as stable.

## Contributing

Issues and pull requests welcome, particularly corrections to verified-platform-behavior sections where vendor behavior has shifted. Operator-practice sections are more opinion-shaped and will be reviewed on a higher bar.

## License

See the [LICENSE](./LICENSE) file.
