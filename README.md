<div align="center">

# CC Switch CodexCont + Grok Build

### 基于 CC Switch v3.17.0 的 Codex reasoning 自动续写与 Grok Build 增强版

[![Base](https://img.shields.io/badge/base-CC%20Switch%203.17.0-blue)](https://github.com/farion1231/cc-switch/tree/v3.17.0)
[![CodexCont](https://img.shields.io/badge/feature-CodexCont-purple)](https://github.com/neteroster/CodexCont)
[![Grok Build](https://img.shields.io/badge/feature-Grok%20Build-black)](https://github.com/1parado/grok-build-switch)
[![Platform](https://img.shields.io/badge/platform-Windows-lightgrey)](https://github.com/2836048681/cc-switch-codexcont/releases/tag/v3.17.0-codexcont-grok)
[![Built with Tauri](https://img.shields.io/badge/built%20with-Tauri%202-orange)](https://tauri.app/)

[下载 Release](https://github.com/2836048681/cc-switch-codexcont/releases/tag/v3.17.0-codexcont-grok) · [上游 CC Switch](https://github.com/farion1231/cc-switch) · [CodexCont](https://github.com/neteroster/CodexCont) · [Grok Build Switch](https://github.com/1parado/grok-build-switch)

</div>

## 这个仓库是什么

这是一个基于 [CC Switch](https://github.com/farion1231/cc-switch) `v3.17.0` 发布提交
[`3d176b9`](https://github.com/farion1231/cc-switch/commit/3d176b98cc0bfd151a42882e88ab59b62083b92f)
的个人增强构建。

本构建保留 CC Switch v3.17.0 原有的项目快照、供应商管理、代理路由、故障转移、官方 ChatGPT 订阅代理、原生 Anthropic Messages，以及 Codex / Claude / Gemini / OpenCode / OpenClaw / Hermes 配置管理能力，并额外加入两个功能：

- CodexCont 风格的 **Codex reasoning 自动续写**。
- 独立的 **Grok Build 供应商、配置管理和代理路由**。

核心链路保持为：

```text
Codex -> CC Switch 本地代理 -> 多上游路由 / 故障转移 -> 模型服务
Grok  -> CC Switch Grok 路由 -> xAI / Grok 兼容上游
```

Codex 续写逻辑和 Grok 路由都集成在 CC Switch 内部，不需要额外运行独立转发服务。

## 相对上游 CC Switch 的新增改动

### 1. 新增 CodexCont UI 开关

在设置页加入独立开关：

```text
设置 -> 代理 -> Codex reasoning 自动续写（CodexCont）
```

可以在 UI 中控制是否启用 Codex reasoning 自动续写，不需要手动修改配置文件。

### 2. 支持持久化 CodexCont 配置

新增数据库配置项：

```text
codex_continue_config
```

配置由 CC Switch 自身保存和读取，重启后继续生效。

### 3. `/v1/responses` 自动续写处理

当 Codex 请求经过 CC Switch 的 `/v1/responses` 时，后端会读取 CodexCont 配置，并在满足条件时自动执行 reasoning continuation。

续写请求仍然使用 CC Switch 原有的 provider 转发和 retry/failover 流程：

```text
RequestForwarder::forward_with_retry(... providers)
```

因此不会绕开 CC Switch 的路由、故障转移和用量记录。

### 4. 保留 CC Switch 路由和故障转移体验

本构建没有把 Codex provider 固定到单独服务，也不要求把 Codex 指向额外端口。推荐配置仍然是 CC Switch 本地代理，例如：

```toml
base_url = "http://127.0.0.1:15721/v1"
```

### 5. native Responses provider 安全门控

只有候选 providers 都使用原生 Responses 链路、不需要 Responses -> Chat Completions 转换时，才启用 Codex reasoning continuation。

这样可以避免把 `reasoning.encrypted_content` 注入不兼容的 Chat provider。

### 6. CodexCont 环境变量覆盖

除 UI 配置外，还支持环境变量覆盖运行参数：

```text
CCSWITCH_CODEX_CONTINUE
CCSWITCH_CODEX_CONTINUE_MAX
CCSWITCH_CODEX_CONTINUE_STEP
CCSWITCH_CODEX_CONTINUE_MARKER
```

### 7. 独立 Grok Build 应用

Grok 作为独立顶层应用加入 CC Switch，不复用 Codex 的通用配置。供应商、当前选中项、目录设置和代理状态均按 Grok 独立管理。

默认扫描并管理：

```text
~/.grok/config.toml
```

也可以在设置中覆盖 Grok 配置目录。

### 8. 原生 Responses 与 Chat Completions 双接口

Grok 路由同时提供：

```text
/grok/v1/responses
/grok/v1/chat/completions
```

xAI 原生 Responses 请求直接以 Responses 格式转发，不转换为 Chat Completions。需要兼容 OpenAI Chat 格式的客户端可以选择 `/grok/v1/chat/completions`。

### 9. Grok 多模型与高级配置

Grok 供应商支持：

- 多模型 Profile 和默认模型选择。
- `web_search`。
- Subagents。
- `extra_headers`。
- 隐私保护配置。
- 官方 OAuth 回退。
- 配置写入前自动备份，保留最近 10 份。

### 10. 数据库 v14 迁移

本构建数据库版本为 `14`。从 CC Switch v3.17.0 的 schema v13 启动时会原地升级到 v14，并保留已有供应商、项目、统计和 Grok 代理配置。

不会降级或重建用户数据库。

## 下载

当前构建版本：`v3.17.0-codexcont-grok`

| 文件 | 说明 |
| --- | --- |
| `CC-Switch-v3.17.0-Windows-x64-Setup.exe` | Windows x64 安装包 |
| `CC-Switch-v3.17.0-Windows.msi` | Windows MSI 安装包 |
| `CC-Switch-v3.17.0-Windows-Portable.zip` | Windows 便携版 |
| `SHA256SUMS.txt` | SHA-256 校验值 |

下载地址：

[https://github.com/2836048681/cc-switch-codexcont/releases/tag/v3.17.0-codexcont-grok](https://github.com/2836048681/cc-switch-codexcont/releases/tag/v3.17.0-codexcont-grok)

## Codex 使用方式

1. 安装或解压本仓库提供的 Windows 构建。
2. 在 CC Switch 中配置 Codex provider、路由和故障转移规则。
3. 确认 Codex 指向 CC Switch 本地代理：

   ```toml
   base_url = "http://127.0.0.1:15721/v1"
   ```

4. 打开：

   ```text
   设置 -> 代理 -> Codex reasoning 自动续写（CodexCont）
   ```

5. 开启开关并按需调整最大续写次数、步长和结束标记。

## Grok 使用方式

1. 在应用切换器中选择 Grok。
2. 添加 xAI 或 Grok 兼容供应商，填写 API 地址、密钥和模型配置。
3. 切换到目标 Grok 供应商，CC Switch 会管理 `~/.grok/config.toml`。
4. 在代理页面启用 Grok 本地路由。
5. 原生 Responses 客户端使用 `/grok/v1/responses`；Chat 格式客户端使用 `/grok/v1/chat/completions`。

## 构建

```bash
pnpm install
pnpm tauri build
```

Windows 构建建议使用 Visual Studio Build Tools 的 x64 Native Tools 环境，避免 Rust 链接阶段缺少 MSVC 和 Windows SDK 运行库。

## 与上游的关系

本仓库不是 CC Switch 官方仓库。

- 上游项目：[farion1231/cc-switch](https://github.com/farion1231/cc-switch)
- CodexCont 参考项目：[neteroster/CodexCont](https://github.com/neteroster/CodexCont)
- Grok Build Switch 参考项目：[1parado/grok-build-switch](https://github.com/1parado/grok-build-switch)

本仓库只记录在 CC Switch `v3.17.0` 基础上集成 Codex reasoning continuation 和 Grok Build 管理能力的改动及对应打包产物。CC Switch 的完整功能说明、原始文档、贡献指南和许可信息请以上游仓库为准。

## 许可证

本仓库沿用上游项目的 MIT License。详见 [`LICENSE`](LICENSE)。

## 致谢

本项目是在以下上游项目和社区讨论基础上完成的衍生集成版本：

- 感谢 [farion1231/cc-switch](https://github.com/farion1231/cc-switch) 提供 CC Switch 的主体应用、代理、路由和故障转移能力。
- 感谢 [neteroster/CodexCont](https://github.com/neteroster/CodexCont) 提供 Codex reasoning continuation 的核心思路与参考实现。
- 感谢 [1parado/grok-build-switch](https://github.com/1parado/grok-build-switch) 提供 Grok Build 配置管理和切换流程参考。
- 感谢 [LINUX DO 社区](https://linux.do/) 对相关工具链、使用场景和实践经验的讨论与分享。
