# 2026-11-23 重复 `EXPORTER_NODE_ID` 导致 Edge Checkpoint 乒乓循环与账单虚增复盘

- **状态**: 已修复并上线防护
- **日期**: 2026-11-23
- **影响范围**: `jp-xconnect-01.svc.plus`、`jp-xconnect-02.svc.plus`、计费实时增量入库与桌面端连接可用性
- **关联 PR**:
  - [`ai-workspace-xstream/xconnect-edge-agent#167`](https://github.com/ai-workspace-xstream/xconnect-edge-agent/pull/167) (`setup-observability.sh` 显式绑定 `agent_proxy_domain` 与 `xray_exporter_node_id`)
  - [`ai-workspace-infra/playbooks#1046`](https://github.com/ai-workspace-infra/playbooks/pull/1046) (`roles/vhosts/xray-exporter` 渲染前强制校验 `EXPORTER_NODE_ID == AGENT_PROXY_DOMAIN`)
- **关联 ORG 文档**:
  - `ai-workspace-services/knowledge`: `docs/02-accounts/Runbook/Fix-Duplicate-Node-ID-Billing-Pingpong.md`
  - `ai-workspace-infra/docs`: `runbooks/2026-11-23-exporter-node-id-uniqueness-and-billing-pingpong.md`

---

## 现象

1. 某测试/运营账号最近 24 小时计费面板显示 **`75.82 GiB`**（本月累计 **`482.84 GiB`**），触发 `50 GiB` 配额耗尽拦截。
2. 核查两台节点本地 `/var/lib/xray-exporter/xhttp-snapshots.json` 原始计数器：
   - `jp-xconnect-01.svc.plus` 最近 24 小时真实流量约为 `19.20 GiB`，`jp-xconnect-02.svc.plus` 约为 `0.09 GiB`，叠加其余节点全天真实流量合计仅约 `21.78 GiB`。
3. 客户端表现出“**macOS 桌面端连接 `jp-xconnect-01.svc.plus` 异常断开，而手机端 APP 可连接**”的分裂现象。

---

## 根因

1. **Edge Exporter `node_id` 冲突**：
   - `jp-xconnect-01.svc.plus` 的 `/etc/xray-exporter.env` 中误配为 `EXPORTER_NODE_ID=jp-xconnect-02.svc.plus`，且 `/etc/systemd/system/xray-exporter-{xhttp,tcp}.service` 中的 `--node-id` 同样指向 `jp-xconnect-02.svc.plus-{xhttp,tcp}`。
   - 两台物理机每分钟错开约 30 秒通过本地 Vector 向 `billing-service` 上报同一个 `node_id`。
   - 由于两台机上的 Xray 用户累计计数器不同（例如 `16:59 UTC` 时 `jp-01` 累计 `1.177 GB`、`jp-02` 累计 `44.5 MB`），较小值先到触发 `reset_epoch++` 覆盖 checkpoint，随后较大值到达立即将整段历史差值（`1.132 GB`）作为单分钟新增流量入账，形成每分钟重复扣减的“乒乓循环”。
2. **macOS 与手机端表现差异**：
   - 虚假计费耗尽配额后，`accounts` 下发 `403 quota_exhausted` 触发 macOS `DesktopSyncService` 的 `quotaPaused` 状态，且 `jp-xconnect-01` 同步移除了用户 UUID（Xray 日志报 `rejected proxy/vless/encoding: invalid request user id`）；而 `jp-xconnect-02` 保留了 `/etc/agent/account-agent.yaml` 的 `allowedEmails` 白名单兜底，手机端恰好路由到 `jp-02` 因此未中断。
   - 另外 `jp-xconnect-01.svc.plus:1443`（`xray-tcp.service`）公网安全组未放行导致 TCP Vision 探测超时，仅 `443`（HAProxy XHTTP `/split`）可达。

---

## 处置与验证

1. **全量 6 台在线边缘节点核验与修复**：
   - 停用 `jp-xconnect-01` exporter → 修正 `/etc/xray-exporter.env` 与 systemd `--node-id` 为 `jp-xconnect-01.svc.plus` → 重写本地 72h 缓存快照的 `node_id` 并同步数据库 `traffic_stat_checkpoints` 水位 → 重启生效。
   - 核验 `jp-xconnect-01`、`jp-xconnect-02`、`jp-xconnect-free-01`、`ph-xconnect`、`us-xconnect-los-angeles-custom-01`、`us-xconnect` 共 6 台节点的 `AGENT_PROXY_DOMAIN == EXPORTER_NODE_ID` 均已唯一且服务 `active`。
2. **数据库清理与恢复**：
   - 备份 11 月原始分钟桶至 `traffic_minute_buckets_backup_20261123`，基于本地原始快照重建最近 72h 增量并去重历史乒乓桶，重算全部 `41` 个账户配额，24h 用量从 `75.82 GiB` 恢复至 `20.65 GiB`，本月从 `482.84 GiB` 恢复至 `205.19 GiB`。
3. **代码防复发**：
   - `xconnect-edge-agent#167` 在 `scripts/setup-observability.sh` 中显式将 `AGENT_PROXY_DOMAIN` 绑定至 `xray_exporter_node_id`。

---

## 后续防范

- 任何节点不得跨机复制 `/etc/xray-exporter.env` 或 `/etc/systemd/system/xray-exporter*.service`。
- 通过 `setup-observability.sh` 或 Ansible role 变更时，必须通过 `AGENT_PROXY_DOMAIN == EXPORTER_NODE_ID` 断言门禁。
