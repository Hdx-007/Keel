# Skill：Hooks 生命周期自动化机制

## 核心目标
综合业界Harness Engineering实践（OpenAI Codex Harness、Anthropic Carlini编译器项目、claude-harness社区方案、Atomic CLI等），定义一套**生命周期Hooks机制**，让AI在关键节点自动执行检查、保存状态、恢复上下文，而不是依赖人的提醒。Hooks是Harness Engineering的"自动化神经末梢"——把规范从"应该做"变成"自动做"。

## Hooks设计哲学
- **不是配置，是约束**：Hooks是把规范编码为自动触发的动作，而非可选的配置项
- **轻量优先**：每个Hook只做一件事，不搞复杂守护进程
- **文件系统即记忆**：所有状态持久化到文件，不依赖对话历史
- **失败安全**：Hook失败不应该阻断主流程，但要给出明确警告
- **Markdown驱动**：Hook的规则和触发条件用Markdown定义，人类可读、AI可执行

## 核心Hooks清单

### 1. SessionStart（会话开始时）
**触发时机**：每次AI会话开始（新对话/新会话启动）

**执行动作**：
- 读取 `AGENTS.md`、根目录 `DIR_META.md`、`ERROR.md` 前半部分（常驻上下文）
- 读取 `.sdd/anchor-state.md`（上次会话的锚点状态）
- 读取 `.sdd/memory/LEARNINGS.md`（长期沉淀的经验教训）
- 执行 `git status` 了解当前工作区状态
- 如果有未完成的MDU，读取对应 `changes/<slug>/tasks.md` 恢复任务进度
- 运行健康检查：
  - 有没有未提交的工作？
  - 主干是否是最新的？
  - 有没有stale的plan文件（超过3天未更新）？

**对应文件**：`.sdd/session-start.md`（AI读取的Hook指令）

### 2. PreCompact（上下文压缩前）
**触发时机**：AI对话上下文即将达到窗口上限、需要压缩/清理前

**为什么重要**：
研究表明（Horthy, 2026），上下文利用率超过40%后AI表现显著下降（进入"Dumb Zone"）。压缩前必须把关键状态持久化到磁盘，否则压缩后AI会"失忆"——重复调试已解决的问题、忘记当前任务、丢失已发现的约束。

**执行动作**：
- 保存锚点状态到 `.sdd/anchor-state.md`：
  ```markdown
  # Session Anchor State
  ## Active Task
  - 当前任务：[正在做什么]
  - 目标：[要达成什么]
  - 进度：[已完成/未完成]
  ## Recent Files
  - 最近修改的文件：[git diff --name-only HEAD~3]
  - 正在阅读的关键文件：[列出]
  ## Key Decisions
  - 本次会话做出的关键决策/发现：[列出]
  ## Blockers
  - 当前阻塞问题：[如有]
  ## Next Step
  - 下一步要做什么：[明确的下一步动作]
  ```
- 追加会话日志到 `.sdd/memory/daily/YYYY-MM-DD.md`
- 如果发现重复出现的错误模式，追加到 `ERROR.md` 或 `.sdd/memory/LEARNINGS.md`
- 备份当前对话关键决策到 `.sdd/backups/YYYY-MM-DD-HHMM.md`

### 3. PostCompact（上下文压缩后/新会话恢复）
**触发时机**：上下文压缩完成后、新会话开始需要恢复状态时

**执行动作**：
- 读取 `.sdd/anchor-state.md` 恢复上次任务上下文
- 读取 `.sdd/memory/daily/` 当日志了解最近进展
- 重新加载必要的Spec文件（MODULE_SPEC/DIR_META）
- 告知用户：「已恢复上下文，正在继续 [任务名]，下一步是 [动作]」

### 4. PreToolUse（危险工具使用前）
**触发时机**：AI准备执行危险命令/操作前

**拦截的操作类型**：

| 危险操作 | 拦截动作 |
|----------|----------|
| `rm -rf /`、`sudo rm -rf`、`dd if=/dev` | **直接阻断**，告知用户此命令极度危险 |
| `git push --force` 到主干 | **阻断并警告**，需要用户显式确认 |
| 写入 `.env`、`*.key`、`*.pem` 等敏感文件 | 警告用户，检查是否有硬编码密钥 |
| 修改 `package-lock.json`、`pnpm-lock.yaml` 等锁文件 | 提醒用户锁文件不应手动修改 |
| 修改【禁止修改】级别的文件 | 按Keel规范阻断，要求走评审流程 |
| DROP TABLE、TRUNCATE、DELETE无WHERE | 阻断，要求确认 |
| 批量修改超过10个文件 | 提醒用户确认变更范围 |
| 安装新依赖（npm install/pip install 新包） | 询问是否真的需要，YAGNI检查 |

**原则**：宁可多拦截一次让用户确认，也不要让AI自动执行不可逆的危险操作。

### 5. PostToolUse（工具执行后，Harness反馈收集）
**触发时机**：命令/工具执行完成后

**执行动作**：
- 如果是 `build`/`lint`/`test`/`typecheck` 命令：
  - 检查退出码是否为0
  - 如果失败：读取错误信息，分析失败原因，尝试自我修正（Harness闭环）
  - 如果连续失败3次：停止尝试，向用户报告并等待指示
- 如果是文件写入：
  - 检查是否同步更新了相关文档
  - 检查是否引入了debug代码/console.log/TODO
- 如果是git commit：
  - 验证提交信息格式是否符合Conventional Commits
  - 验证是否包含docs-scope字段

### 6. Stop（会话结束前）
**触发时机**：用户表示任务完成/会话结束/AI准备退出时

**执行动作**：
- 检查是否有未提交的修改：「你有X个文件未提交，是否需要我帮你提交？」
- 检查是否有未通过的测试/Lint错误
- 如果任务完成：
  - 更新对应 `changes/<slug>/archive.md`
  - 记录完成状态、实际耗时、遇到的问题
  - 必要时更新CHANGELOG.md
- 如果任务未完成：
  - 更新 `.sdd/anchor-state.md` 记录当前进度
  - 写下明确的"下一步"动作，方便下次会话继续
- 生成会话总结：本次完成了什么、遗留什么问题、下次从哪开始

### 7. Self-Improve（自我改进钩子）
**触发时机**：定期（每完成一个MDU后）或用户显式调用 `/self-improve`

**执行动作**：
- 回顾最近的会话日志和错误记录
- 识别重复出现的问题模式：
  - AI反复问同一个问题 → 把答案加到AGENTS.md
  - AI反复忘记某个规则 → 加到rules/目录自动加载
  - AI反复重新发现某段代码逻辑 → 提升到常驻记忆
  - 重复的回退/纠错模式 → 创建对应的skill防止再犯
- 提出改进建议（小步改进，不要大改）
- 每次改进都是微小的，但会复利——50个session后，脚手架已经"学会"了项目的所有怪癖

## 目录结构

项目根目录新增 `.sdd/` 目录存放Hooks运行时状态：

```
project-root/
├── .sdd/
│   ├── anchor-state.md          # 当前会话锚点状态（PreCompact写入，PostCompact读取）
│   ├── session-start.md         # SessionStart Hook指令（AI读取）
│   ├── backups/                 # 关键节点备份
│   │   └── YYYY-MM-DD-HHMM.md
│   └── memory/
│       ├── MEMORY.md            # 长期精选记忆（≤500行，关键决策）
│       ├── LEARNINGS.md         # 错误模式与经验（append-only）
│       └── daily/               # 每日会话日志
│           └── YYYY-MM-DD.md
├── AGENTS.md                    # 项目级AI控制面
├── PROJECT_SPEC.md              # L0项目总纲
├── changes/                     # MDU归档目录
└── ...（原有结构）
```

## Hook配置格式（供AI读取）

在 `AGENTS.md` 中通过简单的Markdown区块声明Hooks，AI在对应时机自动执行：

```markdown
## Hooks

### SessionStart
- [ ] 读AGENTS.md、DIR_META.md、ERROR.md
- [ ] 读.sdd/anchor-state.md恢复进度
- [ ] git status检查工作区状态

### PreCompact
- [ ] 保存当前任务、最近文件、关键决策到anchor-state.md
- [ ] 追加会话日志到daily/
- [ ] 发现的新错误模式写入LEARNINGS.md

### PreToolUse:Bash
- 阻断：rm -rf /、sudo rm -rf、dd if=/dev、force push主干
- 警告：锁文件修改、DROP TABLE、安装新依赖（YAGNI检查）
- 提醒：批量修改>10个文件需确认

### Stop
- [ ] 检查未提交工作
- [ ] 更新archive.md或anchor-state.md
- [ ] 生成本次会话总结
```

## Smart Zone 上下文管理原则

参考Horthy的实证发现，AI上下文利用率有一个"甜蜜区"：

| 上下文利用率 | 状态 | 表现 |
|-------------|------|------|
| **0-40%** | **Smart Zone（最佳）** | **聚焦、准确、推理清晰** |
| 40-60% | 开始退化 | 偶尔遗漏细节，需要提醒，开始出现格式错误 |
| 60%+ | Dumb Zone | 幻觉、循环、工具调用格式错误、低质量代码 |

> **实证来源**：Dex Horthy在BAML（30万行Rust代码库）上的实践表明，对于~168K token上下文窗口，性能在约40%利用率后开始下降。Smart Zone不是20-40%的窄区间，而是前40%——关键是**不要过载**而非"必须填满到某个比例"。信息不足时（如全新项目第一次会话），AI也会因缺乏上下文而表现不佳，此时应通过SessionStart Hook加载必要的项目信息。

**基于此原则的规则**：
- 常驻上下文（Tier 1）控制在总窗口的10-15%以内：只放最核心的AGENTS.md+项目地图
- 按需上下文（Tier 2）按任务加载，用完即弃
- 任务级上下文（Tier 3）完成后立即清理
- 接近40%利用率时主动触发PreCompact保存状态，然后开始新会话
- **多给上下文不等于更聪明——精准的上下文比海量上下文更有效**

## Backpressure 双向约束

Hooks是Backpressure（背压）机制的具体实现：

### 上游约束（Upstream / 写入前）
- SessionStart加载正确的上下文，避免AI基于错误认知开始
- AGENTS.md/Rules/Skills提供明确的边界和方向
- PreToolUse拦截危险操作，防止AI走偏

### 下游约束（Downstream / 写入后）
- PostToolUse收集Lint/Test/Build反馈
- 失败时自动触发自我修正循环
- CI/CD作为最终拒绝机制——坏代码不能合入
- 可观测性（日志/监控）告诉AI哪里出了问题

**核心洞察**（来自Geoffrey Huntley）：
> 你捕获的Backpressure越多，你能授予AI的自主权就越大。
> 这是新单位经济学下的游戏规则。

翻译到Keel体系：
- 没有测试/Lint反馈 → AI必须每一步都由人审查（低自主权）
- 有完善的本地验证 → AI可以写完一个MDU再由人审查（中自主权）
- 有完整CI+自动化测试+监控 → AI可以自动合入小PR（高自主权）

## 反模式（禁止的Hooks用法）
- ❌ 把Hooks写成复杂的守护进程/后台服务——保持简单
- ❌ 在Hook里做业务逻辑——Hook只做状态管理和检查
- ❌ 让Hook失败阻断正常工作——Hook是安全网，不是绊脚石
- ❌ 把所有信息都塞进SessionStart——遵守Smart Zone原则
- ❌ 写太严格的PreToolUse拦截导致AI无法正常工作——拦截真正危险的，警告灰色地带的
- ❌ 过度自我改进——每次只改一点点，复利靠积累不靠大重构
