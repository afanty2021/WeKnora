[根目录](../../CLAUDE.md) > [internal](../CLAUDE.md) > [application](../application/CLAUDE.md) > **repository**

# repository 模块

## 模块职责

`repository` 模块是数据访问层（Data Access Layer），负责与各种数据存储引擎交互，提供统一的数据 CRUD 接口。支持 PostgreSQL（主数据库）、Qdrant、Elasticsearch、Neo4j 等多种存储后端。

## 模块结构

```
repository/
├── knowledgebase.go       # 知识库数据访问
├── knowledge.go           # 知识点数据访问
├── chunk.go              # 分块数据访问
├── session.go            # 会话数据访问
├── message.go            # 消息数据访问
├── user.go               # 用户数据访问
├── tenant.go             # 租户数据访问
├── model.go              # 模型数据访问
├── tag.go                # 标签数据访问
├── custom_agent.go       # 自定义 Agent 数据访问
├── mcp_service.go        # MCP 服务数据访问
└── retriever/            # 检索引擎实现
    ├── qdrant/           # Qdrant 向量数据库
    ├── elasticsearch/     # Elasticsearch 搜索引擎
    │   ├── v7/           # Elasticsearch v7
    │   ├── v8/           # Elasticsearch v8
    │   └── structs.go    # 通用结构
    ├── postgres/         # PostgreSQL + pgvector
    ├── neo4j/            # Neo4j 图数据库
    └── structs.go        # 检索器通用结构
```

## 对外接口

### 1. 知识库 Repository (KnowledgeBaseRepository)

```go
type KnowledgeBaseRepository interface {
    CreateKnowledgeBase(ctx context.Context, kb *types.KnowledgeBase) error
    GetKnowledgeBaseByID(ctx context.Context, id string) (*types.KnowledgeBase, error)
    GetKnowledgeBaseByIDs(ctx context.Context, ids []string) ([]*types.KnowledgeBase, error)
    ListKnowledgeBases(ctx context.Context) ([]*types.KnowledgeBase, error)
    ListKnowledgeBasesByTenantID(ctx context.Context, tenantID uint64) ([]*types.KnowledgeBase, error)
    UpdateKnowledgeBase(ctx context.Context, kb *types.KnowledgeBase) error
    DeleteKnowledgeBase(ctx context.Context, id string) error
}
```

### 2. 会话 Repository (SessionRepository)

```go
type SessionRepository interface {
    Create(ctx context.Context, session *types.Session) (*types.Session, error)
    Get(ctx context.Context, tenantID uint64, id string) (*types.Session, error)
    GetByTenantID(ctx context.Context, tenantID uint64) ([]*types.Session, error)
    GetPagedByTenantID(ctx context.Context, tenantID uint64, page *types.Pagination) ([]*types.Session, int64, error)
    Update(ctx context.Context, session *types.Session) error
    Delete(ctx context.Context, tenantID uint64, id string) error
}
```

### 3. 消息 Repository (MessageRepository)

```go
type MessageRepository interface {
    CreateMessage(ctx context.Context, message *types.Message) (*types.Message, error)
    GetMessage(ctx context.Context, sessionID string, messageID string) (*types.Message, error)
    GetMessagesBySession(ctx context.Context, sessionID string, page int, pageSize int) ([]*types.Message, error)
    GetRecentMessagesBySession(ctx context.Context, sessionID string, limit int) ([]*types.Message, error)
    GetMessagesBySessionBeforeTime(ctx context.Context, sessionID string, beforeTime time.Time, limit int) ([]*types.Message, error)
    UpdateMessage(ctx context.Context, message *types.Message) error
    DeleteMessage(ctx context.Context, sessionID string, messageID string) error
    GetFirstMessageOfUser(ctx context.Context, sessionID string) (*types.Message, error)
    GetMessageByRequestID(ctx context.Context, sessionID string, requestID string) (*types.Message, error)
}
```

### 4. 检索引擎 Repository (RetrieveEngineRepository)

```go
type RetrieveEngineRepository interface {
    EngineType() types.RetrieverEngineType
    Support() []types.RetrieverType
    EstimateStorageSize(ctx context.Context, indexInfoList []*types.IndexInfo, params map[string]any) int64
    Save(ctx context.Context, indexInfo *types.IndexInfo, additionalParams map[string]any) error
    BatchSave(ctx context.Context, indexInfoList []*types.IndexInfo, additionalParams map[string]any) error
    DeleteByChunkIDList(ctx context.Context, chunkIDList []string, dimension int, knowledgeType string) error
    DeleteBySourceIDList(ctx context.Context, sourceIDList []string, dimension int, knowledgeType string) error
    DeleteByKnowledgeIDList(ctx context.Context, knowledgeIDList []string, dimension int, knowledgeType string) error
    Retrieve(ctx context.Context, params types.RetrieveParams) ([]*types.RetrieveResult, error)
    CopyIndices(ctx context.Context, sourceKnowledgeBaseID string, sourceToTargetKBIDMap map[string]string, sourceToTargetChunkIDMap map[string]string, targetKnowledgeBaseID string, dimension int, knowledgeType string) error
    BatchUpdateChunkEnabledStatus(ctx context.Context, chunkStatusMap map[string]bool) error
    BatchUpdateChunkTagID(ctx context.Context, chunkTagMap map[string]string) error
}
```

## 检索引擎实现

### Qdrant (v1.16.2+)

**特性**:
- 多维度向量存储（动态集合创建）
- 全文关键词检索（支持中文分词）
- 向量相似性搜索（Cosine 距离）
- 批量操作优化

**关键方法**:
- `VectorRetrieve`: 向量相似性搜索
- `KeywordsRetrieve`: 关键词全文检索

### Elasticsearch (v7/v8)

**特性**:
- 全文检索
- script_score 向量搜索（v7）
- dense_vector 向量搜索（v8）
- 批量索引操作

**关键方法**:
- `KeywordsRetrieve`: 全文检索
- `VectorRetrieve`: 向量相似性搜索

### PostgreSQL (pgvector)

**特性**:
- ParadeDB 全文检索
- pgvector 向量相似性搜索
- HNSW 索引优化
- 子查询优化（避免重复计算）

**关键方法**:
- `KeywordsRetrieve`: 使用 `paradedb.match` 全文检索
- `VectorRetrieve`: 使用 `<=>` 余弦距离

### Neo4j

**特性**:
- 知识图谱存储
- APOC 库批量导入
- 节点和关系管理

**关键方法**:
- `AddGraph`: 添加图谱数据
- `DelGraph`: 删除图谱数据
- `SearchNode`: 搜索节点

## 关键依赖与配置

### 依赖模块
- `github.com/Tencent/WeKnora/internal/types`: 数据类型定义
- `github.com/Tencent/WeKnora/internal/types/interfaces`: 接口定义
- `github.com/Tencent/WeKnora/internal/logger`: 日志工具
- `github.com/Tencent/WeKnora/internal/common`: 通用工具

### 外部依赖
- **ORM**: `gorm.io/gorm`
- **Qdrant**: `github.com/tencent/qdrant-go`
- **Elasticsearch**: `github.com/elastic/go-elasticsearch/v7` / `v8`
- **Neo4j**: `github.com/neo4j/neo4j-go-driver/v6`
- **pgvector**: `github.com/pgvector/pgvector-go`

## 数据模型

### PostgreSQL 表结构
主要表包括：
- `tenants`: 租户
- `users`: 用户
- `knowledge_bases`: 知识库
- `knowledge`: 知识点
- `chunks`: 分块
- `sessions`: 会话
- `messages`: 消息
- `models`: 模型
- `tags`: 标签
- `custom_agents`: 自定义 Agent
- `mcp_services`: MCP 服务

### 向量存储结构
不同引擎的向量存储：
- **Qdrant**: Collection with payload
- **Elasticsearch**: Index with dense_vector field
- **PostgreSQL**: embeddings table with pgvector column

## 测试与质量

### 测试策略
- 单元测试：每个 repository 方法
- 集成测试：使用测试数据库
- Mock 测试：使用 gomock

### 代码质量
- 使用 GORM 进行数据库操作
- 错误处理必须包含日志
- 事务处理保证数据一致性

## 常见问题 (FAQ)

### Q: 如何添加新的检索引擎？
A: 在 `retriever/` 下创建新目录，实现 `RetrieveEngineRepository` 接口，并在检索服务中注册。

### Q: 如何切换向量数据库？
A: 修改配置文件中的 `RETRIEVE_DRIVER` 环境变量，支持 `qdrant`、`elasticsearch`、`postgres`。

### Q: 如何处理大批量数据操作？
A: 使用 `BatchSave` 和 `BatchUpdate` 方法，内部自动分批处理（默认 500 条/批）。

## 相关文件清单

### 核心文件
- `knowledgebase.go`: 知识库数据访问
- `session.go`: 会话数据访问
- `message.go`: 消息数据访问
- `retriever/qdrant/repository.go`: Qdrant 检索实现
- `retriever/elasticsearch/v7/repository.go`: Elasticsearch v7 实现
- `retriever/postgres/repository.go`: PostgreSQL + pgvector 实现
- `retriever/neo4j/repository.go`: Neo4j 图数据库实现

## 变更记录 (Changelog)
- 2025-12-31: 创建 repository 模块文档
