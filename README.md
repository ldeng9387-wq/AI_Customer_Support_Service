# AI Customer Support Dashboard (企业 AI 知识库与智能客服后台)

> 面向中小企业的“传统客服 + AI 增强”业务系统样板。将企业私有知识接入 RAG 架构，提供从资料上传、后台异步向量化到前端无感状态刷新的完整 SaaS。
>
> 点击观看项目演示视频 Video Link：
> https://raw.githubusercontent.com/ldeng9387-wq/AI_Customer_Support_Service/refs/heads/main/ai_customer_service(1290-1080).mp4

## 1. 业务价值 (Business Value)

企业日常依赖人工客服，产品知识分散在各类文档中，导致响应慢、培训成本高且答案标准不一。本项目致力于通过技术手段实现业务的数字化与降本增效：
- **自动化知识库**：将传统的“人找答案”转变为“企业私有知识库的 AI 解答”。
- **低成本商业落地**：放弃昂贵的云端向量数据库与硅基流动绑定，提供具有性价比的本地化部署方案。

## 2. 技术栈 (Stack)

*   **前端工程**：React + TypeScript + Vite + Tailwind CSS + shadcn/ui
*   **后端服务**：Node.js (Express) + Multer
*   **RAG 实现框架**：LangChain.js
*   **向量与状态存储**：ChromaDB (本地容器化) + SQLite (元数据与状态机)
*   **全栈通信**：RESTful API + Server-Sent Events (SSE) 单向长链接
*   **Embedding 模型**：BAAI/bge-m3

## 3. 解决方案 (Solution)

项目目的不仅是调用模型 API，更在于利用工程化思想解决 RAG 在企业业务落地时的痛点。

### 3.1 异步防阻塞文件解析
*   **问题**：解析大体积的文档并进行语言模型 Embedding 耗时长。传统同步 HTTP 请求易触发网关超时（如 Serverless 环境），且 Node.js 单线程易被阻塞。
*   **工程实现**：引入状态机设计。前端上传文件后，后端 `Multer` 拦截落盘并写入单文件 `SQLite`（状态标记为 `processing`），立刻返回 200 响应。后台独立调度 LangChain 进行文本切片与 ChromaDB 向量入库，完成后推进状态为 `ready`。有效保障了服务端在高并发文件解析时的稳定性与可用性。

### 3.2 SSE 全栈实时状态同步
*   **问题**：后台采用异步处理后，传统轮询（Polling）机制获取进度严重消耗服务器资源且存在延迟。
*   **工程实现**：在后端引入轻量事件总线，新增 SSE (Server-Sent Events) 端点。当文件状态在 SQLite 中发生变更时，触发事件广播；前端 React 接入 `EventSource` 监听消息，实现状态的实时替换，实现流畅交互体验。

### 3.3 支持私有化部署
*   **问题**：依赖外部高价 SaaS 向量库（如 Pinecone），导致项目使用成本高、客户私有化部署困难。
*   **工程实现**：存储层放弃外部云数据库依赖，业务元数据使用 SQLite，向量检索使用轻量级本地 ChromaDB。通过统一的 Docker 环境，支持在轻量云服务器（如 2核 2G）上部署。

### 3.4 防幻觉与溯源
*   **问题**：企业客服场景无法容忍大模型的“幻觉”（编造不存在的功能或政策）。
*   **工程实现**：通过向量相似度阈值过滤噪音，严格限制 LLM 只能基于召回的上下文进行回答。同时在 UI 侧强制渲染引用片段（Citations），确保每一个业务结论都“有据可查”。

## 4. 数据流向图 (Data Flow)

```text
[ Document (PDF/Word) ]
        │
        ▼ (Multer 落盘 + SQLite 标记 processing)
[ Async Parser & TextSplitter ]
        │
        ▼ (BAAI/bge-m3)
[ ChromaDB (Vector Storage) ] ──► [ SQLite 状态更新 ready ]
        │                                  │
        ▼ (Similarity Search)              ▼ (SSE 广播)
[ Prompt Context Injection ]      [ Frontend UI 无感刷新 ]
        │
        ▼ (LLM ChatModel)
[ Answer + Citations (Streaming) ]
```

## 5. AI-First 辅助开发 (AI Assisted development)

在项目开发过程中，持续使用 Trae、Cursor 等 AI 工具参与需求分析、任务拆解、代码开发和文档整理，并逐步沉淀出一套适合个人项目的开发流程：
- **需求拆解与上下文管理**：将产品需求、技术约束和开发规则整理为项目文档，作为 AI 辅助开发的统一上下文，减少重复沟通和无效生成。
- **开发过程沉淀**：通过统一规则文档和阶段性复盘，约束 AI 生成代码的结构和风格，确保项目实现与预期保持一致。
- **规范与质量控制**：不仅利用 AI 生成代码，更利用 AI 梳理架构文档与商业投递策略，实现技术产出向商业资产的转化。
- **文档协同利用**: AI 辅助整理技术文档、项目说明和交付材料，提高开发和交付效率。
