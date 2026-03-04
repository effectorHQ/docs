# Building Extensions (TypeScript Plugins)

Everything you need to know about building full-featured OpenClaw extensions.

## When to Build an Extension

Build an extension (not a skill) when you need:

- **New messaging channel** (WhatsApp, Telegram, custom platform)
- **Custom configuration UI** (complex setup flows)
- **Complex providers** (databases, payment systems, APIs with special handling)
- **Library for other developers** (shared utilities, patterns)
- **Performance-critical operations** (compiled code, tight loops)
- **TypeScript/JavaScript integration** (monorepo, shared dependencies)

Build a **skill** when you need:
- Simple automation or orchestration
- Language-agnostic flexibility
- Quick prototyping and sharing
- Community-driven distribution (ClawHub)

## Extension vs Skill

| Aspect | Skill | Extension |
|--------|-------|-----------|
| Format | SKILL.md (markdown + YAML) | TypeScript package + manifest |
| Distribution | ClawHub | npm |
| Capabilities | Tool invocation, orchestration | Channels, providers, rich config |
| Complexity | Simple to moderate | Moderate to complex |
| Language | Language-agnostic | TypeScript/JavaScript |
| Setup | Minimal | May require build, config UI |

## Project Structure

A typical extension project:

```
my-extension/
├── src/
│   ├── index.ts              # Entry point, register() export
│   ├── channel.ts            # If implementing a channel
│   ├── provider.ts           # If implementing a provider
│   └── types.ts              # TypeScript types
├── openclaw.plugin.json      # Manifest
├── package.json              # npm metadata
├── tsconfig.json             # TypeScript config
├── README.md
└── LICENSE
```

## openclaw.plugin.json Manifest

The manifest declares your extension's identity and capabilities:

```json
{
  "id": "@myorg/my-extension",
  "name": "My Extension",
  "version": "1.0.0",
  "description": "Does amazing things",
  "configSchema": {
    "type": "object",
    "properties": {
      "apiKey": {
        "type": "string",
        "description": "API key for authentication"
      }
    },
    "required": ["apiKey"]
  },
  "kind": "plugin",
  "channels": ["whatsapp", "telegram"],
  "providers": ["database", "cache"],
  "skills": ["send-message", "handle-webhook"],
  "uiHints": {
    "configFields": [
      {
        "name": "apiKey",
        "type": "password",
        "label": "API Key"
      }
    ]
  }
}
```

### Manifest Fields Reference

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | Yes | Unique identifier (e.g., `@myorg/my-extension`) |
| `name` | string | Yes | Human-readable name |
| `version` | string | Yes | Semantic version (e.g., `1.0.0`) |
| `description` | string | Yes | Short description of what it does |
| `configSchema` | object | Yes | JSON Schema for configuration |
| `kind` | string | No | Type of plugin (usually `"plugin"`) |
| `channels` | array | No | List of channels implemented: `whatsapp`, `telegram`, `slack`, etc. |
| `providers` | array | No | List of providers: `database`, `cache`, `auth`, etc. |
| `skills` | array | No | Skills this extension provides/enables |
| `uiHints` | object | No | UI metadata for configuration UI |

### Config Schema

The `configSchema` is a JSON Schema object defining required configuration:

```json
{
  "configSchema": {
    "type": "object",
    "properties": {
      "apiKey": {
        "type": "string",
        "description": "API key for authentication"
      },
      "endpoint": {
        "type": "string",
        "description": "API endpoint URL",
        "default": "https://api.example.com"
      },
      "timeout": {
        "type": "integer",
        "description": "Request timeout in milliseconds",
        "default": 30000
      },
      "debug": {
        "type": "boolean",
        "description": "Enable debug logging",
        "default": false
      }
    },
    "required": ["apiKey"]
  }
}
```

### UI Hints

Optional hints for configuration UI:

```json
{
  "uiHints": {
    "configFields": [
      {
        "name": "apiKey",
        "type": "password",
        "label": "API Key",
        "help": "Get your key from https://example.com/settings"
      },
      {
        "name": "endpoint",
        "type": "text",
        "label": "API Endpoint"
      },
      {
        "name": "debug",
        "type": "checkbox",
        "label": "Enable debugging"
      }
    ]
  }
}
```

## Entry Point: Registering Your Extension

Create `src/index.ts`:

```typescript
import { OpenClawPluginApi } from '@openclaw/plugin-api'

export default {
  id: '@myorg/my-extension',
  name: 'My Extension',
  description: 'Does amazing things',
  version: '1.0.0',
  configSchema: {
    type: 'object',
    properties: {
      apiKey: { type: 'string' }
    },
    required: ['apiKey']
  },
  register(api: OpenClawPluginApi) {
    // Register channels
    if (config.channels?.includes('whatsapp')) {
      api.registerChannel(new WhatsAppChannel(config))
    }

    // Register providers
    api.registerProvider('database', new DatabaseProvider(config))

    // Register middleware
    api.registerMiddleware(authMiddleware)
  }
}
```

## OpenClawPluginApi Surface

The API injected into `register()`:

```typescript
interface OpenClawPluginApi {
  registerChannel(channel: ChannelConfig): void
  registerProvider(name: string, provider: Provider): void
  registerMiddleware(middleware: Middleware): void
  registerSkill(skill: SkillDefinition): void
  runtime: AgentRuntime
  config: ConfigManager
  logger: Logger
}
```

**Main methods:**

### registerChannel(channel: ChannelConfig)

Register a new messaging channel:

```typescript
api.registerChannel({
  id: 'whatsapp',
  name: 'WhatsApp',
  handler: async (message) => {
    // Process inbound WhatsApp message
    await handleWhatsAppMessage(message)
  }
})
```

### registerProvider(name: string, provider: Provider)

Register a provider (database, cache, auth):

```typescript
api.registerProvider('database', {
  connect: async () => { /* ... */ },
  query: async (sql) => { /* ... */ },
  disconnect: async () => { /* ... */ }
})
```

### registerMiddleware(middleware: Middleware)

Register request/response middleware:

```typescript
api.registerMiddleware({
  onRequest: async (req) => {
    // Intercept, validate, transform requests
  },
  onResponse: async (res) => {
    // Intercept, transform responses
  }
})
```

### registerSkill(skill: SkillDefinition)

Register a skill this extension provides:

```typescript
api.registerSkill({
  name: 'send-email',
  description: 'Send an email',
  execute: async (params) => {
    // Execute skill
  }
})
```

### api.runtime

Access to the Agent Runtime:

```typescript
api.runtime.executeSkill(skillName, params)
api.runtime.emitEvent(eventName, data)
```

### api.config

Access configuration:

```typescript
const apiKey = api.config.get('apiKey')
const allConfig = api.config.getAll()
```

### api.logger

Structured logging:

```typescript
api.logger.info('Extension started')
api.logger.error('Failed to connect', { error })
api.logger.debug('Request details', { request })
```

## Implementation Example: WhatsApp Channel

```typescript
import { ChannelConfig, InboundMessage } from '@openclaw/types'

export class WhatsAppChannel implements ChannelConfig {
  id = 'whatsapp'
  name = 'WhatsApp'

  constructor(private config: ExtensionConfig) {}

  async initialize() {
    // Connect to WhatsApp API, set up webhook receiver
  }

  async handleInboundMessage(message: WhatsAppMessage): Promise<InboundMessage> {
    return {
      channelId: 'whatsapp',
      userId: message.from,
      text: message.body,
      timestamp: new Date(),
      metadata: {
        messageId: message.id
      }
    }
  }

  async sendMessage(userId: string, text: string): Promise<void> {
    // Send message back to WhatsApp
  }

  async handleCallback(req: IncomingMessage): Promise<void> {
    // Webhook receiver for WhatsApp status callbacks
  }
}
```

## Testing

Use vitest (configured in OpenClaw monorepo):

```typescript
import { describe, it, expect, vi } from 'vitest'
import { register } from './index'

describe('My Extension', () => {
  it('should register successfully', async () => {
    const mockApi = {
      registerChannel: vi.fn(),
      registerProvider: vi.fn()
    }

    register(mockApi as any)

    expect(mockApi.registerChannel).toHaveBeenCalled()
  })
})
```

Run tests:

```bash
pnpm test
```

## Distribution via npm

1. **Build:**

```bash
pnpm build
```

2. **Version and publish:**

```bash
npm version minor
npm publish
```

3. **Users install with:**

```bash
openclaw plugins install @myorg/my-extension
```

## Installation with openclaw plugins

```bash
# Install from npm
openclaw plugins install @myorg/my-extension

# Install with specific version
openclaw plugins install @myorg/my-extension@2.0.0

# List installed plugins
openclaw plugins list

# Remove
openclaw plugins uninstall @myorg/my-extension
```

## Quality Requirements for Community Listing

If you want your extension listed in the official community registry:

- **Code quality** — TypeScript, type-safe, no `any` types
- **Documentation** — README with examples, API docs
- **Tests** — Minimum 80% coverage
- **Manifest validation** — Must pass `openclaw validate`
- **Error handling** — Graceful errors, clear messages
- **Logging** — Use api.logger for debugging
- **Performance** — No blocking operations, respect timeouts
- **Security** — No hardcoded credentials, input validation

## Common Patterns

### Reading Configuration

```typescript
register(api: OpenClawPluginApi) {
  const apiKey = api.config.get('apiKey')
  const timeout = api.config.get('timeout', 30000)

  if (!apiKey) {
    throw new Error('apiKey is required')
  }
}
```

### Async Initialization

```typescript
register(api: OpenClawPluginApi) {
  // Initialize asynchronously
  api.runtime.onStart(async () => {
    await initializeConnections()
    await validateConfig()
  })
}
```

### Error Handling

```typescript
async handleMessage(message: InboundMessage) {
  try {
    await processMessage(message)
  } catch (err) {
    api.logger.error('Failed to process message', { error: err })
    await notifyError(err)
  }
}
```

## What's Next?

- [Architecture guide](architecture.md) — understand the full extension ecosystem
- [Plugin Manifest Reference](../reference/plugin-manifest.md) — complete field reference
- Official OpenClaw repository — for real-world extension examples
