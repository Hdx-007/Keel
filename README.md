<p align="center">
  <h1 align="center">⚓ Keel</h1>
  <p align="center">
    给AI开发上龙骨——让AI在规矩内高效干活
  </p>
</p>

<p align="center">
  <a href="#核心特性">核心特性</a> •
  <a href="#快速开始">快速开始</a> •
  <a href="#项目结构">项目结构</a> •
  <a href="GETTING_STARTED.md">操作手册</a> •
  <a href="ACKNOWLEDGMENTS.md">致谢</a>
</p>

---

## Keel是什么

**Keel（龙骨）** 是AI原生规格驱动开发范式。龙骨是船的脊梁——没有龙骨，船会散架、会翻；有龙骨，才能乘风破浪。Keel就是你AI辅助开发项目的龙骨：给AI定规矩、定方向、定边界，让AI在规矩内高效干活，人类聚焦工程判断。

Keel不是框架、不是工具，而是一套**规范+技能包+模板**的组合，一个`AGENTS.md`文件就能让AI遵守9条铁律。

### 它解决什么问题

- AI上来就读全仓库，上下文爆炸还瞎改
- 没规格就写代码，出来的东西和想的不一样
- 做的UI千篇一律（紫蓝渐变+三栏卡片），交互反直觉（右键没反应、点了没反馈）
- 改完不验证，相同的坑重复踩
- 文档写完就过期，和代码脱节
- 主干经常挂，没人敢部署
- AI不知道什么时候该做什么事
- Debug全靠猜，找不到根因

## 核心特性

| 特性 | 说明 |
|------|------|
| **9条铁律** | 不可违反的硬约束，违反即严重错误 |
| **场景决策表** | 11种常见场景自动识别，AI不用"悟" |
| **强制自审** | 代码写完必须Reviewer视角过一遍再输出 |
| **四层渐进加载** | 禁止上来读全仓库，控制上下文在Smart Zone |
| **三级规格** | L0项目总纲→L1模块规格→L2接口契约 |
| **10关MDU交付** | 每个最小交付单元过10道门禁才合入 |
| **6个专业角色** | CEO/Designer/Reviewer/QA/Security/Release按需切换 |
| **Anti AI Slop** | 视觉+交互黑名单，消灭AI模板感 |
| **专业工具交互** | 右键菜单/Cmd+K/快捷键/拖拽/乐观UI等桌面级范式 |
| **错误沉淀** | ERROR.md+三道防线，不犯第二次同样的错 |
| **文档代码共生** | 改代码必改文档，Git提交带docs-scope |
| **规模适配** | 个人/小团队/大团队各有裁剪标准 |

## 快速开始

### 30秒安装

1. 下载最新Release解压到项目根目录
2. 确保`AGENTS.md`在项目根目录（AI工具自动读取）
3. 第一次对话跟AI说："读AGENTS.md，按Keel规范来"

### 不同AI工具适配

| 工具 | 自动读取文件 | 操作 |
|------|------------|------|
| **Trae** | `AGENTS.md` | 直接用 ✅ |
| **Cursor** | `.cursorrules` 或 `AGENTS.md` | 新版直接用，旧版复制到`.cursorrules` |
| **Windsurf** | `.windsurfrules` | 复制AGENTS.md内容 |
| **Claude Code** | `CLAUDE.md` | `cp AGENTS.md CLAUDE.md` |
| **GitHub Copilot** | `.github/copilot-instructions.md` | 复制AGENTS.md内容 |

### 初始化项目

安装后第一次跟AI说：
> "按照Keel规范初始化这个项目，帮我创建PROJECT_SPEC.md和DIR_META.md"

AI会读取模板，问你几个问题（项目是什么、技术栈、目录结构），帮你创建好初始文档。

详细操作请见 [GETTING_STARTED.md](GETTING_STARTED.md)。

## 9条铁律

> 这9条是AI违反任意一条即视为严重错误的硬约束：

1. **无规格不开发** — 没有规格先写规格，用户确认再写代码
2. **四层渐进加载** — 禁止上来就读全仓库
3. **遵守可修改性分级** — 【禁止修改】绝对不动，【谨慎修改】先告知风险
4. **代码文档同步** — 改代码必改文档，Git提交含`docs-scope:`
5. **输出完整可运行代码** — 禁止半成品/TODO/废代码
6. **先查ERROR.md再动手** — 避免重复已知错误
7. **修改后必须验证** — 跑Lint/Test/Build，连续失败3次停下来报告
8. **危险操作必须确认** — rm -rf/force push/DROP TABLE/批量改>10文件先问
9. **写完必须自审** — Reviewer视角过一遍，UI类额外过Designer清单

## 项目结构

```
你的项目/
├── AGENTS.md              ← AI入口（自动读取，149行轻量控制面）
├── PROJECT_SPEC.md        ← 项目总纲（从模板创建）
├── DIR_META.md            ← 目录元数据（从模板创建）
├── ERROR.md               ← 错误沉淀（从模板创建）
├── skills/                ← Keel技能包
│   ├── README.md
│   ├── 01-requirement-spec.md    # 需求准入
│   ├── 02-concurrent-dev.md      # 并发开发（含YAGNI懒人哲学）
│   ├── 03-integration-test.md    # 联调测试
│   ├── 04-deploy-release.md      # 部署上线
│   ├── 05-ops-review.md          # 复盘沉淀
│   ├── api-contract.md           # 接口契约
│   ├── doc-binding.md            # 文档代码共生
│   ├── closed-loop.md            # 10关门禁交付
│   ├── error-tracking.md         # 错误沉淀
│   ├── git-management.md         # Git规范
│   ├── ai-loading.md             # 四层加载规则
│   ├── harness-engineering.md    # 反馈闭环
│   ├── hooks.md                  # 生命周期自动化
│   ├── ui-ux.md                  # Anti AI Slop + 专业工具交互
│   └── roles/                    # 6个专业角色
│       ├── ceo.md
│       ├── designer.md
│       ├── reviewer.md
│       ├── qa.md
│       ├── security.md
│       └── release.md
└── templates/             ← 可直接复制的模板
    ├── PROJECT_SPEC.template.md
    ├── MODULE_SPEC.template.md
    ├── API_SPEC.template.md
    ├── DIR_META.template.md
    ├── ERROR.template.md
    ├── CHANGELOG.template.md
    └── ADR.template.md
```

## 设计哲学

> 最好的代码是从未被写出来的代码。

- **YAGNI阶梯**（借鉴[Ponytail](https://github.com/DietrichGebert/ponytail)懒人编码）：需要做吗→已有吗→标准库能做吗→已有依赖能做吗→才写最少代码
- **修根因而不修症状**：grep调用方，在共享层一次性修复
- **最短diff胜出**：前提是真正理解了问题
- **Deletion over addition. Boring over clever.**
- **控制面不是百科全书**：AGENTS.md只写149行，详细规则按需加载
- **好设计是隐形的**：不追求炫，追求清晰、一致、可用

## 不同规模怎么用

| 实践 | 个人/小项目 | 2-5人团队 | 5+人团队 |
|------|------------|-----------|----------|
| 9条铁律 | ✅全部 | ✅全部 | ✅全部 |
| 四层加载 | ✅必须 | ✅必须 | ✅必须 |
| 接口契约 | ✅核心接口 | ✅必须 | ✅必须 |
| 单元测试 | ≥80%核心路径 | ≥90% | ≥90% |
| Code Review | 自我Review | 至少1人 | 至少1人 |
| changes/归档 | 🟢大变更才用 | 🟡推荐 | ✅必须 |
| .sdd/运行时 | 🟢可选 | 🟡推荐 | ✅必须 |

## 致谢

Keel站在巨人肩膀上，核心思想大量借鉴开源社区的优秀实践：
- [Ponytail](https://github.com/DietrichGebert/ponytail) — YAGNI懒人编码哲学
- [GitHub Spec Kit](https://github.com/github/spec-kit)、[OpenSpec](https://github.com/Fission-AI/OpenSpec)、[Kiro](https://kiro.dev) — 规格驱动范式
- [gstack](https://github.com/gstack) — 角色化Skill思想
- [Uncodixfy](https://github.com/cyxzdev/Uncodixfy)、[Hallmark](https://github.com/Nutlope/hallmark)、[ux-skill](https://github.com/Laith0003/ux-skill) — Anti AI Slop
- VSCode、Figma、Linear、Notion — 专业工具交互范式

完整致谢见 [ACKNOWLEDGMENTS.md](ACKNOWLEDGMENTS.md)。

## License

MIT ⚓ Keel Contributors
