# Keel 操作手册：从安装到精通

> 版本：v1.8 | 更新时间：2026-07-17

这份手册手把手教你怎么把Keel用起来。从安装、配置到日常开发、进阶使用，覆盖所有场景。

---

## 目录

1. [安装：30秒把Keel放进项目](#1-安装30秒把keel放进项目)
2. [不同AI工具的适配方法](#2-不同ai工具的适配方法)
3. [第一次使用：初始化项目](#3-第一次使用初始化项目)
4. [日常开发流程（最常用）](#4-日常开发流程最常用)
5. [常见场景操作指南](#5-常见场景操作指南)
6. [角色Skill怎么用](#6-角色skill怎么用)
7. [进阶功能（按需启用）](#7-进阶功能按需启用)
8. [GitHub发布指南](#8-github发布指南)
9. [常见问题FAQ](#9-常见问题faq)

---

## 1. 安装：30秒把Keel放进项目

下载最新Release，解压后把所有文件放到**你的项目根目录**：

```
你的项目/
├── AGENTS.md              ← AI自动读这个（149行轻量控制面）
├── README.md              ← 项目介绍（按需修改）
├── LICENSE                ← MIT协议
├── ACKNOWLEDGMENTS.md     ← 致谢清单
├── GETTING_STARTED.md     ← 本手册
├── .gitignore             ← Git忽略规则
├── PROJECT_SPEC.md        ← 从模板创建
├── DIR_META.md            ← 从模板创建（根目录）
├── ERROR.md               ← 从模板创建
├── skills/                ← Keel技能包
│   ├── 01-requirement-spec.md
│   ├── 02-concurrent-dev.md
│   └── ...（所有skill文件）
├── templates/             ← 可复制的模板
│   └── ...
└── src/                   ← 你已有的代码
```

**关键**：`AGENTS.md`必须在项目根目录，AI工具默认读根目录的规则文件。

---

## 2. 不同AI工具的适配方法

| 你用的工具 | 自动读取的文件 | 你需要做的 |
|-----------|---------------|-----------|
| **Trae** | `AGENTS.md`（根目录） | 不用额外操作，直接用 ✅ |
| **Cursor** | `.cursorrules`（新版也支持AGENTS.md） | 两种方式：①新版直接放AGENTS.md；②旧版把AGENTS.md内容复制到`.cursorrules` |
| **Windsurf** | `.windsurfrules` | 把AGENTS.md内容复制到`.windsurfrules` |
| **Claude Code** | `CLAUDE.md` | 执行 `cp AGENTS.md CLAUDE.md` |
| **GitHub Copilot** | `.github/copilot-instructions.md` | 把AGENTS.md内容复制到该文件 |
| **网页版ChatGPT/Claude** | 不会自动读任何文件 | 每次新对话手动把AGENTS.md内容贴进去 |

### 验证是否生效

安装完后，跟AI说："你好，请告诉我你现在遵守的开发规范有哪些铁律？"

如果AI能说出9条铁律，说明生效了。

---

## 3. 第一次使用：初始化项目

安装完AGENTS.md后，第一次跟AI说：

> "按照Keel规范初始化这个项目。先读templates里的模板，帮我创建PROJECT_SPEC.md和根目录的DIR_META.md，以及ERROR.md。"

AI会自动：
1. 从 `templates/` 读取模板
2. 问你几个问题（项目是什么、技术栈、目录结构）
3. 创建好 `PROJECT_SPEC.md`、`DIR_META.md`、`ERROR.md`

### PROJECT_SPEC.md 填写要点
- **项目名称与简介**：一句话说清楚是什么
- **技术栈**：前端/后端/数据库/部署方式
- **核心功能**：列3-5个MVP功能
- **全局约束**：命名规范、API格式、代码风格
- **目录结构说明**：哪些目录是干嘛的

### DIR_META.md 填写要点
标注每个目录的用途和**可修改性级别**：
- 【禁止修改】：架构核心、自动生成的代码、node_modules等
- 【谨慎修改】：公共组件、工具函数、配置文件
- 【可修改】：业务代码，在规格范围内随便改

### ERROR.md
刚开始是空的，开发过程中遇到坑，AI会自动往里记录。

---

## 4. 日常开发流程（最常用）

### 场景1：开发新功能

**你说**："帮我做一个用户登录功能"

**AI自动做**：
1. Step 0识别为"新需求"场景
2. 先不写代码，跟你确认需求范围和验收标准
3. 写简化MODULE_SPEC，你确认后开始写
4. 四层加载读相关文件
5. 写完跑lint/test/build验证
6. 强制自审（铁律9）：Reviewer视角过一遍，UI类过Designer清单
7. 输出：完整代码+文档同步+测试结果+提交信息

### 场景2：修Bug

**你说**："登录接口报错500了"

**AI自动做**：
1. 识别为"Debug"场景
2. 走"理解→复现→假设→验证→源头修复"流程
3. 找根因在共享层修复，不头痛医头
4. 修复后补规格/更新ERROR.md

### 场景3：做UI页面

**你说**："帮我做一个项目列表页"

**AI自动做**：
1. 识别为"UI开发"场景
2. 读`skills/ui-ux.md`，确认页面类型
3. 遵守Design Tokens，避免AI Slop
4. 写完过Designer检查清单
5. 专业工具类产品自动考虑右键/Cmd+K/快捷键等

### 场景4：代码审查

**你说**："帮我review一下刚写的登录代码"

**AI自动做**：
1. 识别为"代码审查"场景
2. 切Reviewer角色
3. 按Bug优先→坏味道→安全→文档同步顺序审查

### 铁律记忆口诀

```
无规格，不动手；四层加载，不读全仓；
改完必验，三败则停；写完自审，再交用户；
改码必改文档，提交带docs-scope。
```

---

## 5. 常见场景操作指南

### 项目已经开发一半了，怎么引入Keel？

不用从头补文档：
1. 复制AGENTS.md到根目录
2. 写简化版PROJECT_SPEC.md（写清楚项目是什么、技术栈、核心约束就行）
3. 写根目录DIR_META.md（标注哪些不能随便改）
4. 创建空的ERROR.md
5. **新代码按Keel走，老代码随做随补**

### 需求很模糊怎么办？

跟AI说："我想做一个XX功能，但还没想清楚，帮我梳理一下需求"

AI会切CEO角色帮你理清：解决什么问题、MVP范围、验收标准。**千万不要在需求模糊时让AI直接写代码**。

### 前后端联调接口对不上？

跟AI说："定义一下XX模块的API接口契约"——AI会读`skills/api-contract.md`帮你定义请求/响应格式、错误码，前后端基于契约并行开发。

### AI不遵守规则怎么办？

1. 第一次对话明确说："读AGENTS.md，严格按Keel规范来"
2. 铁律被违反时直接指出："你违反了铁律1（无规格不开发），先写规格"
3. 用好DIR_META.md的可修改性标注
4. 用强模型（GPT-4o/Claude 3.5/3.7 Sonnet），弱模型指令遵循能力差

### 小改动（改按钮文字/修typo）要不要走完整流程？

不需要。小改动（<50行、单文件、不改接口、不涉及核心路径）可简化，但铁律7（验证）、铁律4（文档同步）、铁律9（自审）仍然要守。

### 做专业工具类产品（IDE/编辑器/写作工具）？

跟AI说："这是专业工具类产品，读ui-ux.md中专业工具交互范式部分"。AI会自动考虑右键菜单、Cmd+K、快捷键、拖拽、乐观UI、撤销重做等。

### 做营销页/落地页也要遵守专业工具交互规范吗？

不需要。右键菜单/Cmd+K等是给专业工具用的，营销页重点是视觉层次和CTA明确。

---

## 6. 角色Skill怎么用

角色是"思维模式切换"，直接告诉AI用哪个角色：

| 你说的话 | 角色 | 做什么 |
|---------|------|--------|
| "用CEO角色帮我评估这个需求值不值得做" | CEO | 质疑需求、定义价值、划定MVP |
| "做UI前先用Designer思维规划一下" | Designer | 确认页面类型、核心动作、布局范式 |
| "用Designer角色审查这个页面" | Designer | 发现AI Slop、交互问题、状态缺失 |
| "用Reviewer角色帮我review这段代码" | Reviewer | 找Bug、坏味道、安全问题 |
| "用QA角色帮我测试这个功能" | QA | 探索边界场景、验证验收标准 |
| "用Security角色审计登录接口" | Security | 检查OWASP Top 10 |
| "用Release角色帮我准备发布" | Release | Go/No-Go门禁、回滚方案 |

如果AI说"这里不好看"但不说怎么改，直接说："用Before/After格式给我具体建议"。

---

## 7. 进阶功能（按需启用）

### .sdd/ 跨会话记忆

什么时候用：AI经常"失忆"，开新对话就忘了之前在做什么。

在项目根目录创建`.sdd/`：
```
.sdd/
├── anchor-state.md
└── memory/
    ├── LEARNINGS.md
    └── daily/
```

AI会自动在会话开始时恢复状态、上下文快满时保存锚点。

### changes/ MDU归档

什么时候用：一个功能跨多个模块、需要追溯"当初为什么这么设计"。

每个大功能建目录：
```
changes/add-user-auth/
├── proposal.md
├── design.md
├── tasks.md
└── archive.md
```

小改动不用建。个人开发者大部分时候不用。

### Hooks自动化

需要工具/框架支持（如Claude Code hooks、自定义CLI）。普通对话场景下Hooks是"AI应该记得做的检查清单"。

### CI门禁

团队2人以上推荐加CI校验：提交信息格式、代码变更必须带文档变更、Lint+Test必须通过。

---

## 8. GitHub发布指南

### 必要文件
确保根目录有：README.md、LICENSE、.gitignore、AGENTS.md、ACKNOWLEDGMENTS.md

### 关于版权

Keel借鉴了很多开源项目，核心来源已在ACKNOWLEDGMENTS.md中列出：
- **Ponytail (DietrichGebert/ponytail)**：YAGNI七阶梯、懒人编码哲学
- GitHub Spec Kit、OpenSpec、Kiro：规格驱动范式
- gstack：角色化Skill思想
- Uncodixfy、Hallmark、ux-skill：Anti AI Slop
- VSCode/Figma/Linear/Notion：专业工具交互范式

MIT协议允许自由使用、修改、分发，包括私有项目和商业项目。

### Git操作
```bash
git init
git add .
git commit -m "feat: initial release of Keel v1.8

docs-scope: AGENTS.md, SPECIFICATION.md, skills/, templates/"
git remote add origin https://github.com/[用户名]/keel.git
git branch -M main
git push -u origin main
git tag v1.8.0
git push --tags
```

---

## 9. 常见问题FAQ

### AGENTS.md有149行，会不会占太多上下文？
不会。149行约500-600 token，只占200K上下文的0.3%。Keel刻意控制AGENTS.md在150行左右，详细规则在skills/按需加载。

### skills/里15个文件，AI会全部读吗？
不会。AI根据场景只读需要的。做UI读ui-ux.md，Debug读error-tracking.md，其他不读。这就是四层渐进加载。

### 不用AI开发，Keel有用吗？
有用。规格驱动、接口契约、闭环交付、错误沉淀这些方法论对纯人类团队也有价值。

### 个人开发者要用全套吗？
不用贪多。推荐先落地：AGENTS.md（9条铁律）、简化PROJECT_SPEC、接口契约、ERROR.md、Git规范。其他按需加。

### 文档写起来太费时间？
前期花时间写规格，省的是后期返工和联调时间。规格不用写很长，简单功能3-5行说清楚关键点就够了。

### AI说"先写代码后面补文档"怎么办？
礼貌但坚定地拒绝。铁律1是无规格不开发。说："先别写代码，帮我写一个简化规格，我确认后再写。"

---

## 快速参考卡

### 9条铁律
1. 无规格不开发
2. 四层渐进加载
3. 遵守可修改性分级
4. 代码文档同步
5. 输出完整可运行代码
6. 先查ERROR.md再动手
7. 修改后必须验证（三败则停）
8. 危险操作必须确认
9. 写完必须自审

### 场景速查
| 你要做什么 | 跟AI说什么 |
|-----------|-----------|
| 新功能 | "帮我做XX，先确认需求范围" |
| 写代码 | "实现XX，按Keel规范来" |
| 修Bug | "XX报错了，找根因修复" |
| 做UI | "做XX页面，读ui-ux.md" |
| Review | "用Reviewer角色审查这段代码" |
| 测试 | "用QA角色测试这个功能" |
| 安全 | "用Security审计XX" |
| 发布 | "用Release角色准备发布" |

---

**祝你用Keel开发愉快！有问题或改进建议，欢迎提Issue。⚓**
