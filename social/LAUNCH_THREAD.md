# Launch Thread (Twitter/X)

## Tweet 1 — Hook

We analyzed 13,729 AI agent tools across ClawHub.

67% have no declared interface. No input types, no output types, no permission contracts.

When agents chain these tools, type mismatches fail silently at runtime.

We built something to fix this. Thread ↓

## Tweet 2 — Problem

MCP tools today = JSON blob in, JSON blob out.

You can't statically check if Tool A's output can feed into Tool B's input. You find out at runtime. Or worse — in production.

This is where 2024-era type safety stopped.

## Tweet 3 — Solution

effector.toml — a sidecar manifest you drop next to any MCP tool:

```toml
[effector.interface]
input  = "CodeDiff"
output = "ReviewReport"

[effector.permissions]
network = true
env-read = ["GITHUB_TOKEN"]
```

Zero code changes. Zero new dependencies. 5 minutes to add.

## Tweet 4 — What you get

From that one file:

✓ Static composition checking (does A → B type-check?)
✓ Permission auditing (declared vs actual behavior)
✓ Cross-runtime compilation (MCP → OpenAI → LangChain)
✓ Interactive capability graph (D3 force visualization)
✓ CI validation (GitHub Action)

## Tweet 5 — Auto-generate

Already have an MCP server? Generate effector.toml automatically:

```bash
effector init --from-mcp .
```

Scans your code, detects tool definitions, infers types and permissions. Review the output, commit, done.

## Tweet 6 — The ecosystem

9 packages, 265+ tests, zero npm dependencies.

- @effectorhq/core — parse, validate, compile
- @effectorhq/graph — D3 interactive capability graph
- @effectorhq/studio — visual tool builder
- effectorHQ/effector-action — CI validation

All Apache 2.0 licensed. All open source.

## Tweet 7 — CTA

We're looking for MCP server maintainers who want to try typed interfaces.

Add effector.toml to your project → get type checking, composition verification, and permission auditing for free.

github.com/effectorHQ
