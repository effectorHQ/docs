# Why add effector.toml to your MCP server?

## The problem

MCP tools have descriptions and JSON Schema inputs — but no **type contracts**. You can't statically verify:

- Whether two tools can compose (does tool A's output match tool B's input?)
- What permissions a tool actually needs (network? filesystem? env vars?)
- Whether a tool is safe to run in a given environment

When agents chain tools, type mismatches fail silently at runtime. Permission overreach goes unnoticed.

## The solution: one file, zero changes

Add an `effector.toml` next to your existing code:

```toml
[effector]
name = "your-tool"
version = "1.0.0"
type = "skill"
description = "What your tool does"

[effector.interface]
input  = "String"           # What goes in
output = "JSON"             # What comes out
context = ["GenericAPIKey"] # What's needed at runtime

[effector.permissions]
network    = true
subprocess = false
env-read   = ["API_KEY"]
```

That's it. Your tool keeps running exactly as before.

## What you get

| Capability | How |
|-----------|-----|
| **Type-safe composition** | `effector check .` verifies your types exist in the 42-type catalog |
| **Permission auditing** | `effector-audit scan .` compares declared permissions against actual code behavior |
| **Cross-runtime compilation** | `effector compile . -t mcp` / `openai-agents` / `langchain` / `json` |
| **CI validation** | Add `effectorHQ/effector-action@v1` to your GitHub Actions |
| **Visual discovery** | Your tool appears in the 42-type Spectrum with its type position |
| **Composition graph** | See how your tool connects to others in a capability graph |

## What it costs

- **1 file** (effector.toml)
- **0 code changes**
- **0 new dependencies**
- **0 runtime changes**
- **5 minutes** to set up (or 30 seconds with `effector init --from-mcp .`)

## 30-second quickstart

```bash
# Auto-generate from your existing MCP server
effector init --from-mcp .

# Review the generated effector.toml (check # TODO comments)
# Then validate:
effector check .
```

## Learn more

- [Effector Spec](https://github.com/effectorHQ/effector-spec) — the full specification
- [Type Catalog](https://github.com/effectorHQ/effector-types) — 42 standard types
- [Migration Guide](https://github.com/effectorHQ/docs/blob/main/guides/mcp-migration.md) — step-by-step
