# RAG-system
基于RAG的知识库管理系统
# PaiSmart - 基于 RAG 的知识库管理系统

PaiSmart 是一个面向团队/组织的知识库问答系统，基于 RAG（Retrieval-Augmented Generation）实现从文档上传、解析、向量化、检索到问答生成的完整闭环。  
系统支持分片上传、异步处理、权限隔离检索、流式对话，适用于企业内部知识问答和文档智能检索场景。

## 功能特性

- 文档分片上传与断点续传（MinIO + Redis 位图）
- 文档自动解析与语义切分（Apache Tika + HanLP）
- 异步任务处理（Kafka）
- 向量化与检索（Embedding + Elasticsearch）
- 混合检索（KNN 向量检索 + BM25 关键词检索）
- 多组织/多租户权限控制（私有、组织、公开）
- WebSocket 流式问答
- 对话历史管理与追溯

## 技术栈与使用工具

### 后端

- Java 17
- Spring Boot 3.4.2（Web / Security / WebSocket / WebFlux）
- Spring Data JPA（ORM）
- MySQL
- Redis（Spring Data Redis）
- Spring Kafka（异步任务消费）
- Elasticsearch 8（向量检索 + BM25 混合检索）
- JWT（jjwt）
- Apache Tika（文档解析）
- HanLP（中文分词）
- MinIO SDK（对象存储）

### 前端

- Vue 3 + TypeScript
- Vite
- Vue Router
- Pinia
- Naive UI
- UnoCSS
- Axios（项目内 `@sa/axios`）
- Markdown-it + Highlight.js

### AI 与外部能力

- DeepSeek API（对话生成）
- DashScope Embedding API（文本向量化）

### 工程与运维工具

- Maven
- pnpm
- Docker Compose（MySQL / Redis / MinIO / Kafka / Elasticsearch）
- Nginx（`docs/nginx.conf`）
- ESLint + vue-tsc

## 项目结构

```text
PaiSmart-main/
├─ src/                 # Java 后端源码
├─ frontend/            # Vue3 管理前端
├─ homepage/            # 官网/展示页
├─ docs/                # 部署与数据库脚本
│  ├─ docker-compose.yaml
│  └─ databases/ddl.sql
└─ pom.xml              # Maven 构建配置
```

## 核心流程（RAG）

1. 客户端分片上传文件到 MinIO，上传状态写入 Redis。  
2. 服务端合并分片并写入文件元数据（MySQL）。  
3. Kafka 消费文件处理任务，触发文档解析和语义切分。  
4. 调用 Embedding 接口生成向量，批量写入 Elasticsearch。  
5. 用户提问时执行混合检索（KNN + BM25），并做权限过滤。  
6. 将检索上下文交给 DeepSeek 生成回答，通过 WebSocket 流式返回。  

## 本地开发环境要求

- JDK 17
- Maven 3.9+
- Node.js 18.20+（推荐）
- pnpm 8.7+
- Docker / Docker Compose

## 快速启动

### 1) 启动基础依赖

在 `docs` 目录下启动容器：

```bash
docker compose -f docker-compose.yaml up -d
```

默认会启动：

- MySQL（3306）
- Redis（6379）
- MinIO（19000/19001）
- Kafka（9092）
- Elasticsearch（9200）

### 2) 初始化数据库

执行 `docs/databases/ddl.sql` 初始化表结构。

### 3) 配置后端参数

编辑 `src/main/resources/application.yml`，按实际环境配置：

- MySQL 连接信息
- Redis 地址
- MinIO 账号与桶
- Kafka 地址和 topic
- Elasticsearch 账号密码
- `DEEPSEEK_API_KEY`
- `DASHSCOPE_API_KEY`

### 4) 启动后端

```bash
mvn spring-boot:run
```

默认端口：`8081`

### 5) 启动前端

```bash
cd frontend
pnpm install
pnpm dev
```

## 常用脚本

### 后端

- 单元测试：`mvn test`
- 打包：`mvn -DskipTests package`

### 前端

- 本地开发：`pnpm dev`
- 生产构建：`pnpm build`
- 类型检查：`pnpm typecheck`
- 代码检查：`pnpm lint`

## 配置说明（关键项）

- `deepseek.api.*`：对话模型配置
- `embedding.api.*`：向量模型配置
- `file.parsing.*`：文本切分与内存阈值配置
- `elasticsearch.*`：ES 连接与鉴权
- `jwt.secret-key`：JWT 密钥

## 安全与权限

- 基于 JWT 的身份认证
- 基于组织标签（org tag）的数据隔离
- 文档级可见性：私有 / 组织 / 公开
- 检索阶段做权限过滤，避免越权召回

## License

本项目使用 [MIT License](LICENSE)。
