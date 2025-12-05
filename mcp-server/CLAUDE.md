[根目录](../../CLAUDE.md) > **mcp-server**

# mcp-server 模块

## 模块职责

`mcp-server` 模块实现了 Model Context Protocol (MCP) 服务器，为 WeKnora Agent 提供扩展能力。MCP 是一个开放协议，允许 AI 模型安全地连接到外部数据源和工具。

## 技术栈

- **语言**: Python 3.8+
- **协议**: Model Context Protocol (MCP)
- **传输层**: 支持多种传输方式（stdio、HTTP、WebSocket）
- **工具启动器**: uvx、npx

## 项目结构

```
mcp-server/
├── main.py                    # MCP 服务器主入口
├── weknora_mcp_server.py      # WeKnora MCP 实现
├── run.py                     # 运行脚本
├── run_server.py              # 服务器启动脚本
├── pyproject.toml             # Python 项目配置
├── requirements.txt           # 依赖列表
├── setup.py                   # 安装配置
├── __init__.py
├── CHANGELOG.md               # 变更日志
├── EXAMPLES.md               # 使用示例
├── INSTALL.md                # 安装指南
├── LICENSE                   # 许可证
├── MCP_CONFIG.md             # MCP 配置说明
├── PROJECT_SUMMARY.md        # 项目总结
├── README.md                 # 项目说明
├── MANIFEST.in               # 打包配置
├── test_imports.py           # 导入测试
└── test_module.py            # 模块测试
```

## 入口与启动

### 作为 MCP 服务器运行
```bash
# 通过 uvx 启动
uvx weknora-mcp-server

# 或通过 Python 直接运行
python main.py
```

### 配置 MCP 客户端
在 MCP 客户端配置中添加：
```json
{
  "mcpServers": {
    "weknora": {
      "command": "uvx",
      "args": ["weknora-mcp-server"],
      "env": {
        "WEKNORA_API_URL": "http://localhost:8080",
        "WEKNORA_API_KEY": "your-api-key"
      }
    }
  }
}
```

## MCP 工具集合

### 1. WeKnora 知识库工具
- `weknora_search_knowledge`: 搜索知识库内容
- `weknora_add_knowledge`: 添加新知识
- `weknora_list_knowledge_bases`: 列出所有知识库
- `weknora_get_knowledge`: 获取特定知识内容

### 2. 对话管理工具
- `weknora_create_session`: 创建新对话会话
- `weknora_chat`: 发送消息并获取回复
- `weknora_get_chat_history`: 获取对话历史

### 3. 文档处理工具
- `weknora_upload_document`: 上传文档
- `weknora_parse_document`: 解析文档内容
- `weknora_extract_metadata`: 提取文档元数据

### 4. 评估工具
- `weknora_create_evaluation`: 创建评估任务
- `weknora_run_evaluation`: 运行评估
- `weknora_get_evaluation_results`: 获取评估结果

## 核心功能

### 1. MCP 协议实现
- 实现 MCP 规范的核心接口
- 支持工具注册和调用
- 错误处理和响应格式化

### 2. 认证与安全
- API 密钥认证
- 请求签名验证
- 敏感信息脱敏

### 3. 配置管理
- 环境变量配置
- 动态配置更新
- 默认值处理

### 4. 日志与监控
- 结构化日志记录
- 性能指标收集
- 错误追踪

## 环境配置

### 必需配置
```bash
WEKNORA_API_URL=http://localhost:8080    # WeKnora API 地址
WEKNORA_API_KEY=your-api-key             # API 密钥
```

### 可选配置
```bash
WEKNORA_TIMEOUT=30                       # 请求超时（秒）
WEKNORA_RETRY=3                          # 重试次数
WEKNORA_LOG_LEVEL=INFO                   # 日志级别
```

## 数据模型

### MCP 工具定义
每个工具遵循 MCP 工具定义格式：
```python
{
    "name": "weknora_search_knowledge",
    "description": "搜索 WeKnora 知识库",
    "inputSchema": {
        "type": "object",
        "properties": {
            "query": {"type": "string"},
            "knowledge_base_id": {"type": "string"}
        },
        "required": ["query"]
    }
}
```

## 测试与质量

### 测试文件
- `test_module.py`: 核心功能测试
- `test_imports.py`: 模块导入测试

### 测试覆盖
- MCP 协议兼容性
- 工具调用流程
- 错误处理场景
- 性能基准测试

## 部署选项

### 1. 作为 Python 包安装
```bash
pip install weknora-mcp-server
```

### 2. 通过 uvx 运行（推荐）
```bash
uvx weknora-mcp-server
```

### 3. Docker 部署
```dockerfile
FROM python:3.8-slim
COPY . /app
WORKDIR /app
RUN pip install -e .
CMD ["python", "main.py"]
```

## 集成示例

### 与 Claude Desktop 集成
```json
{
  "mcpServers": {
    "weknora": {
      "command": "uvx",
      "args": ["weknora-mcp-server"],
      "env": {
        "WEKNORA_API_URL": "http://localhost:8080",
        "WEKNORA_API_KEY": "${WEKNORA_API_KEY}"
      }
    }
  }
}
```

### 与其他 MCP 客户端集成
支持任何符合 MCP 规范的客户端：
- Claude Desktop
- Cursor
- Continue.dev
- 自定义 MCP 客户端

## 常见问题 (FAQ)

### Q: MCP 服务器无法启动？
A: 检查 Python 版本（需要 3.8+）和依赖是否正确安装。

### Q: 工具调用失败？
A: 验证 WEKNORA_API_URL 和 WEKNORA_API_KEY 配置是否正确。

### Q: 如何添加新工具？
A: 在 `weknora_mcp_server.py` 中定义新工具并注册到工具列表。

## 相关文件清单

### 核心文件
- `main.py`: 服务器入口
- `weknora_mcp_server.py`: MCP 实现
- `pyproject.toml`: 项目配置

### 文档文件
- `README.md`: 项目说明
- `INSTALL.md`: 安装指南
- `MCP_CONFIG.md`: MCP 配置说明
- `EXAMPLES.md`: 使用示例

### 测试文件
- `test_module.py`: 功能测试
- `test_imports.py`: 导入测试

## 变更记录 (Changelog)
- 2025-12-05: 初始化 mcp-server 模块文档