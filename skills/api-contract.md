# Skill：前后端接口契约规范

## 核心目标
用标准化的接口契约作为前后端唯一对齐标准，实现完全解耦的并行开发，确保联调零阻塞。

## 文档方向
- **推荐方案：文档优先（API_SPEC定义→生成前后端类型桩）**：先写API_SPEC，用OpenAPI Generator等工具从契约生成前端TypeScript类型和后端接口桩代码，代码必须继承/实现生成的类型，不匹配则编译失败，从根源保证一致。
- **已有项目降级方案：代码注释→工具生成文档**：对于存量项目，后端接口代码加JSDoc/JavaDoc风格注释，由工具自动生成API文档，CI对比生成文档与仓库中API_SPEC的一致性。

## 契约编写规范（OpenAPI/Swagger）
推荐使用OpenAPI 3.0规范编写`API_SPEC.md`，必须包含以下部分：

### 1. 基本信息
```yaml
openapi: 3.0.0
info:
  title: 用户模块API
  version: 1.0.0
  description: 用户注册、登录、信息管理相关接口
servers:
  - url: https://api.example.com/v1
    description: 生产环境
  - url: https://staging-api.example.com/v1
    description: 测试环境
```

### 2. 全局约定
必须在文档开头明确全局约定：
- 鉴权方式：JWT Bearer Token / API Key / Session
- 数据格式：JSON
- 时间戳格式：ISO 8601（如`2026-07-14T10:30:00Z`）或Unix毫秒
- 字段命名规范：统一驼峰（camelCase）或下划线（snake_case），全项目一致
- 分页参数约定：`page`（页码，从1开始）、`pageSize`（每页条数）
- 统一响应结构：
```json
{
  "code": 0,           // 业务错误码，0表示成功
  "message": "success", // 提示信息
  "data": {},          // 响应数据
  "traceId": "xxx"     // 链路追踪ID
}
```

### 3. 接口定义
每个接口必须明确定义：
- 路径与HTTP方法
- 功能描述
- 鉴权要求
- 请求参数（Header、Query、Path、Body）
  - 字段名、类型、是否必填、描述、示例值、取值范围
- 响应结构
  - 成功响应（200）的数据结构
  - 错误响应（400/401/403/404/500等）的错误码与提示
- 字段校验规则（长度、格式、正则等）

接口定义示例：
```yaml
paths:
  /users/login:
    post:
      summary: 用户登录
      description: 手机号/邮箱+密码登录，返回JWT Token
      security: []  # 不需要鉴权
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              required: [account, password]
              properties:
                account:
                  type: string
                  description: 手机号或邮箱
                  example: "13800138000"
                password:
                  type: string
                  description: 密码（MD5加密后传输）
                  minLength: 6
                  maxLength: 32
      responses:
        '200':
          description: 登录成功
          content:
            application/json:
              schema:
                type: object
                properties:
                  code: { type: integer, example: 0 }
                  data:
                    type: object
                    properties:
                      token: { type: string, description: "JWT Token" }
                      expiresIn: { type: integer, description: "过期时间（秒）", example: 7200 }
                      userInfo: { $ref: '#/components/schemas/User' }
        '400':
          description: 参数错误
          content:
            application/json:
              schema:
                type: object
                properties:
                  code: { type: integer, example: 40001 }
                  message: { type: string, example: "账号或密码错误" }
```

### 4. 数据模型（Schema）
- 公共数据模型统一定义在`components/schemas`下，可复用
- 每个字段必须有类型、描述、示例
- 关联关系要明确（如用户对象包含订单列表）
- 枚举值必须列出所有可选值与含义

### 5. 错误码规范
错误码分段管理，全局唯一：
| 错误码段 | 含义 |
|----------|------|
| 0 | 成功 |
| 40000-40099 | 通用参数错误 |
| 40100-40199 | 鉴权相关错误 |
| 40300-40399 | 权限相关错误 |
| 40400-40499 | 资源不存在 |
| 50000-50099 | 服务端错误 |
| 100000+ | 业务模块自定义错误 |

## 契约评审Checklist
- [ ] 所有接口路径、方法定义清晰
- [ ] 所有字段都有类型、是否必填、描述
- [ ] 请求/响应示例完整，可直接用于Mock
- [ ] 错误码定义完整，覆盖所有异常场景
- [ ] 全局约定（命名、时间、分页、响应结构）已明确
- [ ] 鉴权要求每个接口都标注
- [ ] 枚举值全部列出
- [ ] 前后端双方评审通过并签字确认

## 契约变更流程
1. 变更提出方说明变更原因、影响范围
2. 评估对已完成代码、工期、其他模块的影响
3. 前后端测试共同评审
4. 评审通过后先更新API_SPEC.md
5. 更新Mock数据
6. 通知所有相关方
7. 各方同步修改代码
8. 变更记录写入CHANGELOG

## 工具推荐
- 契约编写：Swagger Editor / Apifox
- Mock服务：Apifox Mock / json-server / Mock.js
- 契约校验：OpenAPI Generator / Dredd
- 类型生成：openapi-typescript（生成TS类型）
