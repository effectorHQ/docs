# Why Effector?

## The Problem

AI agents need capabilities (tools, skills, actions). Every framework defines these differently:

- **MCP** gives you a protocol but no type safety or composition checking
- **LangChain** locks you into Python with runtime-only validation
- **CrewAI** forces rigid role hierarchies with no cross-framework interop
- **Custom solutions** mean every team reinvents the wheel

None of them answer: *"Can these two capabilities actually work together?"*

## What Effector Does

Effector is a **typed capability layer** that sits alongside any runtime. It doesn't replace MCP, LangChain, or your custom agent framework — it adds:

1. **Static type checking** for capability interfaces
2. **Composition verification** before execution
3. **Cross-runtime portability** via compiler targets
4. **Type-indexed discovery** to find compatible capabilities

## Comparison

| Feature | Raw MCP | LangChain | CrewAI | Effector |
|---------|---------|-----------|--------|----------|
| Type safety | None | None | None | Full (42 standard types) |
| Composition checking | Manual | Runtime only | Rigid roles | Static verification |
| Cross-runtime | MCP only | Python only | Python only | MCP, OpenAI, LangChain, JSON IR |
| Discovery by type | No | No | No | Yes — find by input/output type |
| Zero dependencies | N/A | Heavy | Heavy | Yes (Node.js built-ins only) |
| Manifest format | JSON schema | Python classes | Python classes | TOML + SKILL.md |
| Permission model | None | None | None | Declared + audited |

## How It Works

### 1. Declare your capability

```toml
# effector.toml
[effector]
name = "code-review"
version = "0.1.0"

[effector.interface]
input = "CodeDiff"
output = "ReviewReport"
context = ["Repository"]

[effector.permissions]
network = false
subprocess = false
```

### 2. Write the instruction

```markdown
<!-- SKILL.md -->
---
name: code-review
description: Reviews code diffs and produces structured review reports
---

## Purpose
Analyze code diffs and produce structured review reports...
```

### 3. Validate

```bash
effector check .    # Validates SKILL.md structure
npx @effectorhq/audit scan .        # Security audit
```

### 4. Compile to any runtime

```bash
npx skill-mcp compile .          # → MCP tool schema
# Or import programmatically for OpenAI, LangChain, etc.
```

### 5. Compose with type checking

The type system knows that `CodeDiff` output from one skill can feed into another skill that accepts `CodeDiff` input. Mismatched types are caught before execution.

## When to Use Effector

- You're building AI agent capabilities that need to work across frameworks
- You want type safety without giving up your existing runtime
- You need to verify that capabilities compose correctly before deploying
- You want a standard way to discover and audit AI capabilities

## When NOT to Use Effector

- You only need a single tool with no composition
- You're building the runtime/orchestrator itself (Effector is a sidecar, not a runtime)
- You need sub-millisecond overhead (Effector adds a compilation step)

## Get Started

```bash
npx @effectorhq/cli init my-first-skill
cd my-first-skill
effector check .
```
