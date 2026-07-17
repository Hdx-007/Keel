# Skill：Git 管理与变更记录规范

## 核心目标
基于Monorepo单体仓库模式，统一前后端版本、文档、流程，通过分支策略和提交规范保证主干稳定、变更可追溯。

## Monorepo目录结构
```
project-root/
├── .github/              # GitHub Actions/GitLab CI配置
├── PROJECT_SPEC.md       # L0 项目总规格
├── ERROR.md              # 错误沉淀库
├── CHANGELOG.md          # 版本变更记录
├── README.md             # 项目说明
├── DIR_META.md           # 根目录元数据
├── docs/                 # 全局文档
│   ├── adr/              # 架构决策记录
│   ├── deployment/       # 部署文档
│   └── api/              # 全局API文档
├── frontend/             # 前端代码
│   ├── package.json
│   ├── DIR_META.md
│   └── src/
├── backend/              # 后端代码
│   ├── package.json / go.mod / pom.xml
│   ├── DIR_META.md
│   └── src/
├── deploy/               # 部署配置
│   ├── docker/
│   ├── nginx/
│   └── DIR_META.md
└── scripts/              # 工具脚本（CI、校验等）
```

## 分支模型（Trunk-Based Development）
```
main (保护分支，始终可发布)
  ↑
  ├── feature/user-login (特性分支，开发完即删)
  ├── feature/backend-user-api
  ├── fix/login-token-expire (修复分支，修复完即删)
  └── hotfix/payment-callback (紧急修复分支)
```

### 分支命名规范
| 分支类型 | 命名格式 | 示例 | 生命周期 |
|----------|----------|------|----------|
| 主干 | `main` / `trunk` | - | 永久 |
| 特性分支 | `feature/[模块]-[功能名]` | `feature/user-login` | 建议≤3天，最长不超过1周，合并即删 |
| 修复分支 | `fix/[问题简述]` | `fix/login-token-expire` | 修复完即删 |
| 热修复分支 | `hotfix/[问题简述]` | `hotfix/payment-callback` | 线上修复完即删 |
| 发布分支 | `release/vX.Y.Z` | `release/v1.2.0` | 发布完即删（可选） |

### 分支规则
- 禁止长期存在的开发分支，所有分支必须有明确的合并/删除时间
- 特性分支从main拉出，开发完成合回main
- 禁止直接push到main，必须通过PR
- 分支开发期间定期（每天至少1次）rebase main，避免大幅冲突
- 分支合并后立即删除本地和远程分支
- **短生命周期原则**：一个特性分支的生命周期建议≤3天，最长不超过1周。长时间不合并的分支意味着任务拆分有问题。

## PR（Pull Request）规范

### 小PR高频合入原则
- PR粒度严格对齐MDU（最小交付单元），一个PR对应一个可独立验证的功能点
- 建议单个PR控制在500行以内，超过800行考虑拆分（但不要为拆而拆）
- **禁止大段重写/推倒重来**：存量代码优先增量修改，先读懂边界再动手
- 高频合入（每天至少合入一次main），避免"攒大招"式的开发

### PR描述必填项
- **做了什么**：简要说明本次变更内容
- **为什么做**：关联的需求/Issue/问题背景
- **怎么验证**：验证步骤、测试用例、截图（UI变更）
- **影响范围**：哪些模块/接口/文件受影响
- **docs-scope**：列出本次变更更新的文档路径

## Conventional Commits 提交规范

### 提交格式
```
&lt;type&gt;(&lt;scope&gt;): &lt;subject&gt;

&lt;body&gt;（可选，详细描述变更内容和原因）

&lt;footer&gt;（可选，BREAKING CHANGE、关闭Issue等）

docs-scope: &lt;影响的文档路径，逗号分隔&gt;
```

### Type 类型说明
| type | 说明 | 示例 |
|------|------|------|
| `feat` | 新功能 | feat(frontend): 新增登录页面 |
| `fix` | bug修复 | fix(backend): 修复分页查询错误 |
| `docs` | 文档变更 | docs(api): 更新登录接口字段说明 |
| `style` | 代码格式（不影响代码运行） | style: 统一缩进为2空格 |
| `refactor` | 重构（不是新增功能，也不是修bug） | refactor(user): 重构用户服务层 |
| `perf` | 性能优化 | perf: 优化用户列表查询速度 |
| `test` | 测试相关 | test(user): 增加登录接口单测 |
| `chore` | 构建/工具/依赖 | chore: 升级React到18.2 |
| `ci` | CI配置变更 | ci: 增加文档一致性校验脚本 |

### Scope 说明
scope用于说明影响范围，一般是模块名：
- 前端：`frontend`、`frontend/user`、`frontend/components`
- 后端：`backend`、`backend/user`、`backend/db`
- 全局：`docs`、`ci`、`deploy`

### Subject 规范
- 用中文清晰描述变更内容
- 结尾不加句号
- 长度不超过50字符
- 动词开头（新增、修复、更新、重构、优化、删除等）

### 提交示例
```
feat(backend): 新增用户手机号登录接口

- 支持手机号+验证码登录方式
- 新增验证码发送与校验接口
- 增加登录日志记录
- 密码错误次数限制

BREAKING CHANGE: 登录接口响应结构调整，增加needVerify字段
Closes #123

docs-scope: docs/api/user.md, backend/user/MODULE_SPEC.md, backend/user/DIR_META.md
```

### 提交禁令
- ❌ 禁止 `update`、`fix bug`、`优化` 这类无意义的描述
- ❌ 禁止一个提交包含多个不相关功能
- ❌ 禁止提交废代码、注释掉的代码、console.log/print调试代码
- ❌ 禁止提交敏感配置、密钥、密码
- ❌ 禁止提交依赖锁文件以外的大文件（超过10MB）

## PR（Pull Request）规范

### PR标题格式
与提交信息格式一致：`&lt;type&gt;(&lt;scope&gt;): &lt;subject&gt;`
示例：`feat(user): 手机号登录功能`

### PR描述模板
```markdown
## 变更说明
（描述本次变更的内容、原因、影响范围）

## 变更类型
- [ ] 新功能
- [ ] Bug修复
- [ ] 重构
- [ ] 文档更新
- [ ] 性能优化
- [ ] 其他

## 测试情况
- [ ] 单元测试全部通过
- [ ] 测试环境验证通过
- [ ] 新增了测试用例覆盖变更

## 文档同步
- [ ] 对应文档已更新
- docs-scope: （列出更新的文档路径）

## Checklist
- [ ] 代码符合规范
- [ ] 没有提交敏感信息
- [ ] 数据库迁移已验证（如有）
- [ ] 已通知相关方（如有接口变更）

## 关联Issue
Closes #XXX
```

### PR Review规则
- 至少1个审批人通过才能合入
- 核心模块、核心接口变更需要2人审批
- Review意见必须全部处理（修改或回复说明）
- CI检查必须全部通过（代码检查、单测、构建、文档校验）
- 不允许「先合进去之后再改」

### PR合并策略
- 推荐使用**Squash and Merge**：把多个提交压缩成一个干净的提交合入main
- 特性分支保持粒度小，一个PR对应一个MDU
- 合并后删除特性分支

## CHANGELOG 维护规范

### 格式
```markdown
# Changelog

## [v1.2.0] - 2026-07-14
### Added
- 用户手机号登录功能
- 验证码发送接口

### Fixed
- 修复用户列表分页查询错误
- 修复Token过期时间不正确问题

### Changed
- 登录接口响应结构调整（BREAKING CHANGE）
- 用户信息接口返回字段新增avatar

### Docs
- 更新API文档v1.2
- 补充部署文档配置说明
```

### 分类说明
- **Added**：新功能
- **Fixed**：bug修复
- **Changed**：现有功能变更（包含不兼容变更需标注BREAKING CHANGE）
- **Deprecated**：即将废弃的功能
- **Removed**：已删除的功能
- **Security**：安全相关修复
- **Docs**：文档变更

### 维护方式
- 推荐用工具自动生成（如standard-version、release-please）
- 从Conventional Commits提交信息自动提取分类
- 版本发布时自动更新CHANGELOG.md和版本tag

## ADR（架构决策记录）规范
重大架构决策、技术选型变更必须记录ADR，永久留存：

### 文件命名
`docs/adr/ADR-001-选择Monorepo架构.md`（编号+标题）

### ADR模板
```markdown
# ADR-001: 选择Monorepo单体仓库架构

## 状态
已采纳

## 上下文
（描述我们面临什么问题、有哪些可选方案、各方案的优缺点）
- 方案1：多仓库（前后端分开）
- 方案2：Monorepo单体仓库

## 决策
选择Monorepo单体仓库架构

## 理由
- 前后端代码、文档统一版本，方便管理
- 接口变更可以在同一个PR中同时修改前后端代码
- 统一CI/CD流程，工具链共享
- 适合小型团队，减少仓库管理成本

## 后果
- 仓库体积会变大
- 需要做好目录权限控制（如需）
- CI需要分模块构建，避免全量构建
```
