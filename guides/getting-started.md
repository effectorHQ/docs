# Your First OpenClaw Skill in 10 Minutes

Build and share your first skill in less than 10 minutes.

## What You'll Build

A simple skill that echoes back a message. You'll create it, test it locally, and publish it to ClawHub.

## Prerequisites

- OpenClaw installed and workspace initialized
- A code editor (VS Code, Vim, etc.)
- 5 minutes

## Step 1: Create Your Skill Directory

OpenClaw auto-discovers skills in your workspace. Create a directory:

```bash
mkdir -p ~/.openclaw/workspace/skills/my-first-skill
cd ~/.openclaw/workspace/skills/my-first-skill
```

## Step 2: Create SKILL.md

Create a file named `SKILL.md` with frontmatter and body:

```markdown
---
name: my-first-skill
description: A simple skill that echoes messages
metadata:
  openclaw:
    emoji: 📢
---

# Echo Skill

This skill echoes back whatever you send it.

## Usage

Send a message like: "echo hello world"

The skill will respond with: "You said: hello world"
```

The frontmatter declares your skill metadata. The body is markdown documentation.

## Step 3: Test It Locally

OpenClaw auto-discovers skills in your workspace. Once `SKILL.md` is in place, the skill is available:

```bash
openclaw skill list
```

You should see `my-first-skill` in the output.

To test execution (refer to OpenClaw docs for the test interface specific to your OpenClaw version).

## Step 4: Add Requirements (Optional)

If your skill needs external binaries or environment variables, declare them in the frontmatter:

```yaml
metadata:
  openclaw:
    requires:
      bins:
        - name: python3
          version: ">=3.9"
      env:
        - name: MY_API_KEY
          description: "API key for the service"
```

## Step 5: Add Installation Instructions (Optional)

If your skill needs setup, add installation methods:

```yaml
metadata:
  openclaw:
    install:
      - id: homebrew
        kind: brew
        formula: python@3.11
        label: "Install via Homebrew"
      - id: ubuntu
        kind: apt
        package: python3.11
        label: "Install via apt"
      - id: manual
        kind: manual
        label: "Manual installation"
```

## Step 6: Publish to ClawHub

When you're ready to share:

```bash
cd ~/.openclaw/workspace/skills/my-first-skill
clawhub publish
```

This publishes your skill to the ClawHub registry with vector search indexing. Users can now discover and install it:

```bash
openclaw skill install my-first-skill
```

## What's Next?

- [Architecture guide](architecture.md) — understand how skills fit into OpenClaw
- [Skill Development guide](skill-development.md) — deep dive into SKILL.md, tiers, discovery
- [ClawHub](https://clawhub.example.com) — browse community skills

Congratulations! You've built and published your first OpenClaw skill.
