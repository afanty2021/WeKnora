[根目录](../../CLAUDE.md) > [internal](../CLAUDE.md) > **middleware**

# middleware 模块

## 模块职责

`middleware` 模块提供 HTTP 请求处理中间件，负责横切关注点处理，包括认证、日志记录、错误处理、链路追踪和异常恢复等。

## 入口与启动

### 中间件列表
- `auth.go`: JWT 和 API Key 认证中间件
- `logger.go`: 请求日志记录中间件
- `trace.go`: OpenTelemetry 链路追踪中间件
- `recovery.go`: 异常恢复中间件
- `error_handler.go`: 统一错误处理中间件

## 对外接口

### 1. Auth 认证中间件

**功能**: 支持 JWT Token 和 API Key 两种认证方式，同时支持跨租户访问控制。

```go
func Auth(
    tenantService interfaces.TenantService,
    userService interfaces.UserService,
    cfg *config.Config,
) gin.HandlerFunc
```

**认证流程**:
1. 检查请求是否在 `noAuthAPI` 白名单中
2. 尝试 JWT Bearer Token 认证
3. 尝试 X-API-Key 认证（兼容模式）
4. 支持通过 X-Tenant-ID 头进行跨租户访问

**无需认证的 API**:
- `/health`: 健康检查
- `/api/v1/auth/register`: 用户注册
- `/api/v1/auth/login`: 用户登录
- `/api/v1/auth/refresh`: 刷新令牌

**跨租户访问条件**:
1. `cfg.Tenant.EnableCrossTenantAccess` 为 true
2. 用户具有 `CanAccessAllTenants` 权限
3. 目标租户存在

### 2. RequestID 请求 ID 中间件

**功能**: 为每个请求生成唯一 ID，用于日志追踪和链路关联。

```go
func RequestID() gin.HandlerFunc
```

**特性**:
- 从 `X-Request-ID` 头获取或生成新的 UUID
- 设置响应头 `X-Request-ID`
- 将 request_id 存入 Gin context 和 Go context

### 3. Logger 日志中间件

**功能**: 记录请求和响应的详细信息。

```go
func Logger() gin.HandlerFunc
```

**记录内容**:
- 请求方法、路径、查询参数
- 请求体（最大 10KB，自动脱敏）
- 响应状态码、大小、延迟
- 响应体（自动脱敏）

**敏感字段脱敏**:
- password, token, access_token, refresh_token
- authorization, api_key, secret, apikey, apisecret

### 4. TracingMiddleware 追踪中间件

**功能**: 基于 OpenTelemetry 的分布式追踪。

```go
func TracingMiddleware() gin.HandlerFunc
```

**记录内容**:
- HTTP 请求/响应头（排除 authorization 和 cookie）
- 请求体（POST/PUT/PATCH）
- 查询参数
- 响应体
- 状态码（>= 400 时标记为错误）

### 5. Recovery 恢复中间件

**功能**: 捕获 panic，记录堆栈信息，返回 500 错误。

```go
func Recovery() gin.HandlerFunc
```

### 6. ErrorHandler 错误处理中间件

**功能**: 统一处理应用错误，返回标准格式的错误响应。

```go
func ErrorHandler() gin.HandlerFunc
```

**错误响应格式**:
```json
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "错误描述",
    "details": {}
  }
}
```

## 关键依赖与配置

### 依赖模块
- `github.com/Tencent/WeKnora/internal/config`: 配置管理
- `github.com/Tencent/WeKnora/internal/types`: 类型定义
- `github.com/Tencent/WeKnora/internal/types/interfaces`: 接口定义
- `github.com/Tencent/WeKnora/internal/logger`: 日志工具
- `github.com/Tencent/WeKnora/internal/tracing`: 追踪工具
- `github.com/Tencent/WeKnora/internal/errors`: 错误定义

### 外部依赖
- `github.com/gin-gonic/gin`: Web 框架
- `go.opentelemetry.io/otel`: OpenTelemetry SDK

## 数据模型

### 上下文存储
中间件将以下信息存入 Gin context:
- `tenant_id`: 租户 ID (uint64)
- `tenant_info`: 租户信息 (*types.Tenant)
- `user`: 用户信息 (*types.User)
- `request_id`: 请求 ID (string)
- `logger`: 请求日志记录器 (*logger.Entry)

## 中间件执行顺序

在 `internal/router/router.go` 中的注册顺序：
1. **CORS** (gin-contrib/cors) - 跨域处理
2. **RequestID** - 生成请求 ID
3. **Logger** - 日志记录
4. **Recovery** - 异常恢复
5. **ErrorHandler** - 错误处理
6. **Auth** - 认证（需在 Recovery 之后）
7. **TracingMiddleware** - 链路追踪

## 测试与质量

### 测试覆盖
- 认证流程测试（JWT/API Key）
- 跨租户访问测试
- 日志脱敏测试
- Panic 恢复测试

### 代码质量
- 必须通过 `golangci-lint` 检查
- 敏感信息必须脱敏
- 错误日志必须包含 request_id

## 常见问题 (FAQ)

### Q: 如何添加无需认证的 API？
A: 在 `auth.go` 的 `noAuthAPI` map 中添加路由和允许的 HTTP 方法。

### Q: 如何自定义日志格式？
A: 修改 `logger.go` 中的 `Logger` 中间件，调整字段记录逻辑。

### Q: 如何禁用某个中间件？
A: 在 `router.go` 中注释掉对应的 `r.Use()` 调用。

### Q: X-Tenant-ID 头如何使用？
A: 设置请求头 `X-Tenant-ID: 目标租户ID`，需要用户有跨租户权限。

## 相关文件清单

### 核心文件
- `auth.go`: 认证中间件（JWT + API Key + 跨租户）
- `logger.go`: 日志中间件（请求响应记录）
- `trace.go`: 追踪中间件（OpenTelemetry）
- `recovery.go`: 恢复中间件（Panic 处理）
- `error_handler.go`: 错误处理中间件

## 变更记录 (Changelog)
- 2025-12-31: 创建 middleware 模块文档
