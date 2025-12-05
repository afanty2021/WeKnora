[根目录](../../CLAUDE.md) > **docreader**

# docreader 模块

## 模块职责

`docreader` 模块是 WeKnora 的文档解析服务，支持多种文档格式的解析、内容提取和结构化处理。通过 gRPC 协议提供服务，采用插件化架构支持扩展。

## 技术栈

- **主服务**: Python 3.8+
- **gRPC 接口**: Protocol Buffers
- **解析器**: 多语言实现（Python/Go）
- **依赖管理**: uv/ pip
- **代码质量**: pylint

## 支持的文档格式

### 文本格式
- TXT: 纯文本文件
- MD: Markdown 文档
- CSV: CSV 数据文件
- HTML: 网页内容

### 办公文档
- DOC: Word 文档（旧版）
- DOCX: Word 文档
- XLS/XLSX: Excel 表格
- PPT/PPTX: PowerPoint 演示文稿

### PDF 文档
- 基础 PDF 文本提取
- OCR 图像识别（通过集成）
- 表格提取
- 版面分析

### 图像格式
- PNG/JPEG/JPG: 图片格式
- 通过 OCR 提取文字内容

## 项目结构

```
docreader/
├── main.py                    # 服务主入口
├── pyproject.toml             # Python 项目配置
├── Makefile                   # 构建脚本
├── client/                    # Go 客户端
│   ├── client.go
│   └── client_test.go
├── models/                    # 数据模型
│   ├── __init__.py
│   ├── document.py
│   └── read_config.py
├── parser/                    # 解析器实现
│   ├── __init__.py
│   ├── base_parser.py         # 解析器基类
│   ├── pdf_parser.py          # PDF 解析器
│   ├── docx_parser.py         # DOCX 解析器
│   ├── doc_parser.py          # DOC 解析器
│   ├── markdown_parser.py     # Markdown 解析器
│   ├── csv_parser.py          # CSV 解析器
│   ├── excel_parser.py        # Excel 解析器
│   ├── image_parser.py        # 图像解析器
│   ├── web_parser.py          # 网页解析器
│   ├── text_parser.py         # 文本解析器
│   ├── caption.py             # 标题提取
│   ├── chain_parser.py        # 链式解析
│   ├── markitdown_parser.py   # MarkItDown 集成
│   ├── mineru_parser.py       # MinerU 集成
│   ├── ocr_engine.py          # OCR 引擎
│   ├── parser.py              # 解析器管理
│   └── storage.py             # 存储适配器
├── splitter/                  # 文档分割器
│   ├── splitter.py
│   └── header_hook.py
├── proto/                     # gRPC 定义
│   ├── docreader.proto
│   ├── docreader.pb.go
│   ├── docreader_grpc.pb.go
│   ├── docreader_pb2.py
│   └── docreader_pb2_grpc.py
├── scripts/                   # 脚本工具
│   ├── download_deps.py
│   └── generate_proto.sh
├── testdata/                  # 测试数据
└── utils/                     # 工具函数
    ├── __init__.py
    ├── endecode.py
    ├── request.py
    ├── split.py
    └── tempfile.py
```

## 入口与启动

### Python 服务入口
```bash
# 安装依赖
pip install -r requirements.txt
# 或使用 uv
uv sync

# 启动服务
python main.py
```

### 默认配置
- **端口**: 50051
- **协议**: gRPC
- **日志级别**: INFO

## gRPC 接口定义

### 服务定义
```protobuf
service DocReader {
    rpc ParseDocument(ParseRequest) returns (ParseResponse);
    rpc ParseStream(stream ParseRequest) returns (stream ParseResponse);
    rpc GetSupportedFormats(GetSupportedFormatsRequest) returns (GetSupportedFormatsResponse);
}
```

### 主要消息类型
- `ParseRequest`: 解析请求
  - 文件路径或内容
  - 解析选项
  - 输出格式

- `ParseResponse`: 解析响应
  - 解析后的内容
  - 元数据信息
  - 错误信息

## 解析器架构

### 基类设计
`BaseParser` 定义了解析器的通用接口：
- `parse()`: 执行解析
- `extract_metadata()`: 提取元数据
- `is_supported()`: 检查格式支持

### 解析器类型

1. **直接解析器**
   - 文本类文档（TXT, MD, CSV）
   - 结构化文档（HTML, XML）

2. **工具集成解析器**
   - `MarkItDown`: 通用文档解析
   - `MinerU`: 高级版式分析
   - OCR 引擎: 图像文字识别

3. **复合解析器**
   - `ChainParser`: 多解析器链式处理
   - `CaptionParser`: 标题层级提取

## 核心功能

### 1. 文档解析流程
1. 识别文档类型
2. 选择合适解析器
3. 执行内容提取
4. 元数据收集
5. 返回结构化结果

### 2. 内容处理
- 文本清理和规范化
- 编码检测和转换
- 特殊字符处理

### 3. 元数据提取
- 文档基本信息（标题、作者等）
- 创建和修改时间
- 文档结构（标题层级）

## 数据模型

### Document 模型
```python
@dataclass
class Document:
    content: str          # 解析后的内容
    metadata: Dict        # 元数据
    chunks: List[str]     # 分块内容
    images: List[bytes]   # 提取的图像
```

### ReadConfig 配置
```python
@dataclass
class ReadConfig:
    extract_images: bool = False    # 是否提取图像
    preserve_format: bool = True    # 是否保留格式
    chunk_size: int = 1000         # 分块大小
```

## 测试与质量

### 测试覆盖
- 单元测试: 每个解析器的核心功能
- 集成测试: gRPC 服务端到端测试
- 性能测试: 大文档处理能力

### 测试数据
`testdata/` 目录包含各种格式的测试文档：
- `test.txt`: 基本文本文档
- `test.md`: Markdown 文档
- `test.html`: HTML 文档

## 部署与集成

### Docker 部署
```dockerfile
FROM python:3.8-slim
COPY . /app
WORKDIR /app
RUN pip install -r requirements.txt
CMD ["python", "main.py"]
```

### 客户端集成
提供 Go 和 Python 客户端：
```go
// Go 客户端
client := docreader.NewClient("localhost:50051")
resp, err := client.ParseDocument(ctx, &docreader.ParseRequest{
    FilePath: "/path/to/document.pdf",
})
```

## 性能优化

### 内存管理
- 流式处理大文件
- 及时释放资源
- 使用生成器模式

### 并发处理
- 支持异步解析
- 并行处理多个文档
- 连接池管理

## 常见问题 (FAQ)

### Q: 如何添加新的文档格式支持？
A:
1. 继承 `BaseParser` 类
2. 实现 `parse` 方法
3. 在 `ParserRegistry` 中注册

### Q: 如何处理大文件？
A: 使用流式解析接口 `ParseStream`，避免内存溢出。

### Q: 如何配置 OCR 引擎？
A: 在 `ocr_engine.py` 中配置 OCR 提供商（如 Tesseract、PaddleOCR）。

## 相关文件清单

### 核心文件
- `main.py`: 服务主程序
- `pyproject.toml`: 项目配置
- `Makefile`: 构建脚本

### 解析器实现
- `parser/base_parser.py`: 解析器基类
- `parser/pdf_parser.py`: PDF 解析器
- `parser/docx_parser.py`: DOCX 解析器
- `parser/markitdown_parser.py`: MarkItDown 集成

### gRPC 相关
- `proto/docreader.proto`: 服务定义
- `client/client.go`: Go 客户端实现

### 工具和脚本
- `scripts/generate_proto.sh`: 生成 gRPC 代码
- `utils/split.py`: 文本分割工具

## 变更记录 (Changelog)
- 2025-12-05: 初始化 docreader 模块文档