[根目录](../../CLAUDE.md) > **internal**

# internal 模块

## 模块职责

`internal` 模块是 WeKnora 的核心业务逻辑层，包含所有内部实现细节，不对外暴露。采用 Clean Architecture 设计模式，分为多个子模块。

## 子模块结构

### 1. agent - Agent 引擎模块
- **入口**: `internal/agent/engine.go`
- **职责**: 实现 ReACT 模式的智能 Agent，支持工具调用和多轮推理
- **关键组件**:
  - `engine.go`: Agent 核心引擎
  - `tools/`: 内置工具集合
  - `prompts.go`: Prompt 模板管理
  - `const.go`: 常量定义

### 2. application - 应用层
- **职责**: 业务逻辑编排，包含 repository 和 service 两层
- **repository**: 数据访问层实现
  - 支持 PostgreSQL、Qdrant、Elasticsearch、Neo4j 等多种存储
  - 检索器实现（retriever）
  - 文件存储抽象
- **service**: 业务服务层
  - `chat_pipeline/`: 对话流水线实现
  - `metric/`: 评估指标计算
  - `web_search/`: 网页搜索集成
  - `llmcontext/`: LLM 上下文管理

### 3. handler - HTTP 处理器
- **职责**: 处理 HTTP 请求，调用 service 层
- **主要处理器**:
  - `KnowledgeBaseHandler`: 知识库管理
  - `KnowledgeHandler`: 知识管理
  - `SessionHandler`: 会话管理（包含聊天）
  - `ModelHandler`: 模型管理
  - `EvaluationHandler`: 评估管理
  - `AuthHandler`: 认证管理

### 4. middleware - 中间件
- **职责**: 横切关注点处理
- **主要中间件**:
  - 认证中间件（JWT）
  - 请求日志中间件
  - 错误处理中间件
  - OpenTelemetry 追踪中间件

### 5. router - 路由定义
- **入口**: `internal/router/router.go`
- **职责**: 定义 API 路由，注册中间件和处理器
- **路由前缀**: `/api/v1`

### 6. types - 类型定义
- **model.go**: 数据模型定义
- **interfaces/**: 接口定义，依赖倒置

### 7. container - 依赖注入容器
- **职责**: 管理依赖关系，实现控制反转
- 使用 `dig` 库进行依赖注入

## 对外接口

内部模块不直接对外暴露接口，通过以下方式与外部交互：

1. **HTTP API**: 通过 `router` 和 `handler` 暴露 RESTful API
2. **gRPC**: DocReader 服务通过 gRPC 提供文档解析能力
3. **Go Client**: 通过 `client` 模块提供 Go SDK

## 关键依赖与配置

### 外部依赖
- **数据库**: PostgreSQL (主数据库)
- **向量数据库**: Qdrant/Elasticsearch/ParadeDB
- **图数据库**: Neo4j (可选)
- **缓存**: Redis
- **消息队列**: Asynq (基于 Redis)
- **对象存储**: MinIO/腾讯云 COS
- **LLM**: OpenAI API/Ollama

### 配置管理
- 配置文件: `config/config.yaml`
- 环境变量优先级高于配置文件
- 支持热更新部分配置

## 数据模型

### 核心实体
1. **Tenant**: 租户
2. **User**: 用户
3. **KnowledgeBase**: 知识库
4. **Knowledge**: 知识点
5. **Chunk**: 文档分块
6. **Session**: 会话
7. **Message**: 消息
8. **Model**: AI 模型
9. **Evaluation**: 评估

### 关系说明
- 租户 -> 用户 (1:N)
- 租户 -> 知识库 (1:N)
- 知识库 -> 知识点 (1:N)
- 知识点 -> 分块 (1:N)
- 会话 -> 消息 (1:N)

## 测试与质量

### 测试覆盖
- 单元测试: `*_test.go` 文件
- 集成测试: 使用 testify 框架
- 重点测试模块:
  - chat_pipeline 核心流程
  - metric 评估指标计算
  - retriever 检索逻辑

### 代码质量
- 使用 `golangci.yml` 配置代码检查
- 必须通过所有检查规则
- 代码覆盖率要求: 核心模块 > 80%

## 常见问题 (FAQ)

### Q: 如何添加新的存储后端？
A: 在 `repository/retriever/` 下实现 `Repository` 接口，并在 `retriever/registry.go` 中注册。

### Q: 如何扩展 Agent 工具？
A: 在 `agent/tools/` 下实现 `Tool` 接口，并在 `agent/tools/registry.go` 中注册。

### Q: 如何添加新的评估指标？
A: 在 `service/metric/` 下实现 `Metric` 接口。

## 相关文件清单

### 核心文件
- `internal/container/container.go`: 依赖注入容器设置
- `internal/router/router.go`: API 路由定义
- `internal/types/model.go`: 数据模型定义
- `internal/config/config.go`: 配置管理

### Agent 相关
- `internal/agent/engine.go`: Agent 引擎核心
- `internal/agent/tools/`: 工具实现

### Service 层
- `internal/application/service/chat_pipeline/`: 对话流水线
- `internal/application/service/retriever/`: 检索服务
- `internal/application/service/graph.go`: 知识图谱服务

### Repository 层
- `internal/application/repository/retriever/`: 检索器实现
- `internal/application/repository/file/`: 文件存储实现

## 变更记录 (Changelog)
- 2025-12-05: 初始化 internal 模块文档