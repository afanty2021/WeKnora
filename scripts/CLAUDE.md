[根目录](../../CLAUDE.md) > **scripts**

# scripts 模块

## 模块职责

`scripts` 模块包含各种自动化脚本，用于简化开发、构建、部署和运维任务。

## 脚本说明

### 开发相关

#### dev.sh
启动开发环境：
- 启动所有依赖服务（PostgreSQL, Redis, Qdrant等）
- 运行 WeKnora 应用
- 支持热重载

```bash
./scripts/dev.sh
```

#### quick-dev.sh
快速开发模式：
- 最小依赖启动
- 使用内存数据库
- 快速原型验证

```bash
./scripts/quick-dev.sh
```

### 构建相关

#### build_images.sh
构建 Docker 镜像：
- 构建应用镜像
- 构建 DocReader 镜像
- 推送到镜像仓库

```bash
./scripts/build_images.sh
```

### 部署相关

#### start_all.sh
启动完整服务栈：
- 使用 Docker Compose
- 包含所有服务
- 健康检查等待

```bash
./scripts/start_all.sh
```

#### migrate.sh
数据库迁移脚本：
- 执行数据库迁移
- 支持指定版本
- 备份数据

```bash
./scripts/migrate.sh [version]
```

### 工具脚本

#### check-env.sh
检查环境配置：
- 验证必需的环境变量
- 检查依赖服务状态
- 生成配置建议

```bash
./scripts/check-env.sh
```

#### get_version.sh
获取版本信息：
- 从 VERSION 文件读取
- 支持 Git 标签
- 构建版本信息

```bash
./scripts/get_version.sh
```

#### scale_dev_jobs.sh
开发环境任务扩缩容（已忽略）：
- 调整 Worker 数量
- 性能测试支持

## 使用场景

### 1. 本地开发
```bash
# 完整开发环境
./scripts/dev.sh

# 快速验证
./scripts/quick-dev.sh
```

### 2. 生产部署
```bash
# 构建镜像
./scripts/build_images.sh

# 启动服务
./scripts/start_all.sh

# 执行迁移
./scripts/migrate.sh
```

### 3. 环境检查
```bash
# 检查配置
./scripts/check-env.sh

# 查看版本
./scripts/get_version.sh
```

## 常见问题 (FAQ)

### Q: 脚本执行权限问题？
A: 运行 `chmod +x scripts/*.sh` 添加执行权限。

### Q: Docker Compose 未找到？
A: 确保 Docker 和 Docker Compose 已安装。

### Q: 端口冲突？
A: 修改 docker-compose.yml 中的端口映射。

## 相关文件清单
- `dev.sh`: 开发环境启动
- `quick-dev.sh`: 快速开发模式
- `build_images.sh`: 镜像构建
- `start_all.sh`: 完整服务启动
- `migrate.sh`: 数据库迁移
- `check-env.sh`: 环境检查
- `get_version.sh`: 版本获取

## 变更记录 (Changelog)
- 2025-12-05: 初始化 scripts 模块文档