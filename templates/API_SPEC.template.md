# API_SPEC - [模块名称] 接口文档

&gt; OpenAPI 3.0 规范 | 版本：v1.0 | 最后更新：YYYY-MM-DD

---

```yaml
openapi: 3.0.0
info:
  title: [模块名] API
  version: 1.0.0
  description: [模块功能描述]

servers:
  - url: https://api.example.com/v1
    description: 生产环境
  - url: https://staging-api.example.com/v1
    description: 测试环境

# 全局安全定义
security:
  - bearerAuth: []

paths:

  # ==================== 接口示例开始 ====================
  /users/login:
    post:
      summary: 用户登录
      description: 手机号/邮箱+密码登录，返回JWT Token
      security: []  # 不需要鉴权的接口
      tags: [用户认证]
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
                  code:
                    type: integer
                    example: 0
                  message:
                    type: string
                    example: success
                  data:
                    type: object
                    properties:
                      token:
                        type: string
                        description: JWT Token
                        example: "eyJhbGciOiJIUzI1NiIs..."
                      expiresIn:
                        type: integer
                        description: 过期时间（秒）
                        example: 7200
                      userInfo:
                        $ref: '#/components/schemas/User'
        '400':
          description: 参数错误/账号密码错误
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/ErrorResponse'
              examples:
                账号或密码错误:
                  value:
                    code: 40001
                    message: 账号或密码错误
                参数校验失败:
                  value:
                    code: 40000
                    message: 参数错误：password长度不能小于6
  # ==================== 接口示例结束 ====================

  # 在下面继续定义其他接口...

components:
  # 安全方案
  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT

  # 数据模型
  schemas:
    # 统一错误响应
    ErrorResponse:
      type: object
      properties:
        code:
          type: integer
          description: 错误码
        message:
          type: string
          description: 错误提示
        traceId:
          type: string
          description: 链路追踪ID

    # 用户模型示例
    User:
      type: object
      description: 用户信息
      properties:
        id:
          type: integer
          description: 用户ID
          example: 10001
        phone:
          type: string
          description: 手机号
          example: "138****8000"
        email:
          type: string
          description: 邮箱
          example: "u***@example.com"
        nickname:
          type: string
          description: 昵称
          example: "张三"
        avatar:
          type: string
          description: 头像URL
          example: "https://example.com/avatar.jpg"
        createdAt:
          type: string
          format: date-time
          description: 创建时间
          example: "2026-01-01T10:00:00Z"
      required: [id, phone]

    # 在下面继续定义其他数据模型...
```

---

## 接口变更记录

| 版本 | 日期 | 变更内容 | 变更人 |
|------|------|----------|--------|
| v1.0 | YYYY-MM-DD | 初始版本 |  |
