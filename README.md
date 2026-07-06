<div align="center">

# CC Switch CodexCont Build

### 基于 CC Switch v3.16.5 的 Codex reasoning 自动续写增强版

[![Base](https://img.shields.io/badge/base-CC%20Switch%203.16.5-blue)](https://github.com/farion1231/cc-switch)
[![CodexCont](https://img.shields.io/badge/feature-CodexCont-purple)](https://github.com/neteroster/CodexCont)
[![Platform](https://img.shields.io/badge/platform-Windows-lightgrey)](https://github.com/2836048681/cc-switch-codexcont/releases/tag/v3.16.5-codexcont)
[![Built with Tauri](https://img.shields.io/badge/built%20with-Tauri%202-orange)](https://tauri.app/)

[下载 Release](https://github.com/2836048681/cc-switch-codexcont/releases/tag/v3.16.5-codexcont) · [上游 CC Switch](https://github.com/farion1231/cc-switch) · [CodexCont](https://github.com/neteroster/CodexCont)

</div>

## 这个仓库是什么

这是一个基于 [CC Switch](https://github.com/farion1231/cc-switch) `v3.16.5` 的个人增强构建，保留 CC Switch 原有的多上游管理、代理路由、故障转移、Codex / Claude / Gemini / OpenCode / OpenClaw / Hermes 配置管理能力，并额外接入了 CodexCont 风格的 **Codex reasoning 自动续写** 功能。

核心目标是让链路保持为：

```text
Codex -> CC Switch 本地代理 -> 多上游路由 / 故障转移 -> 模型服务
```

而不是把 Codex 直接改到单独的 CodexCont 服务端口。这样可以同时保留 CC Switch 的路由、故障转移和可视化配置能力。

## 相对上游 CC Switch 的新增改动

### 1. 新增 CodexCont UI 开关

在设置页加入了独立开关：

```text
设置 -> 代理 -> Codex reasoning 自动续写（CodexCont）
```

可以在 UI 中控制是否启用 Codex reasoning 自动续写，不需要手动改配置文件。

### 2. 支持持久化 CodexCont 配置

新增数据库配置项：

```text
codex_continue_config
```

配置会由 CC Switch 自身保存和读取，重启后继续生效。

### 3. `/v1/responses` 自动续写处理

当 Codex 请求走 CC Switch 的 `/v1/responses` 时，后端会读取 CodexCont 配置，并在合适条件下对 reasoning continuation 做自动处理。

续写请求仍然走 CC Switch 原本的 provider 转发和 retry/failover 流程：

```text
RequestForwarder::forward_with_retry(... providers)
```

因此不会绕开 CC Switch 的路由策略。

### 4. 保留 CC Switch 路由和故障转移体验

本构建没有把 Codex provider 固定到某个单独服务，也没有要求把 Codex 配置改成 CodexCont 端口。推荐配置仍然是让 Codex 指向 CC Switch 本地代理端口，例如：

```toml
base_url = "http://127.0.0.1:15721/v1"
```

这样 CodexCont 逻辑作为 CC Switch 内部增强存在，CC Switch 的多 provider、路由规则和故障转移仍然可用。

### 5. native Responses provider 安全门控

只有当候选 providers 都是原生 Responses 链路、不需要 Responses -> Chat Completions 转换时，才启用 CodexCont reasoning continuation。

这样可以避免把 `reasoning.encrypted_content` 预注入到不兼容的 Chat provider，降低异常请求和格式错配风险。

### 6. 环境变量覆盖

除 UI 配置外，还支持环境变量覆盖运行参数：

```text
CCSWITCH_CODEX_CONTINUE
CCSWITCH_CODEX_CONTINUE_MAX
CCSWITCH_CODEX_CONTINUE_STEP
CCSWITCH_CODEX_CONTINUE_MARKER
```

适合临时调试、灰度测试或在不改数据库配置的情况下验证行为。

### 7. 改进端口占用提示

对 Windows 常见的 `os error 10048` / `AddrInUse` 场景增加了更明确的错误提示，方便判断是否是端口被占用、重复启动或旧进程未释放。

## 下载

当前打包版本：`v3.16.5-codexcont`

| 文件 | 说明 |
| --- | --- |
| `CC-Switch-v3.16.5-Windows-x64-Setup.exe` | Windows x64 安装包 |
| `CC-Switch-v3.16.5-Windows.msi` | Windows MSI 安装包 |
| `CC-Switch-v3.16.5-Windows-Portable.zip` | 便携版 |
| `SHA256SUMS.txt` | 文件校验值 |

下载地址：

[https://github.com/2836048681/cc-switch-codexcont/releases/tag/v3.16.5-codexcont](https://github.com/2836048681/cc-switch-codexcont/releases/tag/v3.16.5-codexcont)

## 使用方式

1. 安装或解压本仓库 Release 中的 Windows 版本。
2. 在 CC Switch 中配置你的 Codex provider、路由和故障转移规则。
3. 确认 Codex 指向 CC Switch 本地代理，例如：

   ```toml
   base_url = "http://127.0.0.1:15721/v1"
   ```

4. 打开：

   ```text
   设置 -> 代理 -> Codex reasoning 自动续写（CodexCont）
   ```

5. 开启开关并按需调整最大续写次数、步长、结束标记等参数。

## 构建

```bash
pnpm install
pnpm tauri build
```

Windows 构建建议使用 Visual Studio Build Tools 的 x64 Native Tools 环境，避免 Rust 链接阶段缺少 MSVC 运行库。

## 与上游的关系

本仓库不是 CC Switch 官方仓库。

- 上游项目：[farion1231/cc-switch](https://github.com/farion1231/cc-switch)
- CodexCont 参考项目：[neteroster/CodexCont](https://github.com/neteroster/CodexCont)

本仓库只记录在 CC Switch `v3.16.5` 基础上接入 CodexCont 体验的改动和对应打包产物。CC Switch 的完整功能说明、原始文档、贡献指南和许可信息请以上游仓库为准。

## 许可证

本仓库沿用上游项目的 MIT License。详见 [`LICENSE`](LICENSE)。
