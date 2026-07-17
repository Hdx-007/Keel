# Acknowledgments 致谢与参考来源

Keel（龙骨）在形成过程中大量借鉴了开源社区的优秀实践和思想。没有这些先驱项目的探索，就没有Keel的今天。在此向所有相关开源项目和作者致以诚挚的感谢。

## 核心方法论参考

### 规格驱动开发范式
| 项目 | 作者/组织 | 借鉴内容 |
|------|----------|---------|
| [GitHub Spec Kit](https://github.com/github/spec-kit) | GitHub | 规格优先开发理念、Spec作为代码和开发之间的契约层 |
| [OpenSpec](https://github.com/Fission-AI/OpenSpec) | Fission AI | changes/目录归档机制、提案→设计→任务的MDU拆分思路 |
| [Kiro](https://kiro.dev) | Amazon | Spec驱动开发的AI原生工具链实践 |
| [gstack](https://github.com/gstack) | gstack社区 | 角色化Skill思想（CEO/Designer/Reviewer/QA等专业化分工） |

### AI编码哲学（YAGNI与懒人编码）
| 项目 | 作者 | 借鉴内容 |
|------|------|---------|
| [Ponytail](https://github.com/DietrichGebert/ponytail) | Dietrich Gebert | **核心编码哲学来源**：YAGNI七阶梯（需要做吗→已有吗→标准库能做吗→...才写最少代码）、"Deletion over addition. Boring over clever."、"最好的代码是从未被写出来的代码"等核心原则。 |
| [Cursor Rules 社区](https://cursor.directory) | 社区 | AGENTS.md作为项目级AI控制面的惯例 |
| [Claude Code](https://docs.anthropic.com/en/docs/claude-code) | Anthropic | CLAUDE.md约定、Hooks生命周期机制思想 |

### Harness与反馈闭环
| 项目 | 作者 | 借鉴内容 |
|------|------|---------|
| [claude-harness](https://github.com/teknium1/claude-harness) | Teknium | Harness Engineering核心概念：AI在"目标-状态-执行-反馈"四象限中自我修正 |
| [Atomic CLI](https://github.com/atticusofsparta/atomic) | Atticus | 自我修复CLI、测试反馈驱动开发 |
| [self-healing-cli](https://github.com/anthropics/self-healing-cli) | Anthropic | 代码生成→验证→错误修复的自动闭环 |
| Horthy et al. (2026) | 学术研究 | Smart Zone上下文利用率研究（0-40%为最优区间） |

## UI/UX 与 Anti AI Slop 参考
| 项目 | 作者 | 借鉴内容 |
|------|------|---------|
| [Uncodixfy](https://github.com/cyxzdev/Uncodixfy) | cyxzdev | 最简洁的Anti AI-UI规则集：直接告诉AI什么不要做 |
| [Hallmark](https://github.com/Nutlope/hallmark) | Nutlope | 65道Slop闸门检测、组件8状态标准（default/hover/focus/active/disabled/loading/error/success）、设计DNA提取方法 |
| [ux-skill](https://github.com/Laith0003/ux-skill) | Laith0003 | 145+条反模式规则、品牌规范、设计合成器思想 |
| [antigravity-awesome-skills](https://github.com/sickn33/antigravity-awesome-skills) | sickn33 | 社区Skill合集整理思路 |
| [awesome-cursorrules](https://github.com/PatrickJS/awesome-cursorrules) | PatrickJS | 170+ Cursor Rules合集参考 |
| [Blake Crosley设计分析](https://blakecrosley.com) | Blake Crosley | Linear/Figma/VSCode专业工具交互范式深度分析（右键菜单、命令面板、快捷键体系、乐观UI等） |

## 产品交互设计范式
Keel中关于专业工具交互的规范（右键菜单、Cmd+K命令面板、键盘快捷键三层体系、拖拽反馈、乐观UI、内联编辑等）参考了以下顶级产品的实际交互模式：
- **VSCode** - 编辑器类产品的右键菜单、命令面板、快捷键体系标准
- **Figma** - 设计工具的多选批量操作、拖拽反馈、画布交互
- **Linear** - 项目管理工具的乐观UI、键盘优先导航、优雅过渡动画
- **Notion** - 块编辑器的内联编辑、悬停揭示控件、斜杠命令菜单

## 工程化最佳实践
| 来源 | 借鉴内容 |
|------|---------|
| [Conventional Commits](https://www.conventionalcommits.org) | Git提交信息规范 |
| [Semantic Versioning](https://semver.org) | 语义化版本号规范 |
| [Keep a Changelog](https://keepachangelog.com) | CHANGELOG.md格式规范 |
| [OWASP Top 10](https://owasp.org/www-project-top-ten/) | Web应用安全检查清单 |
| [ADR (Architecture Decision Records)](https://adr.github.io) | 架构决策记录模板思想 |

## 声明
Keel是一个站在巨人肩膀上的项目，所有核心思想都源于开源社区的集体智慧。Keel本身也是开源的（MIT协议），欢迎自由使用、修改和分发。

如果你的项目被借鉴但没有被列出，欢迎提Issue或PR补充。
