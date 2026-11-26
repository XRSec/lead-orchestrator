# lead-orchestrator

[English](./README.md) | 简体中文

基于**「三级启发式路由 + 多 Tree 拓扑」**的跨宿主多智能体编排技能，支持 **Claude Code (2.1+)**、**Pi（`pi-subagents` 与 Cindy `cindy_subagent`）** 以及 **Antigravity (AGY)**。技能内置 **5 份可移植专家角色 Profile**，彻底替代零散的外部 `~/.agent/agent/*.md` 定义文件。

---

## 核心架构：启发式 + 多 Tree 结构

传统单体编排技能往往将所有平台语法与固定六步瀑布流塞进同一份长文档，既浪费上下文，又容易在脏工作区或跨宿主调用时报错。`lead-orchestrator` 通过 **55 行微内核根路由（`SKILL.md`）** 与三道零开销启发式（`H1`–`H3`），按需仅加载当前命中的单一路径：

```text
lead-orchestrator/
├── SKILL.md                    # 启发式根路由（H1 宿主指纹 × H2 任务拓扑 × H3 隔离策略）
├── agents/
│   └── openai.yaml             # 显式调用策略元数据
├── runtimes/                   # Tree A：宿主运行时适配树（每次仅读当前宿主 1 份）
│   ├── claude-code.md          # Claude Code 2.1+（Agent + SendMessage + worktree 门禁）
│   ├── pi.md                   # 原生 Pi（pi-subagents / js workflow）与 Cindy（cindy_subagent）
│   └── antigravity.md          # Antigravity（define_subagent / invoke_subagent / run_workflow）
├── modes/                      # Tree B：执行拓扑树（按任务复杂度仅读 1 份）
│   ├── focus.md                # 单车道深闭环（原样意图 -> 实施/调查 -> 独立验收 -> 热会话纠偏）
│   ├── swarm.md                # 多树并行分片、Best-of-N 锦标赛多 Worktree、多视角只读审计
│   └── team.md                 # 多波次有向无环图（探路 -> 澄清门禁 -> 免追踪清单 -> Oracle -> 集成）
└── profiles/                   # Tree C：内置专家角色树（替代外部 ~/.agent/agent/*.md）
    ├── scout.md                # 只读侦察、依赖与并行 Seam 切分、工作区干净度探测
    ├── investigator.md         # 假设证伪型根因排查 & 动工前架构挑刺（Oracle）
    ├── implementer.md          # 严格锁定文件/Worktree 边界的代码实施者 + 本地自检
    ├── verifier.md             # 强制 fresh 上下文的独立对抗验收官（严禁改文件）
    └── integrator.md           # PASS 后多 Worktree 合入、文档同步与受控授权提交
```

## 关键机制亮点

1. **主线程零代码触碰（Zero Inline Code-Touch）**：委派模式下主会话只负责分流、澄清、验收裁决与中文交付，绝不在主线程猜代码或直接改文件。
2. **用户意图零失真传递（Verbatim Fidelity）**：冷启动 Brief 强制内嵌未改写的用户原话与边界坐标，防止多层转述丢失细节约束。
3. **自适应多 Worktree 与脏区降级（Heuristic `H3`）**：
   - **干净 `HEAD` + 构建/测试状态竞争或 Best-of-N 比武**：自动启用独立 Git Worktree（`isolation: "worktree"` / `worktree: true` / `Workspace: "share"`）。
   - **存在未提交改动的脏工作区或共享目录宿主**：自动降级为**共享树无交集 Seam 隔离**（严格锁定 `Authorized Scope` 且禁跑 `.git/index` 命令），避免 Pi 脏区拒跑或 Claude Code 从 `HEAD` 切分支丢掉本地改动。
4. **独立冷启动验收 + 同会话热修复**：`verifier` 恒以 `fresh` 上下文运行且**零文件修改**（彻底消除作者与审查者双写冲突）；未通过项沿原热会话（`SendMessage` / `resume` / `steer` / `send_message`）定向回传修复，上限 3 轮。
5. **免污染暂存清单**：临时 Manifest 统一写入 `.pi/subagents/`、`$TMPDIR` 或 `<ArtifactDir>/scratch/`，绝不在业务仓库根目录遗留 `.t3/` 未跟踪文件。

---

## 安装方式

### Skills CLI（通用安装）
```bash
npx skills add XRSec/lead-orchestrator
```

### Pi Coding Agent 原生包安装
```bash
pi install git:github.com/XRSec/lead-orchestrator
```

### 手动克隆与多端软链
```bash
git clone https://github.com/XRSec/lead-orchestrator.git ~/.agents/skills/lead-orchestrator
ln -s ~/.agents/skills/lead-orchestrator ~/.claude/skills/lead-orchestrator
ln -s ~/.agents/skills/lead-orchestrator ~/.gemini/config/skills/lead-orchestrator
```

---

## 使用指南

通过 `/lead-orchestrator`（Pi 中支持 `/skill:lead-orchestrator`）显式触发；支持自动分流或使用方括号修饰符强制指定拓扑：

| 调用示例 | 行为说明 |
| --- | --- |
| `/lead-orchestrator <任务描述>` | 按 `H1`–`H3` 自动路由至 `FastPath`、`Focus`、`Swarm` 或 `Team` |
| `/lead-orchestrator [focus] <任务描述>` | 强制单车道高保真实施/根因排查 + `fresh verifier` 闭环 |
| `/lead-orchestrator [swarm] <任务描述>` | 强制无冲突多 Seam 并行树 + 独立验收官（每 2–4 名作者配 1 名） |
| `/lead-orchestrator [best-of-n] <任务描述>` | 在多棵独立 Worktree 中并行实现备选方案，实测比武后合入优胜树 |
| `/lead-orchestrator [team] <任务描述>` | 完整多阶段 DAG：`scout` 探路 -> 澄清门禁 -> Scratch Manifest -> 分波次实施 |
| `/lead-orchestrator [skip-interview] <任务描述>` | 在 `Team` 模式下跳过前置提问，直接记录显式假设推进 |

## 开源协议

[MIT](../LICENSE) © XRSec
