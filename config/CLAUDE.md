[根目录](../../CLAUDE.md) > **config**

# config 模块

## 模块职责

`config` 模块包含 WeKnora 应用的所有配置文件，定义了系统运行参数、服务配置和业务规则。

## 文件说明

### config.yaml
主配置文件，包含以下配置节：

#### 1. 服务器配置
```yaml
server:
  port: 8080          # 服务端口
  host: "0.0.0.0"     # 监听地址
```

#### 2. 对话服务配置
```yaml
conversation:
  max_rounds: 5                    # 最大对话轮数
  keyword_threshold: 0.3           # 关键词阈值
  embedding_top_k: 10              # 向量检索 top-k
  vector_threshold: 0.5            # 向量相似度阈值
  rerank_threshold: 0.5            # 重排序阈值
  rerank_top_k: 5                  # 重排序 top-k
  enable_rewrite: true             # 启用查询重写
  enable_query_expansion: true     # 启用查询扩展
  enable_rerank: true              # 启用重排序
```

#### 3. Prompt 配置
包含各类系统 Prompt：
- 回退策略 Prompt
- 查询重写 Prompt
- 对话配置 Prompt

#### 4. Agent 配置
```yaml
agent:
  max_iterations: 10               # 最大迭代次数
  temperature: 0.7                 # 生成温度
  enable_reflection: true          # 启用反思
```

## 配置优先级

1. 环境变量（最高优先级）
2. 配置文件（config.yaml）
3. 默认值（最低优先级）

## 环境变量映射

配置项可以通过环境变量覆盖：
- `WEKNORA_SERVER_PORT` → server.port
- `WEKNORA_CONVERSATION_MAX_ROUNDS` → conversation.max_rounds
- 其他配置项类似映射

## 配置验证

应用启动时会验证配置的合法性：
- 必填项检查
- 数值范围验证
- 格式正确性验证

## 常见问题 (FAQ)

### Q: 如何修改向量检索阈值？
A: 修改 `conversation.vector_threshold` 值（0-1之间）。

### Q: 如何启用知识图谱？
A: 设置 `ENABLE_GRAPH_RAG=true` 环境变量。

### Q: Prompt 模板如何自定义？
A: 直接修改 config.yaml 中对应的 prompt 内容。

## 相关文件清单
- `config.yaml`: 主配置文件

## 变更记录 (Changelog)
- 2025-12-05: 初始化 config 模块文档