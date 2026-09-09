# XConnect One：一条命令接入 Gateway 的产品化计划

- 状态：已批准的产品方向，待按阶段实现
- 日期：2026-09-09
- 范围：XConnect One、XConnect Gateway、Zero/Accounts 集成与 UAT 验证
- 不在范围：将 XConnect APP、Zero Portal 或 Gateway 合并成单一进程

## 核心产品边界

用户面对的对象是“接入一个获授权的 XConnect Gateway”，而不是 Xray、WireGuard、
VLESS、peer、端口、私钥、路由或平台服务管理。

```text
install One
  → join Gateway
  → status / diagnose
  → sync
  → leave
```

One 在 macOS、Linux、Windows 上保持相同生命周期含义：

- `join`：验证短期邀请或获批自注册，完成 Zero enrollment，生成本机密钥；
- `sync`：取得、验证和应用新的签名配置，续期会话并 ACK；
- `status` / `diagnose`：展示 One、transport、WireGuard、精确 peer handshake 与授权连通性；
- `down`：停止 One 所有的本地运行时，保留 enrollment；
- `leave`：请求远端撤销后清理 One 所有本地状态。

One 只管理自己拥有的本机运行时，不接管 XConnect APP 的 TUN、账号、进程或状态。
Gateway 是独立 Linux relay/service；Zero/Accounts 是网络、设备、邀请、策略和签名
配置的唯一来源。

## 运行时策略

当前 `v0.1.9` 需要节点管理员预装外部 Xray/WireGuard。这是过渡状态，不能作为
最终体验。

目标是“受管外部运行时”：

1. One 通过版本化清单取得受批准的 Xray/WireGuard 运行时来源、版本、sha256 与平台约束。
2. 交互式 `join` 在运行时缺失时请求授权并完成 bootstrap；无终端场景要求显式
   `--bootstrap`，避免 CI 静默下载或提升权限。
3. 运行时与生成配置存入 One 自己的受保护状态目录；不覆写用户其他 VPN、Xray 或
   WireGuard 配置。
4. macOS 通过受控的 root helper/`wireguard-go` 适配器，Linux 使用受控 Xray 和
   系统 WireGuard 内核路径，Windows 使用受控签名 Xray 与 WireGuard for Windows
   服务适配器。
5. 所有下载必须校验签名/sha256、支持显式版本固定、支持镜像来源覆盖，并记录
   可审计的运行时版本。

Homebrew、`curl install.svc.plus/...` 与 Windows PowerShell 仅是 CLI 分发方式；
它们不替代 One 的 runtime bootstrap 语义。

## 仓库分工

| 仓库 | 负责内容 | 不负责内容 |
| --- | --- | --- |
| `XConnect-One` | 跨平台 CLI、受管运行时、平台适配器、安装器、端到端客户端测试 | Gateway 服务、Zero 策略签发、XConnect APP 核心 |
| `XConnect-Gateway` | Linux relay、Gateway Xray/WireGuard、peer 应用、结构化就绪/诊断 | One 桌面运行时、Portal UI |
| `accounts` / `portal` | 网络/设备/邀请/策略/签名配置/ACK 与用户隔离 UI | 运行 Xray/WireGuard |
| `iac_modules` | Spot、实例、安全组等可复用资源模块 | OS 角色、设备凭据、客户端逻辑 |
| `gitops/vpn-overlay` | 非敏感版本、拓扑和环境声明 | 私钥、邀请、节点运行状态 |
| `playbooks` | Gateway/Linux/Windows/macOS 的 OS 级安装角色 | Zero 策略、CLI 产品状态 |
| `platform-ops-toolkit` | 受控 UAT 部署、验证与租约回收 | 构建 One/Gateway 发布制品 |

## 交付阶段

### P0：契约与可观测性

- 为 One 定义跨平台 runtime manifest、所有权目录、状态 schema 与 machine-readable
  `status` / `diagnose` 契约。
- Gateway 提供稳定的 peer、Xray、WireGuard 与配置 generation 就绪状态。
- 明确 UAT 断言：配置代数、精确 Gateway public key、精确 One public key、握手时间、
  ping 与 HTTP marker。

### P1：Linux 受管接入

- 实现 `xconnect runtime bootstrap`，并让交互式 `join` 可调用它。
- 将 Xray transport 配置、WireGuard 配置、进程/服务生命周期完全收口到 One。
- 用现有一小时 Spot Gateway + Linux One 实现无人值守 handshake/ping/HTTP。

### P2：macOS 受管接入

- 实现 macOS runtime resolver、权限提示、`wireguard-go` 与 Xray adapter。
- 验证不触碰 XConnect APP；只使用独立 One state directory。
- 在本机以一次性 UAT invitation 完成 join、sync、down、sync 恢复与撤销测试。

### P3：Windows 受管接入

- 实现签名运行时下载、WireGuard for Windows 服务/adapter、Xray adapter 与管理员权限流。
- 在局域网 Windows 首先验证，再用 Windows Spot 进行可复现云端验证。

### P4：正式 Zero 接管

- transport-only 基线通过后，切换为 Accounts 网络/邀请/enrollment/signed config/ACK。
- Portal 保持现有布局，显示用户隔离的 Gateway/One 配置状态；数据面在线状态只能来自
  可信 runtime report，不能由 ACK 推断。

## 验收标准

- 用户无需编辑 Xray JSON、WireGuard 配置、peer 或路由；
- One 可在 macOS、Linux、Windows 以相同命令完成 join、sync、status、down、leave；
- Gateway/One 精确 peer handshake、私网 ping 与 HTTP marker 通过；
- 运行时下载、私钥与状态目录可验证且最小权限；
- 设备撤销后 Gateway peer 收敛，One 无法再次 sync 获取有效配置；
- XConnect APP 未被修改或接管。

## 交付追踪

- [One 产品化 Epic #22](https://github.com/ai-workspace-xstream/XConnect-One/issues/22)
- [Linux 受管运行时 #23](https://github.com/ai-workspace-xstream/XConnect-One/issues/23)
- [macOS One runtime adapter #24](https://github.com/ai-workspace-xstream/XConnect-One/issues/24)
- [Windows One runtime adapter #25](https://github.com/ai-workspace-xstream/XConnect-One/issues/25)
- [Gateway readiness/peer evidence #9](https://github.com/ai-workspace-xstream/XConnect-Gateway/issues/9)
- [UAT 分阶段验证 #632](https://github.com/ai-workspace-infra/platform-ops-toolkit/issues/632)

## 风险与约束

- WireGuard 驱动、root helper 和 Windows 服务安装需要各平台管理员权限，不能伪装成
  无权限操作。
- 运行时下载属于供应链边界，必须固定版本、校验完整性并支持受控镜像。
- 不能把 Zero 长期凭据、Vault 值或私钥作为 bootstrap 输入。
- `WireGuard over VLESS` 仍要求 Gateway Xray 将专用 VLESS 流量转发到同机
  `127.0.0.1:51820`；公网不开放 WireGuard UDP。
