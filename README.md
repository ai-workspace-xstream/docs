# AI Workspace XStream 项目文档

本仓库用于集中管理 AI Workspace XStream 的需求、设计、实施计划、架构决策、运行手册、故障记录和发布说明。当前仓库仅保存 Markdown 文档，不存放业务代码、密钥、数据库备份或生成产物。

## 目录

| 目录 | 用途 |
| --- | --- |
| [`requirements/`](requirements/README.md) | 已确认或待排期的产品及技术需求 |
| [`plans/`](plans/README.md) | 版本、里程碑和实施计划 |
| [`architecture/`](architecture/README.md) | 当前架构、组件关系和接口契约 |
| [`decisions/`](decisions/README.md) | 重要架构及产品决策记录 |
| [`runbooks/`](runbooks/README.md) | 可执行的部署、运维、应急和回滚步骤 |
| [`incidents/`](incidents/README.md) | 故障、修复快照和复盘记录 |
| [`releases/`](releases/README.md) | 各版本发布范围、验证结果和已知问题 |

## 文档约定

- 文件统一采用 Markdown，扩展名为 `.md`。
- 主题文档使用 `YYYY-MM-DD-简短主题.md` 命名。
- 每份文档应写明状态、日期、范围和验收或结论。
- 需求描述“做什么”和验收边界；实施细节写入 `plans/`。
- 已执行的应急操作写入 `incidents/`；可重复执行的标准操作写入 `runbooks/`。
- 不提交密码、令牌、私钥、数据库转储或真实用户凭证。

## 当前重点

- [下一版本账户暂停与代理 UUID 管理需求](requirements/2026-07-05-account-suspension-and-proxy-uuid.md)
- [svc.plus 修复快照](incidents/2026-07-05-svc-plus-repair-snapshot.md)
