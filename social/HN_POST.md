# Show HN: effectorHQ — Typed interfaces for AI agent tools (MCP, OpenAI, LangChain)

## Title options (pick one)

- Show HN: Typed interfaces for AI agent tools — effector.toml
- Show HN: We added type safety to MCP tool composition
- Show HN: effectorHQ — A type system for AI agent capabilities

## Body

Hey HN — I built a typed interface layer for AI agent tools.

**The problem:** MCP tools (and OpenAI function calls, LangChain tools, etc.) have descriptions and JSON Schema inputs, but no type contracts. When you compose tools in a chain, there's no way to statically verify that Tool A's output matches Tool B's expected input. Type mismatches fail silently at runtime.

**The solution:** `effector.toml` — a sidecar manifest you drop next to any tool:

```toml
[effector]
name = "code-review"
type = "skill"
description = "Review code diffs"

[effector.interface]
input  = "CodeDiff"
output = "ReviewReport"

[effector.permissions]
network = false
```

From this file, the toolchain gives you:

- **Type checking** — verify interfaces against a 40-type catalog
- **Composition checking** — statically verify tool chains before running them
- **Permission auditing** — compare declared permissions against actual code behavior
- **Cross-runtime compilation** — compile to MCP, OpenAI Agents, or LangChain from one source
- **Interactive graph** — D3 force-directed visualization of the capability graph
- **CI validation** — GitHub Action that posts inline annotations on PRs

Auto-generate from an existing MCP server: `effector init --from-mcp .`

The whole thing is 9 packages, 265+ tests, zero npm dependencies (all Node.js built-ins). Apache 2.0 licensed.

Looking for feedback and MCP server maintainers willing to try it out.

GitHub: https://github.com/effectorHQ
