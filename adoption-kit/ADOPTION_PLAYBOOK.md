# Adoption Playbook

Step-by-step guide for effectorHQ contributors doing outreach to external MCP server projects.

## 1. Pick a target repo

Good candidates:
- Popular MCP servers with 50+ stars
- Repos that already have good documentation
- Projects with active maintainers (recent commits in the last 30 days)
- Servers with multiple tools (more value from type composition)

Where to find them:
- [awesome-mcp-servers](https://github.com/punkpeye/awesome-mcp-servers)
- [MCP Server Registry](https://github.com/modelcontextprotocol/servers)
- GitHub topic: `mcp-server`

## 2. Generate effector.toml

```bash
# Clone the target repo
git clone <repo-url> && cd <repo-name>

# Auto-generate from the MCP server code
effector init --from-mcp .

# Review and refine the generated file
# - Check input/output types (replace "String" / "JSON" with more specific types)
# - Verify permissions match actual code behavior
# - Add tags, description, emoji
```

## 3. Validate

```bash
effector check .
effector check .
```

## 4. File the PR

Use the PR template from `docs/adoption-kit/PR_TEMPLATE.md`.

Key points to emphasize:
- **Zero runtime changes** — this is a metadata file
- **Zero new dependencies** — nothing to install
- **Immediate value** — type checking, composition verification, permission audit
- **Optional CI** — they can add the GitHub Action later if they want

## 5. Respond to feedback

Common questions and answers:

**"Why not just use the MCP tool description?"**
> MCP descriptions are natural language. effector.toml adds machine-readable type contracts that enable static composition checking — you can verify two tools connect before running them.

**"We don't want extra files in our repo"**
> Totally understand. effector.toml is like a `tsconfig.json` or `.eslintrc` — it's opt-in metadata that tooling can use. If it doesn't add value for your workflow, no pressure.

**"What's the maintenance burden?"**
> Near zero. The file only changes when you change your tool's interface (new input types, new permissions). Most projects never touch it after the initial setup.

**"Is this an official MCP thing?"**
> No — effector is an independent open standard that works alongside MCP. It adds a typed layer that MCP doesn't have today.

## 6. Track adoption

After the PR is merged:
- Add the repo to `awesome-openclaw`
- Update the type coverage stats
- Consider writing a case study if the integration is interesting
