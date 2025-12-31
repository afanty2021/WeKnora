[根目录](../CLAUDE.md) > **helm**

# helm 模块

## 模块职责

`helm` 模块包含 WeKnora 的 Kubernetes 部署 Helm Chart，支持在 Kubernetes 集群上快速部署和管理 WeKnora 服务。Chart 遵循 Helm 最佳实践，支持生产级别的配置、安全性和可扩展性。

## 技术栈

- **Helm**: v2 (Chart API version)
- **Kubernetes**: >= 1.25.0
- **模板引擎**: Go Template
- **应用版本**: v0.2.6
- **Chart 版本**: 0.1.0

## 项目结构

```
helm/
├── Chart.yaml              # Chart 元数据和配置
├── values.yaml             # 默认配置值
├── README.md               # Helm Chart 使用文档
├── .helmignore             # Helm 忽略文件配置
├── templates/              # Kubernetes 资源模板
│   ├── NOTES.txt           # 安装后提示信息
│   ├── _helpers.tpl        # 模板辅助函数
│   ├── app.yaml            # 后端应用部署
│   ├── docreader.yaml      # 文档解析服务部署
│   ├── frontend.yaml       # 前端应用部署
│   ├── postgres.yaml       # PostgreSQL 数据库
│   ├── redis.yaml          # Redis 缓存/队列
│   ├── neo4j.yaml          # Neo4j 图数据库（可选）
│   ├── pvc.yaml            # 持久化存储卷
│   ├── secrets.yaml        # 密钥配置
│   ├── serviceaccount.yaml # 服务账户
│   └── ingress.yaml        # Ingress 配置（可选）
```

## 核心组件

### 1. app (后端服务)
- **镜像**: `wechatopenai/weknora-app:v0.2.6`
- **端口**: 8080
- **副本数**: 1 (可配置)
- **健康检查**: HTTP /health 端点

### 2. frontend (前端服务)
- **镜像**: `wechatopenai/weknora-ui:latest`
- **端口**: 80
- **副本数**: 1 (可配置)

### 3. docreader (文档解析服务)
- **镜像**: `wechatopenai/weknora-docreader:latest`
- **端口**: 50051 (gRPC)
- **副本数**: 1 (可配置)

### 4. postgresql (数据库)
- **镜像**: `paradedb/paradedb:v0.18.9-pg17`
- **端口**: 5432
- **持久化**: 10Gi PVC

### 5. redis (缓存/队列)
- **镜像**: `redis:7-alpine`
- **端口**: 6379
- **持久化**: 1Gi PVC

### 6. neo4j (图数据库，可选)
- **镜像**: `neo4j:2025.10.1`
- **端口**: 7474 (HTTP), 7687 (Bolt)
- **持久化**: 10Gi PVC
- **用途**: GraphRAG 功能支持

## 入口与启动

### 前置要求
- Kubernetes 1.25+
- Helm 3.10+
- PV provisioner 支持

### 基础安装
```bash
# 创建命名空间并安装
helm install weknora ./helm \
  --namespace weknora \
  --create-namespace \
  --set secrets.dbPassword=secure-password \
  --set secrets.redisPassword=secure-password \
  --set secrets.jwtSecret=$(openssl rand -base64 32)
```

### 启用 Ingress
```bash
helm install weknora ./helm \
  --namespace weknora \
  --create-namespace \
  --set ingress.enabled=true \
  --set ingress.host=weknora.example.com \
  --set ingress.tls.enabled=true \
  --set secrets.dbPassword=secure-password \
  --set secrets.redisPassword=secure-password \
  --set secrets.jwtSecret=$(openssl rand -base64 32)
```

### 启用 Neo4j (GraphRAG)
```bash
helm install weknora ./helm \
  --namespace weknora \
  --create-namespace \
  --set neo4j.enabled=true \
  --set neo4j.password=secure-password \
  --set app.env.ENABLE_GRAPH_RAG=true \
  --set secrets.dbPassword=secure-password \
  --set secrets.redisPassword=secure-password \
  --set secrets.jwtSecret=$(openssl rand -base64 32)
```

### 生产环境部署
```bash
# 使用自定义 values 文件
helm install weknora ./helm \
  --namespace weknora \
  --create-namespace \
  -f values-production.yaml
```

## 配置说明

### 全局配置
```yaml
global:
  storageClass: ""           # 存储类，空值使用集群默认
  imagePullSecrets: []       # 镜像拉取密钥
  podSecurityContext:        # Pod 安全上下文
    seccompProfile:
      type: RuntimeDefault
  containerSecurityContext:  # 容器安全上下文
    allowPrivilegeEscalation: false
```

### 密钥配置（生产环境必须修改）
```yaml
secrets:
  dbUser: postgres
  dbPassword: ""             # 必须设置
  dbName: weknora
  redisPassword: ""          # 必须设置
  jwtSecret: ""              # 必须设置
  tenantAesKey: ""
  existingSecret: ""         # 使用现有 Secret
```

### 可选组件（对应 docker-compose profiles）
```yaml
minio:
  enabled: false             # 对应 --profile minio

neo4j:
  enabled: false             # 对应 --profile neo4j
  # GraphRAG 功能需要启用

qdrant:
  enabled: false             # 对应 --profile qdrant
  # 向量数据库替代方案

jaeger:
  enabled: false             # 对应 --profile jaeger
  # 分布式追踪支持
```

## 安全最佳实践

### 密钥管理
1. **使用 --set 参数**（仅用于测试）
2. **使用 External Secrets Operator**（推荐生产环境）
3. **使用 Sealed Secrets**（用于 GitOps）

### Pod 安全
- 遵循 CNCF 安全最佳实践
- 以非 root 用户运行（如镜像支持）
- 只读根文件系统（如可能）
- 使用 seccomp 配置文件

### 安全上下文
```yaml
# 注意：官方镜像（nginx, postgres, redis）默认以 root 运行
# 仅在使用非 root 兼容镜像时启用 runAsNonRoot
podSecurityContext:
  seccompProfile:
    type: RuntimeDefault

containerSecurityContext:
  allowPrivilegeEscalation: false
  # runAsNonRoot: true  # 需要兼容的镜像
```

## 升级与维护

### 升级
```bash
helm upgrade weknora ./helm \
  --namespace weknora \
  --reuse-values
```

### 回滚
```bash
# 查看历史版本
helm history weknora -n weknora

# 回滚到上一个版本
helm rollback weknora -n weknora

# 回滚到指定版本
helm rollback weknora 2 -n weknora
```

### 卸载
```bash
# 卸载 Helm release
helm uninstall weknora --namespace weknora

# 可选：删除 PVCs
kubectl delete pvc -n weknora -l app.kubernetes.io/instance=weknora
```

## 常见问题 (FAQ)

### Q: 如何使用外部数据库？
A: 设置 `postgresql.enabled=false` 并通过 `app.extraEnv` 配置数据库连接字符串。

### Q: 如何配置持久化存储？
A: 通过 `global.storageClass` 设置存储类，或为每个组件单独配置 PVC。

### Q: 如何启用 Grafana 监控？
A: 需要额外安装 ServiceMonitor 和 Prometheus Operator，在 values.yaml 中添加相应配置。

### Q: 如何配置资源限制？
A: 在 values.yaml 中为每个组件设置 `resources.requests` 和 `resources.limits`。

### Q: 生产环境推荐配置？
A:
1. 使用 External Secrets 管理密钥
2. 启用 Pod 反亲和性确保高可用
3. 配置资源限制防止资源耗尽
4. 使用 HPA 自动扩展副本数
5. 配置 PDB 确保服务可用性

## 相关文件清单

### 核心文件
- `Chart.yaml`: Chart 元数据
- `values.yaml`: 默认配置值
- `README.md`: 详细使用文档

### 模板文件
- `templates/app.yaml`: 后端应用部署配置
- `templates/frontend.yaml`: 前端应用部署配置
- `templates/docreader.yaml`: 文档解析服务部署配置
- `templates/postgres.yaml`: PostgreSQL 数据库配置
- `templates/redis.yaml`: Redis 缓存配置
- `templates/neo4j.yaml`: Neo4j 图数据库配置（可选）
- `templates/ingress.yaml`: Ingress 配置
- `templates/secrets.yaml`: 密钥配置
- `templates/pvc.yaml`: 持久化存储卷配置
- `templates/serviceaccount.yaml`: 服务账户配置
- `templates/_helpers.tpl`: 模板辅助函数

## 变更记录 (Changelog)
- 2025-12-31: 创建 helm 模块文档
