[根目录](../../CLAUDE.md) > **docs**

# docs 模块

## 模块职责

`docs` 模块包含 WeKnora 项目的所有文档，包括用户指南、API 参考、开发文档和架构说明。

## 文档分类

### 用户文档

#### README 系列
- `README.md`: 英文版项目介绍
- `README_CN.md`: 中文版项目介绍
- `README_JA.md`: 日文版项目介绍

#### 使用指南
- `QA.md`: 常见问题解答
- `API.md`: API 接口文档
- `WeKnora.md`: 项目详细介绍

### 开发文档

#### 开发指南
- `开发指南.md`: 开发环境搭建和规范
- `快速开发模式说明.md`: 快速开发模式使用
- `开启知识图谱功能.md`: 知识图谱配置指南

#### 架构文档
- `BUILTIN_MODELS.md`: 内置模型说明
- `KnowledgeGraph.md`: 知识图谱设计文档
- `MCP功能使用说明.md`: MCP 功能使用指南

#### 数据库相关
- `使用其他向量数据库.md`: 向量数据库集成指南

### API 文档

#### API 详情
- `api/` 目录包含详细的 API 文档：
  - `chat.md`: 聊天 API
  - `chunk.md`: 分块 API
  - `evaluation.md`: 评估 API
  - `faq.md`: FAQ API
  - `knowledge-base.md`: 知识库 API
  - `knowledge.md`: 知识管理 API
  - `message.md`: 消息 API
  - `model.md`: 模型 API
  - `session.md`: 会话 API
  - `tag.md`: 标签 API
  - `tenant.md`: 租户 API

### 资源文件

#### 图片资源
- `images/` 目录包含各种架构图和截图：
  - `architecture.png`: 系统架构图
  - `logo.png`: 项目 Logo
  - `pipeline.png/jpg/jpeg`: 处理流水线图
  - `*.png`: 各功能界面截图

#### PDF 文档
- `WeKnora.pdf`: 项目 PDF 版本文档

## 文档维护

### 文档更新流程
1. 功能开发时同步更新文档
2. API 变更必须更新 API 文档
3. 定期审查和优化文档结构

### 多语言支持
- 中文为首要文档语言
- 重要文档提供英文版
- 根据需求添加其他语言版本

## 文档规范

### Markdown 规范
- 使用标准 Markdown 语法
- 代码块标注语言类型
- 使用相对路径引用图片
- 目录结构清晰

### API 文档规范
- 包含完整的请求/响应示例
- 说明所有参数类型和含义
- 提供错误码说明
- 包含认证方式说明

## 常见问题 (FAQ)

### Q: 如何贡献文档？
A: 提交 PR 到相应文档文件，遵循文档规范。

### Q: 文档中的图片如何处理？
A: 图片存放在 `images/` 目录，使用相对路径引用。

### Q: API 文档如何生成？
A: 目前手动维护，未来考虑自动化生成。

## 相关文件清单
### 核心文档
- `README.md`: 项目主介绍
- `API.md`: API 总览
- `开发指南.md`: 开发指南
- `QA.md`: 常见问题

### API 文档
- `api/chat.md`: 聊天 API
- `api/knowledge-base.md`: 知识库 API
- 其他 API 文档...

### 架构文档
- `KnowledgeGraph.md`: 知识图谱
- `BUILTIN_MODELS.md`: 内置模型

## 变更记录 (Changelog)
- 2025-12-05: 初始化 docs 模块文档