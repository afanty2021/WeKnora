[根目录](../../CLAUSE.md) > **docker**

# docker 模块

## 模块职责

`docker` 模块包含 WeKnora 项目的容器化配置，用于构建和部署各个服务组件。

## 目录结构

```
docker/
├── Dockerfile.app         # 主应用 Docker 镜像
├── Dockerfile.docreader   # DocReader 服务 Docker 镜像
└── config/
    └── supervisord.conf   # 进程管理配置
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

在根目录的 `docker-compose.yml` 中使用这些镜像：
- frontend: 使用 weknora-ui 镜像
- app: 使用 weknora-app 镜像
- docreader: 使用 weknora-docreader 镜像

## 常见问题 (FAQ)

### Q: 镜像构建失败？
A: 检查 Dockerfile 中的依赖版本和路径是否正确。

### Q: 服务无法启动？
A: 查看容器日志，确认环境变量和配置文件。

## 相关文件清单
- `Dockerfile.app`: 主应用镜像定义
- `Dockerfile.docreader`: DocReader 镜像定义
- `config/supervisord.conf`: 进程管理配置

## 变更记录 (Changelog)
- 2025-12-05: 初始化 docker 模块文档