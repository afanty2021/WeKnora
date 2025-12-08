# API参考

<cite>
**本文档引用的文件**   
- [README.md](file://docs/api/README.md)
- [API.md](file://docs/API.md)
- [chat.md](file://docs/api/chat.md)
- [session.md](file://docs/api/session.md)
- [knowledge-base.md](file://docs/api/knowledge-base.md)
- [knowledge.md](file://docs/api/knowledge.md)
- [model.md](file://docs/api/model.md)
- [message.md](file://docs/api/message.md)
- [tag.md](file://docs/api/tag.md)
- [faq.md](file://docs/api/faq.md)
- [evaluation.md](file://docs/api/evaluation.md)
- [handler.go](file://internal/handler/session/handler.go)
- [knowledgebase.go](file://internal/handler/knowledgebase.go)
- [chunk.go](file://internal/handler/chunk.go)
- [knowledge.go](file://internal/handler/knowledge.go)
- [session.go](file://internal/types/session.go)
- [knowledgebase.go](file://internal/types/knowledgebase.go)
- [knowledge.go](file://internal/types/knowledge.go)
- [chunk.go](file://internal/types/chunk.go)
- [message.go](file://internal/types/message.go)
</cite>

## 目录
1. [概述](#概述)
2. [基础信息](#基础信息)
3. [认证机制](#认证机制)
4. [错误处理](#错误处理)
5. [API概览](#api概览)
6. [会话管理API](#会话管理api)
7. [知识库管理API](#知识库管理api)
8. [知识管理API](#知识管理api)
9. [分块管理API](#分块管理api)
10. [聊天功能API](#聊天功能api)
11. [消息管理API](#消息管理api)
12. [模型管理API](#模型管理api)
13. [标签管理API](#标签管理api)
14. [FAQ管理API](#faq管理api)
15. [评估功能API](#评估功能api)

## 概述

WeKnora提供了一系列RESTful API，用于创建和管理知识库、检索知识，以及进行基于知识的问答。本文档详细描述了这些API的使用方式。

## 基础信息

- **基础URL**: `/api/v1`
- **响应格式**: JSON
- **认证方式**: API Key

## 认证机制

所有API请求需要在HTTP请求头中包含`X-API-Key`进行身份认证：

```
X-API-Key: your_api_key
```

为便于问题追踪和调试，建议每个请求的HTTP请求头中添加`X-Request-ID`：

```
X-Request-ID: unique_request_id
```

### 获取API Key

在web页面完成账户注册后，请前往账户信息页面获取您的API Key。

请妥善保管您的API Key，避免泄露。API Key代表您的账户身份，拥有完整的API访问权限。

## 错误处理

所有API使用标准的HTTP状态码表示请求状态，并返回统一的错误响应格式：

```json
{
  "success": false,
  "error": {
    "code": "错误代码",
    "message": "错误信息",
    "details": "错误详情"
  }
}
```

## API概览

WeKnora API按功能分为以下几类：

| 分类 | 描述 | 文档链接 |
|------|------|----------|
| 租户管理 | 创建和管理租户账户 | [tenant.md](./tenant.md) |
| 知识库管理 | 创建、查询和管理知识库 | [knowledge-base.md](./knowledge-base.md) |
| 知识管理 | 上传、检索和管理知识内容 | [knowledge.md](./knowledge.md) |
| 模型管理 | 配置和管理各种AI模型 | [model.md](./model.md) |
| 分块管理 | 管理知识的分块内容 | [chunk.md](./chunk.md) |
| 标签管理 | 管理知识库的标签分类 | [tag.md](./tag.md) |
| FAQ管理 | 管理FAQ问答对 | [faq.md](./faq.md) |
| 会话管理 | 创建和管理对话会话 | [session.md](./session.md) |
| 聊天功能 | 基于知识库和Agent进行问答 | [chat.md](./chat.md) |
| 消息管理 | 获取和管理对话消息 | [message.md](./message.md) |
| 评估功能 | 评估模型性能 | [evaluation.md](./evaluation.md) |

**Section sources**
- [README.md](file://docs/api/README.md#L1-L73)

## 会话管理API

会话（Session）是WeKnora中对话的核心单元，用于维护用户与系统交互的上下文。每个会话都关联特定的配置策略，如最大轮数、改写开关、兜底策略等。

### POST `/sessions` - 创建会话

创建一个新的对话会话。

**请求参数**:

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| knowledge_base_id | string | 否 | 关联的知识库ID |
| session_strategy | object | 否 | 会话策略配置，若不提供则使用租户默认配置 |

**session_strategy对象**:

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| max_rounds | int | 否 | 多轮保持轮数 |
| enable_rewrite | bool | 否 | 多轮改写开关 |
| fallback_strategy | string | 否 | 兜底策略，可选值：`FIXED_RESPONSE`, `MODEL` |
| fallback_response | string | 否 | 固定回复内容 |
| embedding_top_k | int | 否 | 向量召回TopK |
| keyword_threshold | float | 否 | 关键词召回阈值 |
| vector_threshold | float | 否 | 向量召回阈值 |
| rerank_model_id | string | 否 | 排序模型ID |
| rerank_top_k | int | 否 | 排序TopK |
| rerank_threshold | float | 否 | 排序阈值 |
| summary_model_id | string | 否 | 总结模型ID |
| summary_parameters | object | 否 | 总结模型参数 |

**请求示例**:
```curl
curl --location 'http://localhost:8080/api/v1/sessions' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--header 'Content-Type: application/json' \
--data '{
    "knowledge_base_id": "kb-00000001",
    "session_strategy": {
        "max_rounds": 5,
        "enable_rewrite": true,
        "fallback_strategy": "FIXED_RESPONSE",
        "fallback_response": "对不起，我无法回答这个问题",
        "embedding_top_k": 10,
        "keyword_threshold": 0.5,
        "vector_threshold": 0.7,
        "rerank_model_id": "排序模型ID",
        "rerank_top_k": 3,
        "rerank_threshold": 0.7,
        "summary_model_id": "8aea788c-bb30-4898-809e-e40c14ffb48c",
        "summary_parameters": {
            "max_tokens": 0,
            "repeat_penalty": 1,
            "top_k": 0,
            "top_p": 0,
            "frequency_penalty": 0,
            "presence_penalty": 0,
            "prompt": "这是用户和助手之间的对话。xxx",
            "context_template": "你是一个专业的智能信息检索助手xxx",
            "no_match_prefix": "NO_MATCH",
            "temperature": 0.3,
            "seed": 0,
            "max_completion_tokens": 2048
        }
    }
}'
```

**响应示例**:
```json
{
    "data": {
        "id": "411d6b70-9a85-4d03-bb74-aab0fd8bd12f",
        "title": "",
        "description": "",
        "tenant_id": 1,
        "knowledge_base_id": "kb-00000001",
        "max_rounds": 5,
        "enable_rewrite": true,
        "fallback_strategy": "FIXED_RESPONSE",
        "fallback_response": "对不起，我无法回答这个问题",
        "embedding_top_k": 10,
        "keyword_threshold": 0.5,
        "vector_threshold": 0.7,
        "rerank_model_id": "排序模型ID",
        "rerank_top_k": 3,
        "rerank_threshold": 0.7,
        "summary_model_id": "8aea788c-bb30-4898-809e-e40c14ffb48c",
        "summary_parameters": {
            "max_tokens": 0,
            "repeat_penalty": 1,
            "top_k": 0,
            "top_p": 0,
            "frequency_penalty": 0,
            "presence_penalty": 0,
            "prompt": "这是用户和助手之间的对话。xxx",
            "context_template": "你是一个专业的智能信息检索助手xxx",
            "no_match_prefix": "NO_MATCH",
            "temperature": 0.3,
            "seed": 0,
            "max_completion_tokens": 2048
        },
        "agent_config": null,
        "context_config": null,
        "created_at": "2025-08-12T12:26:19.611616669+08:00",
        "updated_at": "2025-08-12T12:26:19.611616919+08:00",
        "deleted_at": null
    },
    "success": true
}
```

**Section sources**
- [session.md](file://docs/api/session.md#L1-L54)
- [handler.go](file://internal/handler/session/handler.go#L42-L149)
- [session.go](file://internal/types/session.go#L73-L106)

### GET `/sessions/:id` - 获取会话详情

根据会话ID获取会话的详细信息。

**请求示例**:
```curl
curl --location 'http://localhost:8080/api/v1/sessions/ceb9babb-1e30-41d7-817d-fd584954304b' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--header 'Content-Type: application/json'
```

**响应示例**:
```json
{
    "data": {
        "id": "ceb9babb-1e30-41d7-817d-fd584954304b",
        "title": "模型优化策略",
        "description": "",
        "tenant_id": 1,
        "knowledge_base_id": "kb-00000001",
        "max_rounds": 5,
        "enable_rewrite": true,
        "fallback_strategy": "fixed",
        "fallback_response": "抱歉，我无法回答这个问题。",
        "embedding_top_k": 10,
        "keyword_threshold": 0.3,
        "vector_threshold": 0.5,
        "rerank_model_id": "",
        "rerank_top_k": 5,
        "rerank_threshold": 0.7,
        "summary_model_id": "8aea788c-bb30-4898-809e-e40c14ffb48c",
        "summary_parameters": {
            "max_tokens": 0,
            "repeat_penalty": 1,
            "top_k": 0,
            "top_p": 0,
            "frequency_penalty": 0,
            "presence_penalty": 0,
            "prompt": "这是用户和助手之间的对话",
            "context_template": "你是一个专业的智能信息检索助手",
            "no_match_prefix": "NO_MATCH",
            "temperature": 0.3,
            "seed": 0,
            "max_completion_tokens": 2048
        },
        "agent_config": null,
        "context_config": null,
        "created_at": "2025-08-12T10:24:38.308596+08:00",
        "updated_at": "2025-08-12T10:25:41.317761+08:00",
        "deleted_at": null
    },
    "success": true
}
```

**Section sources**
- [session.md](file://docs/api/session.md#L101-L154)
- [handler.go](file://internal/handler/session/handler.go#L190-L224)

### GET `/sessions` - 获取租户的会话列表

获取当前租户的所有会话，支持分页查询。

**请求参数**:
- `page`: 页码（默认1）
- `page_size`: 每页条数（默认20）

**请求示例**:
```curl
curl --location 'http://localhost:8080/api/v1/sessions?page=1&page_size=1' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--header 'Content-Type: application/json'
```

**响应示例**:
```json
{
    "data": [
        {
            "id": "411d6b70-9a85-4d03-bb74-aab0fd8bd12f",
            "title": "",
            "description": "",
            "tenant_id": 1,
            "knowledge_base_id": "kb-00000001",
            "max_rounds": 5,
            "enable_rewrite": true,
            "fallback_strategy": "FIXED_RESPONSE",
            "fallback_response": "对不起，我无法回答这个问题",
            "embedding_top_k": 10,
            "keyword_threshold": 0.5,
            "vector_threshold": 0.7,
            "rerank_model_id": "排序模型ID",
            "rerank_top_k": 3,
            "rerank_threshold": 0.7,
            "summary_model_id": "8aea788c-bb30-4898-809e-e40c14ffb48c",
            "summary_parameters": {
                "max_tokens": 0,
                "repeat_penalty": 1,
                "top_k": 0,
                "top_p": 0,
                "frequency_penalty": 0,
                "presence_penalty": 0,
                "prompt": "这是用户和助手之间的对话。xxx",
                "context_template": "你是一个专业的智能信息检索助手xxx",
                "no_match_prefix": "NO_MATCH",
                "temperature": 0.3,
                "seed": 0,
                "max_completion_tokens": 2048
            },
            "created_at": "2025-08-12T12:26:19.611616+08:00",
            "updated_at": "2025-08-12T12:26:19.611616+08:00",
            "deleted_at": null
        }
    ],
    "page": 1,
    "page_size": 1,
    "success": true,
    "total": 2
}
```

**Section sources**
- [session.md](file://docs/api/session.md#L156-L212)
- [handler.go](file://internal/handler/session/handler.go#L226-L254)

### PUT `/sessions/:id` - 更新会话

更新现有会话的属性。

**请求示例**:
```curl
curl --location --request PUT 'http://localhost:8080/api/v1/sessions/411d6b70-9a85-4d03-bb74-aab0fd8bd12f' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--header 'Content-Type: application/json' \
--data '{
    "title": "weknora",
    "description": "weknora description",
    "knowledge_base_id": "kb-00000001",
    "max_rounds": 5,
    "enable_rewrite": true,
    "fallback_strategy": "FIXED_RESPONSE",
    "fallback_response": "对不起，我无法回答这个问题",
    "embedding_top_k": 10,
    "keyword_threshold": 0.5,
    "vector_threshold": 0.7,
    "rerank_model_id": "排序模型ID",
    "rerank_top_k": 3,
    "rerank_threshold": 0.7,
    "summary_model_id": "8aea788c-bb30-4898-809e-e40c14ffb48c",
    "summary_parameters": {
        "max_tokens": 0,
        "repeat_penalty": 1,
        "top_k": 0,
        "top_p": 0,
        "frequency_penalty": 0,
        "presence_penalty": 0,
        "prompt": "这是用户和助手之间的对话。xxx",
        "context_template": "你是一个专业的智能信息检索助手xxx",
        "no_match_prefix": "NO_MATCH",
        "temperature": 0.3,
        "seed": 0,
        "max_completion_tokens": 2048
    }
}'
```

**响应示例**:
```json
{
    "data": {
        "id": "411d6b70-9a85-4d03-bb74-aab0fd8bd12f",
        "title": "weknora",
        "description": "weknora description",
        "tenant_id": 1,
        "knowledge_base_id": "kb-00000001",
        "max_rounds": 5,
        "enable_rewrite": true,
        "fallback_strategy": "FIXED_RESPONSE",
        "fallback_response": "对不起，我无法回答这个问题",
        "embedding_top_k": 10,
        "keyword_threshold": 0.5,
        "vector_threshold": 0.7,
        "rerank_model_id": "排序模型ID",
        "rerank_top_k": 3,
        "rerank_threshold": 0.7,
        "summary_model_id": "8aea788c-bb30-4898-809e-e40c14ffb48c",
        "summary_parameters": {
            "max_tokens": 0,
            "repeat_penalty": 1,
            "top_k": 0,
            "top_p": 0,
            "frequency_penalty": 0,
            "presence_penalty": 0,
            "prompt": "这是用户和助手之间的对话。xxx",
            "context_template": "你是一个专业的智能信息检索助手xxx",
            "no_match_prefix": "NO_MATCH",
            "temperature": 0.3,
            "seed": 0,
            "max_completion_tokens": 2048
        },
        "created_at": "0001-01-01T00:00:00Z",
        "updated_at": "2025-08-12T14:20:56.738424351+08:00",
        "deleted_at": null
    },
    "success": true
}
```

**Section sources**
- [session.md](file://docs/api/session.md#L214-L294)
- [handler.go](file://internal/handler/session/handler.go#L256-L305)

### DELETE `/sessions/:id` - 删除会话

根据会话ID删除一个会话。

**请求示例**:
```curl
curl --location --request DELETE 'http://localhost:8080/api/v1/sessions/411d6b70-9a85-4d03-bb74-aab0fd8bd12f' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--header 'Content-Type: application/json'
```

**响应示例**:
```json
{
    "message": "Session deleted successfully",
    "success": true
}
```

**Section sources**
- [session.md](file://docs/api/session.md#L297-L314)
- [handler.go](file://internal/handler/session/handler.go#L307-L336)

### POST `/sessions/:session_id/generate_title` - 生成会话标题

根据会话中的消息历史生成一个会话标题。

**请求参数**:
- `messages`: 消息数组，包含角色和内容

**请求示例**:
```curl
curl --location 'http://localhost:8080/api/v1/sessions/ceb9babb-1e30-41d7-817d-fd584954304b/generate_title' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--header 'Content-Type: application/json' \
--data '{
  "messages": [
    {
      "role": "user",
      "content": "你好，我想了解关于人工智能的知识"
    },
    {
      "role": "assistant",
      "content": "人工智能是计算机科学的一个分支..."
    }
  ]
}'
```

**响应示例**:
```json
{
    "data": "模型优化策略",
    "success": true
}
```

**Section sources**
- [session.md](file://docs/api/session.md#L316-L345)

### GET `/sessions/continue-stream/:session_id` - 继续未完成的会话

继续一个未完成的流式会话。

**查询参数**:
- `message_id`: 从`/messages/:session_id/load`接口中获取的`is_completed`为`false`的消息ID

**请求示例**:
```curl
curl --location 'http://localhost:8080/api/v1/sessions/continue-stream/ceb9babb-1e30-41d7-817d-fd584954304b?message_id=b8b90eeb-7dd5-4cf9-81c6-5ebcbd759451' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--header 'Content-Type: application/json'
```

**响应格式**:
服务器端事件流（Server-Sent Events），与`/knowledge-chat/:session_id`返回结果一致。

**Section sources**
- [session.md](file://docs/api/session.md#L347-L362)

## 知识库管理API

知识库（Knowledge Base）是WeKnora中组织和管理知识的核心容器。每个知识库都有独立的配置，如分块设置、嵌入模型、摘要模型等。

### POST `/knowledge-bases` - 创建知识库

创建一个新的知识库。

**请求参数**:

| 参数 | 类型 | 必填 | 说明 |
|------|------|------|------|
| name | string | 是 | 知识库名称 |
| description | string | 否 | 知识库描述 |
| chunking_config | object | 否 | 分块配置 |
| image_processing_config | object | 否 | 图像处理配置 |
| embedding_model_id | string | 否 | 嵌入模型ID |
| summary_model_id | string | 否 | 摘要模型ID |
| rerank_model_id | string | 否 | 排序模型ID |
| vlm_config | object | 否 | 视觉语言模型配置 |
| cos_config | object | 否 | 存储配置 |

**请求示例**:
```curl
curl --location 'http://localhost:8080/api/v1/knowledge-bases' \
--header 'Content-Type: application/json' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--data '{
    "name": "weknora",
    "description": "weknora description",
    "chunking_config": {
        "chunk_size": 1000,
        "chunk_overlap": 200,
        "separators": [
            "."
        ],
        "enable_multimodal": true
    },
    "image_processing_config": {
        "model_id": "f2083ad7-63e3-486d-a610-e6c56e58d72e"
    },
    "embedding_model_id": "dff7bc94-7885-4dd1-bfd5-bd96e4df2fc3",
    "summary_model_id": "8aea788c-bb30-4898-809e-e40c14ffb48c",
    "rerank_model_id": "b30171a1-787b-426e-a293-735cd5ac16c0",
    "vlm_config": {
        "enabled": true,
        "model_id": "f2083ad7-63e3-486d-a610-e6c56e58d72e"
    },
    "cos_config": {
        "secret_id": "",
        "secret_key": "",
        "region": "",
        "bucket_name": "",
        "app_id": "",
        "path_prefix": ""
    }
}'
```

**响应示例**:
```json
{
    "data": {
        "id": "b5829e4a-3845-4624-a7fb-ea3b35e843b0",
        "name": "weknora",
        "description": "weknora description",
        "tenant_id": 1,
        "chunking_config": {
            "chunk_size": 1000,
            "chunk_overlap": 200,
            "separators": [
                "."
            ],
            "enable_multimodal": true
        },
        "image_processing_config": {
            "model_id": "f2083ad7-63e3-486d-a610-e6c56e58d72e"
        },
        "embedding_model_id": "dff7bc94-7885-4dd1-bfd5-bd96e4df2fc3",
        "summary_model_id": "8aea788c-bb30-4898-809e-e40c14ffb48c",
        "rerank_model_id": "b30171a1-787b-426e-a293-735cd5ac16c0",
        "vlm_config": {
            "enabled": true,
            "model_id": "f2083ad7-63e3-486d-a610-e6c56e58d72e"
        },
        "cos_config": {
            "secret_id": "",
            "secret_key": "",
            "region": "",
            "bucket_name": "",
            "app_id": "",
            "path_prefix": ""
        },
        "created_at": "2025-08-12T11:30:09.206238645+08:00",
        "updated_at": "2025-08-12T11:30:09.206238854+08:00",
        "deleted_at": null
    },
    "success": true
}
```

**Section sources**
- [knowledge-base.md](file://docs/api/knowledge-base.md#L344-L425)
- [knowledgebase.go](file://internal/handler/knowledgebase.go#L71-L105)
- [knowledgebase.go](file://internal/types/knowledgebase.go#L38-L84)

### GET `/knowledge-bases` - 获取知识库列表

获取当前租户的所有知识库。

**请求示例**:
```curl
curl --location 'http://localhost:8080/api/v1/knowledge-bases' \
--header 'Content-Type: application/json' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ'
```

**响应示例**:
```json
{
    "data": [
        {
            "id": "kb-00000001",
            "name": "Default Knowledge Base",
            "description": "System Default Knowledge Base",
            "tenant_id": 1,
            "chunking_config": {
                "chunk_size": 1000,
                "chunk_overlap": 200,
                "separators": [
                    "\n\n",
                    "\n",
                    "。",
                    "！",
                    "？",
                    ";",
                    "；"
                ],
                "enable_multimodal": true
            },
            "image_processing_config": {
                "model_id": ""
            },
            "embedding_model_id": "dff7bc94-7885-4dd1-bfd5-bd96e4df2fc3",
            "summary_model_id": "8aea788c-bb30-4898-809e-e40c14ffb48c",
            "rerank_model_id": "b30171a1-787b-426e-a293-735cd5ac16c0",
            "vlm_config": {
                "enabled": true,
                "model_id": "f2083ad7-63e3-486d-a610-e6c56e58d72e"
            },
            "cos_config": {
                "secret_id": "",
                "secret_key": "",
                "region": "",
                "bucket_name": "",
                "app_id": "",
                "path_prefix": ""
            },
            "created_at": "2025-08-11T20:10:41.817794+08:00",
            "updated_at": "2025-08-12T11:23:00.593097+08:00",
            "deleted_at": null
        }
    ],
    "success": true
}
```

**Section sources**
- [knowledge-base.md](file://docs/api/knowledge-base.md#L427-L486)
- [knowledgebase.go](file://internal/handler/knowledgebase.go#L161-L177)

### GET `/knowledge-bases/:id` - 获取知识库详情

根据知识库ID获取知识库的详细信息。

**请求示例**:
```curl
curl --location 'http://localhost:8080/api/v1/knowledge-bases/kb-00000001' \
--header 'Content-Type: application/json' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ'
```

**响应示例**:
```json
{
    "data": {
        "id": "kb-00000001",
        "name": "Default Knowledge Base",
        "description": "System Default Knowledge Base",
        "tenant_id": 1,
        "chunking_config": {
            "chunk_size": 1000,
            "chunk_overlap": 200,
            "separators": [
                "\n\n",
                "\n",
                "。",
                "！",
                "？",
                ";",
                "；"
            ],
            "enable_multimodal": true
        },
        "image_processing_config": {
            "model_id": ""
        },
        "embedding_model_id": "dff7bc94-7885-4dd1-bfd5-bd96e4df2fc3",
        "summary_model_id": "8aea788c-bb30-4898-809e-e40c14ffb48c",
        "rerank_model_id": "b30171a1-787b-426e-a293-735cd5ac16c0",
        "vlm_config": {
            "enabled": true,
            "model_id": "f2083ad7-63e3-486d-a610-e6c56e58d72e"
        },
        "cos_config": {
            "secret_id": "",
            "secret_key": "",
            "region": "",
            "bucket_name": "",
            "app_id": "",
            "path_prefix": ""
        },
        "created_at": "2025-08-11T20:10:41.817794+08:00",
        "updated_at": "2025-08-12T11:23:00.593097+08:00",
        "deleted_at": null
    },
    "success": true
}
```

**Section sources**
- [knowledge-base.md](file://docs/api/knowledge-base.md#L488-L545)
- [knowledgebase.go](file://internal/handler/knowledgebase.go#L147-L159)

### PUT `/knowledge-bases/:id` - 更新知识库

更新现有知识库的配置。

**请求示例**:
```curl
curl --location --request PUT 'http://localhost:8080/api/v1/knowledge-bases/b5829e4a-3845-4624-a7fb-ea3b35e843b0' \
--header 'Content-Type: application/json' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--data '{
    "name": "weknora new",
    "description": "weknora description new",
    "config": {
        "chunking_config": {
            "chunk_size": 1000,
            "chunk_overlap": 200,
            "separators": [
                "\n\n",
                "\n",
                "。",
                "！",
                "？",
                ";",
                "；"
            ],
            "enable_multimodal": true
        },
        "image_processing_config": {
            "model_id": ""
        }
    }
}'
```

**响应示例**:
```json
{
    "data": {
        "id": "b5829e4a-3845-4624-a7fb-ea3b35e843b0",
        "name": "weknora new",
        "description": "weknora description new",
        "tenant_id": 1,
        "chunking_config": {
            "chunk_size": 1000,
            "chunk_overlap": 200,
            "separators": [
                "\n\n",
                "\n",
                "。",
                "！",
                "？",
                ";",
                "；"
            ],
            "enable_multimodal": true
        },
        "image_processing_config": {
            "model_id": ""
        },
        "embedding_model_id": "dff7bc94-7885-4dd1-bfd5-bd96e4df2fc3",
        "summary_model_id": "8aea788c-bb30-4898-809e-e40c14ffb48c",
        "rerank_model_id": "b30171a1-787b-426e-a293-735cd5ac16c0",
        "vlm_config": {
            "enabled": true,
            "model_id": "f2083ad7-63e3-486d-a610-e6c56e58d72e"
        },
        "cos_config": {
            "secret_id": "",
            "secret_key": "",
            "region": "",
            "bucket_name": "",
            "app_id": "",
            "path_prefix": ""
        },
        "created_at": "2025-08-12T11:30:09.206238+08:00",
        "updated_at": "2025-08-12T11:36:09.083577609+08:00",
        "deleted_at": null
    },
    "success": true
}
```

**Section sources**
- [knowledge-base.md](file://docs/api/knowledge-base.md#L547-L627)
- [knowledgebase.go](file://internal/handler/knowledgebase.go#L186-L223)

### DELETE `/knowledge-bases/:id` - 删除知识库

根据知识库ID删除一个知识库。

**请求示例**:
```curl
curl --location --request DELETE 'http://localhost:8080/api/v1/knowledge-bases/b5829e4a-3845-4624-a7fb-ea3b35e843b0' \
--header 'Content-Type: application/json' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ'
```

**响应示例**:
```json
{
    "message": "Knowledge base deleted successfully",
    "success": true
}
```

**Section sources**
- [knowledge-base.md](file://docs/api/knowledge-base.md#L629-L646)
- [knowledgebase.go](file://internal/handler/knowledgebase.go#L225-L253)

### GET `/knowledge-bases/:id/hybrid-search` - 混合搜索

执行向量搜索和关键词搜索的混合检索。

**注意**：此接口使用GET方法但需要JSON请求体。

**请求参数**：
- `query_text`: 搜索查询文本（必填）
- `vector_threshold`: 向量相似度阈值（0-1，可选）
- `keyword_threshold`: 关键词匹配阈值（可选）
- `match_count`: 返回结果数量（可选）
- `disable_keywords_match`: 是否禁用关键词匹配（可选）
- `disable_vector_match`: 是否禁用向量匹配（可选）

**请求示例**:
```curl
curl --location --request GET 'http://localhost:8080/api/v1/knowledge-bases/kb-00000001/hybrid-search' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--header 'Content-Type: application/json' \
--data '{
    "query_text": "如何使用知识库",
    "vector_threshold": 0.5,
    "match_count": 10
}'
```

**响应示例**:
```json
{
    "data": [
        {
            "id": "chunk-00000001",
            "content": "知识库是用于存储和检索知识的系统...",
            "knowledge_id": "knowledge-00000001",
            "chunk_index": 0,
            "knowledge_title": "知识库使用指南",
            "start_at": 0,
            "end_at": 500,
            "seq": 1,
            "score": 0.95,
            "chunk_type": "text",
            "image_info": "",
            "metadata": {},
            "knowledge_filename": "guide.pdf",
            "knowledge_source": "file"
        }
    ],
    "success": true
}
```

**Section sources**
- [knowledge-base.md](file://docs/api/knowledge-base.md#L648-L699)
- [knowledgebase.go](file://internal/handler/knowledgebase.go#L30-L69)

## 知识管理API

知识（Knowledge）是WeKnora中存储原始内容的单元，可以是文件、URL或手动创建的Markdown内容。

### POST `/knowledge-bases/:id/knowledge/file` - 从文件创建知识

从上传的文件创建知识。

**表单参数**：
- `file`: 上传的文件（必填）
- `metadata`: JSON格式的元数据（可选）
- `enable_multimodel`: 是否启用多模态处理（可选，true/false）
- `fileName`: 自定义文件名，用于文件夹上传时保留路径（可选）

**请求示例**:
```curl
curl --location 'http://localhost:8080/api/v1/knowledge-bases/kb-00000001/knowledge/file' \
--header 'Content-Type: application/json' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--form 'file=@"/Users/xxxx/tests/彗星.txt"' \
--form 'enable_multimodel="true"'
```

**响应示例**:
```json
{
    "data": {
        "id": "4c4e7c1a-09cf-485b-a7b5-24b8cdc5acf5",
        "tenant_id": 1,
        "knowledge_base_id": "kb-00000001",
        "type": "file",
        "title": "彗星.txt",
        "description": "",
        "source": "",
        "parse_status": "processing",
        "enable_status": "disabled",
        "embedding_model_id": "dff7bc94-7885-4dd1-bfd5-bd96e4df2fc3",
        "file_name": "彗星.txt",
        "file_type": "txt",
        "file_size": 7710,
        "file_hash": "d69476ddbba45223a5e97e786539952c",
        "file_path": "data/files/1/4c4e7c1a-09cf-485b-a7b5-24b8cdc5acf5/1754970756171067621.txt",
        "storage_size": 0,
        "metadata": null,
        "created_at": "2025-08-12T11:52:36.168632288+08:00",
        "updated_at": "2025-08-12T11:52:36.173612121+08:00",
        "processed_at": null,
        "error_message": "",
        "deleted_at": null
    },
    "success": true
}
```

**Section sources**
- [knowledge.md](file://docs/api/knowledge.md#L720-L768)
- [knowledge.go](file://internal/handler/knowledge.go#L85-L168)

### POST `/knowledge-bases/:id/knowledge/url` - 从URL创建知识

从指定的URL创建知识。

**请求示例**:
```curl
curl --location 'http://localhost:8080/api/v1/knowledge-bases/kb-00000001/knowledge/url' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--header 'Content-Type: application/json' \
--data '{
    "url":"https://github.com/Tencent/WeKnora",
    "enable_multimodel":true
}'
```

**响应示例**:
```json
{
    "data": {
        "id": "9c8af585-ae15-44ce-8f73-45ad18394651",
        "tenant_id": 1,
        "knowledge_base_id": "kb-00000001",
        "type": "url",
        "title": "",
        "description": "",
        "source": "https://github.com/Tencent/WeKnora",
        "parse_status": "processing",
        "enable_status": "disabled",
        "embedding_model_id": "dff7bc94-7885-4dd1-bfd5-bd96e4df2fc3",
        "file_name": "",
        "file_type": "",
        "file_size": 0,
        "file_hash": "",
        "file_path": "",
        "storage_size": 0,
        "metadata": null,
        "created_at": "2025-08-12T11:55:04.169527445+08:00",
        "updated_at": "2025-08-12T11:55:04.171525455+08:00",
        "processed_at": null,
        "error_message": "",
        "deleted_at": null
    },
    "success": true
}
```

**Section sources**
- [knowledge.md](file://docs/api/knowledge.md#L770-L786)
- [knowledge.go](file://internal/handler/knowledge.go#L170-L224)

### POST `/knowledge-bases/:id/knowledge/manual` - 创建手工Markdown知识

创建手动输入的Markdown知识。

**请求示例**:
```curl
curl --location 'http://localhost:8080/api/v1/knowledge-bases/kb-00000001/knowledge/manual' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--header 'Content-Type: application/json' \
--data '{
    "title": "快速入门指南",
    "content": "# WeKnora 快速入门\n\n欢迎使用WeKnora...",
    "status": "publish"
}'
```

**响应示例**:
```json
{
    "data": {
        "id": "manual-00000001",
        "tenant_id": 1,
        "knowledge_base_id": "kb-00000001",
        "type": "manual",
        "title": "快速入门指南",
        "description": "",
        "source": "manual",
        "parse_status": "completed",
        "enable_status": "enabled",
        "embedding_model_id": "dff7bc94-7885-4dd1-bfd5-bd96e4df2fc3",
        "file_name": "manual",
        "file_type": "manual",
        "file_size": 0,
        "file_hash": "",
        "file_path": "",
        "storage_size": 0,
        "metadata": "{\"content\":\"# WeKnora 快速入门\\n\\n欢迎使用WeKnora...\",\"format\":\"markdown\",\"status\":\"publish\",\"version\":1,\"updated_at\":\"2025-08-12T12:00:00Z\"}",
        "created_at": "2025-08-12T12:00:00.000000+08:00",
        "updated_at": "2025-08-12T12:00:00.000000+08:00",
        "processed_at": "2025-08-12T12:00:00.000000+08:00",
        "error_message": "",
        "deleted_at": null
    },
    "success": true
}
```

**Section sources**
- [knowledge.md](file://docs/api/knowledge.md#L226-L263)
- [knowledge.go](file://internal/handler/knowledge.go#L226-L263)

### GET `/knowledge-bases/:id/knowledge` - 获取知识库下的知识列表

获取指定知识库下的所有知识，支持分页和过滤。

**请求参数**:
- `page`: 页码（默认1）
- `page_size`: 每页条数（默认20）
- `tag_id`: 按标签ID筛选（可选）
- `keyword`: 按标题或内容关键字搜索（可选）
- `file_type`: 按文件类型筛选（可选）

**请求示例**:
```curl
curl --location 'http://localhost:8080/api/v1/knowledge-bases/kb-00000001/knowledge?page=1&page_size=10' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--header 'Content-Type: application/json'
```

**响应示例**:
```json
{
    "data": [
        {
            "id": "4c4e7c1a-09cf-485b-a7b5-24b8cdc5acf5",
            "tenant_id": 1,
            "knowledge_base_id": "kb-00000001",
            "type": "file",
            "title": "彗星.txt",
            "description": "",
            "source": "",
            "parse_status": "completed",
            "enable_status": "enabled",
            "embedding_model_id": "dff7bc94-7885-4dd1-bfd5-bd96e4df2fc3",
            "file_name": "彗星.txt",
            "file_type": "txt",
            "file_size": 7710,
            "file_hash": "d69476ddbba45223a5e97e786539952c",
            "file_path": "data/files/1/4c4e7c1a-09cf-485b-a7b5-24b8cdc5acf5/1754970756171067621.txt",
            "storage_size": 7710,
            "metadata": null,
            "created_at": "2025-08-12T11:52:36.168632+08:00",
            "updated_at": "2025-08-12T11:52:36.173612+08:00",
            "processed_at": "2025-08-12T11:52:36.173612+08:00",
            "error_message": "",
            "deleted_at": null
        }
    ],
    "page": 1,
    "page_size": 10,
    "success": true,
    "total": 1
}
```

**Section sources**
- [knowledge.md](file://docs/api/knowledge.md#L299-L357)
- [knowledge.go](file://internal/handler/knowledge.go#L299-L357)

### GET `/knowledge/:id` - 获取知识详情

根据知识ID获取知识的详细信息。

**请求示例**:
```curl
curl --location 'http://localhost:8080/api/v1/knowledge/4c4e7c1a-09cf-485b-a7b5-24b8cdc5acf5' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--header 'Content-Type: application/json'
```

**响应示例**:
```json
{
    "data": {
        "id": "4c4e7c1a-09cf-485b-a7b5-24b8cdc5acf5",
        "tenant_id": 1,
        "knowledge_base_id": "kb-00000001",
        "type": "file",
        "title": "彗星.txt",
        "description": "",
        "source": "",
        "parse_status": "completed",
        "enable_status": "enabled",
        "embedding_model_id": "dff7bc94-7885-4dd1-bfd5-bd96e4df2fc3",
        "file_name": "彗星.txt",
        "file_type": "txt",
        "file_size": 7710,
        "file_hash": "d69476ddbba45223a5e97e786539952c",
        "file_path": "data/files/1/4c4e7c1a-09cf-485b-a7b5-24b8cdc5acf5/1754970756171067621.txt",
        "storage_size": 7710,
        "metadata": null,
        "created_at": "2025-08-12T11:52:36.168632+08:00",
        "updated_at": "2025-08-12T11:52:36.173612+08:00",
        "processed_at": "2025-08-12T11:52:36.173612+08:00",
        "error_message": "",
        "deleted_at": null
    },
    "success": true
}
```

**Section sources**
- [knowledge.md](file://docs/api/knowledge.md#L265-L297)
- [knowledge.go](file://internal/handler/knowledge.go#L266-L297)

### DELETE `/knowledge/:id` - 删除知识

根据知识ID删除一个知识。

**请求示例**:
```curl
curl --location --request DELETE 'http://localhost:8080/api/v1/knowledge/4c4e7c1a-09cf-485b-a7b5-24b8cdc5acf5' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--header 'Content-Type: application/json'
```

**响应示例**:
```json
{
    "success": true,
    "message": "Deleted successfully"
}
```

**Section sources**
- [knowledge.md](file://docs/api/knowledge.md#L359-L386)
- [knowledge.go](file://internal/handler/knowledge.go#L360-L386)

### GET `/knowledge/:id/download` - 下载知识文件

下载与知识关联的原始文件。

**请求示例**:
```curl
curl --location 'http://localhost:8080/api/v1/knowledge/4c4e7c1a-09cf-485b-a7b5-24b8cdc5acf5/download' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--header 'Content-Type: application/json'
```

**响应**:
返回文件流，Content-Type为`application/octet-stream`。

**Section sources**
- [knowledge.md](file://docs/api/knowledge.md#L388-L438)
- [knowledge.go](file://internal/handler/knowledge.go#L389-L438)

### PUT `/knowledge/:id` - 更新知识

更新现有知识的元数据。

**请求示例**:
```curl
curl --location --request PUT 'http://localhost:8080/api/v1/knowledge/4c4e7c1a-09cf-485b-a7b5-24b8cdc5acf5' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--header 'Content-Type: application/json' \
--data '{
    "title": "更新后的标题",
    "description": "更新后的描述"
}'
```

**响应示例**:
```json
{
    "success": true,
    "message": "Knowledge chunk updated successfully"
}
```

**Section sources**
- [knowledge.md](file://docs/api/knowledge.md#L492-L520)
- [knowledge.go](file://internal/handler/knowledge.go#L492-L520)

### PUT `/knowledge/manual/:id` - 更新手工Markdown知识

更新手动创建的Markdown知识内容。

**请求示例**:
```curl
curl --location --request PUT 'http://localhost:8080/api/v1/knowledge/manual/manual-00000001' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--header 'Content-Type: application/json' \
--data '{
    "title": "更新后的快速入门指南",
    "content": "# WeKnora 快速入门\n\n欢迎使用更新版的WeKnora...",
    "status": "publish"
}'
```

**响应示例**:
```json
{
    "data": {
        "id": "manual-00000001",
        "tenant_id": 1,
        "knowledge_base_id": "kb-00000001",
        "type": "manual",
        "title": "更新后的快速入门指南",
        "description": "",
        "source": "manual",
        "parse_status": "completed",
        "enable_status": "enabled",
        "embedding_model_id": "dff7bc94-7885-4dd1-bfd5-bd96e4df2fc3",
        "file_name": "manual",
        "file_type": "manual",
        "file_size": 0,
        "file_hash": "",
        "file_path": "",
        "storage_size": 0,
        "metadata": "{\"content\":\"# WeKnora 快速入门\\n\\n欢迎使用更新版的WeKnora...\",\"format\":\"markdown\",\"status\":\"publish\",\"version\":2,\"updated_at\":\"2025-08-12T12:05:00Z\"}",
        "created_at": "2025-08-12T12:00:00.000000+08:00",
        "updated_at": "2025-08-12T12:05:00.000000+08:00",
        "processed_at": "2025-08-12T12:05:00.000000+08:00",
        "error_message": "",
        "deleted_at": null
    },
    "success": true
}
```

**Section sources**
- [knowledge.md](file://docs/api/knowledge.md#L522-L559)
- [knowledge.go](file://internal/handler/knowledge.go#L522-L559)

### PUT `/knowledge/tags` - 批量更新知识标签

批量更新多个知识的标签。

**请求示例**:
```curl
curl --location --request PUT 'http://localhost:8080/api/v1/knowledge/tags' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--header 'Content-Type: application/json' \
--data '{
    "updates": {
        "knowledge-00000001": "tag-00000001",
        "knowledge-00000002": "tag-00000002"
    }
}'
```

**响应示例**:
```json
{
    "success": true
}
```

**Section sources**
- [knowledge.md](file://docs/api/knowledge.md#L565-L582)
- [knowledge.go](file://internal/handler/knowledge.go#L566-L582)

### GET `/knowledge/batch` - 批量获取知识

批量获取多个知识的详细信息。

**请求参数**:
- `ids`: 知识ID数组（必填）

**请求示例**:
```curl
curl --location 'http://localhost:8080/api/v1/knowledge/batch?ids=knowledge-00000001&ids=knowledge-00000002' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--header 'Content-Type: application/json'
```

**响应示例**:
```json
{
    "data": [
        {
            "id": "knowledge-00000001",
            "tenant_id": 1,
            "knowledge_base_id": "kb-00000001",
            "type": "file",
            "title": "文档1.txt",
            "description": "",
            "source": "",
            "parse_status": "completed",
            "enable_status": "enabled",
            "embedding_model_id": "dff7bc94-7885-4dd1-bfd5-bd96e4df2fc3",
            "file_name": "文档1.txt",
            "file_type": "txt",
            "file_size": 1000,
            "file_hash": "hash1",
            "file_path": "path1",
            "storage_size": 1000,
            "metadata": null,
            "created_at": "2025-08-12T12:10:00.000000+08:00",
            "updated_at": "2025-08-12T12:10:00.000000+08:00",
            "processed_at": "2025-08-12T12:10:00.000000+08:00",
            "error_message": "",
            "deleted_at": null
        },
        {
            "id": "knowledge-00000002",
            "tenant_id": 1,
            "knowledge_base_id": "kb-00000001",
            "type": "file",
            "title": "文档2.txt",
            "description": "",
            "source": "",
            "parse_status": "completed",
            "enable_status": "enabled",
            "embedding_model_id": "dff7bc94-7885-4dd1-bfd5-bd96e4df2fc3",
            "file_name": "文档2.txt",
            "file_type": "txt",
            "file_size": 2000,
            "file_hash": "hash2",
            "file_path": "path2",
            "storage_size": 2000,
            "metadata": null,
            "created_at": "2025-08-12T12:11:00.000000+08:00",
            "updated_at": "2025-08-12T12:11:00.000000+08:00",
            "processed_at": "2025-08-12T12:11:00.000000+08:00",
            "error_message": "",
            "deleted_at": null
        }
    ],
    "success": true
}
```

**Section sources**
- [knowledge.md](file://docs/api/knowledge.md#L440-L490)
- [knowledge.go](file://internal/handler/knowledge.go#L446-L490)

## 分块管理API

分块（Chunk）是知识库中最小的检索单元，由原始知识内容分割而成。

### GET `/knowledge-bases/:id/chunks/:chunk_id` - 获取分块详情

根据分块ID获取分块的详细信息。

**请求示例**:
```curl
curl --location 'http://localhost:8080/api/v1/knowledge-bases/kb-00000001/chunks/chunk-00000001' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--header 'Content-Type: application/json'
```

**响应示例**:
```json
{
    "data": {
        "id": "chunk-00000001",
        "tenant_id": 1,
        "knowledge_id": "knowledge-00000001",
        "knowledge_base_id": "kb-00000001",
        "content": "这是分块的内容...",
        "chunk_index": 0,
        "is_enabled": true,
        "status": 2,
        "start_at": 0,
        "end_at": 100,
        "pre_chunk_id": "",
        "next_chunk_id": "chunk-00000002",
        "chunk_type": "text",
        "parent_chunk_id": "",
        "relation_chunks": [],
        "indirect_relation_chunks": [],
        "metadata": {},
        "content_hash": "",
        "image_info": "",
        "created_at": "2025-08-12T12:15:00.000000+08:00",
        "updated_at": "2025-08-12T12:15:00.000000+08:00",
        "deleted_at": null
    },
    "success": true
}
```

**Section sources**
- [chunk.go](file://internal/handler/chunk.go#L25-L80)

### GET `/knowledge/:knowledge_id/chunks` - 获取知识的分块列表

获取指定知识下的所有分块，支持分页。

**请求参数**:
- `page`: 页码（默认1）
- `page_size`: 每页条数（默认20）

**请求示例**:
```curl
curl --location 'http://localhost:8080/api/v1/knowledge/knowledge-00000001/chunks?page=1&page_size=10' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--header 'Content-Type: application/json'
```

**响应示例**:
```json
{
    "data": [
        {
            "id": "chunk-00000001",
            "tenant_id": 1,
            "knowledge_id": "knowledge-00000001",
            "knowledge_base_id": "kb-00000001",
            "content": "这是分块的内容...",
            "chunk_index": 0,
            "is_enabled": true,
            "status": 2,
            "start_at": 0,
            "end_at": 100,
            "pre_chunk_id": "",
            "next_chunk_id": "chunk-00000002",
            "chunk_type": "text",
            "parent_chunk_id": "",
            "relation_chunks": [],
            "indirect_relation_chunks": [],
            "metadata": {},
            "content_hash": "",
            "image_info": "",
            "created_at": "2025-08-12T12:15:00.000000+08:00",
            "updated_at": "2025-08-12T12:15:00.000000+08:00",
            "deleted_at": null
        }
    ],
    "page": 1,
    "page_size": 10,
    "success": true,
    "total": 1
}
```

**Section sources**
- [chunk.go](file://internal/handler/chunk.go#L82-L135)

### PUT `/knowledge/:knowledge_id/chunks/:chunk_id` - 更新分块

更新现有分块的属性。

**请求示例**:
```curl
curl --location --request PUT 'http://localhost:8080/api/v1/knowledge/knowledge-00000001/chunks/chunk-00000001' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--header 'Content-Type: application/json' \
--data '{
    "content": "更新后的分块内容...",
    "is_enabled": false
}'
```

**响应示例**:
```json
{
    "data": {
        "id": "chunk-00000001",
        "tenant_id": 1,
        "knowledge_id": "knowledge-00000001",
        "knowledge_base_id": "kb-00000001",
        "content": "更新后的分块内容...",
        "chunk_index": 0,
        "is_enabled": false,
        "status": 2,
        "start_at": 0,
        "end_at": 100,
        "pre_chunk_id": "",
        "next_chunk_id": "chunk-00000002",
        "chunk_type": "text",
        "parent_chunk_id": "",
        "relation_chunks": [],
        "indirect_relation_chunks": [],
        "metadata": {},
        "content_hash": "",
        "image_info": "",
        "created_at": "2025-08-12T12:15:00.000000+08:00",
        "updated_at": "2025-08-12T12:20:00.000000+08:00",
        "deleted_at": null
    },
    "success": true
}
```

**Section sources**
- [chunk.go](file://internal/handler/chunk.go#L203-L240)

### DELETE `/knowledge/:knowledge_id/chunks/:chunk_id` - 删除分块

根据分块ID删除一个分块。

**请求示例**:
```curl
curl --location --request DELETE 'http://localhost:8080/api/v1/knowledge/knowledge-00000001/chunks/chunk-00000001' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--header 'Content-Type: application/json'
```

**响应示例**:
```json
{
    "success": true,
    "message": "Chunk deleted"
}
```

**Section sources**
- [chunk.go](file://internal/handler/chunk.go#L242-L264)

### DELETE `/knowledge/:knowledge_id/chunks` - 删除知识的所有分块

删除指定知识下的所有分块。

**请求示例**:
```curl
curl --location --request DELETE 'http://localhost:8080/api/v1/knowledge/knowledge-00000001/chunks' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--header 'Content-Type: application/json'
```

**响应示例**:
```json
{
    "success": true,
    "message": "All chunks under knowledge deleted"
}
```

**Section sources**
- [chunk.go](file://internal/handler/chunk.go#L266-L290)

### DELETE `/knowledge/chunks/:chunk_id/generated-question` - 删除生成的问题

删除由LLM为分块生成的问题。

**请求参数**:
- `question_id`: 要删除的问题ID

**请求示例**:
```curl
curl --location --request DELETE 'http://localhost:8080/api/v1/knowledge/chunks/chunk-00000001/generated-question' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--header 'Content-Type: application/json' \
--data '{
    "question_id": "question-00000001"
}'
```

**响应示例**:
```json
{
    "success": true,
    "message": "Generated question deleted"
}
```

**Section sources**
- [chunk.go](file://internal/handler/chunk.go#L292-L354)

## 聊天功能API

聊天功能API提供了基于知识库和Agent的问答能力。

### POST `/knowledge-chat/:session_id` - 基于知识库的问答

基于知识库进行问答，返回流式响应。

**请求参数**:
- `query`: 查询文本（必填）

**请求示例**:
```curl
curl --location 'http://localhost:8080/api/v1/knowledge-chat/ceb9babb-1e30-41d7-817d-fd584954304b' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--header 'Content-Type: application/json' \
--data '{
    "query": "彗尾的形状"
}'
```

**响应格式**:
服务器端事件流（Server-Sent Events，Content-Type: text/event-stream）

**响应示例**:
```
event: message
data: {"id":"3475c004-0ada-4306-9d30-d7f5efce50d2","response_type":"references","content":"","done":false,"knowledge_references":[{"id":"c8347bef-127f-4a22-b962-edf5a75386ec","content":"彗星xxx。","knowledge_id":"a6790b93-4700-4676-bd48-0d4804e1456b","chunk_index":0,"knowledge_title":"彗星.txt","start_at":0,"end_at":2760,"seq":0,"score":4.038836479187012,"match_type":3,"sub_chunk_id":["688821f0-40bf-428e-8cb6-541531ebeb76","c1e9903e-2b4d-4281-be15-0149288d45c2","7d955251-3f79-4fd5-a6aa-02f81e044091"],"metadata":{},"chunk_type":"text","parent_chunk_id":"","image_info":"","knowledge_filename":"彗星.txt","knowledge_source":""},{"id":"fa3aadee-cadb-4a84-9941-c839edc3e626","content":"# 文档名称\n彗星.txt\n\n# 摘要\n彗星是由冰和尘埃构成的太阳系小天体，接近太阳时会释放气体形成彗发和彗尾。其轨道周期差异大，来源包括柯伊伯带和奥尔特云。彗星与小行星的区别逐渐模糊，部分彗星已失去挥发物质，类似小行星。目前已知彗星数量众多，且存在系外彗星。彗星在古代被视为凶兆，现代研究揭示其复杂结构与起源。","knowledge_id":"a6790b93-4700-4676-bd48-0d4804e1456b","chunk_index":6,"knowledge_title":"彗星.txt","start_at":0,"end_at":0,"seq":6,"score":0.6131043121858466,"match_type":3,"sub_chunk_id":null,"metadata":{},"chunk_type":"summary","parent_chunk_id":"c8347bef-127f-4a22-b962-edf5a75386ec","image_info":"","knowledge_filename":"彗星.txt","knowledge_source":""}]}

event: message
data: {"id":"3475c004-0ada-4306-9d30-d7f5efce50d2","response_type":"answer","content":"表现为","done":false,"knowledge_references":null}

event: message
data: {"id":"3475c004-0ada-4306-9d30-d7f5efce50d2","response_type":"answer","content":"结构","done":false,"knowledge_references":null}

event: message
data: {"id":"3475c004-0ada-4306-9d30-d7f5efce50d2","response_type":"answer","content":"。","done":false,"knowledge_references":null}

event: message
data: {"id":"3475c004-0ada-4306-9d30-d7f5efce50d2","response_type":"answer","content":"","done":true,"knowledge_references":null}
```

**Section sources**
- [chat.md](file://docs/api/chat.md#L11-L44)

### POST `/agent-chat/:session_id` - 基于Agent的智能问答

使用Agent模式进行智能问答，支持工具调用、网络搜索等能力。

**请求参数**：
- `query`: 查询文本（必填）
- `knowledge_base_ids`: 知识库ID数组，可动态指定本次查询使用的知识库（可选）
- `agent_enabled`: 是否启用Agent模式（可选，默认false）
- `web_search_enabled`: 是否启用网络搜索（可选，默认false）
- `summary_model_id`: 覆盖会话默认的摘要模型ID（可选）
- `mcp_service_ids`: MCP服务白名单（可选）

**请求示例**:
```curl
curl --location 'http://localhost:8080/api/v1/agent-chat/ceb9babb-1e30-41d7-817d-fd584954304b' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--header 'Content-Type: application/json' \
--data '{
    "query": "帮我查询今天的天气",
    "agent_enabled": true,
    "web_search_enabled": true,
    "knowledge_base_ids": ["kb-00000001"]
}'
```

**响应格式**:
服务器端事件流（Server-Sent Events，Content-Type: text/event-stream）

**响应类型说明**：

| response_type | 描述 |
|---------------|------|
| `thinking` | Agent思考过程 |
| `tool_call` | 工具调用信息 |
| `tool_result` | 工具调用结果 |
| `references` | 知识库检索引用 |
| `answer` | 最终回答内容 |
| `reflection` | Agent反思内容 |
| `error` | 错误信息 |

**响应示例**:
```
event: message
data: {"id":"agent-001","response_type":"thinking","content":"用户想查询天气，我需要使用网络搜索工具...","done":false,"knowledge_references":null}

event: message
data: {"id":"agent-001","response_type":"tool_call","content":"","done":false,"knowledge_references":null,"data":{"tool_name":"web_search","arguments":{"query":"今天天气"}}}

event: message
data: {"id":"agent-001","response_type":"tool_result","content":"搜索结果：今天晴，气温25°C...","done":false,"knowledge_references":null}

event: message
data: {"id":"agent-001","response_type":"answer","content":"根据查询结果，今天天气晴朗，气温约25°C。","done":false,"knowledge_references":null}

event: message
data: {"id":"agent-001","response_type":"answer","content":"","done":true,"knowledge_references":null}
```

**Section sources**
- [chat.md](file://docs/api/chat.md#L46-L104)

### POST `/knowledge-search` - 基于知识库的搜索知识

仅执行知识库搜索，不生成回答。

**请求示例**:
```curl
curl --location 'http://localhost:8080/api/v1/knowledge-search' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--header 'Content-Type: application/json' \
--data '{
    "query": "彗尾的形状",
    "knowledge_base_id": "kb-00000001"
}'
```

**响应示例**:
```json
{
    "data": [
        {
            "id": "chunk-00000001",
            "content": "彗星xxx。",
            "knowledge_id": "a6790b93-4700-4676-bd48-0d4804e1456b",
            "chunk_index": 0,
            "knowledge_title": "彗星.txt",
            "start_at": 0,
            "end_at": 2760,
            "seq": 0,
            "score": 4.038836479187012,
            "match_type": 3,
            "sub_chunk_id": [
                "688821f0-40bf-428e-8cb6-541531ebeb76",
                "c1e9903e-2b4d-4281-be15-0149288d45c2",
                "7d955251-3f79-4fd5-a6aa-02f81e044091"
            ],
            "metadata": {},
            "chunk_type": "text",
            "parent_chunk_id": "",
            "image_info": "",
            "knowledge_filename": "彗星.txt",
            "knowledge_source": ""
        }
    ],
    "success": true
}
```

**Section sources**
- [chat.md](file://docs/api/chat.md#L9-L10)

## 消息管理API

消息（Message）是会话中的具体交互内容，记录了用户和系统的对话历史。

### GET `/messages/:session_id/load` - 获取最近的会话消息列表

获取指定会话的最近消息列表，支持分页。

**请求参数**:
- `before_time`: 上一次拉取的最早一条消息的created_at字段，为空拉取最近的消息
- `limit`: 每页条数（默认20）

**请求示例**:
```curl
curl --location --request GET 'http://localhost:8080/api/v1/messages/ceb9babb-1e30-41d7-817d-fd584954304b/load?limit=3&before_time=2030-08-12T14%3A35%3A42.123456789Z' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--header 'Content-Type: application/json'
```

**响应示例**:
```json
{
    "data": [
        {
            "id": "b8b90eeb-7dd5-4cf9-81c6-5ebcbd759451",
            "session_id": "ceb9babb-1e30-41d7-817d-fd584954304b",
            "request_id": "hCA8SDjxcAvv",
            "content": "好的",
            "role": "assistant",
            "knowledge_references": [
                {
                    "id": "c8347bef-127f-4a22-b962-edf5a75386ec",
                    "content": "彗星xxx",
                    "knowledge_id": "a6790b93-4700-4676-bd48-0d4804e1456b",
                    "chunk_index": 0,
                    "knowledge_title": "彗星.txt",
                    "start_at": 0,
                    "end_at": 2760,
                    "seq": 0,
                    "score": 4.038836479187012,
                    "match_type": 4,
                    "sub_chunk_id": [
                        "688821f0-40bf-428e-8cb6-541531ebeb76",
                        "c1e9903e-2b4d-4281-be15-0149288d45c2",
                        "7d955251-3f79-4fd5-a6aa-02f81e044091"
                    ],
                    "metadata": {},
                    "chunk_type": "text",
                    "parent_chunk_id": "",
                    "image_info": "",
                    "knowledge_filename": "彗星.txt",
                    "knowledge_source": ""
                }
            ],
            "agent_steps": [],
            "is_completed": true,
            "created_at": "2025-08-12T10:24:38.370548+08:00",
            "updated_at": "2025-08-12T10:25:40.416382+08:00",
            "deleted_at": null
        },
        {
            "id": "7fa136ae-a045-424e-baac-52113d92ae94",
            "session_id": "ceb9babb-1e30-41d7-817d-fd584954304b",
            "request_id": "3475c004-0ada-4306-9d30-d7f5efce50d2",
            "content": "彗尾的形状",
            "role": "user",
            "knowledge_references": [],
            "agent_steps": [],
            "is_completed": true,
            "created_at": "2025-08-12T14:30:39.732246+08:00",
            "updated_at": "2025-08-12T14:30:39.733277+08:00",
            "deleted_at": null
        }
    ],
    "success": true
}
```

**Section sources**
- [message.md](file://docs/api/message.md#L10-L160)
- [message.go](file://internal/handler/message.go#L25-L120)

### DELETE `/messages/:session_id/:id` - 删除消息

根据消息ID删除一条消息。

**请求示例**:
```curl
curl --location --request DELETE 'http://localhost:8080/api/v1/messages/ceb9babb-1e30-41d7-817d-fd584954304b/9bcafbcf-a758-40af-a9a3-c4d8e0f49439' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--header 'Content-Type: application/json'
```

**响应示例**:
```json
{
    "message": "Message deleted successfully",
    "success": true
}
```

**Section sources**
- [message.md](file://docs/api/message.md#L163-L180)
- [message.go](file://internal/handler/message.go#L122-L142)

## 模型管理API

模型管理API用于配置和管理各种AI模型，包括对话模型、嵌入模型和排序模型。

### POST `/models` - 创建模型

创建一个新的AI模型。

**请求参数**:
- `name`: 模型名称
- `type`: 模型类型（KnowledgeQA, Embedding, Rerank）
- `source`: 模型来源（local, remote）
- `description`: 模型描述
- `parameters`: 模型参数
- `is_default`: 是否为默认模型

**创建对话模型（KnowledgeQA）示例**:
```curl
curl --location 'http://localhost:8080/api/v1/models' \
--header 'Content-Type: application/json' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--data '{
    "name": "qwen3:8b",
    "type": "KnowledgeQA",
    "source": "local",
    "description": "LLM Model for Knowledge QA",
    "parameters": {
        "base_url": "",
        "api_key": ""
    },
    "is_default": false
}'
```

**创建嵌入模型（Embedding）示例**:
```curl
curl --location 'http://localhost:8080/api/v1/models' \
--header 'Content-Type: application/json' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--data '{
    "name": "nomic-embed-text:latest",
    "type": "Embedding",
    "source": "local",
    "description": "Embedding Model",
    "parameters": {
        "base_url": "",
        "api_key": "",
        "embedding_parameters": {
            "dimension": 768,
            "truncate_prompt_tokens": 0
        }
    },
    "is_default": false
}'
```

**创建排序模型（Rerank）示例**:
```curl
curl --location 'http://localhost:8080/api/v1/models' \
--header 'Content-Type: application/json' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--data '{
    "name": "linux6200/bge-reranker-v2-m3:latest",
    "type": "Rerank",
    "source": "local",
    "description": "Rerank Model for Knowledge QA",
    "parameters": {
        "base_url": "",
        "api_key": ""
    },
    "is_default": false
}'
```

**响应示例**:
```json
{
    "data": {
        "id": "09c5a1d6-ee8b-4657-9a17-d3dcbd5c70cb",
        "tenant_id": 1,
        "name": "nomic-embed-text:latest3",
        "type": "Embedding",
        "source": "local",
        "description": "Embedding Model",
        "parameters": {
            "base_url": "",
            "api_key": "",
            "embedding_parameters": {
                "dimension": 768,
                "truncate_prompt_tokens": 0
            }
        },
        "is_default": false,
        "status": "downloading",
        "created_at": "2025-08-12T10:39:01.454591766+08:00",
        "updated_at": "2025-08-12T10:39:01.454591766+08:00",
        "deleted_at": null
    },
    "success": true
}
```

**Section sources**
- [model.md](file://docs/api/model.md#L13-L76)
- [model.go](file://internal/handler/model.go#L25-L105)

### GET `/models` - 获取模型列表

获取当前租户的所有模型。

**请求示例**:
```curl
curl --location 'http://localhost:8080/api/v1/models' \
--header 'Content-Type: application/json' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ'
```

**响应示例**:
```json
{
    "data": [
        {
            "id": "dff7bc94-7885-4dd1-bfd5-bd96e4df2fc3",
            "tenant_id": 1,
            "name": "nomic-embed-text:latest",
            "type": "Embedding",
            "source": "local",
            "description": "Embedding Model",
            "parameters": {
                "base_url": "",
                "api_key": "",
                "embedding_parameters": {
                    "dimension": 768,
                    "truncate_prompt_tokens": 0
                }
            },
            "is_default": true,
            "status": "active",
            "created_at": "2025-08-11T20:10:41.813832+08:00",
            "updated_at": "2025-08-11T20:10:41.822354+08:00",
            "deleted_at": null
        },
        {
            "id": "8aea788c-bb30-4898-809e-e40c14ffb48c",
            "tenant_id": 1,
            "name": "qwen3:8b",
            "type": "KnowledgeQA",
            "source": "local",
            "description": "LLM Model for Knowledge QA",
            "parameters": {
                "base_url": "",
                "api_key": "",
                "embedding_parameters": {
                    "dimension": 0,
                    "truncate_prompt_tokens": 0
                }
            },
            "is_default": true,
            "status": "active",
            "created_at": "2025-08-11T20:10:41.811761+08:00",
            "updated_at": "2025-08-11T20:10:41.825381+08:00",
            "deleted_at": null
        }
    ],
    "success": true
}
```

**Section sources**
- [model.md](file://docs/api/model.md#L105-L164)
- [model.go](file://internal/handler/model.go#L107-L135)

### GET `/models/:id` - 获取模型详情

根据模型ID获取模型的详细信息。

**请求示例**:
```curl
curl --location 'http://localhost:8080/api/v1/models/dff7bc94-7885-4dd1-bfd5-bd96e4df2fc3' \
--header 'Content-Type: application/json' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ'
```

**响应示例**:
```json
{
    "data": {
        "id": "dff7bc94-7885-4dd1-bfd5-bd96e4df2fc3",
        "tenant_id": 1,
        "name": "nomic-embed-text:latest",
        "type": "Embedding",
        "source": "local",
        "description": "Embedding Model",
        "parameters": {
            "base_url": "",
            "api_key": "",
            "embedding_parameters": {
                "dimension": 768,
                "truncate_prompt_tokens": 0
            }
        },
        "is_default": true,
        "status": "active",
        "created_at": "2025-08-11T20:10:41.813832+08:00",
        "updated_at": "2025-08-11T20:10:41.822354+08:00",
        "deleted_at": null
    },
    "success": true
}
```

**Section sources**
- [model.md](file://docs/api/model.md#L167-L204)
- [model.go](file://internal/handler/model.go#L137-L165)

### PUT `/models/:id` - 更新模型

更新现有模型的配置。

**请求示例**:
```curl
curl --location --request PUT 'http://localhost:8080/api/v1/models/8fdc464d-8eaa-44d4-a85b-094b28af5330' \
--header 'Content-Type: application/json' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--data '{
    "name": "linux6200/bge-reranker-v2-m3:latest",
    "description": "Rerank Model for Knowledge QA new",
    "parameters": {
        "base_url": "",
        "api_key": ""
    },
    "is_default": false
}'
```

**响应示例**:
```json
{
    "data": {
        "id": "8fdc464d-8eaa-44d4-a85b-094b28af5330",
        "tenant_id": 1,
        "name": "linux6200/bge-reranker-v2-m3:latest",
        "type": "Rerank",
        "source": "local",
        "description": "Rerank Model for Knowledge QA new",
        "parameters": {
            "base_url": "",
            "api_key": "",
            "embedding_parameters": {
                "dimension": 0,
                "truncate_prompt_tokens": 0
            }
        },
        "is_default": false,
        "status": "active",
        "created_at": "2025-08-12T10:57:39.512681+08:00",
        "updated_at": "2025-08-12T11:00:27.271678+08:00",
        "deleted_at": null
    },
    "success": true
}
```

**Section sources**
- [model.md](file://docs/api/model.md#L206-L252)
- [model.go](file://internal/handler/model.go#L167-L195)

### DELETE `/models/:id` - 删除模型

根据模型ID删除一个模型。

**请求示例**:
```curl
curl --location --request DELETE 'http://localhost:8080/api/v1/models/8fdc464d-8eaa-44d4-a85b-094b28af5330' \
--header 'Content-Type: application/json' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ'
```

**响应示例**:
```json
{
    "message": "Model deleted",
    "success": true
}
```

**Section sources**
- [model.md](file://docs/api/model.md#L254-L271)
- [model.go](file://internal/handler/model.go#L197-L217)

## 标签管理API

标签（Tag）用于对知识库中的知识进行分类和组织。

### GET `/knowledge-bases/:id/tags` - 获取知识库标签列表

获取指定知识库下的所有标签，支持分页和搜索。

**请求参数**:
- `page`: 页码（默认1）
- `page_size`: 每页条数（默认20）
- `keyword`: 标签名称关键字搜索（可选）

**请求示例**:
```curl
curl --location 'http://localhost:8080/api/v1/knowledge-bases/kb-00000001/tags?page=1&page_size=10' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--header 'Content-Type: application/json'
```

**响应示例**:
```json
{
    "data": {
        "total": 2,
        "page": 1,
        "page_size": 10,
        "data": [
            {
                "id": "tag-00000001",
                "tenant_id": 1,
                "knowledge_base_id": "kb-00000001",
                "name": "技术文档",
                "color": "#1890ff",
                "sort_order": 1,
                "created_at": "2025-08-12T10:00:00+08:00",
                "updated_at": "2025-08-12T10:00:00+08:00",
                "knowledge_count": 5,
                "chunk_count": 120
            },
            {
                "id": "tag-00000002",
                "tenant_id": 1,
                "knowledge_base_id": "kb-00000001",
                "name": "常见问题",
                "color": "#52c41a",
                "sort_order": 2,
                "created_at": "2025-08-12T10:00:00+08:00",
                "updated_at": "2025-08-12T10:00:00+08:00",
                "knowledge_count": 3,
                "chunk_count": 45
            }
        ]
    },
    "success": true
}
```

**Section sources**
- [tag.md](file://docs/api/tag.md#L12-L63)
- [tag.go](file://internal/handler/tag.go#L25-L105)

### POST `/knowledge-bases/:id/tags` - 创建标签

在指定知识库下创建一个新的标签。

**请求示例**:
```curl
curl --location 'http://localhost:8080/api/v1/knowledge-bases/kb-00000001/tags' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--header 'Content-Type: application/json' \
--data '{
    "name": "产品手册",
    "color": "#faad14",
    "sort_order": 3
}'
```

**响应示例**:
```json
{
    "data": {
        "id": "tag-00000003",
        "tenant_id": 1,
        "knowledge_base_id": "kb-00000001",
        "name": "产品手册",
        "color": "#faad14",
        "sort_order": 3,
        "created_at": "2025-08-12T11:00:00+08:00",
        "updated_at": "2025-08-12T11:00:00+08:00"
    },
    "success": true
}
```

**Section sources**
- [tag.md](file://docs/api/tag.md#L66-L96)
- [tag.go](file://internal/handler/tag.go#L107-L135)

### PUT `/knowledge-bases/:id/tags/:tag_id` - 更新标签

更新现有标签的属性。

**请求示例**:
```curl
curl --location --request PUT 'http://localhost:8080/api/v1/knowledge-bases/kb-00000001/tags/tag-00000003' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--header 'Content-Type: application/json' \
--data '{
    "name": "产品手册更新",
    "color": "#ff4d4f"
}'
```

**响应示例**:
```json
{
    "data": {
        "id": "tag-00000003",
        "tenant_id": 1,
        "knowledge_base_id": "kb-00000001",
        "name": "产品手册更新",
        "color": "#ff4d4f",
        "sort_order": 3,
        "created_at": "2025-08-12T11:00:00+08:00",
        "updated_at": "2025-08-12T11:30:00+08:00"
    },
    "success": true
}
```

**Section sources**
- [tag.md](file://docs/api/tag.md#L99-L129)
- [tag.go](file://internal/handler/tag.go#L137-L165)

### DELETE `/knowledge-bases/:id/tags/:tag_id` - 删除标签

根据标签ID删除一个标签。

**查询参数**:
- `force`: 设置为`true`时强制删除（即使标签被引用）

**请求示例**:
```curl
curl --location --request DELETE 'http://localhost:8080/api/v1/knowledge-bases/kb-00000001/tags/tag-00000003?force=true' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--header 'Content-Type: application/json'
```

**响应示例**:
```json
{
    "success": true
}
```

**Section sources**
- [tag.md](file://docs/api/tag.md#L131-L150)
- [tag.go](file://internal/handler/tag.go#L167-L195)

## FAQ管理API

FAQ管理API用于管理知识库中的常见问答对。

### GET `/knowledge-bases/:id/faq/entries` - 获取FAQ条目列表

获取指定知识库下的所有FAQ条目，支持分页和过滤。

**请求参数**:
- `page`: 页码（默认1）
- `page_size`: 每页条数（默认20）
- `tag_id`: 按标签ID筛选（可选）
- `keyword`: 关键字搜索（可选）

**请求示例**:
```curl
curl --location 'http://localhost:8080/api/v1/knowledge-bases/kb-00000001/faq/entries?page=1&page_size=10' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--header 'Content-Type: application/json'
```

**响应示例**:
```json
{
    "data": {
        "total": 100,
        "page": 1,
        "page_size": 10,
        "data": [
            {
                "id": "faq-00000001",
                "chunk_id": "chunk-00000001",
                "knowledge_id": "knowledge-00000001",
                "knowledge_base_id": "kb-00000001",
                "tag_id": "tag-00000001",
                "is_enabled": true,
                "standard_question": "如何重置密码？",
                "similar_questions": ["忘记密码怎么办", "密码找回"],
                "negative_questions": ["如何修改用户名"],
                "answers": ["您可以通过点击登录页面的'忘记密码'链接来重置密码。"],
                "index_mode": "hybrid",
                "chunk_type": "faq",
                "created_at": "2025-08-12T10:00:00+08:00",
                "updated_at": "2025-08-12T10:00:00+08:00"
            }
        ]
    },
    "success": true
}
```

**Section sources**
- [faq.md](file://docs/api/faq.md#L16-L61)
- [faq.go](file://internal/handler/faq.go#L25-L105)

### POST `/knowledge-bases/:id/faq/entries` - 批量导入FAQ条目

批量导入FAQ条目，支持追加或替换模式。

**请求参数**:
- `mode`: 导入模式，`append`（追加）或`replace`（替换）
- `entries`: FAQ条目数组
- `knowledge_id`: 关联的知识ID（可选）

**请求示例**:
```curl
curl --location 'http://localhost:8080/api/v1/knowledge-bases/kb-00000001/faq/entries' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--header 'Content-Type: application/json' \
--data '{
    "mode": "append",
    "entries": [
        {
            "standard_question": "如何联系客服？",
            "similar_questions": ["客服电话", "在线客服"],
            "answers": ["您可以通过拨打400-xxx-xxxx联系我们的客服。"],
            "tag_id": "tag-00000001"
        },
        {
            "standard_question": "退款政策是什么？",
            "answers": ["我们提供7天无理由退款服务。"]
        }
    ]
}'
```

**响应示例**:
```json
{
    "data": {
        "task_id": "task-00000001"
    },
    "success": true
}
```

**注**：批量导入为异步操作，返回任务ID用于追踪进度。

**Section sources**
- [faq.md](file://docs/api/faq.md#L63-L103)
- [faq.go](file://internal/handler/faq.go#L107-L155)

### POST `/knowledge-bases/:id/faq/entry` - 创建单个FAQ条目

同步创建单个FAQ条目。

**请求参数**:
- `standard_question`: 标准问（必填）
- `similar_questions`: 相似问数组（可选）
- `negative_questions`: 反例问题数组（可选）
- `answers`: 答案数组（必填）
- `tag_id`: 标签ID（可选）
- `is_enabled`: 是否启用（可选，默认true）

**请求示例**:
```curl
curl --location 'http://localhost:8080/api/v1/knowledge-bases/kb-00000001/faq/entry' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--header 'Content-Type: application/json' \
--data '{
    "standard_question": "如何联系客服？",
    "similar_questions": ["客服电话", "在线客服"],
    "answers": ["您可以通过拨打400-xxx-xxxx联系我们的客服。"],
    "tag_id": "tag-00000001",
    "is_enabled": true
}'
```

**响应示例**:
```json
{
    "data": {
        "id": "faq-00000001",
        "chunk_id": "chunk-00000001",
        "knowledge_id": "knowledge-00000001",
        "knowledge_base_id": "kb-00000001",
        "tag_id": "tag-00000001",
        "is_enabled": true,
        "standard_question": "如何联系客服？",
        "similar_questions": ["客服电话", "在线客服"],
        "negative_questions": [],
        "answers": ["您可以通过拨打400-xxx-xxxx联系我们的客服。"],
        "index_mode": "hybrid",
        "chunk_type": "faq",
        "created_at": "2025-08-12T10:00:00+08:00",
        "updated_at": "2025-08-12T10:00:00+08:00"
    },
    "success": true
}
```

**错误响应**（标准问或相似问重复时）:
```json
{
    "success": false,
    "error": {
        "code": "BAD_REQUEST",
        "message": "标准问与已有FAQ重复"
    }
}
```

**Section sources**
- [faq.md](file://docs/api/faq.md#L106-L155)
- [faq.go](file://internal/handler/faq.go#L157-L215)

### PUT `/knowledge-bases/:id/faq/entries/:entry_id` - 更新单个FAQ条目

更新现有FAQ条目的内容。

**请求示例**:
```curl
curl --location --request PUT 'http://localhost:8080/api/v1/knowledge-bases/kb-00000001/faq/entries/faq-00000001' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--header 'Content-Type: application/json' \
--data '{
    "standard_question": "如何重置账户密码？",
    "similar_questions": ["忘记密码怎么办", "密码找回", "重置密码"],
    "answers": ["您可以通过以下步骤重置密码：1. 点击登录页面的\"忘记密码\" 2. 输入注册邮箱 3. 查收重置邮件"],
    "is_enabled": true
}'
```

**响应示例**:
```json
{
    "success": true
}
```

**Section sources**
- [faq.md](file://docs/api/faq.md#L169-L191)
- [faq.go](file://internal/handler/faq.go#L217-L245)

### PUT `/knowledge-bases/:id/faq/entries/status` - 批量更新FAQ启用状态

批量更新多个FAQ条目的启用状态。

**请求示例**:
```curl
curl --location --request PUT 'http://localhost:8080/api/v1/knowledge-bases/kb-00000001/faq/entries/status' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--header 'Content-Type: application/json' \
--data '{
    "updates": {
        "faq-00000001": true,
        "faq-00000002": false,
        "faq-00000003": true
    }
}'
```

**响应示例**:
```json
{
    "success": true
}
```

**Section sources**
- [faq.md](file://docs/api/faq.md#L193-L216)
- [faq.go](file://internal/handler/faq.go#L247-L275)

### PUT `/knowledge-bases/:id/faq/entries/tags` - 批量更新FAQ标签

批量更新多个FAQ条目的标签。

**请求示例**:
```curl
curl --location --request PUT 'http://localhost:8080/api/v1/knowledge-bases/kb-00000001/faq/entries/tags' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--header 'Content-Type: application/json' \
--data '{
    "updates": {
        "faq-00000001": "tag-00000001",
        "faq-00000002": "tag-00000002",
        "faq-00000003": null
    }
}'
```

**注**：设置为`null`可清除标签关联。

**响应示例**:
```json
{
    "success": true
}
```

**Section sources**
- [faq.md](file://docs/api/faq.md#L218-L243)
- [faq.go](file://internal/handler/faq.go#L277-L305)

### DELETE `/knowledge-bases/:id/faq/entries` - 批量删除FAQ条目

批量删除多个FAQ条目。

**请求示例**:
```curl
curl --location --request DELETE 'http://localhost:8080/api/v1/knowledge-bases/kb-00000001/faq/entries' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--header 'Content-Type: application/json' \
--data '{
    "ids": ["faq-00000001", "faq-00000002"]
}'
```

**响应示例**:
```json
{
    "success": true
}
```

**Section sources**
- [faq.md](file://docs/api/faq.md#L245-L264)
- [faq.go](file://internal/handler/faq.go#L307-L335)

### POST `/knowledge-bases/:id/faq/search` - 混合搜索FAQ

执行向量搜索和关键词搜索的混合检索FAQ。

**请求参数**:
- `query_text`: 搜索查询文本
- `vector_threshold`: 向量相似度阈值（0-1）
- `match_count`: 返回结果数量（最大200）

**请求示例**:
```curl
curl --location 'http://localhost:8080/api/v1/knowledge-bases/kb-00000001/faq/search' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--header 'Content-Type: application/json' \
--data '{
    "query_text": "如何重置密码",
    "vector_threshold": 0.5,
    "match_count": 10
}'
```

**响应示例**:
```json
{
    "data": [
        {
            "id": "faq-00000001",
            "chunk_id": "chunk-00000001",
            "knowledge_id": "knowledge-00000001",
            "knowledge_base_id": "kb-00000001",
            "tag_id": "tag-00000001",
            "is_enabled": true,
            "standard_question": "如何重置密码？",
            "similar_questions": ["忘记密码怎么办", "密码找回"],
            "answers": ["您可以通过点击登录页面的'忘记密码'链接来重置密码。"],
            "chunk_type": "faq",
            "score": 0.95,
            "match_type": "vector",
            "created_at": "2025-08-12T10:00:00+08:00",
            "updated_at": "2025-08-12T10:00:00+08:00"
        }
    ],
    "success": true
}
```

**Section sources**
- [faq.md](file://docs/api/faq.md#L266-L310)
- [faq.go](file://internal/handler/faq.go#L337-L385)

## 评估功能API

评估功能API用于评估模型在问答任务中的性能。

### GET `/evaluation` - 获取评估任务

根据任务ID获取评估任务的详细信息。

**请求参数**:
- `task_id`: 从`POST /evaluation`接口中获取到的任务ID

**请求示例**:
```bash
curl --location 'http://localhost:8080/api/v1/evaluation?task_id=c34563ad-b09f-4858-b72e-e92beb80becb' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--header 'Content-Type: application/json'
```

**响应示例**:
```json
{
    "data": {
        "task": {
            "id": "c34563ad-b09f-4858-b72e-e92beb80becb",
            "tenant_id": 1,
            "dataset_id": "default",
            "start_time": "2025-08-12T14:54:26.221804768+08:00",
            "status": 2,
            "total": 1,
            "finished": 1
        },
        "params": {
            "session_id": "",
            "knowledge_base_id": "2ef57434-8c8d-4442-b967-2f7fc578a2fc",
            "vector_threshold": 0.5,
            "keyword_threshold": 0.3,
            "embedding_top_k": 10,
            "vector_database": "",
            "rerank_model_id": "b30171a1-787b-426e-a293-735cd5ac16c0",
            "rerank_top_k": 5,
            "rerank_threshold": 0.7,
            "chat_model_id": "8aea788c-bb30-4898-809e-e40c14ffb48c",
            "summary_config": {
                "max_tokens": 0,
                "repeat_penalty": 1,
                "top_k": 0,
                "top_p": 0,
                "frequency_penalty": 0,
                "presence_penalty": 0,
                "prompt": "这是用户和助手之间的对话。",
                "context_template": "你是一个专业的智能信息检索助手",
                "no_match_prefix": "NO_MATCH",
                "temperature": 0.3,
                "seed": 0,
                "max_completion_tokens": 2048
            },
            "fallback_strategy": "",
            "fallback_response": "抱歉，我无法回答这个问题。"
        },
        "metric": {
            "retrieval_metrics": {
                "precision": 0,
                "recall": 0,
                "ndcg3": 0,
                "ndcg10": 0,
                "mrr": 0,
                "map": 0
            },
            "generation_metrics": {
                "bleu1": 0.037656734016532384,
                "bleu2": 0.04067392145167686,
                "bleu4": 0.048963321289052536,
                "rouge1": 0,
                "rouge2": 0,
                "rougel": 0
            }
        }
    },
    "success": true
}
```

**Section sources**
- [evaluation.md](file://docs/api/evaluation.md#L10-L87)
- [evaluation.go](file://internal/handler/evaluation.go#L25-L120)

### POST `/evaluation` - 创建评估任务

创建一个新的评估任务。

**请求参数**:
- `dataset_id`: 评估使用的数据集，暂时只支持官方测试数据集`default`
- `knowledge_base_id`: 评估使用的知识库
- `chat_id`: 评估使用的对话模型
- `rerank_id`: 评估使用的重排序模型

**请求示例**:
```bash
curl --location 'http://localhost:8080/api/v1/evaluation' \
--header 'X-API-Key: sk-vQHV2NZI_LK5W7wHQvH3yGYExX8YnhaHwZipUYbiZKCYJbBQ' \
--header 'Content-Type: application/json' \
--data '{
    "dataset_id": "default",
    "knowledge_base_id": "kb-00000001",
    "chat_id": "8aea788c-bb30-4898-809e-e40c14ffb48c",
    "rerank_id": "b30171a1-787b-426e-a293-735cd5ac16c0"
}'
```

**响应示例**:
```json
{
    "data": {
        "task": {
            "id": "c34563ad-b09f-4858-b72e-e92beb80becb",
            "tenant_id": 1,
            "dataset_id": "default",
            "start_time": "2025-08-12T14:54:26.221804768+08:00",
            "status": 1
        },
        "params": {
            "session_id": "",
            "knowledge_base_id": "2ef57434-8c8d-4442-b967-2f7fc578a2fc",
            "vector_threshold": 0.5,
            "keyword_threshold": 0.3,
            "embedding_top_k": 10,
            "vector_database": "",
            "rerank_model_id": "b30171a1-787b-426e-a293-735cd5ac16c0",
            "rerank_top_k": 5,
            "rerank_threshold": 0.7,
            "chat_model_id": "8aea788c-bb30-4898-809e-e40c14ffb48c",
            "summary_config": {
                "max_tokens": 0,
                "repeat_penalty": 1,
                "top_k": 0,
                "top_p": 0,
                "frequency_penalty": 0,
                "presence_penalty": 0,
                "prompt": "这是用户和助手之间的对话。",
                "context_template": "你是一个专业的智能信息检索助手，xxx",
                "no_match_prefix": "NO_MATCH",
                "temperature": 0.3,
                "seed": 0,
                "max_completion_tokens": 2048
            },
            "fallback_strategy": "",
            "fallback_response": "抱歉，我无法回答这个问题。"
        }
    },
    "success": true
}
```

**Section sources**
- [evaluation.md](file://docs/api/evaluation.md#L89-L154)
- [evaluation.go](file://internal/handler/evaluation.go#L122-L170)