# MySQL配置

<cite>
**本文档引用的文件**
- [.env.example](file://.env.example)
- [docker-compose.yml](file://docker-compose.yml)
- [docker-compose.dev.yml](file://docker-compose.dev.yml)
- [migrations/mysql/00-init-db.sql](file://migrations/mysql/00-init-db.sql)
- [scripts/check-env.sh](file://scripts/check-env.sh)
- [internal/container/container.go](file://internal/container/container.go)
- [internal/database/migration.go](file://internal/database/migration.go)
- [go.mod](file://go.mod)
</cite>

## 目录
1. [简介](#简介)
2. [环境变量配置](#环境变量配置)
3. [Docker Compose配置](#docker-compose配置)
4. [MySQL初始化脚本](#mysql初始化脚本)
5. [开发环境配置](#开发环境配置)
6. [环境检查脚本](#环境检查脚本)
7. [数据库连接实现](#数据库连接实现)
8. [依赖管理](#依赖管理)
9. [总结](#总结)

## 简介
本文档详细说明了如何在Weknora项目中将MySQL配置为主数据库。通过分析项目中的关键文件，我们将了解如何通过设置环境变量来启用MySQL支持，配置数据库连接参数，调整Docker Compose文件以适应MySQL部署，以及理解MySQL初始化脚本的结构和作用。

## 环境变量配置
在Weknora项目中，数据库配置主要通过`.env`文件中的环境变量来管理。`.env.example`文件提供了所有必要的环境变量示例。

### 主要数据库配置
数据库类型通过`DB_DRIVER`环境变量指定，支持`postgres`和`mysql`两种数据库类型。

```env
# 存储配置
# 主数据库类型(postgres/mysql)
DB_DRIVER=postgres

# 数据库用户名
DB_USER=postgres

# 数据库密码
DB_PASSWORD=postgres123!@#

# 数据库名称
DB_NAME=WeKnora

# 数据库主机地址
DB_HOST=localhost

# 数据库端口
DB_PORT=5432
```

要将数据库从PostgreSQL切换到MySQL，需要将`DB_DRIVER`的值从`postgres`改为`mysql`，并相应地调整其他数据库连接参数。

**Section sources**
- [.env.example](file://.env.example#L14-L43)

## Docker Compose配置
Docker Compose文件定义了项目中各个服务的配置和依赖关系。在Weknora项目中，`docker-compose.yml`和`docker-compose.dev.yml`文件用于管理服务。

### 服务依赖和网络配置
在`docker-compose.yml`文件中，`app`服务依赖于`postgres`服务，并通过健康检查确保数据库服务已准备好。

```yaml
services:
  app:
    # ... 其他配置
    depends_on:
      postgres:
        condition: service_healthy
    # ... 其他配置
    environment:
      - DB_DRIVER=postgres
      - DB_HOST=postgres
      - DB_PORT=5432
      - DB_USER=${DB_USER:-}
      - DB_PASSWORD=${DB_PASSWORD:-}
      - DB_NAME=${DB_NAME:-}
    # ... 其他配置
```

对于开发环境，`docker-compose.dev.yml`文件提供了类似的配置，但使用了不同的容器名称和端口映射。

```yaml
services:
  postgres:
    container_name: WeKnora-postgres-dev
    ports:
      - "${DB_PORT:-5432}:5432"
    # ... 其他配置
```

**Section sources**
- [docker-compose.yml](file://docker-compose.yml#L16-L98)
- [docker-compose.dev.yml](file://docker-compose.dev.yml#L4-L27)

## MySQL初始化脚本
Weknora项目提供了MySQL数据库的初始化脚本，位于`migrations/mysql/00-init-db.sql`文件中。

### 数据库模式和表结构
初始化脚本首先删除可能存在的表，然后创建新的表结构。以下是主要表的创建语句：

```sql
CREATE TABLE tenants (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    api_key VARCHAR(64) NOT NULL,
    retriever_engines JSON NOT NULL,
    status VARCHAR(50) DEFAULT 'active',
    business VARCHAR(255) NOT NULL,
    storage_quota BIGINT NOT NULL DEFAULT 10737418240,
    storage_used BIGINT NOT NULL DEFAULT 0,
    agent_config JSON DEFAULT NULL COMMENT 'Tenant-level agent configuration in JSON format',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP NULL DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 AUTO_INCREMENT=10000;

CREATE TABLE models (
    id VARCHAR(64) PRIMARY KEY,
    tenant_id INT NOT NULL,
    name VARCHAR(255) NOT NULL,
    type VARCHAR(50) NOT NULL,
    source VARCHAR(50) NOT NULL,
    description TEXT,
    parameters JSON NOT NULL,
    is_default BOOLEAN NOT NULL DEFAULT FALSE,
    status VARCHAR(50) NOT NULL DEFAULT 'active',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP NULL DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;  
```

### 索引创建
脚本还为关键字段创建了索引，以提高查询性能：

```sql
CREATE INDEX idx_models_tenant_source_type ON models(tenant_id, source, type);
CREATE INDEX idx_knowledge_bases_tenant_name ON knowledge_bases(tenant_id, name);
CREATE INDEX idx_knowledges_tenant_id ON knowledges(tenant_id, knowledge_base_id);
CREATE INDEX idx_sessions_tenant_id ON sessions(tenant_id);
CREATE INDEX idx_messages_session_role ON messages(session_id, role); 
CREATE INDEX idx_chunks_tenant_knowledge ON chunks(tenant_id, knowledge_id);
CREATE INDEX idx_chunks_parent_id ON chunks(parent_chunk_id);
CREATE INDEX idx_chunks_chunk_type ON chunks(chunk_type);
```

**Section sources**
- [migrations/mysql/00-init-db.sql](file://migrations/mysql/00-init-db.sql#L1-L157)

## 开发环境配置
开发环境的配置与生产环境类似，但使用了不同的Docker Compose文件和端口映射。

### 开发环境变量
在开发环境中，可以通过`.env`文件覆盖默认的环境变量值。例如，可以将`DB_PORT`设置为`3306`以使用MySQL的默认端口。

```env
DB_DRIVER=mysql
DB_HOST=localhost
DB_PORT=3306
DB_USER=root
DB_PASSWORD=root123!@#
DB_NAME=WeKnora
```

### 开发Docker Compose
`docker-compose.dev.yml`文件为开发环境提供了基础设施服务，包括数据库、Redis、MinIO等。

```yaml
services:
  postgres:
    image: paradedb/paradedb:v0.18.9-pg17
    container_name: WeKnora-postgres-dev
    ports:
      - "${DB_PORT:-5432}:5432"
    environment:
      - POSTGRES_USER=${DB_USER}
      - POSTGRES_PASSWORD=${DB_PASSWORD}
      - POSTGRES_DB=${DB_NAME}
    volumes:
      - postgres-data-dev:/var/lib/postgresql/data
      - ./migrations/paradedb/00-init-db.sql:/docker-entrypoint-initdb.d/00-init-db.sql
      - ./migrations/paradedb/01-migrate-to-paradedb.sql:/docker-entrypoint-initdb.d/01-migrate-to-paradedb.sql
    # ... 其他配置
```

**Section sources**
- [docker-compose.dev.yml](file://docker-compose.dev.yml#L4-L27)

## 环境检查脚本
`scripts/check-env.sh`脚本用于检查开发环境的配置是否正确。

### 检查必要的环境变量
脚本首先检查`.env`文件是否存在，然后加载其中的环境变量。

```bash
# 检查 .env 文件
if [ -f ".env" ]; then
    log_success ".env 文件存在"
else
    log_error ".env 文件不存在"
    exit 1
fi

# 加载 .env 文件
set -a
source .env
set +a
```

### 数据库配置检查
脚本检查必要的数据库环境变量是否已设置：

```bash
# 数据库配置
log_info "数据库配置:"
check_var "DB_DRIVER"
check_var "DB_HOST"
check_var "DB_PORT"
check_var "DB_USER"
check_var "DB_PASSWORD"
check_var "DB_NAME"
```

脚本还提供了友好的错误提示，帮助开发者快速解决问题。

**Section sources**
- [scripts/check-env.sh](file://scripts/check-env.sh#L39-L83)

## 数据库连接实现
在Weknora项目的Go代码中，数据库连接的初始化逻辑位于`internal/container/container.go`文件中。

### 数据库驱动支持
`initDatabase`函数根据`DB_DRIVER`环境变量的值选择相应的数据库驱动：

```go
func initDatabase(cfg *config.Config) (*gorm.DB, error) {
	var dialector gorm.Dialector
	var migrateDSN string
	switch os.Getenv("DB_DRIVER") {
	case "postgres":
		// PostgreSQL配置
		gormDSN := fmt.Sprintf(
			"host=%s port=%s user=%s password=%s dbname=%s sslmode=%s",
			os.Getenv("DB_HOST"),
			os.Getenv("DB_PORT"),
			os.Getenv("DB_USER"),
			os.Getenv("DB_PASSWORD"),
			os.Getenv("DB_NAME"),
			"disable",
		)
		dialector = postgres.Open(gormDSN)
		// ... 其他配置
	default:
		return nil, fmt.Errorf("unsupported database driver: %s", os.Getenv("DB_DRIVER"))
	}
	// ... 其他逻辑
}
```

### 迁移管理
项目使用`golang-migrate/migrate/v4`库来管理数据库迁移。迁移脚本位于`migrations/versioned`目录中。

```go
func RunMigrations(dsn string) error {
	migrationsPath := "file://migrations/versioned"
	m, err := migrate.New(migrationsPath, dsn)
	if err != nil {
		return fmt.Errorf("failed to create migrate instance: %w", err)
	}
	defer m.Close()
	
	if err := m.Up(); err != nil && err != migrate.ErrNoChange {
		return fmt.Errorf("failed to run migrations: %w", err)
	}
	// ... 其他逻辑
}
```

**Section sources**
- [internal/container/container.go](file://internal/container/container.go#L237-L307)
- [internal/database/migration.go](file://internal/database/migration.go#L14-L127)

## 依赖管理
项目的依赖管理通过`go.mod`文件进行。

### 数据库相关依赖
`go.mod`文件中列出了项目依赖的数据库驱动和ORM库：

```go
require (
	github.com/golang-migrate/migrate/v4 v4.19.0
	github.com/neo4j/neo4j-go-driver/v6 v6.0.0-alpha.1
	github.com/redis/go-redis/v9 v9.14.0
	gorm.io/driver/postgres v1.5.11
	gorm.io/gorm v1.25.12
)
```

这些依赖支持PostgreSQL、Neo4j和Redis，但没有直接的MySQL驱动依赖。这表明项目可能通过GORM的通用接口支持MySQL，或者需要在运行时动态加载MySQL驱动。

**Section sources**
- [go.mod](file://go.mod#L16-L28)

## 总结
通过分析Weknora项目的配置文件和代码，我们了解到如何将MySQL配置为主数据库。关键步骤包括：

1. 在`.env`文件中将`DB_DRIVER`设置为`mysql`。
2. 配置`DB_HOST`、`DB_PORT`、`DB_USER`、`DB_PASSWORD`和`DB_NAME`等连接参数。
3. 确保`docker-compose.yml`文件中的服务依赖和网络配置正确。
4. 使用`migrations/mysql/00-init-db.sql`脚本初始化MySQL数据库。
5. 在开发环境中使用`scripts/check-env.sh`脚本验证配置。

尽管项目当前主要使用PostgreSQL，但通过适当的配置和可能的代码调整，可以支持MySQL作为主数据库。

**Section sources**
- [.env.example](file://.env.example#L14-L43)
- [docker-compose.yml](file://docker-compose.yml#L16-L98)
- [migrations/mysql/00-init-db.sql](file://migrations/mysql/00-init-db.sql#L1-L157)
- [scripts/check-env.sh](file://scripts/check-env.sh#L39-L83)
- [internal/container/container.go](file://internal/container/container.go#L237-L307)
- [internal/database/migration.go](file://internal/database/migration.go#L14-L127)
- [go.mod](file://go.mod#L16-L28)