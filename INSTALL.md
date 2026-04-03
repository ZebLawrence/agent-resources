# Installation Guide

This repo acts as both a **plugin** and a **marketplace** for Claude Code and GitHub Copilot. Install once on each machine and all skills, agents, and MCP servers are available.

---

## What's included

| Resource | Location | Description |
|---|---|---|
| `skills/template-skill` | `skills/` | Template for building new skills |
| Playwright MCP | `.mcp.json` | Browser automation via Playwright |
| Plugin manifest | `plugin.json` | Shared manifest for both platforms |
| Claude marketplace | `.claude-plugin/marketplace.json` | Claude Code marketplace registry |
| Copilot marketplace | `.github/plugin/marketplace.json` | Copilot marketplace registry |

---

## Claude Code (CLI and Desktop)

Claude Code CLI and Desktop share the same configuration, so these steps work for both.

### Option A — Install as a marketplace (recommended)

This registers the repo as a marketplace and lets you install/update the plugin with one command.

```bash
# 1. Add this repo as a marketplace source
/plugin marketplace add ZebLawrence/agent-resources

# 2. Install the plugin from the marketplace
/plugin install shared-agent-resources@ZebLawrence-agent-resources

# 3. Verify skills are loaded
/skills list
```

To update when new skills or MCP servers are added:
```bash
/plugin update shared-agent-resources
```

To remove:
```bash
/plugin uninstall shared-agent-resources
/plugin marketplace remove ZebLawrence-agent-resources
```

### Option B — Install directly as a plugin

If you don't want the marketplace layer:

```bash
/plugin install ZebLawrence/agent-resources
```

### MCP servers (Claude Code)

MCP servers defined in `.mcp.json` are installed automatically when the plugin is installed. To verify Playwright is active, check:

```bash
/mcp
```

You should see `playwright` listed as an active server.

---

## GitHub Copilot CLI

The standalone GitHub Copilot CLI uses the same plugin/marketplace commands as Claude Code but with the `copilot` prefix.

> **Which CLI?** These commands are for the standalone **GitHub Copilot CLI** (not the `gh copilot` extension). Install it from [cli.github.com/copilot](https://cli.github.com/copilot).

### Option A — Install as a marketplace (recommended)

```bash
# 1. Add this repo as a marketplace source
copilot plugin marketplace add ZebLawrence/agent-resources

# 2. Browse available plugins (optional)
copilot plugin marketplace browse ZebLawrence-agent-resources

# 3. Install the plugin
copilot plugin install shared-agent-resources@ZebLawrence-agent-resources

# 4. Verify
/skills list
```

To update:
```bash
copilot plugin update shared-agent-resources
```

### Option B — Install directly as a plugin

```bash
copilot plugin install ZebLawrence/agent-resources
```

### MCP servers (Copilot CLI)

MCP servers in `.mcp.json` are installed when the plugin is installed. To verify:
```bash
/mcp show playwright
```

---

## GitHub Copilot in VS Code

VS Code does not use the plugin/marketplace system. Skills and agents must be installed manually, and MCP servers are configured separately.

### MCP servers (VS Code)

**Option A — Workspace config** (per-project, committed to the repo):

The repo already includes `.vscode/mcp.json`. If you open this repo in VS Code, Playwright MCP is available automatically.

For other projects, copy `.vscode/mcp.json` into the project root or add the server to your user settings (see Option B).

**Option B — User settings** (all projects on this machine):

1. Open VS Code settings: `Ctrl+Shift+P` → `Preferences: Open User Settings (JSON)`
2. Add the following inside the root JSON object:

```json
"mcp": {
  "servers": {
    "playwright": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@playwright/mcp@latest"]
    }
  }
}
```

3. Reload VS Code. Copilot will prompt you to start the MCP server on first use.

### Skills and agents (VS Code)

VS Code Copilot reads skills and agents from:
- **Project-level:** `.github/skills/<skill-name>/SKILL.md`
- **Personal:** `~/.copilot/skills/<skill-name>/SKILL.md` (Windows: `%USERPROFILE%\.copilot\skills\`)

To install skills from this repo manually:

```bash
# PowerShell — install all skills to your personal Copilot folder
$dest = "$env:USERPROFILE\.copilot\skills"
New-Item -ItemType Directory -Force -Path $dest
git clone https://github.com/ZebLawrence/agent-resources /tmp/agent-resources
Copy-Item -Recurse /tmp/agent-resources/skills/* $dest
```

Or clone the repo and symlink the skills folder for automatic updates:

```powershell
# Clone once
git clone https://github.com/ZebLawrence/agent-resources "$env:USERPROFILE\agent-resources"

# Symlink (run as Administrator)
New-Item -ItemType Junction -Path "$env:USERPROFILE\.copilot\skills" -Target "$env:USERPROFILE\agent-resources\skills"
```

---

## Adding new skills

1. Copy `skills/template-skill/` to a new folder: `skills/my-new-skill/`
2. Edit `skills/my-new-skill/SKILL.md` — update `name`, `description`, and content
3. Commit and push to GitHub
4. On each machine, run `/plugin update shared-agent-resources` (Claude/Copilot CLI) or re-pull and re-copy (VS Code)

## Adding new MCP servers

1. Add an entry to `.mcp.json`
2. Commit and push
3. Run `/plugin update shared-agent-resources` on each machine

---

## Troubleshooting

| Problem | Fix |
|---|---|
| Skill not showing up | Run `/skills reload` or restart the CLI |
| MCP server not starting | Ensure `npx` is on your PATH; run `npx -y @playwright/mcp@latest` directly to test |
| Plugin install fails | Verify you're authenticated: `gh auth status` |
| VS Code MCP server offline | Click the MCP indicator in the status bar → Start server |

---

## Notes on accuracy

The Claude Code plugin/marketplace system (`/plugin` commands) is a relatively new feature. If a command fails, verify the current syntax in the [Claude Code docs](https://docs.anthropic.com/claude-code). The Copilot CLI plugin system mirrors Claude's closely — both use the same `plugin.json` and `marketplace.json` schema.
