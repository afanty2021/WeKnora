# WeKnora - AI 驱动的文档理解与检索框架

> **最新版本**: v0.2.1 (2025-12-08)
> **项目状态**: 活跃开发中，支持 Qdrant 向量数据库和增强的检索引擎
>
> **变更记录 (Changelog)**
> - 2025-12-08: 同步 v0.2.1 版本更新，新增 Qdrant 向量数据库支持
> - 2025-12-05 20:16: 深度补捞完成，覆盖率提升至 95%+，新增核心子模块文档
> - 2025-12-05: 初始化架构文档，完成全模块扫描与索引生成

## 项目愿景

WeKnora 是一个基于大语言模型（LLM）的文档理解与智能检索框架，采用 RAG（检索增强生成）范式实现高质量的上下文感知问答。项目致力于为企业提供安全、可控、可扩展的知识管理解决方案，支持多模态文档处理、智能推理和多轮对话。

## 架构总览

WeKnora 采用现代化的微服务架构设计，主要包含以下核心组件：

- **文档解析服务 (DocReader)**: 支持 PDF、Word、Excel、CSV、图片等多种格式的文档解析与结构化提取
- **向量存储引擎**:
  - **Qdrant** (v1.16.2): 专业向量数据库，支持多维度向量存储和全文检索
  - **Elasticsearch**: 全文检索引擎
  - **PostgreSQL (pgvector)**: 关系型数据库向量扩展
- **知识图谱模块**: 基于 Neo4j 的知识图谱构建与检索（可选）
- **Agent 引擎**: ReACT 模式的智能代理，支持工具调用、MCP 扩展和多轮推理
- **Web 界面**: 基于 Vue 3 + TDesign 的现代化管理界面，支持多语言
- **API 网关**: 基于 Gin 的 RESTful API 服务
- **MCP 服务器**: Model Context Protocol 支持，扩展 Agent 能力
- **异步任务队列**: 基于 MQ 的任务状态管理，确保服务重启后的任务完整性

## 模块结构图

```mermaid
graph TD
    A["(根) WeKnora"] --> B["cmd"];
    A --> C["internal"];
    A --> D["frontend"];
    A --> E["client"];
    A --> F["docreader"];
    A --> G["mcp-server"];
    A --> H["docker"];
    A --> I["config"];
    A --> J["migrations"];
    A --> K["scripts"];
    A --> L["docs"];

    C --> C1["agent"];
    C --> C2["application"];
    C --> C3["handler"];
    C --> C4["middleware"];
    C --> C5["router"];
    C --> C6["types"];
    C --> C7["container"];
    C --> C8["config"];
    C --> C9["common"];
    C --> C10["runtime"];
    C --> C11["tracing"];

    C2 --> C2a["repository"];
    C2 --> C2b["service"];

    C3 --> C3a["session"];
    C2b --> C2b1["chat_pipeline"];
    C2b --> C2b2["metric"];
    C2b --> C2b3["llmcontext"];
    C2b --> C2b4["retriever"];
    C2b --> C2b5["file"];
    C2b --> C2b6["web_search"];

    C2a --> C2a1["retriever"];
    C2a1 --> C2a1a["qdrant"];
    C2a1 --> C2a1b["elasticsearch"];
    C2a1 --> C2a1c["postgres"];
    C2a1 --> C2a1d["neo4j"];

    click B "./cmd/CLAUDE.md" "查看 cmd 模块文档"
    click C "./internal/CLAUDE.md" "查看 internal 模块文档"
    click D "./frontend/CLAUDE.md" "查看 frontend 模块文档"
    click E "./client/CLAUDE.md" "查看 client 模块文档"
    click F "./docreader/CLAUDE.md" "查看 docreader 模块文档"
    click G "./mcp-server/CLAUDE.md" "查看 mcp-server 模块文档"
    click H "./docker/CLAUDE.md" "查看 docker 模块文档"
    click I "./config/CLAUDE.md" "查看 config 模块文档"
    click J "./migrations/CLAUDE.md" "查看 migrations 模块文档"
    click K "./scripts/CLAUDE.md" "查看 scripts 模块文档"
    click L "./docs/CLAUDE.md" "查看 docs 模块文档"
```

## 模块索引

| 模块 | 路径 | 语言 | 职责 | 覆盖率 |
|------|------|------|------|--------|
| 应用入口 | [cmd](./cmd/CLAUDE.md) | Go | 服务启动与生命周期管理 | 100% |
| 核心业务 | [internal](./internal/CLAUDE.md) | Go | 业务逻辑实现 | 95% |
| Web界面 | [frontend](./frontend/CLAUDE.md) | TypeScript/Vue | 用户管理界面 | 85% |
| Go客户端 | [client](./client/CLAUDE.md) | Go | SDK客户端库 | 90% |
| 文档解析 | [docreader](./docreader/CLAUDE.md) | Python/Go | 多格式文档解析 | 85% |
| MCP服务 | [mcp-server](./mcp-server/CLAUDE.md) | Python | MCP协议服务 | 80% |
| 容器化 | [docker](./docker/CLAUDE.md) | Dockerfile | 容器化配置 | 100% |
| 配置文件 | [config](./config/CLAUDE.md) | YAML | 应用配置 | 100% |
| 数据库迁移 | [migrations](./migrations/CLAUDE.md) | SQL | 数据库脚本 | 100% |
| 构建脚本 | [scripts](./scripts/CLAUDE.md) | Shell | 构建和部署 | 100% |
| 项目文档 | [docs](./docs/CLAUDE.md) | Markdown | 项目文档 | 75% |

## 核心子模块文档

### internal 模块深度解析

| 子模块 | 文档路径 | 说明 |
|--------|----------|------|
| Agent引擎 | [internal/agent](./internal/agent/CLAUDE.md) | ReACT模式智能代理实现 |
| 业务服务层 | [internal/application/service](./internal/application/service/CLAUDE.md) | 核心业务逻辑与流水线 |
| HTTP处理器 | [internal/handler](./internal/handler/CLAUDE.md) | RESTful API请求处理 |

## 运行与开发

### 环境要求
- Go 1.21+
- Python 3.8+
- Node.js 18+
- Docker & Docker Compose

### 快速启动
```bash
# 克隆项目
git clone https://github.com/Tencent/WeKnora.git
cd WeKnora

# 启动所有服务（开发环境）
./scripts/quick-dev.sh

# 或使用 Docker Compose（带 Qdrant 支持）
docker-compose --profile qdrant up -d

# 启动完整服务栈（包括所有可选服务）
docker-compose --profile full up -d
```

### 开发模式
```bash
# 启动后端服务
cd cmd/server
go run main.go

# 启动前端（新终端）
cd frontend
npm run dev

# 启动文档解析服务（新终端）
cd docreader
python main.py
```

## 测试策略

### 单元测试
- Go: `go test ./...`
- Python: `pytest docreader/`
- TypeScript: `npm run test`

### 集成测试
```bash
# 运行集成测试
./scripts/test-integration.sh

# 性能测试
./scripts/test-performance.sh
```

### 测试覆盖目标
- 核心业务逻辑: > 90%
- Agent 工具: > 85%
- API 接口: > 80%
- 前端组件: > 70%

## 编码规范

### Go 代码规范
- 遵循 `golangci.yml` 配置
- 行宽限制: 120 字符
- 必须通过 `go vet` 和 `golint` 检查

### Python 代码规范
- 遵循 PEP 8
- 使用 black 格式化
- 类型注解覆盖率 > 90%

### TypeScript 代码规范
- 使用 ESLint + Prettier
- 严格模式 TypeScript
- 单个文件不超过 300 行

## AI 使用指引

### 代码生成
1. 优先使用已定义的接口和类型
2. 遵循现有的设计模式
3. 添加适当的错误处理
4. 包含必要的日志记录

### 文档更新
1. 更新相关模块的 CLAUDE.md
2. 记录重要的设计决策
3. 添加使用示例
4. 更新变更记录

### 新功能开发
1. 先阅读相关模块文档
2. 理解现有架构和约束
3. 遵循依赖注入原则
4. 编写相应测试用例

## 当前扫描状态

### 覆盖率统计
- **总体覆盖率**: 95.2% (810/850 文件)
- **已扫描模块**: 11/11
- **已完成子模块文档**: 3/3
- **测试文件覆盖**: 10/10

### 本次深度补捞成果
- ✅ 深度分析了 Agent 工具实现（14个工具）
- ✅ 详尽解析了对话流水线服务（15个核心文件）
- ✅ 全扫描了 HTTP 处理器实现（20+处理器）
- ✅ 完善了前端组件结构分析（60+组件）
- ✅ 补充了 repository 层检索器实现细节
- ✅ 新增了3个核心子模块的 CLAUDE.md

### 剩余缺口
- docreader/parser/ 下的部分解析器实现细节（低优先级）
- 部分工具函数和辅助类（不影响主流程）

## v0.2.1 新特性 (2025-12-08)

### 🚀 Qdrant 向量数据库支持
- **完整集成**: Qdrant 作为主要检索引擎
- **向量相似性搜索**: 基于嵌入维度的动态集合创建
- **全文关键词搜索**: 支持中文/日文/韩文等多语言分词
- **专业中文分词**: 使用 jieba 进行关键词查询优化

### ⚡ 基础设施增强
- **Docker Compose 配置优化**: 支持服务配置文件（minio、qdrant、neo4j、jaeger、full）
- **增强的 dev.sh 脚本**: 新增 `--minio`、`--qdrant`、`--neo4j`、`--jaeger`、`--full` 参数
- **数据库迁移系统**: 自动脏状态恢复和 Neo4j 连接重试机制
- **检索引擎自动配置**: 通过 `RETRIEVE_DRIVER` 环境变量自动配置

### 🐛 问题修复
- 修复 Qdrant 中文查询返回空结果的问题
- 简化图片 URL 验证逻辑，提高兼容性

## 变更记录 (Changelog)

### 2025-12-08: 同步 v0.2.1 更新
- 更新项目版本信息和最新功能
- 新增 Qdrant 相关配置说明
- 更新 Docker Compose 启动命令

### 2025-12-05 20:16: 深度补捞完成
- 覆盖率从 45.2% 提升至 95.2%
- 新增 `internal/agent/CLAUDE.md`
- 新增 `internal/application/service/CLAUDE.md`
- 新增 `internal/handler/CLAUDE.md`
- 深度分析了 ReACT Agent 实现细节
- 详尽解析了 RAG 对话流水线架构
- 全面扫描了 RESTful API 处理器

### 2025-12-05: 初始化完成
- 完成所有 11 个主要模块的 CLAUDE.md 文档创建
- 生成了带导航的 Mermaid 架构图
- 建立了 .claude/index.json 覆盖率追踪系统
- 实现了可续跑的增量更新机制