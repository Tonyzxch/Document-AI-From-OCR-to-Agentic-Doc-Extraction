# 📚 Document AI From OCR to Agentic Doc Extraction 学习笔记

从 OCR 到 Agentic Document Extraction 的完整学习路径笔记仓库。

本仓库以 Notebook 实战为主，覆盖以下主线：
- OCR 基础能力与文档解析
- Agentic Document Understanding
- 基于 ADE 的 RAG（向量检索 + LLM 生成）
- 云端自动化流水线（AWS Lambda + S3 + Bedrock Knowledge Base + Agent）

## 学习路线（Labs）

按顺序建议如下：

1. **Lab 1**：Document Processing with OCR
2. **Lab 2**：Document Processing with PaddleOCR
3. **Lab 3**：Building Agentic Document Understanding
4. **Lab 4 (1)**：Document Understanding with Agentic Document Extraction
5. **Lab 4 (2)**：Document Understanding with Agentic Document Extraction II
6. **Lab 5**：Agentic Document Extraction for RAG
7. **Lab 6（`sc-landingai/Lab-6.ipynb`）**：Building AWS Pipelines with Agentic Document Extraction

## 目录结构

```text
.
├── lab1.ipynb
├── lab2.ipynb
├── lab3.ipynb
├── lab4(1).ipynb
├── lab4(2).ipynb
├── lab5.ipynb
└── sc-landingai/
    ├── Lab-6.ipynb
    ├── ade_s3_handler.py
    ├── lambda_helpers.py
    ├── visual_grounding_helper.py
    ├── medical/
    ├── images/
    └── README.md
```

## 技术栈概览

### 本地侧（Lab 1 - Lab 5）
- Python / Jupyter Notebook
- OCR / PaddleOCR
- Agentic Document Extraction（ADE）
- ChromaDB（向量库）
- OpenAI Embeddings + LLM
- LangChain

### 云端侧（Lab 6）
- AWS S3（文档输入输出）
- AWS Lambda（事件驱动处理）
- Amazon Bedrock Knowledge Base（知识库索引与检索）
- Strands Agents + Bedrock Memory（对话 Agent）
- LandingAI ADE（文档结构化解析）

## 快速开始

### 1) 克隆仓库

```bash
git clone <your-repo-url>
cd "Document AI From OCR to Agentic Doc Extraction"
```

### 2) 创建虚拟环境（推荐）

```bash
python -m venv .venv
# Windows
.venv\Scripts\activate
# macOS / Linux
source .venv/bin/activate
```

### 3) 安装基础依赖（本地 Labs 常用）

```bash
pip install jupyter python-dotenv
pip install openai chromadb langchain langchain-openai langchain-community
pip install Pillow PyMuPDF
```

> 说明：不同 Lab 依赖不完全相同，建议按 Notebook 中的导入与注释补充安装。

### 4) 运行 Notebook

```bash
jupyter lab
```

## 环境变量建议

你可以在仓库根目录或 `sc-landingai/` 下维护 `.env`（按实际 Lab 需要填写）：

```env
# OpenAI (Lab 3/5 常见)
OPENAI_API_KEY=your_openai_api_key

# AWS (Lab 6)
AWS_ACCESS_KEY_ID=your_access_key
AWS_SECRET_ACCESS_KEY=your_secret_key
AWS_REGION=us-west-2
S3_BUCKET=your_bucket
BEDROCK_MODEL_ID=your_model_id
BEDROCK_KB_ID=your_kb_id

# LandingAI ADE
VISION_AGENT_API_KEY=your_landingai_api_key
```

## Lab 5 -> Lab 6 的关键衔接

- **Lab 5**：在本地构建 RAG，核心是“解析 -> 向量化 -> 检索 -> 生成”。
- **Lab 6**：将流程迁移到云端，以事件驱动自动化替代手动流程：
  - 文档上传到 S3
  - 触发 Lambda 调用 ADE 解析
  - 结果写回 S3（markdown / grounding / chunks）
  - Bedrock Knowledge Base 索引 chunks
  - Strands Agent 进行问答与记忆管理

## 子目录说明：`sc-landingai/`

- `ade_s3_handler.py`：Lambda 入口，负责文档解析与结果落盘
- `lambda_helpers.py`：Lambda 打包、部署与触发器配置辅助函数
- `visual_grounding_helper.py`：基于 bbox 进行可视化定位与裁剪
- `Lab-6.ipynb`：完整云端流水线实验
- `README.md`：Lab 6 更细粒度操作说明

## 注意事项

- Lab 6 依赖 AWS 资源，可能产生云服务费用。
- 请勿将真实密钥提交到仓库（建议使用 `.env` + `.gitignore`）。
- 若在 Windows 打包 Lambda，请关注二进制依赖的平台兼容性（manylinux 轮子）。

## 参考

- LandingAI ADE
- AWS Lambda / S3 / Bedrock
- Strands Agents
- ChromaDB / OpenAI / LangChain