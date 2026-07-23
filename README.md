# 项目级多 Agent 开发工作区

这是一个只在当前仓库生效的 Codex 多 Agent 协作模板。它不会修改
`~/.codex`、系统环境或其他项目。

母版不包含前端、后端或其他业务代码占位目录。复制到具体项目后，应根据真实目录结构修改
根 `AGENTS.md` 的文件所有权，以及 `.codex/agents/` 中实施角色的路径说明。

## 复制范围

最小配置只需要复制：

- 根 `AGENTS.md`
- 整个 `.codex/` 目录

如果需要完整的 Agent 通信、任务管理和检查功能，再复制：

- `coordination/`
- `scripts/`
- `tasks/`
- `contracts/`
- 相关 `docs/` 模板

建议放在目标项目根目录：

```text
F:\projects\my-app\
├─ AGENTS.md
├─ .codex\
│  ├─ config.toml
│  ├─ agents\
│  └─ rules\
├─ coordination\
├─ scripts\
├─ tasks\
├─ contracts\
└─ 项目原有代码……
```

## 已配置的团队

- 主 Agent：需求澄清、架构决策、任务拆分、冲突处理和最终验收。
- `frontend`：只负责前端实现及前端测试。
- `backend`：只负责后端、数据模型和服务测试。
- `tester`：独立补充测试并提供可复现证据，不替开发 Agent 宣布完成。
- `security_reviewer`：只读安全审查。
- `docs_writer`：维护文档、接口说明和架构决策记录。

主 Agent 最多并行运行 4 个子 Agent。Agent 默认继承当前会话的模型，
所以模板不会因为模型版本更新而过期。

## 安全边界

项目默认使用：

- `workspace-write` 沙箱；
- `on-request` 审批策略；
- 沙箱内禁用网络；
- 用户作为审批人；
- 对推送、合并、发布、危险 Git 操作、部署和基础设施变更增加项目级命令规则。
- 连接器写操作默认进入用户审批；
- 子进程只继承核心环境变量，且不使用登录 Shell。

运行时在 Codex 界面中选择的权限模式优先于项目默认值。因此启动任务前，
请确认界面中的权限模式仍然符合预期。

命令规则只控制申请在沙箱外运行的命令，不能替代沙箱和人工审查。

## 第一次使用

1. 将母版需要的目录复制到具体项目根目录，不要覆盖项目已有文件。
2. 按真实源码和测试目录修改根 `AGENTS.md` 与专业 Agent 配置。
3. 在 Codex 中打开具体项目并将其标记为可信；未信任的项目不会加载项目 `.codex/` 配置。
4. 开启一个新任务，让 Codex 重新读取项目配置。
5. 验证工作区：

   ```powershell
   python .\scripts\validate_workspace.py
   ```

6. 新建任务卡：

   ```powershell
   python .\scripts\new_task.py user-profile "实现用户资料编辑"
   ```

7. 对主 Agent 发出请求，例如：

   ```text
   按本项目的多 Agent 工作流实现用户资料编辑。
   先建立任务卡和接口契约，再并行安排前端、后端与测试；
   最后让安全审查 Agent 检查权限边界并由你统一验收。
   ```

## 目录

```text
.codex/
  config.toml            项目级权限和并发配置
  agents/                专业 Agent 定义
  rules/                 敏感命令审批规则
AGENTS.md                主 Agent 的长期治理规则
contracts/               前后端共享契约
  CONTRACT_TEMPLATE.yaml 可复制的共享契约模板
tests/                   协作工具自检和测试目录规则
docs/                    文档和架构决策
coordination/            持久消息箱与确认记录
tasks/
  active/                正在执行的任务卡
  approvals/             待用户决定的审批卡
  done/                  已验收任务
scripts/                 工作区辅助脚本
```

前端和后端目录故意不在母版中创建，避免把示例目录误复制进采用其他布局的项目。

## 推荐节奏

1. 主 Agent 创建任务卡并写清验收标准。
2. 先冻结 `contracts/` 中的共享契约。
3. 主 Agent 按目录所有权分派互不重叠的子任务。
4. 专业 Agent 并行实现，测试和安全角色保持独立。
5. 主 Agent 检查 diff、构建、测试和风险报告。
6. 敏感操作生成审批卡，等待用户明确批准。
7. 验收后通过 `scripts/complete_task.py` 完成机械验收门并移动任务卡。

Agent 的实时交流由 Codex 原生子 Agent 线程完成；`coordination/` 提供不可变、可审计的
持久消息。它不会后台轮询或自动唤醒 Agent。

更完整的状态流见 [docs/OPERATING_MODEL.md](docs/OPERATING_MODEL.md)。
安全审计、已修复问题和剩余边界见 [docs/SECURITY_AUDIT.md](docs/SECURITY_AUDIT.md)。
