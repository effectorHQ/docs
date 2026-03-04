# openclaw.plugin.json Reference

Complete reference for the `openclaw.plugin.json` manifest file that defines OpenClaw extensions.

## Overview

The `openclaw.plugin.json` file is a JSON manifest that declares:
- Plugin identity and metadata
- Capabilities (channels, providers, skills)
- Configuration requirements
- UI hints for configuration

## File Location

```
my-extension/
├── openclaw.plugin.json     # ← Here
├── src/
│   └── index.ts
├── package.json
└── README.md
```

## Basic Structure

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
        "type": "string"
      }
    },
    "required": ["apiKey"]
  }
}
```

## Complete Field Reference

### id (Required)

Unique identifier for the extension.

**Type:** string

**Format:** npm package name format or custom identifier

**Examples:**
```json
{
  "id": "@myorg/my-extension",
  "id": "my-extension",
  "id": "@openclaw/whatsapp-channel"
}
```

**Notes:**
- Must be unique within OpenClaw registry
- If distributed via npm, should match npm package name
- Recommend using scoped names for organization clarity

### name (Required)

Human-readable name of the extension.

**Type:** string

**Example:**
```json
{
  "name": "WhatsApp Channel"
}
```

**Used for:**
- Extension listing UIs
- Install instructions
- Plugin logs

### version (Required)

Semantic version of the extension.

**Type:** string (semver format)

**Format:** MAJOR.MINOR.PATCH

**Examples:**
```json
{
  "version": "1.0.0",
  "version": "2.3.1",
  "version": "0.1.0"
}
```

**Notes:**
- Should match the version in `package.json`
- Used for version constraints when installing

### description (Required)

Short description of what the extension does.

**Type:** string

**Example:**
```json
{
  "description": "WhatsApp channel integration with webhook support"
}
```

**Used for:**
- Extension listing and search
- Help text
- Plugin registry

### configSchema (Required)

JSON Schema defining the extension's configuration.

**Type:** object (JSON Schema)

**Format:** Valid JSON Schema Draft 7+

**Example:**
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
        "minimum": 1000,
        "maximum": 60000,
        "default": 30000
      },
      "debug": {
        "type": "boolean",
        "description": "Enable debug logging",
        "default": false
      }
    },
    "required": ["apiKey"],
    "additionalProperties": false
  }
}
```

**Supported types:**
- `string` — Text values
- `integer` — Whole numbers
- `number` — Floating point numbers
- `boolean` — true/false
- `array` — Lists
- `object` — Nested objects

**Common schema keywords:**
- `type` — Data type
- `description` — Human-readable description
- `default` — Default value if not provided
- `required` (array at root) — Required property names
- `minimum` / `maximum` — Number bounds
- `minLength` / `maxLength` — String length bounds
- `pattern` — Regex pattern for validation
- `enum` — Allowed values

**Notes:**
- Configuration must be JSON Schema compliant
- Required fields should be in the `required` array at root level
- Use `additionalProperties: false` to disallow unexpected config keys

### kind (Optional)

Type of plugin. Usually `"plugin"`.

**Type:** string

**Example:**
```json
{
  "kind": "plugin"
}
```

**Valid values:**
- `"plugin"` — Standard plugin (default)
- `"channel"` — Channel implementation
- `"provider"` — Provider implementation

### channels (Optional)

List of messaging channels this extension implements.

**Type:** array of strings

**Example:**
```json
{
  "channels": ["whatsapp", "telegram", "slack"]
}
```

**Common channel types:**
- `"whatsapp"` — WhatsApp integration
- `"telegram"` — Telegram Bot
- `"slack"` — Slack workspace
- `"discord"` — Discord server
- `"email"` — Email inbox
- `"sms"` — SMS/text messaging
- `"custom"` — Custom channel type

**Notes:**
- If implementing channels, should have corresponding channel handler in code
- Used for capability discovery

### providers (Optional)

List of providers this extension implements.

**Type:** array of strings

**Example:**
```json
{
  "providers": ["database", "cache", "authentication"]
}
```

**Common provider types:**
- `"database"` — Database (SQL, NoSQL, etc.)
- `"cache"` — Caching system (Redis, Memcached)
- `"authentication"` — Auth provider (OAuth, JWT, etc.)
- `"storage"` — Object storage (S3, GCS, etc.)
- `"queue"` — Message queue (RabbitMQ, SQS, etc.)
- `"custom"` — Custom provider type

**Notes:**
- Declares what providers are available
- Used for dependency resolution

### skills (Optional)

List of skills this extension provides or enables.

**Type:** array of strings

**Example:**
```json
{
  "skills": ["send-message", "receive-webhook", "handle-callback"]
}
```

**Notes:**
- Skills provided by the extension
- Used for capability discovery
- Skill definitions should be registered in code via `api.registerSkill()`

### uiHints (Optional)

Hints for UI rendering and configuration forms.

**Type:** object

**Structure:**

```json
{
  "uiHints": {
    "configFields": [
      {
        "name": "apiKey",
        "type": "password",
        "label": "API Key",
        "help": "Get your key from https://example.com/settings",
        "placeholder": "sk-..."
      },
      {
        "name": "endpoint",
        "type": "text",
        "label": "API Endpoint",
        "required": true
      },
      {
        "name": "timeout",
        "type": "number",
        "label": "Timeout (ms)",
        "min": 1000,
        "max": 60000
      },
      {
        "name": "debug",
        "type": "checkbox",
        "label": "Enable debugging"
      },
      {
        "name": "logLevel",
        "type": "select",
        "label": "Log Level",
        "options": [
          { "value": "debug", "label": "Debug" },
          { "value": "info", "label": "Info" },
          { "value": "warn", "label": "Warning" },
          { "value": "error", "label": "Error" }
        ]
      }
    ]
  }
}
```

**Field properties:**
- `name` (required) — Config property name
- `type` (required) — Input type: `text`, `password`, `number`, `checkbox`, `select`, `textarea`
- `label` (required) — Human-readable label
- `help` (optional) — Help text or hint
- `placeholder` (optional) — Placeholder text
- `required` (optional) — Whether field is required (UI validation)
- `min` / `max` (optional) — Number bounds
- `minLength` / `maxLength` (optional) — String length
- `options` (for select) — Array of `{ value, label }` pairs

**UI field types:**
- `"text"` — Single-line text input
- `"password"` — Password input (masked)
- `"number"` — Number input
- `"checkbox"` — Boolean checkbox
- `"select"` — Dropdown select
- `"textarea"` — Multi-line text

### author (Optional)

Author(s) of the extension.

**Type:** string

**Example:**
```json
{
  "author": "Jane Doe"
}
```

### license (Optional)

License identifier.

**Type:** string

**Example:**
```json
{
  "license": "MIT"
}
```

**Common values:**
- `"MIT"`
- `"Apache-2.0"`
- `"GPL-3.0"`
- `"BSD-3-Clause"`

### repository (Optional)

Repository URL.

**Type:** string or object

**Examples:**
```json
{
  "repository": "https://github.com/myorg/my-extension"
}
```

or

```json
{
  "repository": {
    "type": "git",
    "url": "https://github.com/myorg/my-extension"
  }
}
```

### homepage (Optional)

Homepage or documentation URL.

**Type:** string

**Example:**
```json
{
  "homepage": "https://docs.example.com/my-extension"
}
```

### keywords (Optional)

Search keywords for discovery.

**Type:** array of strings

**Example:**
```json
{
  "keywords": ["whatsapp", "messaging", "channel", "webhook"]
}
```

## Complete Example: WhatsApp Channel Extension

```json
{
  "id": "@openclaw/whatsapp-channel",
  "name": "WhatsApp Channel",
  "version": "1.2.0",
  "description": "WhatsApp messaging channel integration with webhook support and media handling",
  "author": "OpenClaw Team",
  "license": "MIT",
  "homepage": "https://docs.openclaw.com/channels/whatsapp",
  "repository": "https://github.com/OpenClawHQ/channel-whatsapp",
  "keywords": ["whatsapp", "messaging", "channel"],
  "kind": "plugin",
  "channels": ["whatsapp"],
  "providers": [],
  "skills": ["send-message", "handle-webhook"],
  "configSchema": {
    "type": "object",
    "properties": {
      "accountId": {
        "type": "string",
        "description": "WhatsApp Business Account ID"
      },
      "apiKey": {
        "type": "string",
        "description": "WhatsApp API key for authentication"
      },
      "phoneNumber": {
        "type": "string",
        "description": "WhatsApp Business Phone Number",
        "pattern": "^\\+?[1-9]\\d{1,14}$"
      },
      "webhookSecret": {
        "type": "string",
        "description": "Secret key for webhook signature verification"
      },
      "webhookUrl": {
        "type": "string",
        "description": "Public URL where WhatsApp will send webhooks",
        "format": "uri"
      },
      "maxMediaSize": {
        "type": "integer",
        "description": "Maximum media file size in MB",
        "default": 100,
        "minimum": 1,
        "maximum": 5000
      },
      "messageTimeout": {
        "type": "integer",
        "description": "Message delivery timeout in seconds",
        "default": 30,
        "minimum": 5,
        "maximum": 300
      },
      "debug": {
        "type": "boolean",
        "description": "Enable debug logging",
        "default": false
      }
    },
    "required": ["accountId", "apiKey", "phoneNumber", "webhookSecret", "webhookUrl"],
    "additionalProperties": false
  },
  "uiHints": {
    "configFields": [
      {
        "name": "accountId",
        "type": "text",
        "label": "Account ID",
        "help": "Your WhatsApp Business Account ID",
        "required": true
      },
      {
        "name": "apiKey",
        "type": "password",
        "label": "API Key",
        "help": "Get this from WhatsApp Business Manager",
        "required": true
      },
      {
        "name": "phoneNumber",
        "type": "text",
        "label": "Phone Number",
        "placeholder": "+1234567890",
        "help": "Your WhatsApp Business phone number",
        "required": true
      },
      {
        "name": "webhookSecret",
        "type": "password",
        "label": "Webhook Secret",
        "help": "Secret key for verifying webhook requests",
        "required": true
      },
      {
        "name": "webhookUrl",
        "type": "text",
        "label": "Webhook URL",
        "placeholder": "https://your-domain.com/webhooks/whatsapp",
        "help": "Public URL where WhatsApp sends updates",
        "required": true
      },
      {
        "name": "maxMediaSize",
        "type": "number",
        "label": "Max Media Size (MB)",
        "min": 1,
        "max": 5000,
        "help": "Maximum file size for media attachments"
      },
      {
        "name": "messageTimeout",
        "type": "number",
        "label": "Message Timeout (sec)",
        "min": 5,
        "max": 300,
        "help": "How long to wait for delivery confirmation"
      },
      {
        "name": "debug",
        "type": "checkbox",
        "label": "Enable Debug Logging"
      }
    ]
  }
}
```

## Validation Rules

When validating a manifest, check:

1. **Required fields present:** `id`, `name`, `version`, `description`, `configSchema`
2. **Valid semver:** `version` must be MAJOR.MINOR.PATCH
3. **Valid JSON Schema:** `configSchema` must be valid JSON Schema
4. **Valid field types:** All field values match their declared types
5. **No unknown fields:** Only recognized fields are used (depends on schema version)

Validation command:

```bash
openclaw validate
```

## Notes for Manifest Maintainers

- Keep `version` in sync with `package.json`
- Update `configSchema` when adding new configuration options
- Update `channels` when implementing new channels
- Use `uiHints` to improve user experience for configuration
- Consider backwards compatibility when changing `configSchema`
- Document all required configuration in `description`

## What's Next?

- [Extension Development guide](../guides/extension-development.md) — full tutorial on building extensions
- [Architecture](../guides/architecture.md) — understand how extensions fit into OpenClaw
- Official OpenClaw documentation — for latest schema updates
