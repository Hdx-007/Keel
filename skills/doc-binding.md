# Skill：代码-文档双向绑定机制

## 核心目标
从流程、工具、校验三个维度确保「改代码必改文档，改文档必影响代码」，彻底杜绝文档与代码脱节。

## Code First原则：文档只写代码推断不出的信息

文档的目标是**降低维护成本、提升知识复用**，而不是成为另一种负担。

### 代码本身能表达的，不要重复写进文档
- 函数签名、入参出参类型（TypeScript/静态类型语言已明确）
- 字段名、枚举值（代码即真相）
- import依赖关系、调用关系（IDE能直接查看）
- 简单逻辑、直接的控制流（读代码就能理解）

### 文档应该重点记录的
- **业务规则**：代码背后的业务逻辑、决策背景、为什么这么写（代码无法表达"为什么"）
- **边界与约束**：哪些值是非法的、为什么禁止某种用法、不变量是什么
- **历史债与坑**：历史决策、踩过的坑、已知的workaround（防止后人重蹈覆辙）
- **约定俗成**：团队里大家都知道但代码里看不出来的约定
- **模块职责边界**：这个模块负责什么、不负责什么、和谁协作
- **关键接口的契约细节**：错误码语义、幂等性要求、并发语义、性能约束

### 一句话原则
**让代码读得懂的别写文档；让文档补代码读不懂的，然后多做索引。** 文档要像维基百科一样有超链接，方便AI和人按图索骥，而不是大段重复代码。

## 存储绑定：文档与代码同仓共存

### 目录结构约定
```
project-root/
├── PROJECT_SPEC.md      # L0 项目总纲（根目录）
├── ERROR.md             # 错误沉淀库（根目录）
├── CHANGELOG.md         # 变更日志（根目录）
├── DIR_META.md          # 根目录元数据
├── docs/
│   ├── adr/             # 架构决策记录
│   ├── deployment/      # 部署文档
│   └── api/             # 全局API文档
├── frontend/
│   ├── DIR_META.md      # 前端目录元数据
│   ├── src/
│   │   ├── user/        # 用户模块
│   │   │   ├── MODULE_SPEC.md  # L1 模块规格
│   │   │   ├── DIR_META.md     # 模块目录元数据
│   │   │   ├── components/
│   │   │   └── pages/
├── backend/
│   ├── DIR_META.md      # 后端目录元数据
│   ├── src/
│   │   ├── user/
│   │   │   ├── MODULE_SPEC.md  # L1 模块规格
│   │   │   ├── DIR_META.md     # 模块目录元数据
│   │   │   ├── controller/
│   │   │   ├── service/
│   │   │   └── entity/
└── deploy/
    └── DIR_META.md
```

### 代码注释绑定接口文档
后端接口代码必须有JSDoc/JavaDoc风格注释，可由工具自动生成API文档：
```javascript
/**
 * 用户登录
 * @param {string} account - 手机号或邮箱
 * @param {string} password - 密码（MD5加密）
 * @returns {Object} 登录结果，包含token和用户信息
 * @throws {40001} 账号或密码错误
 * @throws {40002} 账号已被锁定
 */
async login(account, password) {
  // 实现逻辑
}
```

## 提交绑定：Git提交强制关联文档

### 提交信息规范（Conventional Commits扩展）
```
&lt;type&gt;(&lt;scope&gt;): &lt;subject&gt;

&lt;body&gt;（可选，详细描述）

docs-scope: &lt;影响的文档路径，逗号分隔&gt;
```

type类型：
- `feat`: 新功能
- `fix`: bug修复
- `docs`: 文档变更
- `style`: 代码格式（不影响代码运行）
- `refactor`: 重构（既不是新增功能，也不是修改bug）
- `perf`: 性能优化
- `test`: 测试相关
- `chore`: 构建/工具/依赖相关
- `ci`: CI配置变更

示例：
```
feat(backend): 新增用户手机号登录接口

- 支持手机号+验证码登录
- 新增验证码发送接口
- 登录日志记录

docs-scope: docs/api/user.md, backend/user/MODULE_SPEC.md, backend/user/DIR_META.md
```

### PR门禁规则（CI自动校验）
PR创建时CI自动检查以下规则，不通过则禁止合入：

1. **代码变更必须带文档变更**：
   - 如果PR修改了`.ts`/`.js`/`.py`/`.java`等代码文件，必须同时修改对应路径下的`.md`文档（MODULE_SPEC/DIR_META/API文档）
   - 纯重构（不改变功能）可在PR描述中标注「无文档变更」，需1人以上审核确认

2. **文档变更必须关联代码或计划**：
   - 纯文档PR如果修改了接口/逻辑契约，必须关联对应代码PR或Issue
   - 纯排版/错别字修正可直接合入

3. **核心接口变更三方同步**：
   - 接口变更必须同时更新：后端代码、API文档、前端TypeScript类型定义
   - 缺一方CI拦截

## 自动化校验：CI层面一致性检查

### 1. 接口层双向校验
- 方案A（文档优先）：用OpenAPI Generator从API_SPEC生成前端TS类型和后端接口桩代码，代码必须继承生成的类型，不匹配则编译失败
- 方案B（代码优先）：从后端代码注释自动生成OpenAPI文档，CI对比生成的文档与仓库中的API_SPEC.md，不一致则拦截
- 推荐方案A，契约先行，从根源保证一致

### 2. DIR_META元数据校验
CI脚本检查：
- 每个目录是否有DIR_META.md
- DIR_META.md中列出的核心文件是否存在
- 目录中新增/删除文件后，DIR_META.md是否同步更新
- 「禁止修改」的文件是否被改动（改动需PR中特殊说明）

### 3. 提交信息校验
- 提交信息是否符合Conventional Commits格式
- 是否包含docs-scope字段（纯chore/test/style除外）
- docs-scope中列出的文档路径是否在本次PR中有变更

### 4. 错误记录校验
- 修复BUG的PR必须在ERROR.md中记录或关联已有记录
- 线上BUG修复必须更新ERROR.md

## 文档同步Checklist（每次代码提交前自查）
- [ ] 修改的代码对应MODULE_SPEC.md是否需要更新？
- [ ] 新增/删除文件是否更新了DIR_META.md？
- [ ] 接口变更是否更新了API_SPEC.md？
- [ ] 前端类型定义是否同步更新？
- [ ] 提交信息是否包含docs-scope？
- [ ] 是否有需要记录到ERROR.md的问题？
- [ ] CHANGELOG.md是否需要更新？
