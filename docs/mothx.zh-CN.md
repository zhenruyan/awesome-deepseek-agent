[English](./mothx.md) | [简体中文](./mothx.zh-CN.md) · [← Back](../README.zh-CN.md)

# 集成 MothX（默思）

MothX（原名 **VibeCoding**，中文名 **默思**）是一款用纯 Go 编写的全能型终端 AI 编码助手。它内置 20+ 供应商适配器、沙箱、SQLite 会话、技能、工作流和 OpenAI 兼容的 serve 模式，全部打包为单个二进制文件，零外部依赖。**DeepSeek 为默认供应商**，模型目录已预置最新的 `deepseek-v4-pro` / `deepseek-v4-flash` 名称、100 万 token 上下文以及 `max` 推理强度。

- **GitHub：** <https://github.com/startvibecoding/mothx>
- **国内镜像（Gitee）：** <https://gitee.com/startvibecoding/mothx>

> **改名说明：** 本过渡版本仍保留 `vibecoding` 命令、旧安装包名以及 `VIBECODING_*` 环境变量作为兼容入口；发现旧 `.vibecoding` 和 `.vibe` 目录时会自动迁移到 `.mothx`。新安装请直接使用 `mothx`。

#### 1. 安装 MothX

一键安装脚本会优先使用已有的 Node.js LTS；若未检测到，则先自动安装对应系统的 Node.js，再通过 npm 安装最新版 `mothx`。

Linux / macOS / FreeBSD：

```bash
curl -fsSL https://mothx.net/install.sh | bash
```

Windows（命令提示符）：

```bat
curl.exe -fsSL https://mothx.net/install.bat -o install.bat && install.bat
```

Docker（GHCR，Linux amd64 / arm64）：

```bash
docker run --rm -it -v "$PWD:/workspace" -w /workspace \
  ghcr.io/startvibecoding/mothx:latest
```

验证安装是否成功：

```bash
mothx --version
```

#### 2. 配置 DeepSeek

从 [DeepSeek 开放平台](https://platform.deepseek.com/api_keys) 获取你的 API Key 并导出。MothX 默认读取 `DEEPSEEK_API_KEY`。

Linux / macOS：

```bash
export DEEPSEEK_API_KEY="sk-..."
```

Windows（PowerShell）：

```powershell
$env:DEEPSEEK_API_KEY="sk-..."
```

MothX 内置两个 DeepSeek 供应商，按你偏好的 API 形态二选一即可：

| 供应商名称            | 接口地址                              | API 形态            |
| --------------------- | ------------------------------------- | ------------------- |
| `deepseek-openai`     | `https://api.deepseek.com`            | OpenAI Chat Completions |
| `deepseek-anthropic`  | `https://api.deepseek.com/anthropic`  | Anthropic Messages  |

两个供应商都已预置 `deepseek-v4-pro` 和 `deepseek-v4-flash`（100 万上下文、384K 最大输出、开启推理）。如需锁定 Pro 模型并启用 `max` 推理强度，创建 `~/.mothx/settings.json`（Windows：`%APPDATA%\mothx\settings.json`）：

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

**关键配置项：**

| 配置项                  | 说明                                                                                                         |
| ----------------------- | ------------------------------------------------------------------------------------------------------------ |
| `defaultProvider`       | `deepseek-openai`（默认）或 `deepseek-anthropic`                                                            |
| `defaultModel`          | `deepseek-v4-pro` 或 `deepseek-v4-flash`                                                                    |
| `defaultThinkingLevel`  | `off`、`minimal`、`low`、`medium`、`high`、`xhigh`。`xhigh` 对应 DeepSeek 的 `reasoning_effort: "max"`。   |
| `maxContextTokens`      | DeepSeek V4 最高支持 100 万 token；设为 `1000000` 即可启用完整窗口。                                        |
| `sandbox.enabled`       | 在 Linux 上启用 bwrap 沙箱，安全隔离文件与网络。                                                            |
| `compaction.enabled`    | 自动压缩长会话以保持在上下文预算内。                                                                         |

> OpenAI 兼容端点会在请求体里发送 `reasoning_effort`；Anthropic 兼容端点使用 DeepSeek 原生的 `output_config: { effort: "max" }`。两种方式都由 MothX 自动处理，无需手工修补。

#### 3. 运行并切换模型

```bash
cd /path/to/my-project
mothx
```

进入 TUI 后，可使用以下斜杠命令与快捷键：

| 命令 / 按键             | 功能                                                |
| ----------------------- | --------------------------------------------------- |
| `/model`                | 打开模型切换器（选择 V4-Pro 或 V4-Flash）。          |
| `/think`                | 循环切换思考级别（`off → … → xhigh`）。              |
| `/mode plan\|agent\|yolo` | 切换沙箱/安全模式。                                  |
| `Tab`                   | 快速循环切换思考级别。                              |
| `/clear`、`/quit`       | 清空对话、退出 MothX。                               |

非交互式一次性执行：

```bash
# 在单次运行中指定供应商/模型/思考级别
mothx --provider deepseek-openai --model deepseek-v4-pro -t xhigh -P "把这个函数重构为泛型版本"
```

#### 定价（DeepSeek V4）

最新价格请以 [DeepSeek 定价页](https://api-docs.deepseek.com/zh-cn/quick_start/pricing) 为准。下表为参考快照：

| 模型                | 输入 / 百万 token | 输出 / 百万 token | 缓存命中 / 百万 token |
| ------------------- | ----------------- | ----------------- | -------------------- |
| `deepseek-v4-pro`   | $0.435            | $0.87             | $0.003625            |
| `deepseek-v4-flash` | $0.14             | $0.28             | $0.0028              |

#### 更多

- [MothX 文档](https://github.com/startvibecoding/mothx#readme) — 供应商指南、serve 模式、ACP/A2A、沙箱、技能、工作流。
- 卸载：`npm uninstall -g mothx-installer`（或 `curl -fsSL https://mothx.net/install.sh | bash -s -- --uninstall`）。
