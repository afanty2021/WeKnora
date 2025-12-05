[根目录](../../CLAUDE.md) > **migrations**

# migrations 模块

## 模块职责

`migrations` 模块包含数据库迁移脚本，用于初始化和升级 WeKnora 的数据库结构。

## 目录结构

```
migrations/
├── mysql/                    # MySQL 数据库脚本
│   └── 00-init-db.sql       # MySQL 初始化脚本
├── paradedb/                # ParadeDB 向量数据库脚本
│   ├── 00-init-db.sql       # ParadeDB 初始化
│   └── 01-migrate-to-paradedb.sql  # 迁移到 ParadeDB
└── versioned/               # 版本化迁移（golang-migrate）
    ├── 000001_agent.up.sql
    └── 000001_agent.down.sql
```

## 数据库支持

### 1. PostgreSQL (主数据库)
- 用于存储核心业务数据
- 支持事务和关系查询
- 使用 GORM 进行 ORM 操作

### 2. ParadeDB (向量数据库)
- 基于 PostgreSQL 的向量扩展
- 支持混合检索（关键词+向量）
- 可选使用（与 Qdrant 二选一）

### 3. 版本化迁移
- 使用 golang-migrate 工具管理
- 支持升级和回滚
- 自动执行数据库版本控制

## 核心表结构

### 主要数据表
1. **tenants** - 租户表
2. **users** - 用户表
3. **knowledge_bases** - 知识库表
4. **knowledges** - 知识点表
5. **chunks** - 文档分块表
6. **sessions** - 会话表
7. **messages** - 消息表
8. **models** - 模型表
9. **evaluations** - 评估表
10. **agents** - Agent 配置表（新增）

## 迁移执行

### 自动迁移
WeKnora 支持应用启动时自动执行数据库迁移：
- 检测数据库版本
- 执行待应用的迁移
- 记录迁移历史

### 手动迁移
```bash
# 使用 migrate 工具
migrate -path migrations/versioned -database "postgres://..." up
```

## 索引优化

### 性能索引
- 主键索引
- 外键索引
- 查询优化索引
- 唯一性约束

### 向量索引
- pgvector 扩展的向量索引
- HNSW 算法优化

## 常见问题 (FAQ)

### Q: 如何切换数据库？
A: 修改环境变量 `DB_DRIVER` 和相应的连接配置。

### Q: 迁移失败怎么办？
A: 检查迁移脚本语法，确保数据库权限足够。

### Q: 如何添加新表？
A: 创建新的版本化迁移文件（up/down）。

## 相关文件清单
- `mysql/00-init-db.sql`: MySQL 初始化
- `paradedb/00-init-db.sql`: ParadeDB 初始化
- `versioned/000001_agent.up.sql`: Agent 功能迁移

## 变更记录 (Changelog)
- 2025-12-05: 初始化 migrations 模块文档