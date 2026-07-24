[English](./mothx.md) | [简体中文](./mothx.zh-CN.md) · [← Back](../README.md)

# Integrate with MothX (默思)

MothX (formerly **VibeCoding**) is an all-in-one terminal AI coding assistant written in pure Go. It ships with 20+ provider adapters, an integrated sandbox, SQLite sessions, skills, workflows, and an OpenAI-compatible serve mode — packed into a single binary with zero external dependencies. **DeepSeek is the default provider**, and the model catalog ships preconfigured with the current `deepseek-v4-pro` / `deepseek-v4-flash` names, 1M context window, and `max` reasoning effort.

- **GitHub:** <https://github.com/startvibecoding/mothx>
- **Chinese mirror (Gitee):** <https://gitee.com/startvibecoding/mothx>

> **Rename notice:** The `vibecoding` command, the old installer package names, and `VIBECODING_*` environment variables are kept for compatibility. Legacy `.vibecoding` and `.vibe` directories are automatically migrated to `.mothx` when found. Use `mothx` for new setups.

#### 1. Install MothX

The one-line installer prefers an existing Node.js LTS install; if none is found it installs Node.js for you and then installs the latest `mothx` via npm.

Linux / macOS / FreeBSD:

```bash
curl -fsSL https://mothx.net/install.sh | bash
```

Windows (Command Prompt):

```bat
curl.exe -fsSL https://mothx.net/install.bat -o install.bat && install.bat
```

Docker (GHCR, Linux amd64 / arm64):

```bash
docker run --rm -it -v "$PWD:/workspace" -w /workspace \
  ghcr.io/startvibecoding/mothx:latest
```

Verify the installation:

```bash
mothx --version
```

#### 2. Configure DeepSeek

Get your API Key from the [DeepSeek Platform](https://platform.deepseek.com/api_keys) and export it. MothX reads `DEEPSEEK_API_KEY` by default.

Linux / macOS:

```bash
export DEEPSEEK_API_KEY="sk-..."
```

Windows (PowerShell):

```powershell
$env:DEEPSEEK_API_KEY="sk-..."
```

MothX ships two built-in DeepSeek providers — pick whichever API shape your workflow prefers:

| Provider name          | Endpoint                              | API shape            |
| ---------------------- | ------------------------------------- | -------------------- |
| `deepseek-openai`      | `https://api.deepseek.com`            | OpenAI Chat Completions |
| `deepseek-anthropic`   | `https://api.deepseek.com/anthropic`  | Anthropic Messages   |

Both providers are preconfigured with `deepseek-v4-pro` and `deepseek-v4-flash` (1M context, 384K max output, reasoning enabled). To pin the Pro model and enable `max` thinking effort, create `~/.mothx/settings.json` (Windows: `%APPDATA%\mothx\settings.json`):

```json
{
  "defaultProvider": "deepseek-openai",
  "defaultModel": "deepseek-v4-pro",
  "defaultThinkingLevel": "xhigh",
  "defaultMode": "agent",
  "maxContextTokens": 1000000,
  "compaction": {
    "enabled": true,
    "reserveTokens": 16384,
    "keepRecentTokens": 20000
  },
  "sandbox": {
    "enabled": true,
    "level": "standard",
    "allowNetwork": false
  }
}
```

**Key settings:**

| Setting                  | Description                                                                                                  |
| ------------------------ | ------------------------------------------------------------------------------------------------------------ |
| `defaultProvider`        | `deepseek-openai` (default) or `deepseek-anthropic`                                                          |
| `defaultModel`           | `deepseek-v4-pro` or `deepseek-v4-flash`                                                                     |
| `defaultThinkingLevel`   | `off`, `minimal`, `low`, `medium`, `high`, `xhigh`. `xhigh` maps to DeepSeek's `reasoning_effort: "max"`.   |
| `maxContextTokens`       | DeepSeek V4 supports up to 1,000,000 tokens; set this to `1000000` to use the full window.                  |
| `sandbox.enabled`        | Enable bwrap sandbox on Linux for safe file/network isolation.                                              |
| `compaction.enabled`     | Auto-compact long sessions to stay within the context budget.                                                |

> The OpenAI-compatible endpoint sends `reasoning_effort` in the request body; the Anthropic-compatible endpoint uses DeepSeek's native `output_config: { effort: "max" }`. Both are handled automatically by MothX — no manual patching required.

#### 3. Run and switch models

```bash
cd /path/to/my-project
mothx
```

Inside the TUI, use these slash commands and shortcuts:

| Command / Key          | Action                                             |
| ---------------------- | -------------------------------------------------- |
| `/model`               | Open the model switcher (pick V4-Pro or V4-Flash). |
| `/think`               | Cycle thinking levels (`off → … → xhigh`).         |
| `/mode plan\|agent\|yolo` | Switch sandbox/safety mode.                       |
| `Tab`                  | Cycle thinking levels quickly.                     |
| `/clear`, `/quit`      | Clear conversation, quit MothX.                    |

Non-interactive one-shot:

```bash
# Pin provider/model/thinking on a single run
mothx --provider deepseek-openai --model deepseek-v4-pro -t xhigh -P "Refactor this function to use generics"
```

#### Pricing (DeepSeek V4)

Verify the latest numbers on the [DeepSeek pricing page](https://api-docs.deepseek.com/quick_start/pricing). Snapshot for reference:

| Model             | Input / M tokens | Output / M tokens | Cache Hit / M tokens |
| ----------------- | ----------------- | ----------------- | -------------------- |
| `deepseek-v4-pro`   | $0.435            | $0.87             | $0.003625            |
| `deepseek-v4-flash` | $0.14             | $0.28             | $0.0028              |

#### More

- [MothX docs](https://github.com/startvibecoding/mothx#readme) — provider guide, serve mode, ACP/A2A, sandbox, skills, workflows.
- Uninstall: `npm uninstall -g mothx-installer` (or `curl -fsSL https://mothx.net/install.sh | bash -s -- --uninstall`).
