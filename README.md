# OpenClawHQ Documentation

The knowledge base for building skills and extensions with OpenClaw.

## Quick Navigation

| Guide | For who | What you'll learn |
|-------|---------|-------------------|
| [Getting Started](guides/getting-started.md) | New builders | Create your first skill in 10 minutes |
| [Architecture](guides/architecture.md) | Anyone curious | How OpenClaw works: Gateway, Agent, Channels, Skills & Extensions |
| [Skill Development](guides/skill-development.md) | Skill builders | Build language-agnostic skills with SKILL.md |
| [Extension Development](guides/extension-development.md) | TypeScript developers | Build full-featured plugins with TypeScript |
| [SKILL.md Format Reference](reference/skill-format.md) | Everyone | Complete reference for SKILL.md files |
| [Plugin Manifest Reference](reference/plugin-manifest.md) | Extension builders | Complete reference for openclaw.plugin.json |

## Key Concepts

**Skills**: Language-agnostic, markdown-based automation. Live in `~/.openclaw/workspace/skills/`. Published to ClawHub.

**Extensions/Plugins**: Full-featured TypeScript packages. Implement channels, configuration, and complex logic. Distributed via npm.

**ClawHub**: The skill registry. Search, publish, and install community skills.

**Gateway**: WebSocket control plane (`ws://127.0.0.1:18789`) managing sessions, presence, config, webhooks.

**Agent Runtime**: Processes inbound messages, executes tools via RPC.

## Getting Help

- Check the [Architecture guide](guides/architecture.md) to understand how everything fits together
- Browse the [Skill Development guide](guides/skill-development.md) for building skills
- See [Extension Development](guides/extension-development.md) for TypeScript plugins
- Refer to [SKILL.md Format Reference](reference/skill-format.md) for all skill frontmatter fields

## Contributing to Docs

Found an error? Have a better explanation? PRs are welcome.

For doc-specific conventions: write in plain language, use code examples liberally, keep paragraphs short.

---

<sub>OpenClawHQ · Every claw extends the reach.</sub>
