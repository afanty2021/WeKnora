[根目录](../CLAUDE.md) > **dataset**

# dataset 模块

## 模块职责

`dataset` 模块提供 QA 数据集采样和答案生成工具，用于创建高质量的问答评估数据集。支持从大规模数据集（如 MS MARCO）中进行智能采样，并使用 OpenAI GPT 模型自动生成答案。

## 入口与启动

### 主程序
- **入口**: `qa_dataset.py`
- **依赖**: pandas, pyarrow, openai

### 快速启动
```bash
# 1. 采样数据
python dataset/qa_dataset.py sample \
  --queries ~/dataset/mmarco-queries.parquet \
  --corpus ~/dataset/mmarco-corpus.parquet \
  --qrels ~/dataset/mmarco-qrels.parquet \
  --nq 100 \
  --output_dir ./dataset/samples

# 2. 生成答案
export OPENAI_API_KEY="your-key"
python dataset/qa_dataset.py generate \
  --input_dir ./dataset/samples \
  --output_dir ./dataset/samples

# 3. 查看结果
python dataset/qa_dataset.py show \
  --input_dir ./dataset/samples \
  -n 5
```

## 对外接口

### 命令行接口

#### 1. sample 命令
从大型数据集采样子集。

```bash
python dataset/qa_dataset.py sample [OPTIONS]
```

**参数**:
- `--queries`: 查询 parquet 文件路径 (columns: id, text)
- `--corpus`: 语料库 parquet 文件路径 (columns: id, text)
- `--qrels`: 相关性判断 parquet 文件路径 (columns: qid, pid)
- `--nq`: 采样查询数量 (默认: 1000)
- `--output_dir`: 输出目录 (默认: ./save)

**输出文件**:
- `queries.parquet`: 采样的查询子集
- `corpus.parquet`: 采样的文档子集
- `qrels.parquet`: 采样的相关性判断

#### 2. generate 命令
使用 OpenAI API 生成答案。

```bash
python dataset/qa_dataset.py generate [OPTIONS]
```

**参数**:
- `--input_dir`: 包含采样数据的目录
- `--output_dir`: 输出目录 (默认: ./save)

**输出文件**:
- `answers.parquet`: 生成的答案 (columns: id, text)
- `qas.parquet`: 问答映射 (columns: qid, aid)

**特性**:
- 断点续传：自动从上次中断处继续
- 错误重试：失败请求最多重试 3 次
- 进度保存：每个成功生成的答案后保存

#### 3. show 命令
显示生成的问答对。

```bash
python dataset/qa_dataset.py show [OPTIONS]
```

**参数**:
- `--input_dir`: 包含 QA 数据的目录
- `-n`: 显示结果数量 (默认: 5)

### Python API

#### QAAnsweringSystem 类
```python
class QAAnsweringSystem:
    def __init__(self, queries: pd.DataFrame, corpus: pd.DataFrame, qrels: pd.DataFrame)
    def get_context_for_qid(self, qid: str) -> str
    def answer_question(self, qid: str, model: str = "gpt-4o-2024-05-13") -> str
```

**使用示例**:
```python
from dataset.qa_dataset import QAAnsweringSystem

system = QAAnsweringSystem(queries, corpus, qrels)
answer = system.answer_question("query_id", model="gpt-4")
```

## 关键依赖与配置

### 环境变量
```bash
# 必需
export OPENAI_API_KEY="your-openai-api-key"

# 可选
export OPENAI_BASE_URL="https://api.openai.com/v1"
```

### Python 依赖
```bash
pip install pandas pyarrow openai
```

### 支持的 OpenAI 模型
- `gpt-4o-2024-05-13` (默认)
- `gpt-4-turbo`
- `gpt-3.5-turbo`
- 其他 OpenAI 兼容模型

## 数据模型

### 输入数据格式

**queries.parquet**:
| 列名 | 类型 | 描述 |
|------|------|------|
| id | string | 唯一查询标识符 |
| text | string | 问题文本 |

**corpus.parquet**:
| 列名 | 类型 | 描述 |
|------|------|------|
| id | string | 唯一文档标识符 |
| text | string | 文档内容 |

**qrels.parquet**:
| 列名 | 类型 | 描述 |
|------|------|------|
| qid | string | 查询 ID |
| pid | string | 文档 ID |

### 输出数据格式

**answers.parquet**:
| 列名 | 类型 | 描述 |
|------|------|------|
| id | string/int | 答案唯一标识符 |
| text | string | 生成的答案文本 |

**qas.parquet**:
| 列名 | 类型 | 描述 |
|------|------|------|
| qid | string | 查询 ID |
| aid | string/int | 答案 ID |

## 采样算法

### sample_data 函数流程
1. **过滤**: 只保留在 queries 和 corpus 中都存在的 qrels
2. **采样**: 选择关联文档最多的查询（确保多样性）
3. **扩展**: 添加额外文档作为干扰项（20%）
4. **验证**: 确保数据完整性

## 测试与质量

### 测试方法
```bash
# 小规模测试
python dataset/qa_dataset.py sample \
  --queries data/test_queries.parquet \
  --corpus data/test_corpus.parquet \
  --qrels data/test_qrels.parquet \
  --nq 10 \
  --output_dir ./test_output

python dataset/qa_dataset.py show \
  --input_dir ./test_output \
  -n 3
```

### 质量保证
- 数据完整性验证（qid/pid 存在性检查）
- API 错误重试机制
- 进度持久化支持
- 详细日志输出

## 常见问题 (FAQ)

### Q: 如何使用 Azure OpenAI？
A: 设置 `OPENAI_BASE_URL` 为 Azure OpenAI 端点。

### Q: 如何处理内存不足？
A: 减少 `--nq` 参数，分批处理数据。

### Q: 如何自定义 Prompt？
A: 修改 `qa_dataset.py` 中的 `answer_question` 方法中的 prompt 模板。

### Q: 支持哪些数据集格式？
A: 支持 Parquet 格式，需包含指定的列名（id/text 或 qid/pid）。

## 相关文件清单

### 核心文件
- `qa_dataset.py`: 主程序文件
- `README`: 英文使用说明
- `README_zh.md`: 中文使用说明
- `samples/`: 示例数据目录
  - `queries.parquet`: 查询示例
  - `corpus.parquet`: 文档示例
  - `qrels.parquet`: 相关性示例
  - `qas.parquet`: 问答映射示例
  - `answers.parquet`: 答案示例

## 变更记录 (Changelog)
- 2025-12-31: 创建 dataset 模块文档
