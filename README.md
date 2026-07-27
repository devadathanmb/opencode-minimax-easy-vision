# OpenCode Easy Vision

An [OpenCode](https://opencode.ai) plugin that adds **vision support** to models that lack it. Paste images directly into the chat and ask questions, just like you would with Claude or GPT.

> [!IMPORTANT]
>
> **Renamed from `opencode-minimax-easy-vision`:** Existing users only need to replace the old package name in the `plugin` array with `opencode-easy-vision`. Existing plugin configuration continues to work; no other migration is required.

## Table of Contents

- [The Problem](#the-problem)
- [Demo](#demo)
- [Setup](#setup)
  - [Manual Setup](#manual-setup)
  - [Agent-Assisted Setup](#agent-assisted-setup)
- [Usage](#usage)
- [Configuration](#configuration)
- [Supported Image Formats](#supported-image-formats)
- [Troubleshooting](#troubleshooting)
- [Uninstallation](#uninstallation)
- [Contributing](#contributing)
- [License](#license)
- [References](#references)

## The Problem

Many models, especially open-weight ones, have no vision capability. They are text-only and cannot process image input. For some models, such as MiniMax, an MCP tool can read an image through a vision service and return the analysis as text. The tool needs a local file path rather than a clipboard paste, so you would otherwise need to save each screenshot and provide its path yourself.

This plugin automates the entire workaround. It intercepts pasted images, saves them to disk, and injects the instructions the model needs to call the MCP tool with the correct path. You paste an image and ask a question. The plugin handles the rest.

## Demo

| Without the plugin | With the plugin |
| :-: | :-: |
| <img width="100%" alt="Without plugin screenshot" src="https://github.com/user-attachments/assets/eb5c76ce-e1a6-4ad0-aa51-58ce5e8efd8f" /> | <img width="100%" alt="With plugin screenshot" src="https://github.com/user-attachments/assets/b5bcaac4-8ac8-48eb-90cb-59630c26f4e1" /> |
| The image is silently ignored by the model. | The model analyzes the attached image correctly before answering. |


https://github.com/user-attachments/assets/da7d8618-5de1-4a00-9250-a91493e3ea16

https://github.com/user-attachments/assets/fdb68339-b95b-46eb-90d4-ac2dbc0f436e

## Setup

### Manual Setup

#### 1. Configure an MCP image analysis tool

The plugin works with any MCP server that can analyze an image from a local path or URL and return its analysis as text. Add one to your `opencode.json`.

**Default: MiniMax Coding Plan MCP**

```json
{
  "mcp": {
    "MiniMax": {
      "type": "local",
      "command": ["uvx", "minimax-coding-plan-mcp"],
      "environment": {
        "MINIMAX_API_KEY": "your-api-key-here",
        "MINIMAX_API_HOST": "https://api.minimax.io"
      }
    }
  }
}
```

**Alternative: [openrouter-image-mcp](https://github.com/JonathanJude/openrouter-image-mcp)**

Routes image analysis through OpenRouter, giving you access to any vision-capable model including free ones.

```json
{
  "mcp": {
    "openrouter_image": {
      "type": "local",
      "command": ["npx", "openrouter-image-mcp"],
      "environment": {
        "OPENROUTER_API_KEY": "your-api-key-here",
        "OPENROUTER_MODEL": "nvidia/nemotron-3-nano-omni-30b-a3b-reasoning:free"
      }
    }
  }
}
```

> [!TIP]
>
> `nvidia/nemotron-3-nano-omni-30b-a3b-reasoning:free` is a free multimodal model designed for image perception and reasoning. Free model availability changes over time, so check [OpenRouter's free models](https://openrouter.ai/models?fmt=cards&max_price=0) for current alternatives. Free endpoints use shared capacity and may respond more slowly or time out under load. When using this server, set `imageAnalysisTool` to `openrouter_image_analyze_image` in the plugin config.

> [!NOTE]
>
> Any MCP server with an image analysis tool will work. The servers above are examples. For a different tool, set `imageAnalysisTool`; see [Configuration](#configuration).

#### 2. Install the plugin

**With the OpenCode CLI (v1.3.4+):**

Global (all projects):

```bash
opencode plugin opencode-easy-vision --global
```

Project-level (current directory only):

```bash
opencode plugin opencode-easy-vision
```

**Or manually,** add it to the `plugin` array in your `opencode.json`:

```json
{
  "$schema": "https://opencode.ai/config.json",
  "plugin": ["opencode-easy-vision"]
}
```

---

### Agent-Assisted Setup

**Paste this into OpenCode and let your agent handle the rest:**

```
Set up opencode-easy-vision by following https://raw.githubusercontent.com/devadathanmb/opencode-easy-vision/main/AGENT_SETUP.md
```

<details>
<summary><b>What the agent will do</b></summary>

The agent follows the instructions in [AGENT_SETUP.md](./AGENT_SETUP.md). It will:

1. Inspect your existing OpenCode and plugin configuration
2. Ask whether to set up the plugin for all projects or only the current project
3. Ask which MCP image analysis tool and models you want to use
4. Update existing files in place instead of replacing them
5. Summarize the changes and guide you through verification after a restart

</details>

## Usage

1. Select a configured model in OpenCode.
2. Paste an image (`Cmd+V` / `Ctrl+V`).
3. Ask your question, just like you would with Claude or GPT.

## Configuration

> [!IMPORTANT]
>
> By default, this plugin only activates for **MiniMax provider models**, where MiniMax is the direct provider in OpenCode. These IDs match `minimax/*`, `minimax-cn/*`, and related patterns. If you access a MiniMax model through a third-party provider such as OpenRouter, or use another model, add that model's pattern to `models`. See [CONFIGURATION.md](./CONFIGURATION.md).

Each option uses the following priority order:

1. **Project level**: `.opencode/opencode-easy-vision.json` (or `.jsonc`)
2. **User level**: `~/.config/opencode/opencode-easy-vision.json` (or `.jsonc`)

On first load, an example config is created at `~/.config/opencode/opencode-easy-vision.jsonc`. See [CONFIGURATION.md](./CONFIGURATION.md) for config precedence, legacy config support, and the full option reference.

## Supported Image Formats

PNG, JPEG, and WebP. Exact format support depends on the image analysis tool you configure.

## Troubleshooting

**Plugin not updating after a new release?**

OpenCode caches plugins under `~/.cache/opencode/packages/`. If it's still running an old version after a release, clear the cache entry and restart:

```bash
rm -rf ~/.cache/opencode/packages/opencode-easy-vision@latest
```

**Plugin not activating?**

By default, the plugin only activates for MiniMax provider models, with IDs matching `minimax/*`, `minimax-cn/*`, and related patterns. It does not activate for a MiniMax model accessed through OpenRouter or another provider. To use the plugin with a different model or provider, add the model's ID pattern to `models` in your config. See [CONFIGURATION.md](./CONFIGURATION.md).

## Uninstallation

1. Remove `opencode-easy-vision` from the `plugin` array in the `opencode.json` file where you installed it.

2. Delete the plugin config files you no longer need:

```bash
rm -f ~/.config/opencode/opencode-easy-vision.{json,jsonc}
rm -f .opencode/opencode-easy-vision.{json,jsonc}
```

If you migrated from `opencode-minimax-easy-vision` and no longer use it, you can also delete its legacy config files.

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md) for local development setup.

## License

AGPL-3.0. See [LICENSE](./LICENSE).

## References

- [OpenCode Official Website](https://opencode.ai)
- [OpenCode Plugins Documentation](https://opencode.ai/docs/plugins/)
- [MiniMax Coding Plan MCP Repository](https://github.com/MiniMax-AI/MiniMax-Coding-Plan-MCP)
- [openrouter-image-mcp Repository](https://github.com/JonathanJude/openrouter-image-mcp)
