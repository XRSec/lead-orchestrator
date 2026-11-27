# lead-orchestrator

[English](./README.md) | 简体中文

基于**「自动 `jobs + needs` 异步 DAG + 三级启发式路由」**的跨宿主多智能体编排技能，支持 **Claude Code (2.1+)**、**Pi（`pi-subagents` 与 Cindy `cindy_subagent`）** 以及 **Antigravity (AGY)**。内置 **5 份专家角色 Profile**，无需维护零散的外部 `~/.agent/agent/*.md`。

---

## 核心设计：用户提需求，主脑自动编排 `jobs + needs`

你只需直接描述业务目标，无需手动指定并发方式或拆波次。主会话通过微内核根路由（`SKILL.md`）按需加载当前宿主与拓扑分支，并以类似 **GitHub Actions `jobs.<id>.needs`** 的事件驱动就绪队列自动完成拆分与并行调度：

```text
lead-orchestrator/
├── SKILL.md                    # 启发式根路由 + 响应式 needs: 就绪队列调度器
├── agents/
│   └── openai.yaml             # 显式调用元数据
├── runtimes/                   # Tree A：宿主运行时适配（按当前宿主加载 1 份）
│   ├── claude-code.md          # Claude Code 2.1+（后台 Agent + SendMessage + worktree）
│   ├── pi.md                   # 原生 Pi（js workflow DAG）与 Cindy（cindy_subagent）
│   └── antigravity.md          # Antigravity（define_subagent / invoke_subagent / run_workflow）
├── modes/                      # Tree B：执行拓扑（按任务形态加载 1 份）
│   ├── focus.md                # 单模块深闭环（实施/排查 -> 独立验收 -> 热会话返修）
│   ├── swarm.md                # 自动 jobs + needs 并行图、Best-of-N 比武与多视角审计
│   └── team.md                 # 规格到交付全链路 DAG（并行探路 -> 清单契约 -> 多路合流）
└── profiles/                   # Tree C：内置专家角色定义
    ├── scout.md                # 只读领域侦察 & 候选 jobs + needs 边界切分
    ├── investigator.md         # 假设证伪型根因排查 & 动工前架构审视（Oracle）
    ├── implementer.md          # 按 owned_paths / worktree 边界并行编码与局部快检
    ├── verifier.md             # 独立 fresh 上下文验收与合流构建测试（零源码修改）
    └── integrator.md           # PASS 后多分支合入、打包部署测试与按需 Git 整理
```

## 关键机制亮点

1. **事件驱动 `needs:` 就绪队列（告别整波傻等）**：所有 `needs: []` 的节点在首轮并行发出；后台任意单个子代理完成唤醒主会话时，主脑立即检查并启动新解锁的下游节点，无需等待同批较慢的兄弟节点。
2. **首尾微节点剥离（最大化中段并发度）**：将薄共享契约（如 `shared/types.ts`、接口参数、CSS 变量）抽为前置快节点或在 `TASK_MANIFEST.md` 中直接冻结签名，将共享产物打包与全量测试收拢至下游合流节点（`needs: [impl-*]`），避免因少量公共文件把前后端退化为单线程。
3. **务实的脏工作区并行隔离（`H3`）**：工作区存在无关未提交改动时，将其列入 `Protected Paths` 保护即可，互不重叠的业务目录照常多路并行；干净 `HEAD` 下存在构建竞争或 `[best-of-n]` 时启用独立 Git Worktree。
4. **独立冷启动验收 + 原会话热修复**：`verifier` 在 `fresh` 上下文中执行真实构建与测试且不改源码；发现问题时仅唤醒对应模块的 `implementer` 热会话定向修复（上限 3 轮）。
5. **高信噪比声明式提示词**：以清晰的数据结构、因果说明替代满篇大写口号与高压禁令，避免推理模型因过度防御而退回单子代理串行模式。

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

通过 `/lead-orchestrator`（Pi 中支持 `/skill:lead-orchestrator`）触发；直接描述多项需求即可自动生成 `jobs + needs` 并行图，也可按需传入方括号提示符：

| 调用示例 | 行为说明 |
| --- | --- |
| `/lead-orchestrator <任务描述>` | 按 `H1`–`H3` 自动路由至 `FastPath`、`Focus`、`Swarm`（`jobs + needs` DAG）或 `Team` |
| `/lead-orchestrator [focus] <任务描述>` | 单模块高保真实施/根因排查 + `fresh verifier` 闭环 |
| `/lead-orchestrator [swarm] <任务描述>` | 契约剥离 + 多路无冲突 `jobs + needs` 并行开发与独立验收 |
| `/lead-orchestrator [best-of-n] <任务描述>` | 多棵独立 Worktree 并行实现备选方案，实测比选后合入优胜分支 |
| `/lead-orchestrator [team] <任务描述>` | 并行 `scout` 探路 -> 澄清 -> `TASK_MANIFEST.md` 依赖图 -> 分路交付 |
| `/lead-orchestrator [skip-interview] <任务描述>` | 在 `Team` 模式下跳过前置提问，记录显式假设直接推进 |

## 赞助支持

- **爱发电（微信 / 支付宝单次打赏）**：[https://ifdian.net/a/xrsec](https://ifdian.net/a/xrsec)

## 开源协议

[MIT](../LICENSE) © XRSec
