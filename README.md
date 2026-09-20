<p align="center">
  <img src="assets/icon.svg" width="112" height="112" alt="Windsurf++ icon">
</p>

# Windsurf++

> Bring Your Own Key for Windsurf and Devin.

[English](README.md) | [简体中文](README.zh-CN.md)

Windsurf++ lets you use your own model API keys, local inference servers, and compatible gateways in Windsurf/Devin while keeping model selection inside Cascade's native model picker.

It consists of a cross-platform installer CLI, the Windsurf++ extension, and a local proxy runtime. Official traffic remains independent when a request does not target a custom model or when BYOK authorization is unavailable.

> [!IMPORTANT]
> Windsurf++ is not an official product of Windsurf, Devin, Codeium, or any model provider. Installation modifies third-party application files and may be affected by application updates and applicable terms. Use it only on devices, accounts, and APIs you are authorized to access. Do not use it to bypass billing, access controls, or service restrictions.

## Quick Start

Node.js 18 or later and an installed copy of Windsurf or Devin are required.

```bash
# Install
npm install -g @priders/cwindsurf
cwindsurf install

# Check status
cwindsurf status

# Uninstall
cwindsurf uninstall

# Help
cwindsurf help
```

If the application is installed in a non-standard location, set `CWINDSURF_APP_PATH` before installing; its value can be the application directory or an executable path.

- Windows system-level installs may require an administrator terminal.
- On Linux, system directories may require `sudo --preserve-env=HOME $(command -v cwindsurf) install`.
- Linux certificate trust supports Debian/Ubuntu `update-ca-certificates` and Fedora/RHEL `update-ca-trust`.

After installation:

1. Fully quit and reopen Windsurf/Devin.
2. Open **Windsurf++** from the activity bar.
3. Sign in with LinuxDO or GitHub to authorize the device.
4. Add a provider with its API endpoint, API key, and model definitions.
5. Test the connection and enable BYOK.
6. Select the custom model from Cascade's native model picker.

## Features

- **Native model selection** — Custom models appear directly in Cascade's model list, with per-model logos (built-in provider icons, emoji, or uploaded PNG/JPG/WebP images).
- **Multiple protocols** — OpenAI Chat Completions, OpenAI Responses, Anthropic Messages, and Google Gemini APIs, plus an optional Responses-Lite gateway compatibility mode for gateways that only implement that protocol.
- **Local and cloud models** — Ollama, LM Studio, vLLM, llama.cpp, and common compatible cloud services.
- **Agent tool calling** — Streaming text, reasoning/thinking, function calls, tool-result continuation, and multi-turn context.
- **Visual configuration** — Manage providers, models, context limits, output limits, and protocol options from the sidebar.
- **Multi-key rotation and failover** — Configure multiple API keys per provider with polling, load, and session affinity; keys are automatically rotated on auth, quota, or rate-limit errors with cooldown and `Retry-After` support.
- **Custom request headers** — Send the official Devin/Windsurf `User-Agent` by default and attach provider-level custom headers to chat, commit generation, connection tests, health checks, and model listing.
- **Stream reliability** — Idle timeouts, exponential backoff, rate-limit delays, controlled reconnect behavior, network-class auto-retry for Devin Local turns, and status-level retry for official forwarded requests.
- **Usage and cost analytics** — Tokens, estimated cost, latency, throughput, success rate, cache usage, and budget alerts, with model prices synced from models.dev, LiteLLM, and OpenRouter.
- **Local commit messages** — Choose a provider, model, and language for commit-message generation.
- **Dynamic Island notifications** — Cross-platform native task status notifications (macOS, Windows, Linux) with themes, sizes, target screens, notch mode, privacy mode, and custom or built-in sounds.
- **Native Cascade restore** — Reversible patch that restores Devin's native Cascade entry, session picker, and composer, and can set Cascade as the default agent for new conversations.
- **UI language preference** — Choose between system language, Simplified Chinese, and English for the Windsurf++ panel.
- **Device authorization** — LinuxDO/GitHub sign-in, device management, remote revocation, and signed leases.
- **Remote workbench (Beta)** — End-to-end encrypted remote control from a browser or phone: browse computers, projects, and sessions; create, continue, stop, or archive tasks; inspect files, diffs, and terminals. Requires the desktop extension plus a self-hosted or public relay.
- **Cross-platform installer** — macOS, Windows, and Linux.

## Provider Protocols

| Type | Standard endpoint | Typical use |
|---|---|---|
| `openai` | `/chat/completions` | Ollama, LM Studio, vLLM, DeepSeek, OpenRouter, and compatible services |
| `openai-responses` | `/responses` | OpenAI Responses API and compatible gateways |
| `anthropic` | `/messages` | Anthropic Claude API and Messages-compatible services |
| `gemini` | `/models/*:streamGenerateContent` | Google Gemini `v1` / `v1beta` streaming API |

Compatibility depends on whether the target service fully implements streaming, tool calling, tool-result continuation, and usage fields. Run a connection test after adding a provider. For gateways that only support the Responses-Lite protocol, enable the provider- or model-level Responses-Lite compatibility switch to add the required routing header.

## How It Works

```text
Windsurf / Devin
  │
  ├─ Local interception and proxy layer
  │   ├─ Identify official requests and BYOK models
  │   └─ Keep official traffic on an independent pass-through path
  │
  └─ Windsurf++ Extension
      ├─ Provider and model management
      ├─ OpenAI / Responses / Anthropic / Gemini adapters
      ├─ Streaming and tool-call conversion
      └─ Local analytics, authorization, and runtime status
```

The installer creates backups before modifying application files. `cwindsurf uninstall` can back up user data first, then removes the patches, extension, caches, and the entire `~/.cwindsurf/` directory — back up provider keys or statistics you want to keep before uninstalling.

## Local Data and Privacy

Configuration is stored under `~/.cwindsurf/`. Provider API keys stay on the local device; refresh tokens and device private keys are kept in the editor's SecretStorage instead of plain config files. Runtime statistics are aggregated locally by default, and full prompts or responses are not required for normal analytics. Development traffic-capture tooling is not included in public builds; public artifacts are bundled, minified, obfuscated, and signed with Ed25519 release manifests.

Never publish API keys, OAuth tokens, refresh tokens, device private keys, release signing keys, or configuration files containing secrets.

## Platform Support

| Platform | Status |
|---|---|
| macOS Apple Silicon / Intel | Supported |
| Windows | Supported |
| Linux | Supported |

Application updates may overwrite patches. Run `cwindsurf status` after updating Windsurf/Devin and reinstall when necessary.

## Releases

Version history and per-release notes are published with the npm package: see the [CHANGELOG](https://www.npmjs.com/package/@priders/cwindsurf?activeTab=code) in `@priders/cwindsurf`.

## Issues and Feedback

This repository is intended for project information, release notes, and issue tracking. The main project source code is not included.

When opening an issue, include your OS and architecture, application and Windsurf++ versions, provider protocol, model ID, reproduction steps, and sanitized error details.

## License

The project and distributed packages are currently marked `UNLICENSED`. Public availability does not grant permission to copy, modify, redistribute, or use the project commercially unless the maintainers provide an explicit license.
