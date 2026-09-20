<p align="center">
  <img src="assets/icon.svg" width="112" height="112" alt="Windsurf++ 图标">
</p>

# Windsurf++

> 在 Windsurf 和 Devin 中使用你自己的模型 API Key。

[English](README.md) | [简体中文](README.zh-CN.md)

Windsurf++ 让你在 Windsurf/Devin 中使用自己的模型 API Key、本地推理服务和兼容网关，并直接通过 Cascade 原生模型选择器切换模型。

它由跨平台安装 CLI、Windsurf++ 扩展和本地代理运行时组成。未命中自定义模型或授权不可用时，官方请求保持独立运行。

> [!IMPORTANT]
> Windsurf++ 不是 Windsurf、Devin、Codeium 或任何模型厂商的官方产品。安装过程会修改第三方应用文件，可能受应用升级和服务条款影响。请仅在你有权使用的设备、账号和 API 上安装，不得用于绕过计费、访问控制或服务限制。

## 快速开始

需要 Node.js 18 或更高版本，以及已经安装的 Windsurf 或 Devin。

```bash
# 安装
npm install -g @priders/cwindsurf
cwindsurf install

# 查看状态
cwindsurf status

# 卸载
cwindsurf uninstall

# 帮助
cwindsurf help
```

如果应用安装在非标准位置，可在安装前设置 `CWINDSURF_APP_PATH`，其值可以是应用目录或可执行文件路径。

- Windows 系统级安装可能需要管理员终端。
- Linux 系统目录安装可能需要 `sudo --preserve-env=HOME $(command -v cwindsurf) install`。
- Linux 证书信任支持 Debian/Ubuntu `update-ca-certificates` 和 Fedora/RHEL `update-ca-trust`。

安装完成后：

1. 完全退出并重新打开 Windsurf/Devin。
2. 打开活动栏中的 **Windsurf++**。
3. 使用 LinuxDO 或 GitHub 完成设备登录。
4. 添加 Provider，填写 API 地址、API Key 和模型信息。
5. 测试连接并开启 BYOK。
6. 在 Cascade 原生模型选择器中选择自定义模型。

## 功能特点

- **原生模型选择**：自定义模型直接显示在 Cascade 模型列表中，每个模型可独立配置 Logo（Devin 内置 Provider 图标、Emoji 或上传 PNG/JPG/WebP 图片）。
- **多协议支持**：兼容 OpenAI Chat Completions、OpenAI Responses、Anthropic Messages 和 Google Gemini API，并为仅实现 Responses-Lite 协议的网关提供可选兼容模式。
- **本地与云端模型**：支持 Ollama、LM Studio、vLLM、llama.cpp 以及常见云端兼容服务。
- **Agent 与工具调用**：支持流式文本、reasoning/thinking、函数工具调用、工具结果续写和多轮上下文。
- **可视化配置**：在侧边栏中管理 Provider、模型、上下文窗口、输出限制和协议参数。
- **多 API Key 轮换与故障转移**：同一 Provider 可配置多个 API Key，按轮询、并发负载和会话亲和性选择；遇到鉴权、配额或限流错误时自动切换，并按 `Retry-After` 或退避时间冷却故障 Key。
- **自定义请求头**：默认发送 Devin/Windsurf 官方 `User-Agent`，可在 Provider 级配置自定义请求头，统一应用到聊天、提交信息、连接测试、健康检查和模型列表请求。
- **稳定性控制**：提供流式空闲超时、指数退避、限流等待和安全重连；Devin Local 回合的网络类失败自动续发重试，官方转发请求支持状态级自动重放。
- **用量与成本分析**：统计 Token、费用、延迟、吞吐、成功率、缓存命中和预算预警，支持从 models.dev、LiteLLM 和 OpenRouter 同步模型价格。
- **本地 Commit 生成**：可选择自定义 Provider、模型和输出语言生成提交信息。
- **灵动岛通知**：跨平台原生任务状态通知（macOS/Windows/Linux），支持主题、尺寸、目标屏幕、刘海模式、隐私模式和自定义或内置音效。
- **原生 Cascade 恢复**：可回退补丁恢复 Devin 原生 Cascade 入口、会话选择与输入框，并支持将 Cascade 设为新会话默认 Agent。
- **界面语言偏好**：控制面板支持跟随系统、简体中文和英文三种界面语言。
- **设备授权**：支持 LinuxDO/GitHub 登录、设备管理、远程吊销和签名许可。
- **远程工作台（Beta）**：端到端加密的浏览器/手机远程控制：浏览电脑、项目和会话；远程新建、续接、停止或归档任务；查看文件、Diff 和终端。需要桌面扩展配合自托管或公共 Relay 服务。
- **跨平台安装**：支持 macOS、Windows 和 Linux。

## 支持的 Provider 协议

| 类型 | 标准接口 | 常见用途 |
|---|---|---|
| `openai` | `/chat/completions` | Ollama、LM Studio、vLLM、DeepSeek、OpenRouter 等兼容服务 |
| `openai-responses` | `/responses` | OpenAI Responses API 及兼容网关 |
| `anthropic` | `/messages` | Anthropic Claude API 及 Messages 兼容服务 |
| `gemini` | `/models/*:streamGenerateContent` | Google Gemini `v1` / `v1beta` 流式接口 |

实际兼容性取决于目标服务是否完整实现流式输出、工具调用和用量字段。添加 Provider 后，建议先执行连接测试。对于仅支持 Responses-Lite 协议的网关，可开启 Provider 级或模型级的 Responses-Lite 兼容开关，自动注入所需的路由标记头。

## 工作原理

```text
Windsurf / Devin
  │
  ├─ 本地拦截与代理层
  │   ├─ 识别官方请求与 BYOK 模型
  │   └─ 保持官方流量独立直通
  │
  └─ Windsurf++ 扩展
      ├─ Provider 与模型管理
      ├─ OpenAI / Responses / Anthropic / Gemini 协议适配
      ├─ 流式输出与工具调用转换
      └─ 本地统计、认证和运行状态
```

安装程序会备份需要修改的应用文件。执行 `cwindsurf uninstall` 可先选择备份用户数据，然后移除补丁、扩展、缓存和整个 `~/.cwindsurf/` 目录——如需保留 Provider API Key 或统计数据，请在卸载前自行备份。

## 配置与本地数据

配置默认保存在 `~/.cwindsurf/`，Provider API Key 只保存在本机；Refresh Token 和设备私钥保存在编辑器的 SecretStorage，不写入普通配置文件。统计默认本地聚合，不保存完整 prompt 和响应正文；开发抓包工具不包含在公开构建中，公开产物经过 bundle、压缩、混淆和 Ed25519 签名清单校验：

| 路径 | 内容 |
|---|---|
| `providers.json` | Provider、模型和 API Key |
| `routes.json` | 本地服务、BYOK、稳定性和 Commit 配置 |
| `settings.json` | 日志、统计、保留期和预算设置 |
| `auth/lease.json` | 设备签名许可，不包含 Provider API Key |
| `stats.json` / `stats-deltas/` | 本地聚合统计 |

Provider 示例：

```json
{
  "providers": [
    {
      "id": "ollama",
      "name": "Ollama",
      "type": "openai",
      "baseUrl": "http://127.0.0.1:11434/v1",
      "apiKey": "ollama",
      "apiKeys": ["sk-key-1", "sk-key-2"],
      "customHeaders": { "X-My-Gateway": "value" },
      "enabled": true,
      "models": [
        {
          "id": "qwen3:32b",
          "name": "Qwen3 32B",
          "contextWindow": 131072,
          "responsesLiteGateway": false
        }
      ]
    }
  ]
}
```

- `apiKeys` 为可选的多 API Key 列表（轮换与故障转移），未配置时使用单 `apiKey`。
- `customHeaders` 为可选的自定义请求头。
- `responsesLiteGateway`（或 Provider 级 `requestCompatibility: "responses-lite-gateway"`）仅对 `openai-responses` 类型生效，用于仅支持 Responses-Lite 协议的网关。

## 平台支持

| 平台 | 状态 |
|---|---|
| macOS Apple Silicon / Intel | 支持 |
| Windows | 支持 |
| Linux | 支持 |

应用升级可能覆盖补丁。升级 Windsurf/Devin 后，请运行 `cwindsurf status` 检查状态，必要时重新安装。

## 常见问题

| 问题 | 处理方式 |
|---|---|
| 自定义模型未显示 | 确认 Provider 已启用、本地服务正在运行，然后完全重启应用 |
| 模型显示但无法发送 | 检查登录许可、API 地址、API Key、模型 ID 和协议类型 |
| Provider 测试失败 | 先使用服务商官方示例验证 Key 和模型权限，再检查 `baseUrl` |
| 工具调用后没有最终文本 | 确认目标兼容服务支持工具结果续写，而不只是首次函数调用 |
| 模型 Logo 未显示 | 确认 Logo 已保存（Emoji ≤ 8 个码点、图片 ≤ 16 KiB），重新执行 `cwindsurf install` 并重启应用 |
| Devin Local 频繁网络错误 | 网络类失败会自动续发重试（最多 10 次）；持续失败时检查本地代理状态，或用 `localStorage["cwindsurf.acpAutoRetry"]="off"` 停用自动重试 |
| Responses 网关报协议错误 | 网关仅支持 Responses-Lite 时，开启 Provider 级或模型级 Responses-Lite 兼容开关 |
| 应用升级后失效 | 执行 `cwindsurf status`，重新安装并重启应用 |
| 端口冲突 | 检查默认本地端口是否被其他进程占用，或在控制面板中调整设置 |

## 更新日志

各版本的重要修复与改进随 npm 包发布，详见 [`@priders/cwindsurf`](https://www.npmjs.com/package/@priders/cwindsurf) 中的 CHANGELOG。

## 问题反馈

此仓库用于项目介绍、发布说明和问题反馈，不包含主项目源码。

提交 Issue 时请提供：

- 操作系统和架构。
- Windsurf/Devin 与 Windsurf++ 版本。
- Provider 协议类型和模型 ID。
- 可复现步骤及经过脱敏的错误信息。

请勿提交 API Key、OAuth Token、Refresh Token、设备私钥、完整聊天内容或包含秘密的配置文件。

## 授权说明

当前项目和发布包标记为 `UNLICENSED`。公开可访问不代表允许复制、修改、再发布或商业使用，除非项目维护者另行提供明确许可。
