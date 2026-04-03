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
| Copilot/VS Code marketplace | `.github/plugin/marketplace.json` | Copilot CLI + VS Code marketplace registry |
| VS Code workspace config | `.vscode/settings.json` | Auto-registers marketplace when repo is open in VS Code |

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

VS Code supports the same plugin/marketplace system. Skills, agents, and MCP servers all install through it.

### Option A — Register the marketplace in user settings (recommended)

This makes the marketplace available across all your VS Code instances on this machine.

1. Open VS Code settings: `Ctrl+Shift+P` → **Preferences: Open User Settings (JSON)**
2. Add the marketplace to `chat.plugins.marketplaces`:

```json
"chat.plugins.marketplaces": [
    "ZebLawrence/agent-resources"
]
```

3. Open the Extensions panel (`Ctrl+Shift+X`) and search `@agentPlugins`
4. Find **shared-agent-resources** and click **Install**

All skills, agents, and MCP servers (including Playwright) are now active.

### Option B — Install directly from source

Skip the marketplace and install the plugin in one step:

1. Open the Command Palette: `Ctrl+Shift+P`
2. Run **Chat: Install Plugin From Source**
3. Enter: `https://github.com/ZebLawrence/agent-resources`

### Option C — Workspace auto-discovery (for repos that reference this marketplace)

The repo already includes `.vscode/settings.json` with `extraKnownMarketplaces` and `enabledPlugins`. Any project that has these workspace settings will automatically register the marketplace and prompt to enable the plugin when Copilot Chat opens. To use this pattern in your own projects, copy the `.vscode/settings.json` from this repo into the target project:

```json
{
  "extraKnownMarketplaces": {
    "ZebLawrence-agent-resources": {
      "source": {
        "source": "github",
        "repo": "ZebLawrence/agent-resources"
      }
    }
  },
  "enabledPlugins": {
    "shared-agent-resources@ZebLawrence-agent-resources": true
  }
}
```

VS Code will show a notification on the first chat message prompting you to install the plugin.

### Managing installed plugins (VS Code)

- **View installed:** Extensions panel → **Agent Plugins - Installed**
- **Enable/Disable:** Right-click in the installed view or toggle in the Chat Customizations editor
- **Update:** `Ctrl+Shift+P` → **Extensions: Check for Extension Updates** (or auto-updates every 24h)
- **Uninstall:** Right-click in **Agent Plugins - Installed** → Uninstall

### MCP servers (VS Code — manual fallback)

MCP servers are included automatically when the plugin is installed. If you need to add Playwright manually without the plugin system, open user settings and add:

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

VS Code will prompt you to start the server on first use. The repo's `.vscode/mcp.json` also configures Playwright for the workspace scope when this repo is open.

---

## Adding new skills

1. Copy `skills/template-skill/` to a new folder: `skills/my-new-skill/`
2. Edit `skills/my-new-skill/SKILL.md` — update `name`, `description`, and content
3. Commit and push to GitHub
4. On each machine, run `/plugin update shared-agent-resources` (Claude CLI / Copilot CLI) or `Ctrl+Shift+P` → **Extensions: Check for Extension Updates** (VS Code)

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

## Notes

The Claude Code plugin/marketplace system (`/plugin` commands) is a relatively new feature. If a command fails, verify the current syntax in the [Claude Code docs](https://docs.anthropic.com/claude-code). The VS Code plugin system is documented at [code.visualstudio.com/docs/copilot/customization/agent-plugins](https://code.visualstudio.com/docs/copilot/customization/agent-plugins).
