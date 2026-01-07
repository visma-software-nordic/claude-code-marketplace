# Claude Code Marketplace

A Claude Code plugin marketplace for Visma Software Nordic.

## Installation

From inside Claude Code, add the marketplace:

```
/plugin marketplace add visma-software-nordic/claude-code-marketplace
```

Then install plugins:

```
/plugin install ngx-gaia@visma-software-nordic-marketplace
```

## Team Setup

To share this marketplace with your team by default, add this to your project's `.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "visma-software-nordic-marketplace": {
      "source": {
        "source": "github",
        "repo": "visma-software-nordic/claude-code-marketplace"
      }
    }
  },
  "enabledPlugins": {
    "ngx-gaia@visma-software-nordic-marketplace": true
  }
}
```

## Available Plugins

| Plugin | Description |
|--------|-------------|
| **ngx-gaia** | Provides usage guidance for `@vsn-ux/ngx-gaia` Angular components (Gaia design system) |
