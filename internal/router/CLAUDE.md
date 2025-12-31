[根目录](../../CLAUDE.md) > [internal](../CLAUDE.md) > **router**

# router 模块

## 模块职责

`router` 模块负责 API 路由定义和注册，将 HTTP 请求映射到对应的处理器，同时配置中间件链。

## 入口与启动

### 入口文件
- **主入口**: `router.go`
- **任务路由**: `task.go`

### 路由初始化
路由在 `internal/container/container.go` 中通过依赖注入创建。

## 对外接口

### 路由结构

```
/health                    # 健康检查（无需认证）
/swagger/*                 # API 文档（无需认证，非生产环境）
/api/v1/
├── auth/                  # 认证相关
├── tenants/               # 租户管理
├── knowledge-bases/       # 知识库管理
├── knowledge/             # 知识管理
├── chunks/                # 分块管理
├── sessions/              # 会话管理
├── knowledge-chat/        # 知识问答
├── agent-chat/            # Agent 问答
├── knowledge-search/      # 知识检索
├── messages/              # 消息管理
├── models/                # 模型管理
├── evaluation/            # 评估管理
├── initialization/        # 初始化配置
├── system/                # 系统信息
├── mcp-services/          # MCP 服务
├── web-search/            # 网页搜索
└── agents/                # 自定义 Agent
```

## 路由详细列表

### 认证路由 (RegisterAuthRoutes)
```go
POST   /api/v1/auth/register    # 用户注册
POST   /api/v1/auth/login       # 用户登录
POST   /api/v1/auth/refresh     # 刷新令牌
GET    /api/v1/auth/validate    # 验证令牌
POST   /api/v1/auth/logout      # 登出
GET    /api/v1/auth/me          # 获取当前用户
POST   /api/v1/auth/change-password  # 修改密码
```

### 租户路由 (RegisterTenantRoutes)
```go
GET    /api/v1/tenants/all      # 获取所有租户（跨租户权限）
GET    /api/v1/tenants/search   # 搜索租户（跨租户权限）
POST   /api/v1/tenants          # 创建租户
GET    /api/v1/tenants/:id      # 获取租户详情
PUT    /api/v1/tenants/:id      # 更新租户
DELETE /api/v1/tenants/:id      # 删除租户
GET    /api/v1/tenants          # 获取租户列表
GET    /api/v1/tenants/kv/:key  # 获取 KV 配置
PUT    /api/v1/tenants/kv/:key  # 更新 KV 配置
```

### 知识库路由 (RegisterKnowledgeBaseRoutes)
```go
POST   /api/v1/knowledge-bases              # 创建知识库
GET    /api/v1/knowledge-bases              # 获取知识库列表
GET    /api/v1/knowledge-bases/:id          # 获取知识库详情
PUT    /api/v1/knowledge-bases/:id          # 更新知识库
DELETE /api/v1/knowledge-bases/:id          # 删除知识库
GET    /api/v1/knowledge-bases/:id/hybrid-search  # 混合搜索
POST   /api/v1/knowledge-bases/copy         # 复制知识库
GET    /api/v1/knowledge-bases/copy/progress/:task_id  # 获取复制进度
```

### 知识库标签路由 (RegisterKnowledgeTagRoutes)
```go
GET    /api/v1/knowledge-bases/:id/tags     # 获取标签列表
POST   /api/v1/knowledge-bases/:id/tags     # 创建标签
PUT    /api/v1/knowledge-bases/:id/tags/:tag_id  # 更新标签
DELETE /api/v1/knowledge-bases/:id/tags/:tag_id  # 删除标签
```

### 知识路由 (RegisterKnowledgeRoutes)
```go
# 知识库下的知识
POST   /api/v1/knowledge-bases/:id/knowledge/file      # 从文件创建
POST   /api/v1/knowledge-bases/:id/knowledge/url       # 从 URL 创建
POST   /api/v1/knowledge-bases/:id/knowledge/manual    # 手工录入
GET    /api/v1/knowledge-bases/:id/knowledge           # 获取知识列表

# 独立知识操作
GET    /api/v1/knowledge/batch              # 批量获取知识
GET    /api/v1/knowledge/:id                # 获取知识详情
DELETE /api/v1/knowledge/:id                # 删除知识
PUT    /api/v1/knowledge/:id                # 更新知识
PUT    /api/v1/knowledge/manual/:id         # 更新手工知识
GET    /api/v1/knowledge/:id/download       # 下载知识文件
PUT    /api/v1/knowledge/image/:id/:chunk_id  # 更新图片分块
PUT    /api/v1/knowledge/tags               # 批量更新标签
GET    /api/v1/knowledge/search             # 搜索知识
```

### FAQ 路由 (RegisterFAQRoutes)
```go
GET    /api/v1/knowledge-bases/:id/faq/entries        # 获取 FAQ 列表
GET    /api/v1/knowledge-bases/:id/faq/entries/export # 导出 FAQ
GET    /api/v1/knowledge-bases/:id/faq/entries/:entry_id  # 获取 FAQ 详情
POST   /api/v1/knowledge-bases/:id/faq/entries        # 批量创建/更新 FAQ
POST   /api/v1/knowledge-bases/:id/faq/entry          # 创建单个 FAQ
PUT    /api/v1/knowledge-bases/:id/faq/entries/:entry_id  # 更新 FAQ
PUT    /api/v1/knowledge-bases/:id/faq/entries/fields  # 批量更新字段
PUT    /api/v1/knowledge-bases/:id/faq/entries/tags    # 批量更新标签
DELETE /api/v1/knowledge-bases/:id/faq/entries         # 批量删除 FAQ
POST   /api/v1/knowledge-bases/:id/faq/search         # 搜索 FAQ
GET    /api/v1/faq/import/progress/:task_id            # 获取导入进度
```

### 分块路由 (RegisterChunkRoutes)
```go
GET    /api/v1/chunks/:knowledge_id              # 获取分块列表
GET    /api/v1/chunks/by-id/:id                  # 通过 ID 获取分块
DELETE /api/v1/chunks/:knowledge_id/:id          # 删除分块
DELETE /api/v1/chunks/:knowledge_id              # 删除所有分块
PUT    /api/v1/chunks/:knowledge_id/:id          # 更新分块
DELETE /api/v1/chunks/by-id/:id/questions        # 删除生成的问题
```

### 会话路由 (RegisterSessionRoutes)
```go
POST   /api/v1/sessions                         # 创建会话
GET    /api/v1/sessions/:id                     # 获取会话详情
GET    /api/v1/sessions                         # 获取会话列表
PUT    /api/v1/sessions/:id                     # 更新会话
DELETE /api/v1/sessions/:id                     # 删除会话
POST   /api/v1/sessions/:session_id/generate_title  # 生成标题
POST   /api/v1/sessions/:session_id/stop        # 停止生成
GET    /api/v1/sessions/continue-stream/:session_id  # 继续接收流
```

### 聊天路由 (RegisterChatRoutes)
```go
POST   /api/v1/knowledge-chat/:session_id       # 知识问答
POST   /api/v1/agent-chat/:session_id           # Agent 问答
POST   /api/v1/knowledge-search                 # 知识检索（无需会话）
```

### 消息路由 (RegisterMessageRoutes)
```go
GET    /api/v1/messages/:session_id/load        # 加载更多消息
DELETE /api/v1/messages/:session_id/:id         # 删除消息
```

### 模型路由 (RegisterModelRoutes)
```go
GET    /api/v1/models/providers    # 获取模型厂商列表
POST   /api/v1/models              # 创建模型
GET    /api/v1/models              # 获取模型列表
GET    /api/v1/models/:id          # 获取模型详情
PUT    /api/v1/models/:id          # 更新模型
DELETE /api/v1/models/:id          # 删除模型
```

### 评估路由 (RegisterEvaluationRoutes)
```go
POST   /api/v1/evaluation/         # 创建评估
GET    /api/v1/evaluation/         # 获取评估结果
```

### 初始化路由 (RegisterInitializationRoutes)
```go
GET    /api/v1/initialization/config/:kbId              # 获取配置
POST   /api/v1/initialization/initialize/:kbId          # 初始化
PUT    /api/v1/initialization/config/:kbId              # 更新配置

# Ollama 相关
GET    /api/v1/initialization/ollama/status             # 检查状态
GET    /api/v1/initialization/ollama/models             # 列出模型
POST   /api/v1/initialization/ollama/models/check       # 检查模型
POST   /api/v1/initialization/ollama/models/download    # 下载模型
GET    /api/v1/initialization/ollama/download/progress/:taskId  # 下载进度
GET    /api/v1/initialization/ollama/download/tasks     # 下载任务列表

# 远程 API 相关
POST   /api/v1/initialization/remote/check              # 检查远程模型
POST   /api/v1/initialization/embedding/test            # 测试嵌入模型
POST   /api/v1/initialization/rerank/check              # 检查重排序模型
POST   /api/v1/initialization/multimodal/test           # 测试多模态功能

# 信息提取
POST   /api/v1/initialization/extract/text-relation     # 提取文本关系
POST   /api/v1/initialization/extract/fabri-tag         # Fabric 标签提取
POST   /api/v1/initialization/extract/fabri-text        # Fabric 文本提取
```

### 系统路由 (RegisterSystemRoutes)
```go
GET    /api/v1/system/info          # 获取系统信息
GET    /api/v1/system/minio/buckets # 列出 MinIO buckets
```

### MCP 服务路由 (RegisterMCPServiceRoutes)
```go
POST   /api/v1/mcp-services         # 创建 MCP 服务
GET    /api/v1/mcp-services         # 获取 MCP 服务列表
GET    /api/v1/mcp-services/:id     # 获取 MCP 服务详情
PUT    /api/v1/mcp-services/:id     # 更新 MCP 服务
DELETE /api/v1/mcp-services/:id     # 删除 MCP 服务
POST   /api/v1/mcp-services/:id/test      # 测试连接
GET    /api/v1/mcp-services/:id/tools     # 获取工具列表
GET    /api/v1/mcp-services/:id/resources  # 获取资源列表
```

### 网页搜索路由 (RegisterWebSearchRoutes)
```go
GET    /api/v1/web-search/providers  # 获取搜索提供商列表
```

### 自定义 Agent 路由 (RegisterCustomAgentRoutes)
```go
GET    /api/v1/agents/placeholders  # 获取占位符定义
POST   /api/v1/agents               # 创建 Agent
GET    /api/v1/agents               # 获取 Agent 列表
GET    /api/v1/agents/:id           # 获取 Agent 详情
PUT    /api/v1/agents/:id           # 更新 Agent
DELETE /api/v1/agents/:id           # 删除 Agent
POST   /api/v1/agents/:id/copy      # 复制 Agent
```

## 中间件配置

### 中间件执行顺序
1. CORS (gin-contrib/cors)
2. RequestID (middleware.RequestID)
3. Logger (middleware.Logger)
4. Recovery (middleware.Recovery)
5. ErrorHandler (middleware.ErrorHandler)
6. Auth (middleware.Auth) - 需要认证
7. TracingMiddleware (middleware.TracingMiddleware)

### CORS 配置
```go
cors.New(cors.Config{
    AllowOrigins:     []string{"*"},
    AllowMethods:     []string{"GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS"},
    AllowHeaders:     []string{"Origin", "Content-Type", "Accept", "Authorization", "X-API-Key", "X-Request-ID"},
    ExposeHeaders:    []string{"Content-Length", "Access-Control-Allow-Origin"},
    AllowCredentials: true,
    MaxAge:           12 * time.Hour,
})
```

### Swagger 配置
- 仅在非 `release` 模式下启用
- 默认折叠 Models
- 展开模式为 list
- 支持深度链接
- 持久化认证信息

## 关键依赖与配置

### 依赖模块
- `github.com/Tencent/WeKnora/internal/config`: 配置管理
- `github.com/Tencent/WeKnora/internal/handler`: 请求处理器
- `github.com/Tencent/WeKnora/internal/middleware`: 中间件
- `github.com/Tencent/WeKnora/internal/types/interfaces`: 服务接口

### 外部依赖
- `github.com/gin-gonic/gin`: Web 框架
- `github.com/gin-contrib/cors`: CORS 支持
- `github.com/swaggo/gin-swagger`: Swagger 文档
- `go.uber.org/dig`: 依赖注入

## 测试与质量

### 测试策略
- 路由测试：验证路由注册正确
- 中间件测试：验证中间件执行顺序
- 集成测试：端到端 API 测试

### 代码质量
- 路由分组清晰
- 处理器函数命名规范
- 错误处理统一

## 常见问题 (FAQ)

### Q: 如何添加新的路由？
A: 在 `router.go` 中添加新的 `Register*Routes` 函数，并在 `NewRouter` 中调用。

### Q: 如何禁用 Swagger？
A: 设置 `GIN_MODE=release` 环境变量。

### Q: 如何修改 CORS 配置？
A: 修改 `NewRouter` 函数中的 `cors.Config`。

## 相关文件清单

### 核心文件
- `router.go`: 路由主文件
- `task.go`: 任务相关路由

## 变更记录 (Changelog)
- 2025-12-31: 创建 router 模块文档
