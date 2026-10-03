# XConnect 自适应 mesh 与多地域 VLESS relay 实施任务

日期：2026-10-03。状态：待实施；任务登记不代表开发或 UAT 已完成。

架构全文：[设计文档](../../architecture/xconnect-adaptive-mesh-relay-design.zh-CN.md)。
GitHub Project：[XConnect 项目 #1](https://github.com/orgs/ai-workspace-xstream/projects/1)。
总任务：[[EPIC-MESH] N 个 One 与多地域 Gateway 自适应网络（WireGuard over VLESS relay）](https://github.com/ai-workspace-xstream/docs/issues/5)。

## 交付规则

保持三个产品仓库独立；Edge Agent 负责节点注册上报，One／Gateway 保持设备 enrollment 与数据面所有权。One↔One 端到端 WireGuard，加密包 relay 优先 VLESS/XHTTP + TLS。

每个功能交付 PR → 合并 → 不可变 release → UAT → 实际数据面证据。设计任务以文档评审和契约决定验收。未开展实施的任务保持 Todo。Accounts 禁用了 Issues，控制面任务在 docs 跟踪，实际 PR 归属 ai-workspace-services/accounts。

旧 EPIC-M2 的仓库合并方案与本架构不一致，不作为 mesh 实施前置。原任务保留历史，追加替代说明；不自动认定旧任务完成。旧 E-3 Gateway HA 设计关联本次总任务；已有 INF-8／INF-9 基线复用，不重复创建。

## 阶段与依赖

P0 协议／兼容 → P1 单 relay、ACL 和状态 → P2 LAN 自适应 → P3 多地域单 relay → P4 公网直连 → P5 Gateway 联邦与规模验收。

P3、P4 可根据业务调整实施先后。需要覆盖没有共同可达 Gateway 的首批交付，必须包含 P5；否则明确声明该场景未覆盖。

| 任务 | 阶段 | 实施仓库 | 前置任务 | GitHub Issue |
|---|---|---|---|---|
| [MESH-01][P0] 验证 WireGuard over VLESS 双向 relay 承载并定稿会话协议 | P0 | `ai-workspace-xstream/XConnect-Gateway` | — | [MESH-01](https://github.com/ai-workspace-xstream/xconnect-gateway/issues/20) |
| [MESH-22][P0] 定稿 mesh 能力协商、新旧签名契约与灰度回滚方案 | P0 | `ai-workspace-xstream/docs` | MESH-01 | [MESH-22](https://github.com/ai-workspace-xstream/docs/issues/6) |
| [MESH-02][P1] Zero：下发端到端 One peer 与 relay 授权签名配置 | P1 | `ai-workspace-services/accounts` | MESH-22 | [MESH-02](https://github.com/ai-workspace-xstream/docs/issues/7) |
| [MESH-03][P1] 实现多 peer 配置与各平台 WireGuard 增量应用 | P1 | `ai-workspace-xstream/XConnect-One` | MESH-02 | [MESH-03](https://github.com/ai-workspace-xstream/xconnect-one/issues/43) |
| [MESH-04][P1] 实现稳定入口与 VLESS relay 路径管理器 | P1 | `ai-workspace-xstream/XConnect-One` | MESH-01, MESH-03 | [MESH-04](https://github.com/ai-workspace-xstream/xconnect-one/issues/44) |
| [MESH-05][P1] 实现认证会话与 One↔One 透明加密包 relay | P1 | `ai-workspace-xstream/XConnect-Gateway` | MESH-01, MESH-02 | [MESH-05](https://github.com/ai-workspace-xstream/xconnect-gateway/issues/21) |
| [MESH-06][P1] 实现 relay 准入、队列限流、撤销与真实健康 | P1 | `ai-workspace-xstream/XConnect-Gateway` | MESH-05 | [MESH-06](https://github.com/ai-workspace-xstream/xconnect-gateway/issues/22) |
| [MESH-07][P1] 实现解密后 ACL 执行与 peer 撤销 | P1 | `ai-workspace-xstream/XConnect-One` | MESH-02, MESH-03 | [MESH-07](https://github.com/ai-workspace-xstream/xconnect-one/issues/45) |
| [MESH-08][P1] 统一 Gateway／One 注册绑定、能力与真实状态上报 | P1 | `ai-workspace-xstream/xconnect-edge-agent` | MESH-02, MESH-04, MESH-06 | [MESH-08](https://github.com/ai-workspace-xstream/xconnect-edge-agent/issues/88) |
| [MESH-09][P1] 验收 N 个 One 的单 Gateway 端到端 relay 基线 | P1 | `ai-workspace-xstream/docs` | MESH-04, MESH-06, MESH-07, MESH-08 | [MESH-09](https://github.com/ai-workspace-xstream/docs/issues/8) |
| [MESH-10][P2] 实现授权 LAN 候选交换、mDNS 补充与认证 UDP 直连 | P2 | `ai-workspace-xstream/XConnect-One` | MESH-09 | [MESH-10](https://github.com/ai-workspace-xstream/xconnect-one/issues/46) |
| [MESH-11][P2] 实现 LAN／relay 自动切换、防抖与网络变化恢复 | P2 | `ai-workspace-xstream/XConnect-One` | MESH-10 | [MESH-11](https://github.com/ai-workspace-xstream/xconnect-one/issues/47) |
| [MESH-12][P2] 验收 LAN 自动发现、N 节点直连与回退 | P2 | `ai-workspace-xstream/docs` | MESH-11 | [MESH-12](https://github.com/ai-workspace-xstream/docs/issues/9) |
| [MESH-13][P3] Zero：实现多地域 relay 目录与设备 attachment 更新 | P3 | `ai-workspace-services/accounts` | MESH-02, MESH-08 | [MESH-13](https://github.com/ai-workspace-xstream/docs/issues/10) |
| [MESH-14][P3] 实现按 peer 的多 Gateway 协商与完整路径选路 | P3 | `ai-workspace-xstream/XConnect-One` | MESH-09, MESH-13 | [MESH-14](https://github.com/ai-workspace-xstream/xconnect-one/issues/48) |
| [MESH-15][P3] 验收多地域单 relay 选路、故障切换和质量收益 | P3 | `ai-workspace-xstream/docs` | MESH-14 | [MESH-15](https://github.com/ai-workspace-xstream/docs/issues/11) |
| [MESH-16][P4] 实现 IPv6／STUN／ICE 公网直连与 NAT 打洞 | P4 | `ai-workspace-xstream/XConnect-One` | MESH-11, MESH-13 | [MESH-16](https://github.com/ai-workspace-xstream/xconnect-one/issues/49) |
| [MESH-17][P4] 验收异地公网直连与受限网络 relay 兜底 | P4 | `ai-workspace-xstream/docs` | MESH-16 | [MESH-17](https://github.com/ai-workspace-xstream/docs/issues/12) |
| [MESH-18][P5] 实现 VLESS Gateway 联邦与最多两跳中继 | P5 | `ai-workspace-xstream/XConnect-Gateway` | MESH-06, MESH-13 | [MESH-18](https://github.com/ai-workspace-xstream/xconnect-gateway/issues/23) |
| [MESH-19][P5] 实现单／双 relay 候选探测和跨地域自适应 | P5 | `ai-workspace-xstream/XConnect-One` | MESH-14, MESH-18 | [MESH-19](https://github.com/ai-workspace-xstream/xconnect-one/issues/50) |
| [MESH-20][P5] 验收无共同 Gateway 的跨地域联邦 relay | P5 | `ai-workspace-xstream/docs` | MESH-19 | [MESH-20](https://github.com/ai-workspace-xstream/docs/issues/13) |
| [MESH-21][P3] 汇总路径可观察性与 Zero／App 诊断契约 | P3 | `ai-workspace-xstream/xconnect-edge-agent` | MESH-08, MESH-14 | [MESH-21](https://github.com/ai-workspace-xstream/xconnect-edge-agent/issues/89) |
| [MESH-23][P5] 完成 N×M 规模、故障注入与分阶段 release 验收 | P5 | `ai-workspace-xstream/docs` | MESH-12, MESH-15, MESH-17, MESH-20, MESH-21 | [MESH-23](https://github.com/ai-workspace-xstream/docs/issues/14) |

## 任务范围与验收

### [MESH-01][P0] 验证 WireGuard over VLESS 双向 relay 承载并定稿会话协议

实施仓库：`ai-workspace-xstream/XConnect-Gateway`。

范围：

- [ ] 复核现有 Xray/XHTTP/TLS/XUDP profile，明确它与 Gateway 本机 WireGuard 转发的差别。
- [ ] 用两个 One 模拟端到端密文 relay，比较 XUDP 与帧化双向流适配，选择并版本化一套首版协议。
- [ ] 定义 REGISTER/READY/DATA/PROBE/PING/CLOSE、帧边界、最大 payload、身份绑定及背压。

验收：

- [ ] 双方主动发包、空闲后主动回包、传输断线重连均有原型证据。
- [ ] 证明目标设备会话可识别且不同 One 不串流；不能仅使用共享 VLESS UUID 判断身份。
- [ ] 记录实际 HTTP 版本、TCP 承载、MTU、队列压力；给出协议决定与版本化 schema。

### [MESH-22][P0] 定稿 mesh 能力协商、新旧签名契约与灰度回滚方案

实施仓库：`ai-workspace-xstream/docs`。

范围：

- [ ] 分离现有 signed-config v1/v2 和新 mesh schema，不向旧严格 decoder 注入未知字段。
- [ ] 定义新 profile/media type、客户端能力协商、服务端签名序列化、动态候选更新和 generation 规则。
- [ ] 明确旧 Gateway WireGuard 模式与新透明 relay 的独立入口及回滚策略。

验收：

- [ ] 新旧客户端与 Gateway 的兼容矩阵、签名黄金向量计划和部署顺序完整。
- [ ] 整体回滚生成新的有效高 generation 配置；不重放旧签名，不承诺旧模型切换无感。
- [ ] 明确每个阶段关闭证据为 PR、不可变 release、UAT 与实际路径验收。

### [MESH-02][P1] Zero：下发端到端 One peer 与 relay 授权签名配置

实施仓库：`ai-workspace-services/accounts`。

范围：

- [ ] 扩展 accounts overlay 模型与配置生成，支持 One peer 的 device、公钥、overlay /32 或 /128、策略引用。
- [ ] 发布独立 relay descriptor、传输 profile、发现认证绑定、network/device/audience 和有效期。
- [ ] 保留旧单 Gateway 配置生成与敏感 enrollment/session/ACK 身份边界。

验收：

- [ ] 两个 One 得到互相匹配、经过签名验证的端到端 peer 配置。
- [ ] 跨 network、过期、篡改、低 generation、未知 schema 被拒绝；旧客户端仍获兼容配置。
- [ ] 新增控制面 API 契约和测试，无私钥或 VLESS 凭据泄漏到 Portal／日志。

### [MESH-03][P1] 实现多 peer 配置与各平台 WireGuard 增量应用

实施仓库：`ai-workspace-xstream/XConnect-One`。

范围：

- [ ] 扩展 signedconfig Compile、运行时模型与渲染，解除单 Gateway peer 限制。
- [ ] 为每个 One peer 绑定稳定 loopback endpoint，支持新增、撤销和热更新。
- [ ] 明确 One 精确 AllowedIPs 与旧 Gateway 宽网段路由的归属，防止冷 peer 错误投递。

验收：

- [ ] A↔B、A↔C 同时存在独立 WireGuard peer，公钥／地址映射正确。
- [ ] 新增或更新一个 peer 不重建 overlay 接口、不破坏其他活动连接。
- [ ] Linux/macOS/Windows 运行时有适配与验证；不支持的平台显式报告能力。

### [MESH-04][P1] 实现稳定入口与 VLESS relay 路径管理器

实施仓库：`ai-workspace-xstream/XConnect-One`。

范围：

- [ ] 新增 per-peer 路径状态及加密包转发模块，首版仅启用 VLESS relay。
- [ ] 将同一 peer 的所有返回包从稳定本机入口交还 WireGuard；复用 relay 连接并隔离逻辑会话。
- [ ] 集成运行时启动、退出、健康检查、断线恢复及本地状态接口。

验收：

- [ ] WireGuard endpoint／overlay IP 在 relay 重连和路径状态更新时保持稳定。
- [ ] 双向主动发包和空闲恢复正常；多 peer 回包无混淆，socket／队列受限。
- [ ] 路径状态与实际运行一致，故障原因可诊断且不输出凭据。

### [MESH-05][P1] 实现认证会话与 One↔One 透明加密包 relay

实施仓库：`ai-workspace-xstream/XConnect-Gateway`。

范围：

- [ ] 新增 relay 服务，Xray 的新 profile 投递到此服务，旧 profile 保留本机 WireGuard。
- [ ] 实现设备持有证明、短期授权、session epoch、目标设备目录和双向 DATA 投递。
- [ ] 验证源身份取自认证会话；Gateway 不终止 One↔One 的 WireGuard 隧道。

验收：

- [ ] A↔B 端到端 WireGuard 经 VLESS relay 通信，双方可主动发包。
- [ ] 多 One 会话隔离，伪造源设备和越权目标被拒绝。
- [ ] 旧 Gateway 资源访问与 Agent Proxy 配置／入口不受影响。

### [MESH-06][P1] 实现 relay 准入、队列限流、撤销与真实健康

实施仓库：`ai-workspace-xstream/XConnect-Gateway`。

范围：

- [ ] 增加 network/source/target 投递授权、会话期限、撤销、重连旧 epoch 清理。
- [ ] 限制队列、最大帧、速率与慢消费者，并区分准入过载和已有会话健康。
- [ ] 提供受保护本机状态接口及转发／拒绝／丢弃统计。

验收：

- [ ] 越权、过期、重放会话不可投递，授权撤销有可测生效上界。
- [ ] 慢消费者与高流量压力不会产生无界缓存或拖垮全部 peer。
- [ ] 健康接口区分进程、relay READY、投递能力；指标无凭据和高基数全 peer 组合。

### [MESH-07][P1] 实现解密后 ACL 执行与 peer 撤销

实施仓库：`ai-workspace-xstream/XConnect-One`。

范围：

- [ ] 复用现有 policy consumer 的签名引用、digest、generation 和过期验证。
- [ ] 在 WireGuard 解密后入口执行 network/source/destination/protocol/port 策略并绑定可信 peer。
- [ ] 落实各平台执行点、默认拒绝、策略原子更新、撤销和控制面离线租约。

验收：

- [ ] 保存策略文件不能替代执行；允许和拒绝流量均有实际数据面证据。
- [ ] 篡改、低 generation、过期策略被拒绝，在线撤销与离线有效期行为明确。
- [ ] 直连开放前 ACL 基线通过；后续 direct、单 relay、双 relay 权限保持一致。

### [MESH-08][P1] 统一 Gateway／One 注册绑定、能力与真实状态上报

实施仓库：`ai-workspace-xstream/xconnect-edge-agent`。

范围：

- [ ] 保持独立 One/Gateway 数据面仓库，通过受保护本机接口获取运行状态。
- [ ] 校验 agent/node/device/network/role 绑定，上报真实版本、relay 能力、候选摘要和路径事件。
- [ ] 保留 Agent Proxy 同步职责，Gateway/One 角色不覆盖它们的 WireGuard/Xray 文件；为桌面／移动提供嵌入适配。

验收：

- [ ] 注册绑定不可由自报字段越权；Agent 凭据不自动获取任意设备 session。
- [ ] 健康上报不再用同步成功时间代替 One/Gateway 数据面存活。
- [ ] 心跳、配置 ACK、会话 READY、端到端健康可分别观察；敏感密钥留在角色运行时。

### [MESH-09][P1] 验收 N 个 One 的单 Gateway 端到端 relay 基线

实施仓库：`ai-workspace-xstream/docs`。

范围：

- [ ] 建立多 One 的可重复 UAT 拓扑、版本清单和双向业务测试。
- [ ] 覆盖主动发包、空闲、重连、ACL、MTU、并发隔离和旧模式共存。

验收：

- [ ] 证明端到端 WG peer 是 One 公钥，Gateway 只投递内部密文。
- [ ] 记录 RTT p50/p95、吞吐、恢复时间、资源与 Gateway 转发字节。
- [ ] 每个组件提交 PR／release／UAT 证据；配置 ACK 或握手不能单独关闭此任务。

### [MESH-10][P2] 实现授权 LAN 候选交换、mDNS 补充与认证 UDP 直连

实施仓库：`ai-workspace-xstream/XConnect-One`。

范围：

- [ ] 收集受允许物理接口候选，通过 Zero 授权可见范围交换，mDNS 仅作补充。
- [ ] 独立发现凭据绑定设备、network 和 WG 公钥；用 nonce/epoch 验证真实业务 socket 的双向可达。
- [ ] 支持 IPv4 和正确 scope 的 IPv6 LAN 候选，过滤 loopback／overlay／无关网卡。

验收：

- [ ] 同 LAN 两个或多个 One 自动建立直接加密包路径。
- [ ] 同出口、同网段、伪造 mDNS、AP isolation 不会误判身份或直连成功。
- [ ] 组播不可用时控制面候选仍可探测；未授权节点被拒绝且探测限流。

### [MESH-11][P2] 实现 LAN／relay 自动切换、防抖与网络变化恢复

实施仓库：`ai-workspace-xstream/XConnect-One`。

范围：

- [ ] 先建立并验证新路径再切发送出口，允许有效旧／新路径接收。
- [ ] 加入平滑质量、稳定窗口、最短保持、失败回退、退避和 standby relay。
- [ ] Wi-Fi、地址、网卡变化及休眠恢复更新 network epoch，空闲业务不直接判失败。

验收：

- [ ] 阻断 LAN 自动回退 VLESS，恢复后稳定升级；记录连续 TCP/HTTP 的中断分布。
- [ ] 不改 WireGuard peer／IP，不要求双方同时切换，临时非对称路径仍可通信。
- [ ] 网络变化无陈旧候选／socket 泄漏，抖动不导致频繁切换。

### [MESH-12][P2] 验收 LAN 自动发现、N 节点直连与回退

实施仓库：`ai-workspace-xstream/docs`。

范围：

- [ ] 测试同 LAN 多 peer、mDNS 阻断、主机防火墙、AP/VLAN 隔离和链路恢复。
- [ ] 核验实际路径、权限一致、持续业务与 Gateway 流量下降。

验收：

- [ ] 直接路径承载测试业务，Gateway 对该流量的中继字节明显下降。
- [ ] LAN 阻断及恢复行为可复现，包含大包和双向主动业务。
- [ ] 记录故障检测／恢复时间与切换次数，不用小包 ping 替代完整验收。

### [MESH-13][P3] Zero：实现多地域 relay 目录与设备 attachment 更新

实施仓库：`ai-workspace-services/accounts`。

范围：

- [ ] 扩展单 network 单 Gateway 模型，发布受授权有限候选、独立地址、TLS/profile、地域和准入状态。
- [ ] 绑定设备主要／备用 relay session、epoch 和有效期，支持认证动态更新。
- [ ] 分离配置签名、候选有效期、attachment 租约，缓存信息不能扩大权限。

验收：

- [ ] 一个 network 可使用多个授权 Gateway，不同网络目录隔离。
- [ ] 候选／attachment 过期和 session 变化可传播；旧客户端仍获得旧配置。
- [ ] 独立 Gateway 能明确寻址，不依赖无会话目录的随机 LB。

### [MESH-14][P3] 实现按 peer 的多 Gateway 协商与完整路径选路

实施仓库：`ai-workspace-xstream/XConnect-One`。

范围：

- [ ] 分离主要接入 Gateway 与 per-peer 业务 relay，维护少量备用和复用连接。
- [ ] 协商共同可达 relay，通过实际 VLESS 链路测 A↔G↔B 的 RTT／丢包。
- [ ] 实现候选筛选、归一化成本／简单质量排名、置信度和切换防抖。

验收：

- [ ] A→B、A→C 可使用不同 relay；双方原 home relay 不同也能协调共同接入。
- [ ] 存在本机最近但全路径更差的 Gateway 时选择较好业务路径。
- [ ] Gateway 故障自动使用已验证备用，网络恢复无持续切换抖动。

### [MESH-15][P3] 验收多地域单 relay 选路、故障切换和质量收益

实施仓库：`ai-workspace-xstream/docs`。

范围：

- [ ] 准备至少两个地域、不同单段延迟／完整延迟的 Gateway 候选。
- [ ] 注入 Gateway 断连、慢转发、拥塞和目录变化，记录实际 VLESS 业务路径。

验收：

- [ ] 证明使用完整路径选路，不只按地理距离或本机 RTT。
- [ ] 不同 peer 独立选择，主要接入与业务 relay 可不同。
- [ ] 失败回退、恢复防抖、ACL 和持续业务通过；记录版本与量化证据。

### [MESH-16][P4] 实现 IPv6／STUN／ICE 公网直连与 NAT 打洞

实施仓库：`ai-workspace-xstream/XConnect-One`。

范围：

- [ ] 增加 IPv6 global、公网映射候选及认证信令，评估 Pion ICE 的依赖和接口边界。
- [ ] 验证 NAT 穿透，保留 VLESS relay；不把 STUN 成功当成端到端直连成功。
- [ ] 记录能力与实际路径，支持移动网络／出口变化后的重新收集与验证。

验收：

- [ ] IPv6 和可打洞 IPv4 网络端到端 WG 业务直接通信。
- [ ] 限制 NAT、CGNAT 场景和 UDP 阻断能可靠回退；不承诺所有 NAT 可打洞。
- [ ] 公网候选身份绑定、重放防护和策略限制有效。

### [MESH-17][P4] 验收异地公网直连与受限网络 relay 兜底

实施仓库：`ai-workspace-xstream/docs`。

范围：

- [ ] 覆盖 IPv6、可打洞 IPv4、限制 NAT、移动网络和禁 UDP 出口。
- [ ] 比较直连与 VLESS relay 的真实业务质量、MTU和切换行为。

验收：

- [ ] 记录每种网络的实际候选、路径与成功／不支持原因。
- [ ] UDP 受阻时仍双向可用，直连探测有退避，不破坏 relay。
- [ ] 持续 TCP/HTTP、双向 UDP、权限和网络变化均有证据。

### [MESH-18][P5] 实现 VLESS Gateway 联邦与最多两跳中继

实施仓库：`ai-workspace-xstream/XConnect-Gateway`。

范围：

- [ ] 增加授权 Gateway 身份、独立 VLESS/TLS 互联、可信源上下文和远端 attachment 目录。
- [ ] 实现 G1→G2 的密文转发、反向投递、session epoch、有效期和路径／跳数限制。
- [ ] 按需连接或复用地域骨干；对互联限流、背压、目录失效与撤销处理。

验收：

- [ ] A 仅能接入 G1、B 仅能接入 G2 时，经允许互联完成双向 WG 业务。
- [ ] 伪造源身份、跨 network、陈旧 attachment、环路和第三个 Gateway 被拒绝。
- [ ] 空闲主动回包、互联断线重连和慢消费者压力通过，内部 WG 不在 Gateway 解密。

### [MESH-19][P5] 实现单／双 relay 候选探测和跨地域自适应

实施仓库：`ai-workspace-xstream/XConnect-One`。

范围：

- [ ] 加入 Gi→Gj 候选，按真实 A↔Gi↔Gj↔B 链路探测。
- [ ] 无共同 Gateway 时选择双 relay；质量显著更好时允许稳定升级。
- [ ] 与 direct、单 relay 共享每 peer 状态机和安全租约。

验收：

- [ ] 没有共同可达 relay 的节点自动建立双 Gateway 路径。
- [ ] 实际测量完整路径，不以各节点分别最近 Gateway 推断最优。
- [ ] 路径失效自动尝试允许备用，所有 Gateway 不可达时明确报告不可用。

### [MESH-20][P5] 验收无共同 Gateway 的跨地域联邦 relay

实施仓库：`ai-workspace-xstream/docs`。

范围：

- [ ] 搭建限制双方 Gateway 可达范围的网络，覆盖允许互联和互联不可达。
- [ ] 验证端到端身份、ACL、撤销、MTU、主动发包和持续业务。

验收：

- [ ] 无共同 relay 时实际经 G1→G2，恢复单 relay／direct 后可自适应。
- [ ] 互联故障可诊断并回退；不能以 Gateway 服务在线替代 A↔B 可用。
- [ ] 跳数／环路限制、队列压力和不同网络隔离证据完整。

### [MESH-21][P3] 汇总路径可观察性与 Zero／App 诊断契约

实施仓库：`ai-workspace-xstream/xconnect-edge-agent`。

范围：

- [ ] 上报 direct_lan/direct_wan/relay_single/relay_federated 摘要、profile、RTT和切换原因。
- [ ] 版本化本机／控制面诊断契约，明确 Portal/App 读取与展示责任，不持有私钥。
- [ ] 区分注册、心跳、ACK、READY、端到端健康；限制高基数和敏感日志。

验收：

- [ ] 用户可识别活动路径和真实失败原因，默认无需手选 IP／Gateway。
- [ ] 统计与实际流量一致，状态含时间和来源；未知质量不展示为零。
- [ ] 输出可供后续 Portal/App 集成的契约及本地诊断，凭据不可出现在日志／指标。

### [MESH-23][P5] 完成 N×M 规模、故障注入与分阶段 release 验收

实施仓库：`ai-workspace-xstream/docs`。

范围：

- [ ] 建立可参数化 N One、M Gateway 压测与失败注入，按活跃 peer 维护状态。
- [ ] 验证候选裁剪、连接复用、探测退避、重连随机化、CPU／内存／socket／队列上限。
- [ ] 归档各阶段不可变版本、UAT 网络条件、权限／MTU／恢复／吞吐矩阵及回滚演练。

验收：

- [ ] 不会持续高频运行所有 N×N peer／N×M探测／M×M互联。
- [ ] Zero 故障、批量 Gateway 故障和慢消费者下安全期限／资源限制符合目标。
- [ ] 所有阶段记录 p50/p95、丢包、恢复时间与实际路径证据；未覆盖能力明确列出。
