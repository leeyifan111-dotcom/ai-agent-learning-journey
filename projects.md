# 🏗️ 作品集

> 每个项目对应学习路线的一个阶段，是「能力变作品」的证据

## tiny-agent-loop（阶段 1 · ✅ v0.4 已发布）

- **GitHub**：https://github.com/leeyifan111-dotcom/tiny-agent-loop
- **本地路径**：`D:\ai学习\ai-agent\tiny-agent-loop\`
- **起点**：2026-06-04（v0.1） · **v0.4 收官**：2026-06-25
- **简介**：从最小 Agent 循环演化到「反复横跳治愈包」——Phase 1 全部知识落地的实战项目
- **v0.4 六大升级**（每个挂复习钩子，对应 Phase 1 文章）：
  - TOOL_REGISTRY dict 派发（#021）
  - system prompt 六件套 + 自我正当化检测（#059 + #062）
  - 错误分层 dispatch + 重试上限（#006）
  - turn signature 跨轮循环检测（#006 Q1）
  - JSON Mode + llm_reflect 软硬约束双轨（#061 + #062）
  - HITL input() 兜底（#003 + #007）
- **对应知识库**：#001 #002 #003 #006 #007 #021 #022 #027 #059 #061 #062（Phase 1 全部 11 篇）
- **下个里程碑（v0.5 候选）**：把 get_weather 改成 HTTP API → 让 HITL 链路真闭环（Day 5 推导出的 MCP 必然性）

## tiny-rag（阶段 2 · 🟢 v0.1 进行中）

- **GitHub**：https://github.com/leeyifan111-dotcom/tiny-rag
- **本地路径**：`D:\ai学习\ai-agent\tiny-rag\`
- **起点**：2026-06-26（骨架 + 语料就位）
- **简介**：给 LLM 装上「长期记忆」的最小可用 RAG 系统——文档→分块→embedding→检索→生成
- **当前状态**：scaffold 完成（indexer/retriever/rag 三件套 stub）+ 10 篇 RAG 文档当测试语料
- **路线**：
  - v0.1（读完 #013-#015 后）：固定分块 + FAISS 内存版 + 单轮检索生成
  - v0.2（读完 #016-#017 后）：混合检索（BM25 + 向量）+ Re-ranking
  - v0.3（读完 #018-#020 后）：JSON Mode eval + 高级查询变体（HyDE / Decomposition）
  - v0.4：跟 tiny-agent-loop 集成 → Agentic RAG
- **对应知识库**：#011-#020 全部 10 篇
- **金句**：用 RAG 系统答 RAG 知识——能问"什么是 HyDE"让 tiny-rag 自己从语料里答出来就成功

## tiny-multi-agent / 扩展 tiny-rag（阶段 3 · ⚪ 未启动）

- **简介**：在 tiny-rag 基础上引入多 Agent 协作 + 持久化记忆
- **对应知识库**：#031 #033 #037 #040 #044

## tiny-eval + tiny-mcp-server（阶段 4 · ⚪ 未启动）

- **简介**：用 Ragas 评估 RAG 系统 + 实现一个 MCP Server 暴露自定义工具
- **对应知识库**：#020 #027 #075 #086

## 🌟 旗舰项目：企业内部知识库 Agent（贯穿全程）

- **目标**：一个端到端的、能 demo 的、简历首发的项目
- **技术栈预期**：RAG（文档库）+ 工具调用（接业务系统）+ K8s 部署 + 监控告警 + 评估流水线
- **预计完成**：路线 4 阶段结束前
- **价值**：同时展示**数据工程优势**和 **Agent 新技能**，面试一个项目讲穿全栈
