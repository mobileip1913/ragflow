# RAGFlow 使用手册

## 目录

1. [快速开始](#1-快速开始)
2. [系统要求](#2-系统要求)
3. [安装部署](#3-安装部署)
4. [基础配置](#4-基础配置)
5. [核心功能使用](#5-核心功能使用)
6. [高级功能](#6-高级功能)
7. [API 使用](#7-api-使用)
8. [常见问题](#8-常见问题)
9. [故障排除](#9-故障排除)
10. [最佳实践](#10-最佳实践)

---

## 1. 快速开始

### 1.1 什么是 RAGFlow？

RAGFlow 是一个开源的检索增强生成（RAG）引擎，可以帮助您：
- 构建企业知识库
- 实现智能问答系统
- 处理和分析各种文档
- 创建自动化工作流

### 1.2 5 分钟快速体验

1. **启动服务**
   ```bash
   cd ragflow/docker
   docker compose -f docker-compose.yml up -d
   ```

2. **访问系统**
   - 打开浏览器访问 `http://localhost`（或您的服务器 IP）
   - 注册账号并登录

3. **配置模型**
   - 进入"设置" → "模型供应商"
   - 添加您的 LLM API Key
   - 设置默认模型

4. **创建知识库**
   - 点击"知识库" → "新增"
   - 上传文档
   - 等待解析完成

5. **开始对话**
   - 创建对话
   - 选择知识库
   - 开始提问

---

## 2. 系统要求

### 2.1 硬件要求

#### 最低配置
- **CPU**: 4 核
- **内存**: 16 GB
- **磁盘**: 50 GB 可用空间
- **网络**: 稳定的互联网连接（用于下载模型和 API 调用）

#### 推荐配置
- **CPU**: 8 核或更多
- **内存**: 32 GB 或更多
- **磁盘**: 100 GB+ SSD
- **GPU**: 可选，用于加速文档解析（推荐 NVIDIA GPU，8GB+ 显存）

### 2.2 软件要求

- **操作系统**: 
  - Linux (Ubuntu 20.04+, CentOS 7+, 或其他主流发行版)
  - macOS (10.15+)
  - Windows (通过 WSL2 或 Docker Desktop)
  
- **Docker**: >= 24.0.0
- **Docker Compose**: >= v2.26.1

### 2.3 网络要求

- 能够访问 Docker Hub 或镜像仓库
- 能够访问 HuggingFace（或配置镜像站点）
- 能够访问 LLM API（OpenAI、Anthropic 等）

---

## 3. 安装部署

### 3.1 Docker 部署（推荐）

#### 步骤 1: 克隆仓库
```bash
git clone https://github.com/infiniflow/ragflow.git
cd ragflow/docker
```

#### 步骤 2: 配置环境变量
编辑 `.env` 文件，配置必要的参数：
```bash
# MySQL 配置
MYSQL_PASSWORD=your_mysql_password

# MinIO 配置
MINIO_USER=your_minio_user
MINIO_PASSWORD=your_minio_password

# Redis 配置
REDIS_PASSWORD=your_redis_password

# Elasticsearch 配置
ELASTIC_PASSWORD=your_elastic_password
```

#### 步骤 3: 启动服务

**CPU 模式**（默认）:
```bash
docker compose -f docker-compose.yml --profile cpu up -d
```

**GPU 模式**（需要 NVIDIA GPU 和 nvidia-docker）:
```bash
# 首先在 .env 文件中添加: DEVICE=gpu
docker compose -f docker-compose.yml --profile gpu up -d
```

#### 步骤 4: 检查服务状态
```bash
# 查看日志
docker logs -f docker-ragflow-cpu-1

# 检查服务健康状态
docker ps
```

看到以下输出表示启动成功：
```
        ____   ___    ______ ______ __
       / __ \ /   |  / ____// ____// /____  _      __
      / /_/ // /| | / / __ / /_   / // __ \| | /| / /
     / _, _// ___ |/ /_/ // __/  / // /_/ /| |/ |/ /
    /_/ |_|/_/  |_|\____//_/    /_/ \____/ |__/|__/

    * Running on all addresses (0.0.0.0)
```

### 3.2 源码部署（开发环境）

#### 步骤 1: 安装依赖
```bash
# 安装 uv 和 pre-commit
pipx install uv pre-commit

# 克隆代码
git clone https://github.com/infiniflow/ragflow.git
cd ragflow/

# 安装 Python 依赖
uv sync --python 3.10
uv run download_deps.py
pre-commit install
```

#### 步骤 2: 启动基础服务
```bash
docker compose -f docker/docker-compose-base.yml up -d
```

#### 步骤 3: 配置 hosts
在 `/etc/hosts` 中添加：
```
127.0.0.1       es01 infinity mysql minio redis sandbox-executor-manager
```

#### 步骤 4: 启动后端
```bash
source .venv/bin/activate
export PYTHONPATH=$(pwd)
bash docker/launch_backend_service.sh
```

#### 步骤 5: 启动前端
```bash
cd web
npm install
npm run dev
```

### 3.3 使用外部数据库

如果您已有 MySQL 和 Redis，可以：

1. **修改 docker-compose.yml**
   - 移除 `depends_on` 中的 mysql 依赖
   - 注释掉 mysql 和 redis 服务

2. **修改 .env 文件**
   ```env
   MYSQL_HOST=host.docker.internal
   MYSQL_PORT=3306
   MYSQL_USER=your_user
   MYSQL_PASSWORD=your_password
   MYSQL_DBNAME=rag_flow
   
   REDIS_HOST=host.docker.internal
   REDIS_PORT=6379
   REDIS_PASSWORD=your_redis_password
   ```

3. **创建数据库**
   ```sql
   CREATE DATABASE IF NOT EXISTS rag_flow CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
   ```

---

## 4. 基础配置

### 4.1 首次登录配置

#### 4.1.1 注册账号
1. 访问系统首页
2. 点击"Sign Up"注册
3. 填写用户名、邮箱、密码
4. 完成注册

#### 4.1.2 配置模型供应商

**添加 LLM 供应商**:
1. 点击右上角头像 → "设置"
2. 进入"模型供应商"
3. 在"可选模型"中选择 LLM
4. 点击"添加"，输入 API Key
5. 保存

**添加 Embedding 模型**:
1. 在"模型供应商"页面
2. 选择"TEXT EMBEDDING"
3. 选择 Embedding 模型供应商
4. 输入 API Key
5. 保存

**设置默认模型**:
1. 刷新页面
2. 在"设置默认模型"中选择 LLM 和 Embedding 模型
3. 保存设置

### 4.2 系统配置

#### 4.2.1 关闭用户注册
编辑 `docker/.env`:
```env
REGISTER_ENABLED=0
```
重启服务生效。

#### 4.2.2 配置 HTTPS

**使用 Let's Encrypt**:
1. 安装 Certbot
2. 获取证书:
   ```bash
   sudo certbot certonly --standalone -d your-domain.com
   ```
3. 修改 `docker-compose.yml`，添加证书挂载
4. 使用 HTTPS 配置的 nginx 配置

**使用已有证书**:
1. 将证书文件放到可访问位置
2. 在 `docker-compose.yml` 中挂载证书
3. 配置 nginx 使用 HTTPS

#### 4.2.3 配置存储后端

**使用 AWS S3**:
在 `service_conf.yaml.template` 中配置：
```yaml
s3:
  access_key: your_access_key
  secret_key: your_secret_key
  bucket: your_bucket_name
  region: us-east-1
```

**使用 Azure Blob**:
```yaml
azure:
  account_name: your_account_name
  account_key: your_account_key
  container_name: your_container
```

**使用阿里云 OSS**:
```yaml
oss:
  access_key: your_access_key
  secret_key: your_secret_key
  endpoint_url: https://oss-cn-hangzhou.aliyuncs.com
  bucket: your_bucket_name
```

### 4.3 文档引擎配置

#### 4.3.1 使用 Infinity（替代 Elasticsearch）

1. **停止服务**
   ```bash
   docker compose -f docker/docker-compose.yml down -v
   ```

2. **修改 .env**
   ```env
   DOC_ENGINE=infinity
   ```

3. **启动服务**
   ```bash
   docker compose -f docker-compose.yml up -d
   ```

---

## 5. 核心功能使用

### 5.1 知识库管理

#### 5.1.1 创建知识库

1. **创建知识库**
   - 点击"知识库" → "新增"
   - 填写知识库名称和描述
   - 选择解析方法（可选）
   - 点击"保存"

2. **知识库设置**
   - **名称**: 有意义的名称，如"产品文档"
   - **描述**: 详细描述，帮助系统理解知识库用途
   - **解析方法**: 选择适合的解析模板

#### 5.1.2 上传文档

**单个上传**:
1. 进入知识库详情页
2. 点击"新增" → "上传文档"
3. 选择文件
4. 等待上传完成

**批量上传**:
1. 进入知识库详情页
2. 点击"新增" → "批量上传"
3. 选择多个文件
4. 等待上传完成

**支持的文件格式**:
- PDF (.pdf)
- Word (.docx)
- PowerPoint (.pptx)
- Excel (.xlsx)
- 文本 (.txt, .md)
- 图片 (.jpg, .png)
- 其他格式（通过解析器支持）

#### 5.1.3 文档解析

**自动解析**:
- 文档上传后自动开始解析
- 可在文档列表中查看解析状态
- 解析完成后可查看切片

**手动解析**:
1. 在文档列表中找到未解析的文档
2. 点击"解析"按钮
3. 等待解析完成

**解析设置**:
- **切片模板**: 选择适合的切片模板
- **切片大小**: 调整切片大小
- **重叠大小**: 设置切片重叠

#### 5.1.4 查看和编辑切片

1. **查看切片**
   - 进入文档详情
   - 点击"切片"标签
   - 查看所有切片内容

2. **编辑切片**
   - 点击切片进行编辑
   - 修改切片内容
   - 保存更改

3. **删除切片**
   - 选择不需要的切片
   - 点击删除

#### 5.1.5 召回测试

1. **测试检索**
   - 在知识库详情页点击"召回测试"
   - 输入测试查询
   - 查看检索结果

2. **评估效果**
   - 检查检索到的切片是否相关
   - 调整检索参数
   - 优化知识库内容

### 5.2 对话系统

#### 5.2.1 创建对话

1. **新建对话**
   - 点击"对话" → "新建对话"
   - 选择关联的知识库
   - 设置对话名称

2. **对话设置**
   - **系统提示词**: 自定义系统提示
   - **温度**: 控制生成随机性（0-1）
   - **最大 Token**: 限制生成长度
   - **关联知识库**: 选择要使用的知识库

#### 5.2.2 开始对话

1. **提问**
   - 在输入框输入问题
   - 点击发送或按 Enter

2. **查看回答**
   - 实时查看流式输出
   - 查看引用来源
   - 点击引用查看原文

3. **多轮对话**
   - 继续提问，系统会保持上下文
   - 可以引用之前的对话内容

#### 5.2.3 对话管理

**对话历史**:
- 查看所有历史对话
- 搜索对话内容
- 导出对话记录

**对话设置**:
- 修改对话参数
- 更换关联知识库
- 清空对话历史

### 5.3 文档处理

#### 5.3.1 切片模板选择

**通用模板 (naive)**:
- 适用于一般文档
- 按段落和句子切片

**问答模板 (qa)**:
- 适用于问答对格式
- 保持问答对完整性

**表格模板 (table)**:
- 适用于表格数据
- 保持表格结构

**论文模板 (paper)**:
- 适用于学术论文
- 按章节和段落切片

**其他模板**:
- 根据文档类型选择合适模板

#### 5.3.2 切片参数调整

**切片大小**:
- 默认: 512 tokens
- 范围: 128-2048 tokens
- 建议: 根据文档类型调整

**重叠大小**:
- 默认: 50 tokens
- 作用: 保持上下文连续性
- 建议: 切片大小的 10-20%

### 5.4 检索配置

#### 5.4.1 检索策略

**向量检索**:
- 基于语义相似度
- 适合语义查询

**关键词检索**:
- 基于全文搜索
- 适合精确匹配

**混合检索**:
- 结合向量和关键词
- 平衡准确率和召回率

#### 5.4.2 检索参数

**Top K**:
- 返回的切片数量
- 默认: 5
- 范围: 1-20

**相似度阈值**:
- 过滤低相似度结果
- 默认: 0.5
- 范围: 0-1

---

## 6. 高级功能

### 6.1 Agent 工作流

#### 6.1.1 创建工作流

1. **进入工作流页面**
   - 点击"Agent" → "工作流"
   - 点击"新建工作流"

2. **添加节点**
   - 从组件库拖拽节点
   - 连接节点形成流程
   - 配置每个节点的参数

3. **常用节点类型**
   - **开始节点**: 工作流入口
   - **LLM 节点**: 调用大模型
   - **检索节点**: 从知识库检索
   - **工具节点**: 调用外部工具
   - **条件节点**: 条件判断
   - **循环节点**: 循环处理
   - **结束节点**: 工作流结束

#### 6.1.2 工作流模板

**简单问答**:
1. 开始 → 检索 → LLM → 结束

**复杂分析**:
1. 开始 → 检索 → 条件判断
2. 是 → LLM 分析 → 工具调用
3. 否 → 重新检索
4. 结束

**多步骤处理**:
1. 开始 → 检索 → LLM 提取
2. → 代码执行 → LLM 分析
3. → 结果聚合 → 结束

#### 6.1.3 工作流调试

1. **测试运行**
   - 点击"测试"按钮
   - 输入测试数据
   - 查看执行结果

2. **调试模式**
   - 启用调试模式
   - 查看每个节点的输出
   - 检查错误信息

3. **日志查看**
   - 查看执行日志
   - 分析性能瓶颈
   - 优化工作流

### 6.2 数据同步

#### 6.2.1 Confluence 同步

1. **配置连接**
   - 进入"数据源" → "Confluence"
   - 输入 Confluence URL
   - 配置认证信息

2. **选择空间**
   - 选择要同步的空间
   - 设置同步规则
   - 启动同步

#### 6.2.2 S3 同步

1. **配置 S3**
   - 输入 Access Key 和 Secret Key
   - 选择 Bucket
   - 配置同步路径

2. **同步设置**
   - 设置同步频率
   - 配置文件过滤规则
   - 启动同步

#### 6.2.3 其他数据源

- **Notion**: 配置 Notion API
- **Google Drive**: OAuth 认证
- **Discord**: 配置 Webhook

### 6.3 多模态支持

#### 6.3.1 图片理解

1. **上传图片**
   - 支持 JPG、PNG 格式
   - 自动 OCR 识别文字
   - 使用多模态模型理解内容

2. **图片问答**
   - 上传图片提问
   - 系统理解图片内容
   - 返回相关答案

#### 6.3.2 PDF 图片处理

1. **提取图片**
   - PDF 中的图片自动提取
   - 使用多模态模型理解
   - 生成图片描述

2. **图文结合**
   - 结合文字和图片内容
   - 提供更准确的答案

### 6.4 知识图谱

#### 6.4.1 构建知识图谱

1. **启用图谱**
   - 在知识库设置中启用
   - 选择实体识别模型
   - 开始构建

2. **查看图谱**
   - 进入知识图谱视图
   - 查看实体和关系
   - 探索知识网络

#### 6.4.2 图谱查询

1. **实体查询**
   - 搜索特定实体
   - 查看实体详情
   - 查看相关实体

2. **关系查询**
   - 查询实体间关系
   - 发现知识关联
   - 可视化展示

---

## 7. API 使用

### 7.1 API 认证

#### 7.1.1 获取 API Key

1. 登录系统
2. 进入"设置" → "API"
3. 点击"Create new Key"
4. 复制 API Key

#### 7.1.2 使用 API Key

**在请求头中添加**:
```bash
Authorization: Bearer your_api_key
```

### 7.2 Python SDK

#### 7.2.1 安装 SDK

```bash
pip install ragflow-sdk
```

#### 7.2.2 基本使用

```python
from ragflow_sdk import RAGFlow

# 初始化客户端
client = RAGFlow(
    base_url="http://your-ragflow-server:9380",
    api_key="your_api_key"
)

# 创建知识库
kb = client.knowledge_base.create(
    name="我的知识库",
    description="这是一个测试知识库"
)

# 上传文档
file = client.file.upload(
    knowledge_base_id=kb.id,
    file_path="./document.pdf"
)

# 创建对话
conversation = client.conversation.create(
    knowledge_base_ids=[kb.id],
    name="测试对话"
)

# 发送消息
response = client.chat.send(
    conversation_id=conversation.id,
    question="什么是 RAG？"
)

print(response.answer)
```

### 7.3 REST API

#### 7.3.1 知识库 API

**创建知识库**:
```bash
curl -X POST http://your-server:9380/api/v1/datasets \
  -H "Authorization: Bearer your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "我的知识库",
    "description": "测试知识库"
  }'
```

**上传文档**:
```bash
curl -X POST http://your-server:9380/api/v1/datasets/{kb_id}/documents \
  -H "Authorization: Bearer your_api_key" \
  -F "file=@document.pdf"
```

#### 7.3.2 对话 API

**创建对话**:
```bash
curl -X POST http://your-server:9380/api/v1/chats \
  -H "Authorization: Bearer your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "测试对话",
    "dataset_ids": ["kb_id"]
  }'
```

**发送消息**:
```bash
curl -X POST http://your-server:9380/api/v1/chats/{chat_id}/completions \
  -H "Authorization: Bearer your_api_key" \
  -H "Content-Type: application/json" \
  -d '{
    "question": "什么是 RAG？",
    "stream": false
  }'
```

### 7.4 Webhook

#### 7.4.1 配置 Webhook

1. 进入"设置" → "Webhook"
2. 添加 Webhook URL
3. 选择触发事件
4. 保存配置

#### 7.4.2 Webhook 事件

- **文档解析完成**: 文档解析完成后触发
- **对话创建**: 新对话创建时触发
- **错误发生**: 系统错误时触发

---

## 8. 常见问题

### 8.1 安装问题

**Q: Docker 镜像拉取失败？**
A: 
- 检查网络连接
- 使用镜像站点（华为云、阿里云）
- 修改 `.env` 中的 `RAGFLOW_IMAGE`

**Q: 服务启动失败？**
A:
- 检查端口是否被占用
- 查看日志: `docker logs docker-ragflow-cpu-1`
- 检查环境变量配置

**Q: 内存不足？**
A:
- 增加系统内存
- 调整 `MEM_LIMIT` 环境变量
- 减少并发处理数量

### 8.2 配置问题

**Q: 如何更换 LLM 模型？**
A:
1. 进入"设置" → "模型供应商"
2. 添加新的模型供应商
3. 设置默认模型

**Q: API Key 在哪里配置？**
A:
- 在 Web UI 的"设置"中配置
- 或在 `service_conf.yaml.template` 中配置

**Q: 如何修改默认端口？**
A:
- 修改 `docker-compose.yml` 中的端口映射
- 修改 `.env` 中的端口配置

### 8.3 使用问题

**Q: 文档解析很慢？**
A:
- 使用 GPU 加速
- 减少文档大小
- 调整批量处理大小

**Q: 检索结果不准确？**
A:
- 调整检索策略（向量/关键词/混合）
- 优化文档切片
- 调整相似度阈值
- 改进知识库内容质量

**Q: 回答有幻觉？**
A:
- 检查知识库内容是否充足
- 调整系统提示词
- 使用重排序模型
- 查看引用来源

### 8.4 性能问题

**Q: 系统响应慢？**
A:
- 检查网络连接
- 优化数据库查询
- 使用缓存
- 考虑分布式部署

**Q: 内存占用高？**
A:
- 调整 Elasticsearch 内存限制
- 减少并发请求
- 清理无用数据

---

## 9. 故障排除

### 9.1 日志查看

**查看容器日志**:
```bash
# 查看主服务日志
docker logs -f docker-ragflow-cpu-1

# 查看所有服务日志
docker compose logs -f
```

**查看系统日志**:
- 日志位置: `./ragflow-logs/`
- 主要日志文件:
  - `ragflow_server.log`: 主服务日志
  - `task_executor.log`: 任务执行日志

### 9.2 常见错误

**错误: 数据库连接失败**
- 检查 MySQL 服务是否运行
- 验证连接信息
- 检查网络连接

**错误: Elasticsearch 连接失败**
- 检查 Elasticsearch 服务状态
- 验证密码配置
- 查看 Elasticsearch 日志

**错误: 文件上传失败**
- 检查文件大小限制
- 验证存储配置
- 检查磁盘空间

**错误: API 调用失败**
- 验证 API Key
- 检查网络连接
- 查看 API 日志

### 9.3 数据恢复

**备份数据**:
```bash
# 备份 MySQL
docker exec mysql mysqldump -u root -p rag_flow > backup.sql

# 备份 Elasticsearch
# 使用 Elasticsearch 快照功能

# 备份 MinIO
# 使用 MinIO 客户端备份
```

**恢复数据**:
```bash
# 恢复 MySQL
docker exec -i mysql mysql -u root -p rag_flow < backup.sql
```

---

## 10. 最佳实践

### 10.1 知识库构建

1. **文档准备**
   - 使用高质量、结构化的文档
   - 确保文档内容准确
   - 定期更新文档

2. **切片策略**
   - 根据文档类型选择合适模板
   - 调整切片大小和重叠
   - 保持语义完整性

3. **知识库组织**
   - 按主题分类知识库
   - 使用有意义的名称和描述
   - 定期清理无用内容

### 10.2 对话优化

1. **提示词设计**
   - 明确系统角色
   - 指定回答格式
   - 强调引用来源

2. **参数调整**
   - 温度: 0.7-0.9（平衡创造性和准确性）
   - Top K: 5-10（平衡准确率和召回率）
   - 最大 Token: 根据需求调整

3. **上下文管理**
   - 保持对话连贯性
   - 及时清理无关上下文
   - 使用对话历史

### 10.3 性能优化

1. **系统配置**
   - 使用 SSD 存储
   - 配置足够内存
   - 使用 GPU 加速（如可能）

2. **检索优化**
   - 使用混合检索
   - 启用重排序
   - 优化索引结构

3. **缓存策略**
   - 缓存常用查询
   - 缓存模型响应
   - 使用 Redis 缓存

### 10.4 安全实践

1. **访问控制**
   - 使用强密码
   - 启用 HTTPS
   - 配置防火墙

2. **数据安全**
   - 定期备份数据
   - 加密敏感信息
   - 限制 API 访问

3. **监控告警**
   - 监控系统资源
   - 设置告警规则
   - 定期检查日志

### 10.5 维护建议

1. **定期更新**
   - 关注版本更新
   - 及时应用安全补丁
   - 测试新功能

2. **性能监控**
   - 监控系统指标
   - 分析使用情况
   - 优化瓶颈

3. **文档管理**
   - 定期更新知识库
   - 清理过期内容
   - 优化文档结构

---

## 附录

### A. 环境变量参考

详见 `docker/README.md` 中的环境变量说明。

### B. API 参考

完整的 API 文档请访问: `http://your-server:9380/api/docs`

### C. 技术支持

- **GitHub Issues**: https://github.com/infiniflow/ragflow/issues
- **Discord**: https://discord.gg/NjYzJD3GM3
- **文档**: https://ragflow.io/docs/dev/

### D. 更新日志

查看 `docs/release_notes.md` 了解版本更新内容。

---

**手册版本**: v0.22.1
**最后更新**: 2025年1月

