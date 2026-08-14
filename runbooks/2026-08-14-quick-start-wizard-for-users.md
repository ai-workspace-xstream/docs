# 普通用户向导式加速节点与客户端配置手册

- **创建日期**：2026-08-14
- **适用对象**：个人开发者、AI 工具重度用户（Cursor / Claude / ChatGPT / Copilot）
- **状态**：已验证 (Verified)

---

## 🎯 方案概述

针对需要快速打通海外 AI 服务网络加速的用户，提供从**服务器一键部署**到**客户端一键导入连接**的完整向导式闭环流程。

---

## 🧭 操作步骤

### 1. 前置准备
- 拥有 1 台 Linux VPS（推荐 Ubuntu 22.04 / 24.04 或 Debian 12，已开通公网 IP）。
- 拥有 1 个域名（例如 `xhttp.example.com`），并将 A 记录解析到该 VPS 公网 IP。

### 2. 执行一键部署命令
SSH 登录 VPS 执行：
```bash
curl -fsSL https://raw.githubusercontent.com/cloud-neutral-toolkit/agent.svc.plus/main/scripts/setup-proxy.sh | \
  bash -s -- --node xhttp.example.com
```

> **可选参数**：
> - 独立自建（不接入云端控制台）：追加 `--standalone`
> - 二进制升级（保留配置）：追加 `--upgrade-only`

### 3. 获取连接凭证
部署完成后，终端将输出：
- `VLESS XHTTP` 链接（推荐，流式传输体验最佳）
- `VLESS TCP Vision` 链接
- 节点专属 UUID

### 4. 客户端连接与体验

- **方式一（推荐尝鲜自研客户端）**：
  - 前往 **[XConnect App Release main-149](https://github.com/ai-workspace-xstream/xconnect-app/releases/tag/main-149)** 下载对应系统版本（macOS / Windows / iOS / Linux）。
  - 打开应用，点击添加节点或粘贴配置链接即可开启加速。
- **方式二（通用第三方客户端）**：
  - 复制链接至 **OneXray** / **v2rayN** / **Sing-box** / **Surge** 导入并启用系统代理。

---

## ⚡ 免运维云端替代方案

如果不希望自行维护 VPS 或域名，可直接使用官方全托管服务：
- 访问：**[XConnect 控制台 (https://console.svc.plus/products/xconnect)](https://console.svc.plus/products/xconnect)**
- 支持 GitHub / Google OAuth 一键免密登录使用。
