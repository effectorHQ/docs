# How OpenClaw Works

Understanding OpenClaw's architecture: Gateway, Agent Runtime, Channels, Skills, and Extensions.

## The Big Picture

OpenClaw is a distributed system for managing autonomous agents across multiple messaging platforms. Here are the core pieces:

```
┌────────────────────────────────────────────────────────────────┐
│                         Gateway                                │
│           (WebSocket Control Plane: ws://127.0.0.1:18789)     │
│  - Manages sessions, presence, config                         │
│  - Routes webhooks, broadcasts config changes                 │
│  - Central orchestration point                                │
└────────────────────────────────────────────────────────────────┘
                              ▲
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
        ▼                     ▼                     ▼
┌──────────────────┐  ┌──────────────────┐  ┌──────────────────┐
│   Pi Agent 1     │  │   Pi Agent 2     │  │   Pi Agent N     │
│  (Message Proc.) │  │  (Message Proc.) │  │  (Message Proc.) │
│  Executes Tools  │  │  Executes Tools  │  │  Executes Tools  │
└──────────────────┘  └──────────────────┘  └──────────────────┘
        │                     │                     │
        ├─ Inbound Messages ──┤                     │
        │  (20+ Channels:     │                     │
        │   WhatsApp, Slack,  │                     │
        │   Telegram, etc.)   │                     │
        └─────────────────────┘
```

## Key Components

### Gateway (WebSocket Control Plane)

The Gateway is a WebSocket server running on `ws://127.0.0.1:18789` (configurable).

**Responsibilities:**
- Session management (create, update, remove agent sessions)
- Presence tracking (which agents are online)
- Configuration distribution (injected prompt files, agent settings)
- Webhook ingestion (receive inbound messages from channels)
- Config synchronization (YAML/JSON from workspace)

**Configuration files injected by Gateway:**
- `AGENTS.md` — Agent definitions and capabilities
- `SOUL.md` — Agent personality, behavior guidelines
- `TOOLS.md` — Available tools/skills for execution

### Pi Agent Runtime

The Agent Runtime processes inbound messages and executes tools.

**Responsibilities:**
- Receive messages from Gateway via WebSocket
- Parse and route messages to appropriate tools/skills
- Execute skills and extensions via RPC
- Handle tool parameters, validation, execution
- Return results to Gateway

**Message Processing Flow:**
1. Message arrives from a channel (WhatsApp, Slack, etc.)
2. Gateway routes to appropriate agent session
3. Agent Runtime receives message
4. Agent determines which tools/skills to execute
5. Tools execute, return results
6. Results sent back through Gateway to the channel

### Multi-Channel Inbox

OpenClaw integrates 20+ messaging platforms. Each channel is a source of inbound messages:

- **WhatsApp** — Two-way messaging
- **Telegram** — Bots, groups
- **Slack** — Direct messages, channels
- **Discord** — Direct messages, servers
- **Email** — Inbound messages
- **SMS** — Phone messages
- **And 14+ more...**

Channels are configured via extensions/plugins that implement the channel interface.

## Skills vs Extensions vs Plugins

OpenClaw has two extension mechanisms:

### Skills

**What:** Language-agnostic, markdown-based automation.

**Format:** SKILL.md files (YAML frontmatter + markdown body).

**Where:** Live in `~/.openclaw/workspace/skills/<name>/SKILL.md`

**Distribution:** Published to ClawHub registry (community-driven).

**Good for:**
- Simple, self-contained automation
- Quick prototyping and sharing
- Non-programming workflows
- Leverage existing tools via RPC

**Example:** A skill that processes CSV files, sends emails, or orchestrates API calls.

**Three tiers:**
- **Bundled**: Shipped with OpenClaw core
- **Managed**: Published to ClawHub, installed via `openclaw skill install <name>`
- **Workspace**: Your own skills in `~/.openclaw/workspace/skills/`

### Extensions / Plugins

**What:** Full-featured TypeScript packages.

**Format:** TypeScript with `openclaw.plugin.json` manifest.

**Distribution:** npm packages. Installed via `openclaw plugins install <npm-package>`.

**Good for:**
- Channel implementations (WhatsApp, Slack, etc.)
- Complex configuration and setup
- Custom providers (databases, APIs)
- UI hints and rich interactions

**Manifest file:** `openclaw.plugin.json` with fields: `id`, `channels`, `configSchema`, `kind`, `providers`, `skills`, `name`, `description`, `uiHints`, `version`.

**Entry point:** Exports `register(api: OpenClawPluginApi)` function.

**When to use:**
- You need a new messaging channel
- You need custom configuration UI
- You're implementing a complex provider
- You're building a library for other developers

## Configuration & Prompt Injection

OpenClaw uses a workspace-based configuration system. Files in your workspace are injected into the Agent Runtime:

### AGENTS.md

Defines available agents and their capabilities.

```markdown
# Available Agents

## Alice
- Personality: Friendly, helpful
- Capabilities: Email, scheduling, file operations
- Tools: send-email, check-calendar, upload-file

## Bob
- Personality: Technical, detailed
- Capabilities: Code execution, debugging
- Tools: run-python, analyze-logs
```

### SOUL.md

Agent personality and behavior guidelines.

```markdown
# Agent Soul

You are a helpful assistant. You:
- Answer questions clearly and concisely
- Ask clarifying questions when needed
- Respect user privacy
- Never execute dangerous operations
```

### TOOLS.md

Available tools/skills for execution.

```markdown
# Available Tools

## send-email
Send an email to a recipient.

Parameters:
- to: recipient email
- subject: email subject
- body: email body

## check-calendar
Check user's calendar for availability.

Parameters:
- date: date to check (YYYY-MM-DD)
```

### Configuration Hierarchy

Config can be provided at multiple levels:

1. **Gateway config** (`gateway.yaml`)
   - `gateway.address`, `gateway.port`
   - `gateway.webhookUrl`

2. **Channel config** (`channels.*.yaml`)
   - Channel-specific settings (API keys, tokens, etc.)

3. **Agent config** (`agents.*.yaml`)
   - Agent-specific settings and overrides

## ClawHub Registry

ClawHub is the skill registry with vector search.

**Features:**
- Publish skills to a shared registry
- Search for skills using embeddings
- Discover community-built automation
- Version management and updates

**Publish a skill:**

```bash
cd ~/.openclaw/workspace/skills/my-skill
clawhub publish
```

**Install a skill:**

```bash
openclaw skill install <skill-name>
```

## Extension Plugins Architecture

Extensions implement the `OpenClawPluginApi`:

```typescript
interface OpenClawPluginApi {
  registerChannel(channel: ChannelConfig): void
  runtime: AgentRuntime
  config: ConfigManager
  // ... other APIs
}
```

Plugins are distributed via npm and installed with:

```bash
openclaw plugins install @org/my-plugin
```

## Data Flow: Message to Execution

1. **Channel receives message** (WhatsApp, Slack, etc.)
2. **Channel posts to Gateway** via webhook or SDK
3. **Gateway routes to Agent session**
4. **Agent Runtime receives message**
5. **Agent decides which skill/tool to invoke**
6. **Skill executes** (may call external services, databases, etc.)
7. **Result returned to Agent**
8. **Agent formats response**
9. **Response sent back through Gateway**
10. **Gateway routes to channel**
11. **Channel delivers message** to user

## What's Next?

- [Skill Development](skill-development.md) — build skills with SKILL.md
- [Extension Development](extension-development.md) — build TypeScript plugins
- [SKILL.md Reference](../reference/skill-format.md) — all frontmatter fields
- [Plugin Manifest Reference](../reference/plugin-manifest.md) — openclaw.plugin.json fields
