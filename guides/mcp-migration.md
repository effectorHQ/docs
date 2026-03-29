# Migrating an MCP Tool to Effector

Already have an MCP tool? Add types and composition in 5 minutes.

## What You Get

- Type-safe interfaces (input/output/context)
- Static composition checking with other capabilities
- Security audit via `effector-audit`
- Automatic discovery by type in registries

## Step 1: Create effector.toml

Next to your existing MCP tool, create `effector.toml`:

```toml
[effector]
name = "your-tool-name"
version = "0.1.0"
type = "skill"
description = "What your tool does in one line"

[effector.interface]
input = "String"        # or CodeDiff, FilePath, JSON, URL, etc.
output = "Markdown"     # or JSON, ReviewReport, Summary, etc.
context = []            # e.g. ["GitHubCredentials"] if needed

[effector.permissions]
network = false         # true if your tool makes HTTP requests
subprocess = false      # true if your tool spawns processes
```

### Choosing Types

Pick from the 42 standard types in `effector-types`:

**Input types:** String, FilePath, URL, JSON, RepositoryRef, CodeDiff, CodeSnippet, IssueRef, PullRequestRef, CommitRef, TextDocument, DataTable, ImageRef, PatchSet, StructuredData

**Output types:** Markdown, JSON, String, ReviewReport, SecurityReport, Notification, SlackMessage, DiscordMessage, OperationStatus, TestResult, DeploymentStatus, LintReport, Summary, TranslatedText

**Context types:** GitHubCredentials, GenericAPIKey, Docker, Kubernetes, AWSCredentials, SlackCredentials, Repository, CodingStandards, ShellEnvironment, UserPreferences, ConversationHistory, PromptContext, APICredentials

## Step 2: Create SKILL.md (optional but recommended)

If your tool doesn't already have a SKILL.md:

```markdown
---
name: your-tool-name
description: What your tool does in one line
---

## Purpose

Detailed description of what this tool does.

## When to Use

- Scenario 1
- Scenario 2

## Examples

\`\`\`bash
# Example usage
your-tool --input "example"
\`\`\`
```

## Step 3: Validate

```bash
# Install the linter
npm install -g @effectorhq/cli

# Validate your SKILL.md
effector check .

# Run security audit
npx @effectorhq/audit scan .
```

## Step 4: Your tool is now composable

Other effector-typed tools can discover and compose with yours:

```js
import { isTypeCompatible } from '@effectorhq/core/types';

// Check if tool A's output can feed into tool B's input
isTypeCompatible(toolA.interface.output, toolB.interface.input);
// → true or false
```

## What Changes (and What Doesn't)

| Aspect | Before | After |
|--------|--------|-------|
| MCP protocol | Unchanged | Unchanged |
| Tool execution | Unchanged | Unchanged |
| Type safety | None | Full |
| Composability | Manual | Automatic |
| Security audit | Manual | `effector-audit scan` |
| Discovery | By name only | By type |

Your MCP tool continues to work exactly as before. The `effector.toml` is a sidecar manifest that adds metadata without changing behavior.
