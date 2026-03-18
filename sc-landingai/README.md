# Lab 6：使用 Strands Agents 构建研究论文聊天机器人

## 资源

本实验会使用多个 AWS 服务，包括 Lambda、S3、IAM 和 Bedrock。若想进一步了解 AWS 上的云计算，请查看以下资源：

 * <u> **文档** </u>
    * <a href="https://docs.aws.amazon.com/s3/" target="_blank" style="text-decoration: none;">S3</a>
    * <a href="https://docs.aws.amazon.com/lambda/" target="_blank" style="text-decoration: none;">Lambda</a>
    * <a href="https://docs.aws.amazon.com/iam/" target="_blank" style="text-decoration: none;">IAM</a>
    * <a href="https://docs.aws.amazon.com/bedrock/" target="_blank" style="text-decoration: none;">Bedrock</a>
  * <u> **库** </u>
    * <a href="https://docs.aws.amazon.com/pythonsdk/" target="_blank" style="text-decoration: none;">boto3</a>
    * <a href="https://docs.aws.amazon.com/bedrock-agentcore/" target="_blank" style="text-decoration: none;">bedrock-agentcore</a>
    * <a href="https://docs.aws.amazon.com/prescriptive-guidance/latest/agentic-ai-frameworks/strands-agents.html" target="_blank" style="text-decoration: none;">strands-agents</a>


## 概述

该流水线由三个组件组成：

### 1. 文档处理
- **S3 Bucket**：存储上传的 PDF 文档
- **Lambda Function**：在文件上传到 S3 时自动触发
- **LandingAI ADE**：
    - 处理文档并提取带有边界框的块
    - 为每个文档块创建单独的 JSON 文件
- **存储位置**：
  - `output/medical/`：Markdown 文件
  - `output/medical_grounding/`：带边界框的 grounding 数据
  - `output/medical_chunks/`：供 Knowledge Base 使用的独立 chunk JSON 文件
  - `output/medical_chunk_images/`：动态生成的裁剪后 chunk 图像

### 2. 知识库
- **AWS Bedrock Knowledge Base**：为独立的 chunk JSON 文件建立索引
- **元数据**：维护 chunk 类型、页码和边界框坐标

### 3. 聊天机器人
- **Strands Agent Framework**：编排对话流程
- **Bedrock Memory Service**：维护对话上下文
- **Visual Grounding**：
  - 从 PDF 中提取并裁剪特定 chunk 区域
  - 为 chunk 添加红色高亮边框

## 依赖项

若要复现实验，你需要配置你自己的 AWS 账户。

- Python
    - 使用 3.10 版本
- OS
    - 推荐使用 x86_64
- AWS
    - 请准备一个具备以下服务权限的 AWS 账户
        - Lambda
        - S3
        - IAM
        - Bedrock
        - CloudWatch Logs
      - 在你的账户中，你需要配置以下资源
        - S3 Bucket
        - Bedrock Knowledge Base
- LandingAI
    - Vision Agent API Key
    - 你可以在 <a href="https://bit.ly/3Ys8HXL" target="_blank">LandingAI</a> 注册免费账户：

## 文件夹结构

```
sc-landingai/
鈹溾攢鈹€ L6.ipynb                          # 主实验 Notebook
鈹溾攢鈹€ ade_s3_handler.py                 # 用于文档处理的 Lambda 函数
鈹溾攢鈹€ lambda_helpers.py                 # Lambda 部署辅助函数
鈹溾攢鈹€ visual_grounding_helper.py        # 创建裁剪后 chunk 图像的函数
鈹溾攢鈹€ medical/                          # 医疗 PDF 示例文档
鈹?  鈹溾攢鈹€ Common_cold_clinincal_evidence.pdf
鈹?  鈹溾攢鈹€ CT_Study_of_the_Common_Cold.pdf
鈹?  鈹溾攢鈹€ Evaluation_of_echinacea_for_the_prevention_and_treatment_of_the_common_cold.pdf
鈹?  鈹溾攢鈹€ Prevention_and_treatment_of_the_common_cold.pdf
鈹?  鈹溾攢鈹€ The_common_cold_a_review_of_the_literature.pdf
鈹?  鈹溾攢鈹€ Understanding_the_symptoms_of_the_common_cold_and_influenza.pdf
鈹?  鈹溾攢鈹€ Viruses_and_Bacteria_in_the_Etiology_of_the_Common_Cold.pdf
鈹?  鈹斺攢鈹€ Vitamin_C_for_Preventing_and_Treating_the_Common_Cold.pdf
鈹斺攢鈹€ README.md                         # 本文件
```

## 开始使用

### 第 0 步：S3 和 Bedrock

- 在你的 S3 bucket 中创建两个文件夹：`input/` 和 `output/`
- 将 Bedrock Knowledge Base 连接到该文件夹

### 第 1 步：环境设置

创建一个包含凭证的 `.env` 文件：

```bash
AWS_ACCESS_KEY_ID=your_access_key
AWS_SECRET_ACCESS_KEY=your_secret_key
AWS_REGION=us-west-2
S3_BUCKET=your-bucket-name
VISION_AGENT_API_KEY=your_landingai_api_key
BEDROCK_MODEL_ID=us.anthropic.claude-sonnet-4-5-20250929-v1:0
BEDROCK_KB_ID=your_knowledge_base_id
```

### 第 2 步：安装依赖

```bash
pip install boto3 python-dotenv Pillow PyMuPDF landingai-ade typing-extensions
pip install bedrock-agentcore strands-agents pandas
```

### 第 3 步：运行 Notebook

在 Jupyter 中打开 `Lab-6.ipynb`，并按照分步说明执行：
1. 部署 Lambda 函数
2. 设置 S3 触发器
3. 处理医疗文档（自动创建 chunks）
4. 配置 Bedrock Knowledge Base 为 `output/medical_chunks/` 建立索引
5. 使用 `search_medical_chunks()` 测试基于 chunk 的搜索
6. 启动交互式聊天机器人

## 监控与调试

### CloudWatch 日志
在 AWS CloudWatch 中监控 Lambda 执行：
- 每个文档的处理状态
- 错误信息和堆栈跟踪
- 性能指标与耗时

### S3 输出验证
检查处理结果：
```python
# List all processed files
stats = monitor_lambda_processing(logs_client, s3_client, bucket_name)
```

### 知识库同步
验证文档摄取：
```python
response = bedrock_agent.start_ingestion_job(
    knowledgeBaseId=BEDROCK_KB_ID,
    dataSourceId=DATA_SOURCE_ID
)
```

## 故障排查

### 常见问题

1. **Lambda Timeout**：在部署时增大超时时间（默认：900 秒）
2. **Memory Errors**：增加 Lambda 内存（默认：1024MB）
3. **IAM Permissions**：确保角色具有 S3 和 CloudWatch 访问权限
4. **Python Version Mismatch**：使用 Python 3.10 以确保兼容性
5. **Knowledge Base Not Found**：检查 KB ID 和区域设置

### 调试命令

```python
# Check Lambda logs
monitor_lambda_processing(logs_client, s3_client, bucket)

# Verify S3 outputs
s3_client.list_objects_v2(Bucket=bucket, Prefix='output/')

# Test chunk-based search
results = search_medical_chunks("test query", s3_client, bucket)

# Test knowledge base search
test_result = search_knowledge_base("test query")
```

**注意**：本实验需要启用 AWS 服务，可能会产生费用。完成练习后请记得清理相关资源。
