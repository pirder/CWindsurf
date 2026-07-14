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
```

安装完成后：

1. 完全退出并重新打开 Windsurf/Devin。
2. 打开活动栏中的 **Windsurf++**。
3. 使用 LinuxDO 或 GitHub 完成设备登录。
4. 添加 Provider，填写 API 地址、API Key 和模型信息。
5. 测试连接并开启 BYOK。
6. 在 Cascade 原生模型选择器中选择自定义模型。

## 功能特点

- **原生模型选择**：自定义模型直接显示在 Cascade 模型列表中。
- **多协议支持**：兼容 OpenAI Chat Completions、OpenAI Responses、Anthropic Messages 和 Google Gemini API。
- **本地与云端模型**：支持 Ollama、LM Studio、vLLM、llama.cpp 以及常见云端兼容服务。
- **Agent 与工具调用**：支持流式文本、reasoning/thinking、函数工具调用、工具结果续写和多轮上下文。
- **可视化配置**：在侧边栏中管理 Provider、模型、上下文窗口、输出限制和协议参数。
- **稳定性控制**：提供流式空闲超时、指数退避、限流等待和安全重连设置。
- **用量与成本分析**：统计 Token、费用、延迟、吞吐、成功率、缓存命中和预算预警。
- **本地 Commit 生成**：可选择自定义 Provider、模型和输出语言生成提交信息。
- **设备授权**：支持 LinuxDO/GitHub 登录、设备管理、远程吊销和签名许可。
- **跨平台安装**：支持 macOS、Windows 和 Linux。

## 支持的 Provider 协议

| 类型 | 标准接口 | 常见用途 |
|---|---|---|
| `openai` | `/chat/completions` | Ollama、LM Studio、vLLM、DeepSeek、OpenRouter 等兼容服务 |
| `openai-responses` | `/responses` | OpenAI Responses API 及兼容网关 |
| `anthropic` | `/messages` | Anthropic Claude API 及 Messages 兼容服务 |
| `gemini` | `/models/*:streamGenerateContent` | Google Gemini `v1` / `v1beta` 流式接口 |

实际兼容性取决于目标服务是否完整实现流式输出、工具调用和用量字段。添加 Provider 后，建议先执行连接测试。

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

安装程序会备份需要修改的应用文件。执行 `cwindsurf uninstall` 可移除补丁和扩展；卸载前可选择备份用户配置。

## 配置与本地数据

配置默认保存在 `~/.cwindsurf/`：

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
      "enabled": true,
      "models": [
        {
          "id": "qwen3:32b",
          "name": "Qwen3 32B",
          "contextWindow": 131072
        }
      ]
    }
  ]
}
```

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
| 应用升级后失效 | 执行 `cwindsurf status`，重新安装并重启应用 |
| 端口冲突 | 检查默认本地端口是否被其他进程占用，或在控制面板中调整设置 |

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
