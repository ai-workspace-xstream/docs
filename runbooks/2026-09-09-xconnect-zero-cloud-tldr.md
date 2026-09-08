# XConnect Zero Cloud：Gateway / One 自建接入 TL;DR

- 状态：草案，随 XConnect One 与 XConnect Gateway Release 更新
- 日期：2026-09-09
- 范围：UAT 与自建 Linux Gateway、Linux/macOS/Windows One 节点
- 验收：精确 peer handshake、授权私网 ping、私网 HTTP 三项同时通过

## 目标与安全边界

XConnect Zero/Accounts 是网络、设备、邀请、策略与签名配置的唯一权威。
Gateway 与 One 只消费签名配置并运行数据面。Portal 的 ACK 仅代表某一代配置已
确认，不能单独证明数据面已连通。

不将短期邀请、设备凭据、WireGuard 私钥、TLS 私钥、Vault 值或真实地址写入
Git、终端历史、CI 参数或本手册。

## 安装入口

安装器只下载、校验并安装 CLI 二进制；不会加入网络，不会启动 Xray/WireGuard，
也不会创建设备私钥或读取 Vault。

```sh
# Gateway（Linux）
curl -fsSL https://install.svc.plus/xconnect-gateway | \
  XCONNECT_GATEWAY_VERSION=v0.1.5 bash

# One（Linux/macOS）
curl -fsSL https://install.svc.plus/xconnect-one | \
  XCONNECT_ONE_VERSION=v0.1.9 bash
```

Windows 使用管理员 PowerShell：

```powershell
$env:XCONNECT_ONE_VERSION = 'v0.1.9'
irm https://install.svc.plus/xconnect-one.ps1 | iex
```

发布前，`install.svc.plus` 必须仅提供已经审核的安装脚本，并从批准的 Release
或内部镜像拉取同版本资产与 `SHA256SUMS`。私有镜像使用相应的
`XCONNECT_*_RELEASE_BASE_URL` 覆盖变量。

## macOS One

Homebrew 公式合并发布后，可用：

```sh
brew install --formula \
  https://raw.githubusercontent.com/ai-workspace-xstream/XConnect-One/main/Formula/xconnect-one.rb

brew install xray wireguard-tools wireguard-go
```

然后确认运行时：

```sh
XCONNECT_BIN="$(brew --prefix)/bin/xconnect"
sudo "$XCONNECT_BIN" diagnose --state-dir /var/lib/xconnect-one
```

首次加入在受保护终端内使用短期邀请执行。建议使用 XConnect One 的 macOS 加入
脚本，避免把邀请写进 history：

```sh
XCONNECT_BIN="$XCONNECT_BIN" \
XCONNECT_STATE_DIR=/var/lib/xconnect-one \
bash /path/to/xconnect-one-macos-join.sh /path/to/handoff.json
```

`join` 会生成本机受保护状态和 WireGuard 密钥，完成 Zero enrollment，获取并验证
签名配置，生成外部 Xray transport/WireGuard 配置，启动运行时并发送 ACK。
后续更新用 `sync`；停止用 `down`。不要使用离线缓存 `up` 作为恢复手段。

```sh
sudo "$XCONNECT_BIN" sync --state-dir /var/lib/xconnect-one
sudo "$XCONNECT_BIN" status --state-dir /var/lib/xconnect-one
sudo "$XCONNECT_BIN" down --state-dir /var/lib/xconnect-one
```

## Linux Gateway

先从 Vault 以受保护方式注入：

```text
/etc/xconnect-gateway/tls.crt
/etc/xconnect-gateway/tls.key
```

预先安装受信任来源的 `xray`、`wireguard-tools` 和 systemd 运行环境。然后：

```sh
sudo /usr/local/bin/xconnect-gateway diagnose
sudo /usr/local/bin/xconnect-gateway init \
  --state-dir /var/lib/xconnect-gateway \
  --controller https://accounts-uat.example \
  --gateway-id gw-uat-1

# 在 Zero Portal 确认 Gateway 公钥后，使用其短期邀请。
sudo /usr/local/bin/xconnect-gateway join \
  --state-dir /var/lib/xconnect-gateway \
  --gateway-id gw-uat-1 \
  'xconnect://join/SHORT_LIVED_INVITE'

sudo /usr/local/bin/xconnect-gateway up \
  --state-dir /var/lib/xconnect-gateway \
  --tls-cert /etc/xconnect-gateway/tls.crt \
  --tls-key /etc/xconnect-gateway/tls.key
```

`up` 会验证 Gateway 签名配置、生成受保护的运行时文件、启动外部 Xray 与
WireGuard，并发送 ACK。

## WireGuard-over-VLESS 与验证

```text
One WireGuard
  → One 本机 Xray transport
  → VLESS/TLS/XUDP
  → Gateway Xray
  → Gateway 本机 UDP 127.0.0.1:51820
  → Gateway WireGuard
  → 授权私网资源
```

Gateway 对外提供 VLESS/TLS（通常 TCP 443），不暴露公网 WireGuard UDP。
One 的 WireGuard Endpoint 指向本机 Xray transport adapter；NAT 后客户端不需要
入站端口。

```sh
# One
sudo "$XCONNECT_BIN" status --state-dir /var/lib/xconnect-one
sudo wg show xconone0 latest-handshakes
ping -c 3 10.77.0.1
curl --fail --max-time 10 http://10.77.0.1:8080/uat/run

# Gateway
sudo /usr/local/bin/xconnect-gateway status --state-dir /var/lib/xconnect-gateway
sudo wg show xconnect0 latest-handshakes
sudo systemctl is-active xconnect-gateway-xray.service
sudo ss -lntp | grep ':443'
```

通过必须同时满足：Gateway/One 运行时正常、精确对端 peer 的 handshake 足够新、
私网 ping 成功、私网 HTTP 返回本次验证标记。

## 代码与详版文档

- [XConnect One](https://github.com/ai-workspace-xstream/XConnect-One)：跨平台 CLI、安装器与运行时细节。
- [XConnect Gateway](https://github.com/ai-workspace-xstream/XConnect-Gateway)：Linux relay、Xray/WireGuard 运行时细节。
- [XConnect One 自建安装与验证](https://github.com/ai-workspace-xstream/XConnect-One/blob/main/docs/self-hosted-install-and-validation.md)。
- [XConnect Gateway 自建安装与验证](https://github.com/ai-workspace-xstream/XConnect-Gateway/blob/main/docs/self-hosted-install-and-validation.md)。
