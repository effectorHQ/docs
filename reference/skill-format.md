# SKILL.md Format Reference

Complete reference for SKILL.md frontmatter and body conventions.

## Overview

A SKILL.md file has two parts:

1. **Frontmatter (YAML)** — Metadata, requirements, install instructions
2. **Body (Markdown)** — Documentation and usage

```markdown
---
name: my-skill
description: What this skill does
metadata:
  openclaw:
    emoji: 📦
---

# Skill Name

Body content goes here.
```

## Frontmatter Fields

### Top-Level Fields

| Field | Type | Required | Example |
|-------|------|----------|---------|
| `name` | string | Yes | `my-skill` |
| `description` | string | Yes | `Processes CSV files` |
| `metadata` | object | No | See below |
| `version` | string | No | `1.0.0` |
| `author` | string | No | `Jane Doe` |
| `license` | string | No | `MIT` |

### name

Unique identifier for the skill. Used in:
- Workspace path: `~/.openclaw/workspace/skills/{name}/`
- ClawHub registry lookups
- `openclaw skill install {name}`

**Format:** kebab-case, alphanumeric + hyphens

**Example:**
```yaml
name: email-sender
```

### description

Short description of what the skill does. Used for:
- ClawHub search and discovery
- Vector embeddings (skills searchable by keywords in description)
- UI display in skill listings

**Best practices:**
- 2-3 sentences maximum
- Start with action verb: "Sends emails", "Processes CSV", "Checks weather"
- Mention key integrations: "Gmail integration", "Slack updates"
- Be specific: "Monitors GitHub repositories for new releases and sends Slack notifications"

**Example:**
```yaml
description: Sends emails via SMTP or Gmail. Supports attachments and HTML templates.
```

## metadata.openclaw

The `metadata.openclaw` section contains OpenClaw-specific metadata.

### metadata.openclaw.emoji

Single emoji representing the skill visually.

**Type:** string
**Optional:** Yes
**Example:**
```yaml
metadata:
  openclaw:
    emoji: 📧
```

**Common choices:**
- 📧 — Email
- 📊 — Data/spreadsheet
- 🔐 — Security/authentication
- 🌐 — Web/API
- ⏰ — Scheduling/time
- 📱 — Mobile/messaging
- 🗄️ — Database/storage
- 🚀 — Deployment/automation
- 🔔 — Notifications

### metadata.openclaw.requires

Declares what the skill needs to run.

#### metadata.openclaw.requires.bins

Required system binaries.

**Type:** array of objects
**Optional:** Yes

**Fields per binary:**
- `name` (required): Binary name (e.g., `python3`, `node`, `curl`)
- `version` (optional): Version constraint in semver format (e.g., `>=3.9`, `^18.0.0`)

**Example:**
```yaml
metadata:
  openclaw:
    requires:
      bins:
        - name: python3
          version: ">=3.9"
        - name: node
          version: ">=18.0.0"
        - name: curl
```

#### metadata.openclaw.requires.env

Required environment variables.

**Type:** array of objects
**Optional:** Yes

**Fields per variable:**
- `name` (required): Variable name (e.g., `AWS_ACCESS_KEY_ID`)
- `description` (optional): What the variable is for
- `secret` (optional): Boolean flag (true if sensitive data)

**Example:**
```yaml
metadata:
  openclaw:
    requires:
      env:
        - name: AWS_ACCESS_KEY_ID
          description: AWS access key for authentication
          secret: true
        - name: AWS_REGION
          description: AWS region (e.g., us-east-1)
        - name: SLACK_BOT_TOKEN
          secret: true
```

#### metadata.openclaw.requires.config

Configuration options the skill accepts.

**Type:** array of objects
**Optional:** Yes

**Fields per config:**
- `name` (required): Config key
- `type` (optional): `string`, `integer`, `boolean`, `float`
- `description` (optional): What it's for
- `required` (optional): Boolean, true if mandatory
- `default` (optional): Default value if not provided

**Example:**
```yaml
metadata:
  openclaw:
    requires:
      config:
        - name: api_endpoint
          type: string
          description: API endpoint URL
          required: true
        - name: timeout
          type: integer
          description: Request timeout in seconds
          required: false
          default: 30
        - name: debug_mode
          type: boolean
          description: Enable debug logging
          default: false
```

### metadata.openclaw.install

Installation instructions for dependencies.

**Type:** array of objects
**Optional:** Yes

**Common `kind` values:**
- `brew` — Homebrew (macOS)
- `apt` — APT (Debian/Ubuntu)
- `yum` — YUM (RHEL/CentOS)
- `manual` — Manual steps

**Fields per install method:**
- `id` (required): Unique identifier (e.g., `homebrew`, `ubuntu`, `manual`)
- `kind` (required): Installation system
- `formula` or `package` (for package managers): Package name
- `bins` (optional): Binaries that will be available after install
- `label` (required): Human-readable label for UI
- `instructions` (for manual): Step-by-step installation text

**Example:**
```yaml
metadata:
  openclaw:
    install:
      - id: homebrew
        kind: brew
        formula: python@3.11
        bins:
          - python3.11
        label: Install Python 3.11 via Homebrew

      - id: ubuntu
        kind: apt
        package: python3.11
        bins:
          - python3.11
        label: Install Python 3.11 via apt

      - id: manual
        kind: manual
        label: Manual installation
        instructions: |
          1. Visit https://www.python.org/downloads/
          2. Download the latest Python 3.11 installer
          3. Run the installer
          4. Verify: python3 --version
```

## Body Structure

The markdown body (after frontmatter) contains documentation.

### Recommended Sections

#### H1 Heading (Required)

Skill name, matching the `name` field:

```markdown
# Email Sender
```

#### Brief Description

One-paragraph overview:

```markdown
Send emails through SMTP, Gmail, or other mail providers. Supports attachments, HTML templates, and multiple recipients.
```

#### Features (Recommended)

Bullet list of key capabilities:

```markdown
## Features

- Send to single or multiple recipients
- Support for attachments (PDF, images, etc.)
- HTML and plaintext bodies
- Template variables: {name}, {email}, etc.
- Retry on failure
```

#### Usage (Recommended)

How to invoke the skill:

```markdown
## Usage

Invoke the skill with action `send`:

```
action: send
to: user@example.com
subject: Hello World
body: This is my message
```

Required parameters:
- `to` (string or array): Recipient email(s)
- `subject` (string): Email subject
- `body` (string): Email body

Optional parameters:
- `html` (string): HTML body (overrides body if provided)
- `cc` (array): CC recipients
- `bcc` (array): BCC recipients
- `attachments` (array): Files to attach
```

#### Configuration (If Applicable)

Configuration requirements:

```markdown
## Configuration

Set these environment variables:

- `SMTP_HOST` — SMTP server hostname
- `SMTP_PORT` — SMTP port (usually 587 for TLS or 465 for SSL)
- `SMTP_USER` — SMTP username
- `SMTP_PASS` — SMTP password
- `SMTP_FROM` — From address for emails

Or use OAuth2 with Gmail:

- `GMAIL_CLIENT_ID` — OAuth2 client ID
- `GMAIL_CLIENT_SECRET` — OAuth2 client secret
- `GMAIL_REFRESH_TOKEN` — OAuth2 refresh token
```

#### Examples (Recommended)

Real-world usage examples:

```markdown
## Examples

### Simple email

```
action: send
to: alice@example.com
subject: Meeting reminder
body: Don't forget our 3pm call
```

### Multiple recipients with attachment

```
action: send
to: [alice@example.com, bob@example.com]
cc: [manager@example.com]
subject: Report attached
body: Please review the attached report
attachments:
  - path: /tmp/report.pdf
    name: Q1_Report.pdf
```

### HTML email with template

```
action: send
to: user@example.com
subject: Welcome, {name}!
body: |
  <h1>Welcome to our service</h1>
  <p>Hi {name}, thanks for signing up!</p>
  <p><a href="https://example.com/verify?code={code}">Verify your email</a></p>
template_vars:
  name: Alice
  code: abc123xyz
```
```

#### Requirements (If Applicable)

System requirements:

```markdown
## Requirements

- Node.js >= 18 or Python >= 3.9
- SMTP server access or Gmail API credentials
- Network access to mail servers
```

#### Limitations (If Applicable)

What the skill doesn't do:

```markdown
## Limitations

- Does not support IMAP (incoming mail)
- Cannot schedule emails for later delivery
- Attachments limited to 25MB per email
- No read receipts tracking
```

#### Troubleshooting (If Applicable)

Common issues and solutions:

```markdown
## Troubleshooting

### "Connection refused" error

Check that SMTP_HOST and SMTP_PORT are correct. Test with:

```bash
telnet $SMTP_HOST $SMTP_PORT
```

### "Authentication failed"

Verify SMTP_USER and SMTP_PASS are correct. For Gmail, use an App Password, not your regular password.

### "Timeout sending email"

Increase the timeout or check network connectivity to the SMTP server.
```

## Complete Example: CSV Processor

```markdown
---
name: csv-processor
version: 1.0.0
description: Parse, filter, and transform CSV files. Export to JSON, SQL, or send to external services.
author: Jane Doe
license: MIT
metadata:
  openclaw:
    emoji: 📊
    requires:
      bins:
        - name: node
          version: ">=18.0.0"
      env:
        - name: DATABASE_URL
          description: PostgreSQL connection string for loading data
          secret: true
      config:
        - name: output_format
          type: string
          description: Output format (json, sql, csv)
          required: false
          default: json
        - name: delimiter
          type: string
          description: CSV delimiter character
          required: false
          default: ","
    install:
      - id: node
        kind: manual
        label: Install Node.js
        instructions: |
          Visit https://nodejs.org and download the LTS version (>= 18)
---

# CSV Processor

Parse, filter, and transform CSV files. Convert between formats and load into databases.

## Features

- Parse CSV files with auto-detection of delimiters
- Filter rows by column values
- Transform columns (rename, combine, extract)
- Export to JSON, SQL INSERT statements, or CSV
- Load directly into PostgreSQL
- Handles large files with streaming

## Usage

Invoke with action `process`:

```
action: process
file: /path/to/data.csv
output_format: json
filter:
  status: active
```

## Configuration

Set optional environment variable:

```
DATABASE_URL=postgresql://user:pass@localhost/mydb
```

Configure output format:

```yaml
csv_processor:
  output_format: json
  delimiter: ","
```

## Examples

### Convert CSV to JSON

```
action: process
file: /tmp/users.csv
output_format: json
output_file: /tmp/users.json
```

### Filter rows

```
action: process
file: /tmp/sales.csv
filter:
  region: "North America"
  year: "2024"
output_format: csv
```

### Load into PostgreSQL

```
action: process
file: /tmp/customers.csv
action: load_database
table: customers
```

## Requirements

- Node.js >= 18
- For database loading: PostgreSQL connection string

## Limitations

- Large files (>1GB) require streaming mode
- Does not handle nested JSON in CSV cells
- Maximum column count: 10,000
```

## Discovery Best Practices

Your `description` and body content are indexed for ClawHub search. To improve discoverability:

1. **Use specific action verbs** in description: "Sends emails", "Processes CSV", "Monitors logs"
2. **Mention integrations** clearly: "Gmail", "Slack", "PostgreSQL"
3. **Include keywords** in description: "SMTP", "attachments", "templates"
4. **Good examples** in body help users understand what the skill does

Example with good discoverability:

```yaml
name: github-notifications
description: Monitor GitHub repositories for new releases and push notifications. Supports email, Slack, and webhooks.
```

This is discoverable by: "github", "notifications", "releases", "slack", "email", "webhooks", "monitor"

## Validation

Skills are validated with:

```bash
openclaw skill validate
```

Checks:
- SKILL.md exists and is valid YAML frontmatter
- `name` and `description` are present
- `emoji` is a single character if provided
- All config types are valid (`string`, `integer`, `boolean`, `float`)
- Install methods reference valid `kind` values
- Required fields are present

## What's Next?

- [Skill Development guide](../guides/skill-development.md) — full tutorial on building skills
- [Getting Started](../guides/getting-started.md) — create your first skill in 10 minutes
- [Architecture](../guides/architecture.md) — understand OpenClaw's architecture
