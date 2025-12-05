[根目录](../../CLAUDE.md) > **cmd**

# cmd 模块

## 模块职责

`cmd` 模块包含 WeKnora 应用程序的入口点，负责服务的启动、初始化和生命周期管理。

## 目录结构

```
cmd/
└── server/
    └── main.go    # WeKnora 服务器主入口
```

## 入口与启动

### main.go 核心功能
1. **依赖注入容器初始化**
   - 使用 `dig` 库构建依赖关系
   - 注册所有服务组件

2. **服务启动流程**
   - 加载配置文件
   - 初始化数据库连接
   - 设置路由
   - 启动 HTTP 服务器

3. **优雅关闭**
   - 监听系统信号（SIGINT, SIGTERM, SIGHUP）
   - 30秒超时的关闭流程
   - 资源清理和连接关闭

### 启动命令
```bash
# 开发模式
go run cmd/server/main.go

# 生产模式
GIN_MODE=release go run cmd/server/main.go

# 构建后运行
go build -o weknora cmd/server/main.go
./weknora
```

## 核心组件

### 1. 容器构建
```go
c := container.BuildContainer(runtime.GetContainer())
```
构建包含所有依赖的注入容器。

### 2. 服务注册
注册以下核心服务：
- HTTP 路由器
- 配置管理
- 链路追踪
- 资源清理器

### 3. 中间件链
- CORS 跨域处理
- 请求 ID 生成
- 日志记录
- 错误处理
- 认证中间件
- OpenTelemetry 追踪

### 4. 健康检查
```go
GET /health
```
返回服务健康状态。

## 配置管理

### 环境变量
- `GIN_MODE`: 运行模式（debug/release）
- 自动检测并设置 Gin 模式

### 资源清理
实现了完善的资源清理机制：
- 追踪器关闭
- 数据库连接关闭
- 其他注册资源的清理

## 常见问题 (FAQ)

### Q: 如何修改监听端口？
A: 通过 `config/config.yaml` 中的 server.port 配置。

### Q: 如何启用 HTTPS？
A: 需要在配置中添加证书配置，并使用反向代理（如 Nginx）。

### Q: 服务启动失败怎么办？
A: 检查日志输出，确认端口是否被占用，配置文件是否正确。

## 相关文件清单
- `cmd/server/main.go`: 服务器主入口

## 变更记录 (Changelog)
- 2025-12-05: 初始化 cmd 模块文档