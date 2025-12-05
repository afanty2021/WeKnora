[根目录](../../CLAUDE.md) > [frontend](../) > **frontend**

# frontend 模块

## 模块职责

`frontend` 模块是 WeKnora 的 Web 管理界面，基于 Vue 3 + TypeScript + TDesign 构建，提供直观的用户界面用于知识库管理、对话交互和系统配置。

## 技术栈

- **框架**: Vue 3 (Composition API)
- **语言**: TypeScript
- **UI 库**: TDesign Vue Next
- **状态管理**: Pinia
- **路由**: Vue Router 4
- **构建工具**: Vite
- **HTTP 客户端**: Axios
- **Markdown 渲染**: Marked + DOMPurify
- **文件处理**: PapaParse (CSV)、xlsx

## 项目结构

```
frontend/
├── public/              # 静态资源
├── src/
│   ├── api/            # API 接口定义
│   ├── assets/         # 资源文件
│   ├── components/     # 公共组件
│   ├── hooks/          # 组合式函数
│   ├── locales/        # 国际化文件
│   ├── router/         # 路由配置
│   ├── store/          # Pinia 状态管理
│   ├── style/          # 全局样式
│   ├── utils/          # 工具函数
│   ├── views/          # 页面组件
│   ├── App.vue         # 根组件
│   └── main.ts         # 入口文件
├── tests/              # 测试文件
├── package.json        # 项目配置
└── vite.config.ts      # Vite 配置
```

## 核心页面

### 1. 登录页面
- 路径: `/login`
- 功能: 用户认证、租户选择

### 2. 知识库管理
- 路径: `/knowledge`
- 功能:
  - 知识库列表
  - 创建/编辑知识库
  - 文档导入（文件、URL、文本）
  - 标签管理

### 3. 对话界面
- 路径: `/chat`
- 功能:
  - 多轮对话
  - Agent 模式切换
  - 工具调用过程展示
  - 历史会话管理

### 4. 模型管理
- 路径: `/models`
- 功能:
  - 模型列表
  - 添加/编辑模型
  - 模型状态管理

### 5. 系统设置
- 路径: `/settings`
- 功能:
  - 系统配置
  - 用户管理
  - 评估管理

## 对外接口

### API 调用
通过 `src/api/` 模块调用后端 API：

```typescript
// 主要 API 模块
- auth.ts: 认证相关
- knowledge.ts: 知识库管理
- chat.ts: 对话服务
- model.ts: 模型管理
- system.ts: 系统管理
```

### 事件总线
使用 mitt 实现组件间通信：
- 用户登录状态变更
- 语言切换
- 主题切换

## 关键依赖与配置

### 核心依赖
- `vue`: 3.5.13
- `tdesign-vue-next`: 1.17.2
- `pinia`: 3.0.1
- `axios`: 1.8.4
- `vue-router`: 4.5.0

### 开发依赖
- `@vitejs/plugin-vue`: 6.0.0
- `typescript`: 5.8.0
- `vite`: 7.2.2
- `vue-tsc`: 2.2.8

### 环境配置
通过 `.env` 文件配置：
- `VITE_API_BASE_URL`: API 基础地址
- `VITE_APP_TITLE`: 应用标题

## 状态管理

使用 Pinia 进行状态管理，主要 Store：

1. **authStore**: 用户认证状态
2. **knowledgeStore**: 知识库数据
3. **chatStore**: 对话状态
4. **modelStore**: 模型配置
5. **settingsStore**: 系统设置

## 国际化

使用 Vue I18n 支持多语言：
- 中文（简体）
- 英文
- 日文

## 测试策略

### 单元测试
- 使用 Vitest 进行组件测试
- 测试覆盖率目标: 70%

### E2E 测试
- 使用 Playwright 进行端到端测试
- 覆盖核心用户流程

## 开发指南

### 启动开发服务器
```bash
npm run dev
```

### 构建生产版本
```bash
npm run build
```

### 代码规范
- 使用 ESLint + Prettier
- 遵循 Vue 3 官方风格指南
- 使用 TypeScript 严格模式

## 常见问题 (FAQ)

### Q: 如何添加新的页面？
A:
1. 在 `src/views/` 下创建组件
2. 在 `router/index.ts` 中添加路由
3. 如需权限控制，添加路由守卫

### Q: 如何调用新的 API？
A:
1. 在 `src/api/` 下添加接口定义
2. 在组件中使用 `await` 调用
3. 处理错误和加载状态

### Q: 如何添加新的主题样式？
A:
1. 在 `src/style/` 下创建主题文件
2. 使用 CSS 变量定义颜色
3. 在组件中使用 class 绑定

## 相关文件清单

### 入口文件
- `src/main.ts`: 应用入口
- `src/App.vue`: 根组件

### 核心页面
- `src/views/knowledge/`: 知识库管理
- `src/views/chat/`: 对话界面
- `src/views/model/`: 模型管理
- `src/views/login/`: 登录页面

### API 定义
- `src/api/auth.ts`: 认证 API
- `src/api/knowledge.ts`: 知识库 API
- `src/api/chat.ts`: 对话 API

### 配置文件
- `vite.config.ts`: Vite 配置
- `tsconfig.json`: TypeScript 配置
- `package.json`: 项目配置

## 变更记录 (Changelog)
- 2025-12-05: 初始化 frontend 模块文档