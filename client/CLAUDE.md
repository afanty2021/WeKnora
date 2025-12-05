[根目录](../../CLAUDE.md) > **client**

# client 模块

## 模块职责

`client` 模块是 WeKnora 的 Go SDK，提供简洁的客户端接口用于集成 WeKnora 服务。支持知识库管理、对话交互、模型管理等核心功能。

## 入口与启动

### 入口文件
- **主入口**: `client.go`
- **示例代码**: `example.go`

### 初始化客户端
```go
import "github.com/Tencent/WeKnora/client"

c, err := client.NewClient(&client.Config{
    BaseURL: "http://localhost:8080",
    APIKey:  "your-api-key",
})
```

## 对外接口

### 1. 知识库管理 (KnowledgeBase)
```go
// 创建知识库
kb, err := c.KnowledgeBase.Create(ctx, &client.CreateKnowledgeBaseRequest{
    Name:        "我的知识库",
    Description: "知识库描述",
    Type:        "document",
})

// 获取知识库列表
kbs, err := c.KnowledgeBase.List(ctx, &client.ListKnowledgeBaseRequest{
    Page:     1,
    PageSize: 10,
})

// 更新知识库
kb, err := c.KnowledgeBase.Update(ctx, kbID, &client.UpdateKnowledgeBaseRequest{
    Name: "新名称",
})
```

### 2. 知识点管理 (Knowledge)
```go
// 添加文档
knowledge, err := c.Knowledge.AddDocument(ctx, &client.AddDocumentRequest{
    KnowledgeBaseID: kbID,
    Name:           "文档名称",
    Content:        "文档内容",
    Type:           "text",
})

// 搜索知识点
knowledges, err := c.Knowledge.Search(ctx, &client.SearchRequest{
    KnowledgeBaseID: kbID,
    Query:          "搜索关键词",
    TopK:           10,
})
```

### 3. 对话管理 (Session)
```go
// 创建会话
session, err := c.Session.Create(ctx, &client.CreateSessionRequest{
    KnowledgeBaseIDs: []string{kbID},
    ModelID:         modelID,
    AgentMode:       false,
})

// 发送消息
resp, err := c.Chat(ctx, sessionID, &client.ChatRequest{
    Message: "你好",
    Stream:  false,
})
```

### 4. 消息管理 (Message)
```go
// 获取历史消息
messages, err := c.Message.List(ctx, sessionID, &client.ListMessagesRequest{
    Page:     1,
    PageSize: 50,
})
```

### 5. 模型管理 (Model)
```go
// 获取模型列表
models, err := c.Model.List(ctx, &client.ListModelsRequest{
    Type: "KnowledgeQA",
})
```

### 6. 分块管理 (Chunk)
```go
// 获取分块列表
chunks, err := c.Chunk.List(ctx, &client.ListChunksRequest{
    KnowledgeID: knowledgeID,
    Page:        1,
    PageSize:    20,
})
```

### 7. 评估管理 (Evaluation)
```go
// 创建评估
eval, err := c.Evaluation.Create(ctx, &client.CreateEvaluationRequest{
    Name:           "评估任务",
    KnowledgeBaseID: kbID,
    DatasetID:      datasetID,
})
```

### 8. 标签管理 (Tag)
```go
// 创建标签
tag, err := c.Tag.Create(ctx, &client.CreateTagRequest{
    Name:  "标签名",
    Color: "#FF0000",
})
```

### 9. FAQ 管理
```go
// 添加 FAQ
faq, err := c.FAQ.Create(ctx, &client.CreateFAQRequest{
    KnowledgeBaseID: kbID,
    Question:        "问题",
    Answer:          "答案",
})
```

### 10. Agent 功能
```go
// Agent 聊天
resp, err := c.Agent.Chat(ctx, sessionID, &client.AgentChatRequest{
    Message: "请帮我搜索相关资料",
})
```

## 关键依赖与配置

### Go 模块配置
- **Go 版本**: 1.24+
- **主模块**: `github.com/Tencent/WeKnora`

### 依赖包
- `github.com/google/uuid`: UUID 生成
- `github.com/sirupsen/logrus`: 日志记录
- 标准库: `net/http`, `encoding/json`, `context`

### 配置选项
```go
type Config struct {
    BaseURL string  // API 基础地址
    APIKey  string  // API 密钥
    Timeout time.Duration // 请求超时
}
```

## 数据模型

### 请求/响应结构
所有 API 调用都使用结构化的请求和响应对象，位于各自的文件中：
- `knowledgebase.go`: 知识库相关结构
- `knowledge.go`: 知识点相关结构
- `session.go`: 会话相关结构
- `message.go`: 消息相关结构
- 等等...

### 通用字段
- `ID string`: 资源唯一标识
- `CreatedAt time.Time`: 创建时间
- `UpdatedAt time.Time`: 更新时间

## 错误处理

客户端使用标准的 `error` 返回错误信息：
- 网络错误：`context.DeadlineExceeded`, 连接错误等
- API 错误：HTTP 状态码 + 错误消息
- 业务错误：通过 `error` 包装详细信息

## 测试与质量

### 测试策略
- 单元测试：使用 `testing` 包
- Mock 服务：用于独立测试
- 集成测试：连接真实服务测试

### 示例代码
`example.go` 提供了完整的使用示例，包括：
- 客户端初始化
- 创建知识库
- 上传文档
- 进行对话

## 常见问题 (FAQ)

### Q: 如何处理流式响应？
A: 使用 `ChatStream` 方法，它会返回一个 channel 用于接收流式数据。

### Q: 如何上传文件？
A: 使用 `Knowledge.UploadFile` 方法，支持文件路径和 io.Reader。

### Q: 如何设置超时？
A: 在创建客户端时设置 `Timeout` 配置项。

## 相关文件清单

### 核心文件
- `client.go`: 客户端主实现
- `example.go`: 使用示例
- `go.mod`: Go 模块定义
- `README.md`: 使用说明
- `README_EN.md`: 英文说明

### API 接口文件
- `agent.go`: Agent 相关接口
- `chunk.go`: 分块管理接口
- `evaluation.go`: 评估管理接口
- `faq.go`: FAQ 管理接口
- `knowledge.go`: 知识管理接口
- `knowledgebase.go`: 知识库管理接口
- `message.go`: 消息管理接口
- `model.go`: 模型管理接口
- `session.go`: 会话管理接口
- `tag.go`: 标签管理接口
- `tenant.go`: 租户管理接口

## 变更记录 (Changelog)
- 2025-12-05: 初始化 client 模块文档