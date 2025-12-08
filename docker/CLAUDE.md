[根目录](../../CLAUSE.md) > **docker**

# docker 模块

## 模块职责

`docker` 模块包含 WeKnora 项目的容器化配置，用于构建和部署各个服务组件。支持通过 Docker Compose profiles 启动可选服务，如 Qdrant、MinIO、Neo4j 等。

## 目录结构

```
docker/
├── Dockerfile.app         # 主应用 Docker 镜像
├── Dockerfile.docreader   # DocReader 服务 Docker 镜像
└── config/
    └── supervisord.conf   # 进程管理配置

# 根目录配置文件
├── docker-compose.yml          # 生产环境配置
├── docker-compose.dev.yml      # 开发环境配置
└── .env.example               # 环境变量示例
```

## 镜像说明

### 1. Dockerfile.app
构建 WeKnora 主应用镜像：
- **基础镜像**: golang:1.24-alpine
- **构建阶段**: 编译 Go 应用
- **运行阶段**: 最小化运行时镜像
- **暴露端口**: 8080

### 2. Dockerfile.docreader
构建 DocReader 服务镜像：
- **基础镜像**: python:3.8-slim
- **依赖安装**: pip 安装 Python 依赖
- **服务启动**: 启动 gRPC 服务
- **暴露端口**: 50051

### 3. supervisord.conf
进程管理配置，用于管理多个服务进程：
- 主应用进程
- DocReader 进程
- 日志管理
- 自动重启

## 构建命令

```bash
# 构建主应用镜像
docker build -f docker/Dockerfile.app -t weknora-app .

# 构建 DocReader 镜像
docker build -f docker/Dockerfile.docreader -t weknora-docreader .
```

## Docker Compose 集成

### 服务配置
在根目录的 `docker-compose.yml` 中使用这些镜像：
- **frontend**: 使用 weknora-ui 镜像
- **app**: 使用 weknora-app 镜像
- **docreader**: 使用 weknora-docreader 镜像

### 可选服务 (v0.2.1 新增)
支持通过 profiles 启动的可选服务：

#### qdrant profile
```bash
docker-compose --profile qdrant up -d
```
- **qdrant**: 向量数据库服务 (v1.16.2)
- 端口: 6333 (HTTP), 6334 (gRPC)

#### minio profile
```bash
docker-compose --profile minio up -d
```
- **minio**: 对象存储服务
- 端口: 9000 (API), 9001 (Console)

#### neo4j profile
```bash
docker-compose --profile neo4j up -d
```
- **neo4j**: 图数据库服务
- 端口: 7474 (HTTP), 7687 (Bolt)

#### jaeger profile
```bash
docker-compose --profile jaeger up -d
```
- **jaeger**: 链路追踪服务
- 端口: 16686 (UI), 14268 (Collector)

#### full profile
```bash
docker-compose --profile full up -d
```
启动所有可选服务

### 开发环境配置
使用 `docker-compose.dev.yml` 进行开发：
```bash
docker-compose -f docker-compose.dev.yml up -d
```

### 脚本支持
使用 `scripts/dev.sh` 快速启动：
```bash
# 启动基础服务
./scripts/dev.sh

# 启动带 Qdrant 的服务
./scripts/dev.sh --qdrant

# 启动完整服务栈
./scripts/dev.sh --full
```

## 常见问题 (FAQ)

### Q: 镜像构建失败？
A: 检查 Dockerfile 中的依赖版本和路径是否正确。

### Q: 服务无法启动？
A: 查看容器日志，确认环境变量和配置文件。

### Q: Qdrant 服务连接失败？
A: 确保使用 `--profile qdrant` 启动 Qdrant 服务，并检查 6333 端口是否可用。

### Q: 如何查看所有可用服务？
A: 运行 `docker-compose config` 查看完整的服务配置。

## 环境变量配置

复制 `.env.example` 到 `.env` 并修改：
```bash
cp .env.example .env
```

关键配置项：
```bash
# Qdrant 配置
QDRANT_URL=http://qdrant:6333

# MinIO 配置
MINIO_ENDPOINT=minio:9000
MINIO_ACCESS_KEY=minioadmin
MINIO_SECRET_KEY=minioadmin

# Neo4j 配置
NEO4J_URI=bolt://neo4j:7687
NEO4J_USERNAME=neo4j
NEO4J_PASSWORD=neo4j
```

## 相关文件清单
- `Dockerfile.app`: 主应用镜像定义
- `Dockerfile.docreader`: DocReader 镜像定义
- `config/supervisord.conf`: 进程管理配置
- `docker-compose.yml`: 生产环境编排
- `docker-compose.dev.yml`: 开发环境编排
- `.env.example`: 环境变量模板

## 变更记录 (Changelog)

### 2025-12-08: v0.2.1 更新
- ✨ 新增 Docker Compose profiles 支持
- ✨ 添加 Qdrant 服务配置 (v1.16.2)
- ✨ 支持 MinIO、Neo4j、Jaeger 等可选服务
- 📝 更新启动脚本和使用说明

### 2025-12-05: 初始化
- 📝 初始化 docker 模块文档
- 🏗️ 完成基础镜像配置