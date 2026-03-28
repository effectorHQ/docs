# PR Template: Add effector.toml

> Copy this template when filing a PR to add `effector.toml` to an external MCP server project.

---

## Add typed capability manifest (effector.toml)

### What this PR does

Adds an `effector.toml` sidecar manifest that declares this tool's typed interface, permissions, and metadata — following the [Effector specification](https://github.com/effectorHQ/effector-spec).

### Why

MCP tools today are untyped JSON blobs. `effector.toml` adds a thin type layer that enables:

- **Static composition checking** — verify two tools can connect before running them (output type of A matches input type of B)
- **Permission auditing** — `effector-audit` compares declared permissions against what the code actually does
- **Cross-runtime compilation** — compile to MCP, OpenAI Agents, or LangChain from one source of truth
- **Discovery** — search and filter tools by input/output type, not just name/description

### What changes

| File | Description |
|------|-------------|
| `effector.toml` | Typed capability manifest (6 fields minimum) |

### What does NOT change

- **Zero runtime behavior changes** — the MCP protocol, tool execution, and existing tests are completely unaffected
- **Zero new dependencies** — `effector.toml` is a passive metadata file
- **Zero breaking changes** — this is purely additive

### How to verify

```bash
# Install the validator
npm install -g @effectorhq/core

# Validate the manifest
effector check .

# Type-check the interface
effector check .
```

### References

- [Effector Spec](https://github.com/effectorHQ/effector-spec) — the specification
- [effector-types](https://github.com/effectorHQ/effector-types) — 40 standard capability types
- [Why effector.toml?](https://github.com/effectorHQ/docs/blob/main/guides/why-effector.md)
