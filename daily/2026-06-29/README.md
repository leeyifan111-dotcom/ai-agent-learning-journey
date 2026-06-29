# 2026-06-29 · Day 7 · Phase 2 索引侧

**阶段**：2（RAG 猛攻）
**主题**：索引侧的三个关键选型——分块 / 向量库 / Embedding 模型
**产出**：`tiny-rag/indexer.py` 全链路跑通（109 条 chunk → bge-m3 1024 维 → Chroma 持久化）

---

## 🎯 今日目标

- ✅ 精读 #013 #014 #015 三道选型题
- ✅ `tiny-rag/indexer.py` 三个 stub 全部填完（load → chunk → embed → store）
- ✅ 跑通「建索引」全流程：109 chunks × BAAI/bge-m3 1024 维 → Chroma `chroma_store/`

## 🛠️ 产出

- ✅ 精读 3 篇，完成 read_feelings
- ✅ `chunk_text()` — 固定长度 + 句子边界 + overlap（#013）
- ✅ `build_index()` — Chroma PersistentClient 持久化（#014）
- ✅ `embed_texts()` — 硅基流动 BAAI/bge-m3 1024 维（#015）
- ✅ commit `85b2a42` 推送到 GitHub

---

## 📚 今日阅读（全部完成）

| 顺序 | 编号 | 标题               | 配套代码                       | 状态 |
| ---- | ---- | ------------------ | ------------------------------ | ---- |
| 1    | #013 | 分块策略           | `chunk_text()`                 | ✅   |
| 2    | #014 | 向量数据库选型     | `build_index()` 里 Chroma 存取 | ✅   |
| 3    | #015 | Embedding 模型选择 | `embed_texts()`                | ✅   |

---

## 💡 核心收获

1. **分块三策略**：固定长度（快但截断）< 递归（层级保护）< 语义（余弦相邻，recall 90%+）。v0.1 用**固定长度 + 句子边界保护 + overlap 粘合**（#013），v0.2 可换递归。
2. **向量库五行选型**：Pinecone/Weaviate/Milvus/Chroma/Qdrant。v0.1 选 Chroma——Python 原生、零配置、持久化到磁盘（#014）。FAISS 是搜索**库**不是**数据库**，文档没提是对的。
3. **Embedding 三模型权衡**：MiniLM 384 维（快/省）vs text-embedding-3-small 1536 维（精/可降维）vs bge-m3 1024 维（中文最佳/平衡）。v0.1 用硅基流动 `BAAI/bge-m3`（1024 维、免费额度、OpenAI 兼容格式）。
4. **DeepSeek 没有云端 Embedding API**：踩坑验证——端点存在但 chat-only，GitHub 官方 issue 确认。切到硅基流动只改两行 base_url + model。
5. **模型必须一致**（Day 6 hard truth 验证）：Chroma 老索引残留 MiniLM 384 维，新建索引 1024 维直接报 `InvalidArgumentError`。`rm -rf chroma_store` 重建解决。

---

## 📝 读后感（详见 [read_feelings](./read_feelings)）

- **#013 — 分块策略**：固定/递归/语义三种，各有利弊；chunk_size=512 token / overlap=50 从基线开始调
- **#014 — 向量数据库**：个人用 Chroma/Qdrant(free<1GB)，中型 Weaviate，大型 Pinecone(代管)/Milvus(自管)
- **#015 — Embedding 模型**：v0.1 硅基流动 BAAI/bge-m3(1024 维、中文最佳)+ 英文嵌中文=全废

## ❓ 困惑/待消化

- TBD

## 🚀 下一步

- 明天 Day 8 读 #016 #017（混合检索 + Re-ranking），填 `retriever.py`
- 跑通第一条完整 RAG 问答：`python rag.py "什么是 HyDE？"`

## 📝 备忘

- Chroma 清数据用 `rm -r chroma_store`，不能用 Chroma 自己的 `delete_collection`（连 schema 都删了）
- 硅基流动 base_url 要加 `/v1` 后缀：`https://api.siliconflow.cn/v1`
