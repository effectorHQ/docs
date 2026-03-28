---
title: "Why AI Agent Tools Need Types — And How We Built Them"
date: 2026-03-17
author: Jiayi Du
tags: [ai-agents, type-system, mcp, openai, langchain, effector]
---

# Why AI Agent Tools Need Types — And How We Built Them

AI agents are only as reliable as the tools they call. You can have the most capable LLM in the world, but if it chains two tools that are silently incompatible — wrong output format, missing context, permission mismatch — the whole pipeline fails. Quietly. At runtime. With no compiler to catch it.

We built effectorHQ to fix this.

## The problem: 67% of agent tool chains fail

We analyzed [13,729 ClawHub skills](https://github.com/effectorHQ/clawhub-analysis) — real-world tool definitions used by AI agents in production. The failure rate was **67%**. Not "sometimes flaky." Two out of three tool chains produced incorrect results, crashed, or silently returned garbage.

The root causes were consistent:

- **Untyped interfaces** — Tool A outputs a `string`. Tool B expects a `ReviewReport`. Nobody checks.
- **Missing prerequisites** — A tool requires `GitBranch` context but the caller doesn't provide it.
- **Permission mismatches** — A tool declares `network: false` but the downstream tool needs to make HTTP calls.

If this sounds familiar, it should. It's the same problem TypeScript solved for JavaScript. Before 2012, you wrote JavaScript and hoped your function received the right arguments. TypeScript added a type system, and an entire category of bugs disappeared at compile time.

AI agent tools are in the same "JavaScript 2010" era. We're chaining tools with no type checking, no schema validation, no compile-time verification. Just hope.

## What if tools had types?

Imagine every AI tool declares its input type, output type, and context requirements:

```toml
[effector]
name = "code-review"
version = "1.0.0"

[effector.interface]
input = "CodeDiff"
output = "ReviewReport"
context = ["Repository", "GitBranch"]

[effector.permissions]
network = true
subprocess = false
env-read = ["GITHUB_TOKEN"]
```

This is an **effector definition** — a typed sidecar that lives alongside your tool's implementation. It doesn't care what language your tool is written in or what runtime executes it. It just declares the contract.

Now a composition engine can answer questions before a single token is spent:

- *Can `code-review` feed into `slack-notify`?* — Check if `ReviewReport` is compatible with `Notification` (it is, via subtype relationship: `ReviewReport <: Notification`).
- *Does this pipeline have the context it needs?* — `code-review` requires `Repository` and `GitBranch`. Does the caller provide them?
- *Is the permission model consistent?* — `code-review` needs network access. Is that allowed by the orchestrator's policy?

All of this happens at **compile time**. No runtime surprises.

## 40 standard types, derived from real data

We didn't invent types in a vacuum. The [effector type catalog](https://github.com/effectorHQ/effector-types) was derived from statistical analysis of those 13,729 real-world skills:

- **15 input types**: `TextInput`, `CodeDiff`, `ImageInput`, `SearchQuery`, `URL`, `FilePath`, `APIRequest`, `DatabaseQuery`, `Configuration`, `UserMessage`, `Notification`, `Webhook`, `StructuredData`, `BinaryData`, `Credentials`
- **14 output types**: `TextOutput`, `ReviewReport`, `SecurityReport`, `AnalysisResult`, `CodeOutput`, `ImageOutput`, `SearchResults`, `APIResponse`, `DatabaseResult`, `FileOutput`, `StatusMessage`, `Notification`, `StructuredData`, `BinaryData`
- **11 context types**: `Repository`, `GitBranch`, `Workspace`, `Environment`, `UserSession`, `APICredentials`, `GitHubCredentials`, `DatabaseConnection`, `FileSystem`, `NetworkAccess`, `SystemInfo`

The type system includes **subtype relationships**:

```
SecurityReport <: ReviewReport <: AnalysisResult
GitHubCredentials <: APICredentials <: Credentials
```

This means a tool that accepts `ReviewReport` will also accept `SecurityReport` — structural subtyping, just like TypeScript interfaces.

## One definition, four runtimes

The real power isn't just validation — it's **cross-runtime compilation**. Write one effector definition, compile to any runtime:

```js
import { Effector } from '@effectorhq/core';

// Parse, validate, type-check, and compile in one chain
const mcp = Effector.fromDir('./my-tool').validate().compile('mcp');
const openai = Effector.fromDir('./my-tool').validate().compile('openai-agents');
```

The **MCP output** is a JSON-RPC tool schema ready for Claude, Cursor, or Windsurf:

```json
{
  "name": "code_review",
  "description": "Performs automated code review on pull request diffs",
  "inputSchema": {
    "type": "object",
    "properties": {
      "GITHUB_TOKEN": { "type": "string" }
    }
  }
}
```

The **OpenAI Agents output** wraps the same definition in their `FunctionTool` format:

```json
{
  "type": "function",
  "function": {
    "name": "code_review",
    "description": "Performs automated code review on pull request diffs",
    "parameters": { ... }
  }
}
```

Same tool. Same types. Different runtimes. One source of truth.

Need a runtime we don't support? The plugin system lets you register custom compile targets:

```js
import { registerTarget } from '@effectorhq/core';

registerTarget('crewai', (def, skill) => {
  return `from crewai.tools import BaseTool\n\nclass ${def.name}Tool(BaseTool):\n    name = "${def.name}"`;
});
```

## The toolkit: 9 packages, zero dependencies

effectorHQ is a modular ecosystem — use what you need, ignore the rest:

| Package | Purpose |
|---------|---------|
| `@effectorhq/core` | Parse, validate, type-check, compile |
| `@effectorhq/types` | 40 standard capability types |
| `@effectorhq/skill-lint` | Lint SKILL.md structure (16 rules) |
| `@effectorhq/skill-eval` | Quality scoring — A through F |
| `@effectorhq/audit` | Security scanner: injection, exfiltration, drift |
| `@effectorhq/compose` | Type-checked pipeline builder |
| `@effectorhq/graph` | Capability graph queries and visualization |
| `@effectorhq/skill-mcp` | SKILL.md → MCP bridge for Claude/Cursor |
| `@effectorhq/create-effector` | Scaffold any effector type in 30 seconds |

Every package is ESM-only, zero-dependency (only Node.js built-ins), and works on Node 18+. The entire core is 23KB.

## Get started in 30 seconds

```bash
# Install the core toolkit
npm install @effectorhq/core

# Scaffold a new tool
npx @effectorhq/cli init my-first-tool --type skill
cd my-first-tool

# Lint → Evaluate → Validate → Compile
effector check .
npx @effectorhq/skill-eval . --static-only
effector check .
effector compile . -t mcp
```

From zero to a Claude-ready MCP tool in under a minute.

## What's next

This is v1.0 — the foundation. Here's what we're building toward:

- **Runtime type validation** — Middleware that checks actual tool I/O against declared types at execution time, not just compile time.
- **More compile targets** — AutoGen, Semantic Kernel, CrewAI as first-class targets.
- **Discovery protocol** — How do agents find and negotiate capabilities? The spec has a draft; implementation is next.
- **Community type proposals** — The [RFC process](https://github.com/effectorHQ/rfcs) is open. If a type is missing, propose it.

The thesis is simple: AI agent capabilities need the same rigor we brought to web development with TypeScript. Type the tools. Verify the compositions. Compile to any runtime. Make the hands safer.

---

**Links**: [GitHub](https://github.com/effectorHQ) · [npm](https://www.npmjs.com/package/@effectorhq/core) · [v1.0 announcement](https://github.com/effectorHQ/.github/discussions/8) · [Spec](https://github.com/effectorHQ/effector-spec)
