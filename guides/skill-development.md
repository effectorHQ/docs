# Building Skills

Everything you need to know about building, testing, and publishing OpenClaw skills.

## What is a Skill?

A skill is a language-agnostic, markdown-based automation unit. It's a single file: `SKILL.md` containing:

1. **Frontmatter (YAML)** — Metadata, requirements, install instructions
2. **Body (Markdown)** — Documentation and usage instructions

Skills live in `~/.openclaw/workspace/skills/<skill-name>/SKILL.md` and are auto-discovered by OpenClaw.

## The Three Tiers

### Bundled Skills

Shipped with OpenClaw core. Available out-of-the-box. Examples: basic utilities, file operations.

### Managed Skills

Published to ClawHub. Installable via:

```bash
openclaw skill install <skill-name>
```

These are vetted, discoverable, and versioned.

### Workspace Skills

Your personal skills in `~/.openclaw/workspace/skills/`. Auto-discovered locally. Not published.

Good for:
- Development and testing
- Personal automation
- Private/internal workflows

## SKILL.md Format Deep Dive

### Structure

```markdown
---
name: my-skill
description: What this skill does
metadata:
  openclaw:
    emoji: 📦
    requires:
      bins: [...]
      env: [...]
      config: [...]
    install: [...]
---

# Skill Name

Documentation goes here.

## Usage

How to use this skill.

## Examples

Examples of execution.
```

### Frontmatter Fields

| Field | Type | Required | Example |
|-------|------|----------|---------|
| `name` | string | Yes | `my-skill` |
| `description` | string | Yes | `Processes CSV files and uploads to cloud` |
| `metadata.openclaw.emoji` | string | No | `📦`, `🚀`, `📧` |
| `metadata.openclaw.requires.bins` | array | No | See below |
| `metadata.openclaw.requires.env` | array | No | See below |
| `metadata.openclaw.requires.config` | array | No | See below |
| `metadata.openclaw.install` | array | No | See below |

### Requirements Declaration

#### Binary Requirements

```yaml
metadata:
  openclaw:
    requires:
      bins:
        - name: python3
          version: ">=3.9"
        - name: curl
          version: ">=7.0"
```

Fields:
- `name` (required): Binary name
- `version` (optional): Version constraint (semver format)

#### Environment Variable Requirements

```yaml
metadata:
  openclaw:
    requires:
      env:
        - name: AWS_ACCESS_KEY_ID
          description: "AWS access key for authentication"
          secret: true
        - name: DATABASE_URL
          description: "PostgreSQL connection string"
          secret: true
```

Fields:
- `name` (required): Variable name
- `description` (optional): What it's for
- `secret` (optional): Boolean flag (true if sensitive)

#### Configuration Requirements

```yaml
metadata:
  openclaw:
    requires:
      config:
        - name: api_endpoint
          type: string
          description: "API endpoint URL"
          required: true
        - name: timeout
          type: integer
          description: "Request timeout in seconds"
          required: false
          default: 30
```

Fields:
- `name` (required): Config key
- `type` (optional): `string`, `integer`, `boolean`, `float`
- `description` (optional): What it's for
- `required` (optional): Whether it's mandatory
- `default` (optional): Default value if not provided

### Install Instructions

Install instructions tell users how to set up dependencies:

```yaml
metadata:
  openclaw:
    install:
      - id: homebrew
        kind: brew
        formula: python@3.11
        bins: [python3.11]
        label: "Install Python 3.11 via Homebrew"

      - id: ubuntu
        kind: apt
        package: python3.11
        bins: [python3.11]
        label: "Install Python 3.11 via apt"

      - id: manual
        kind: manual
        label: "Manual installation"
        instructions: |
          Visit https://www.python.org/downloads/
          Download and run the installer
```

Fields:
- `id` (required): Unique identifier for this install method
- `kind` (required): `brew`, `apt`, `manual`, etc.
- `formula` or `package` (for brew/apt): Package name
- `bins` (optional): Binaries that will be available after install
- `label` (required): Human-readable label
- `instructions` (for manual): Step-by-step instructions

## Skill Discovery

Skills are discovered via:

1. **Name** — Full-text search on skill name
2. **Description** — Indexed for discovery
3. **Emoji** — Visual categorization
4. **Keywords** — Implicit from description (vector embeddings)

**Best practices for descriptions:**

- Be specific and descriptive (2-3 sentences)
- Include the main action: "Sends emails", "Processes CSV", "Checks weather"
- Mention integrations: "Gmail integration", "Slack updates"
- Example: `"Monitors GitHub repositories for new releases and sends Slack notifications"`

This description helps users discover your skill via ClawHub search.

## Body Structure Conventions

### Required Sections

1. **Heading**: Skill name (H1)
2. **Description**: What it does (brief overview)
3. **Usage**: How to use it

### Optional Sections

- **Features** — Key capabilities
- **Configuration** — Config options
- **Examples** — Usage examples
- **Limitations** — What it doesn't do
- **Requirements** — System requirements
- **Troubleshooting** — Common issues

### Example Body

```markdown
# Email Sender

Send emails through Gmail or SMTP.

## Features

- Send to single or multiple recipients
- Support for attachments
- HTML and plaintext bodies
- Template variables

## Usage

Invoke with action `send`:

```
action: send
to: user@example.com
subject: Hello
body: This is my message
```

## Configuration

Set these environment variables:

- `SMTP_HOST` — SMTP server hostname
- `SMTP_PORT` — SMTP port (usually 587 or 465)
- `SMTP_USER` — SMTP username
- `SMTP_PASS` — SMTP password

## Examples

### Send a simple email

```
to: alice@example.com
subject: Meeting reminder
body: Don't forget our 3pm call
```

### Send with attachments

```
to: alice@example.com, bob@example.com
subject: Report attached
body: Please review
attachments:
  - path: /tmp/report.pdf
```

## Requirements

- SMTP server access or Gmail API credentials
- Bash or Node.js runtime
```

## Complete Example: CSV to Slack

Here's a complete SKILL.md that processes a CSV and sends to Slack:

```yaml
---
name: csv-to-slack
description: Read a CSV file and post rows to a Slack channel
metadata:
  openclaw:
    emoji: 📊
    requires:
      bins:
        - name: node
          version: ">=18.0.0"
      env:
        - name: SLACK_BOT_TOKEN
          description: Slack bot token for posting messages
          secret: true
      config:
        - name: slack_channel
          type: string
          description: Slack channel ID to post to
          required: true
    install:
      - id: node
        kind: manual
        label: Install Node.js
        instructions: "Visit https://nodejs.org and download LTS version"
---

# CSV to Slack

Read CSV files and post their contents to a Slack channel.

## Features

- Parses CSV files
- Posts rows as formatted Slack messages
- Handles large files with pagination
- Supports custom column filtering

## Usage

Invoke with action `upload`:

```
action: upload
file: /path/to/data.csv
columns: [name, email, status]
```

## Configuration

Set environment variable:

```
SLACK_BOT_TOKEN=xoxb-your-token-here
```

In your workspace config, set:

```yaml
csv_to_slack:
  slack_channel: C01234567890
```

## Examples

### Basic usage

```
action: upload
file: /tmp/users.csv
```

### With column selection

```
action: upload
file: /tmp/users.csv
columns: [name, email]
message_template: "New user: {name} ({email})"
```

## Requirements

- Node.js >= 18
- Slack workspace with bot token
```

## Testing Locally

To test a skill before publishing:

1. **Ensure OpenClaw can find it:**

```bash
cd ~/.openclaw/workspace/skills/my-skill
openclaw skill list | grep my-skill
```

2. **Validate the SKILL.md:**

```bash
openclaw skill validate
```

3. **Test execution** (via OpenClaw's test interface — refer to OpenClaw CLI docs for current syntax)

## Publishing to ClawHub

Once your skill is working:

```bash
cd ~/.openclaw/workspace/skills/my-skill
clawhub publish
```

This:
- Validates the SKILL.md
- Uploads to ClawHub
- Generates vector embeddings for discovery
- Makes it installable for others

**First time setup:**

```bash
clawhub login
clawhub publish
```

## Version Management

Skills use semantic versioning. Update the version in your workspace before republishing:

```yaml
name: my-skill
version: 1.0.1  # Update this
description: ...
```

ClawHub tracks all versions and users can install specific versions:

```bash
openclaw skill install my-skill@1.0.0
```

## Best Practices

1. **Keep skills focused** — One main capability per skill
2. **Document clearly** — Describe requirements, configuration, examples
3. **Make descriptions discoverable** — Include keywords relevant to what the skill does
4. **Version responsibly** — Use semver, document breaking changes
5. **Declare requirements upfront** — List all bins, env, config needed
6. **Test thoroughly** — Validate locally before publishing
7. **Use the emoji** — Choose an emoji that visually represents your skill's purpose

## What's Next?

- [Architecture guide](architecture.md) — understand how skills fit into OpenClaw
- [SKILL.md Reference](../reference/skill-format.md) — detailed field reference
- [Getting Started](getting-started.md) — build your first skill in 10 minutes
