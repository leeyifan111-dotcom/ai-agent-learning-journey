# 2026-06-29 · Day 7 · Phase 2 索引侧

**阶段**：2（RAG 猛攻）
**主题**：索引侧的三个关键选型——分块 / 向量库 / Embedding 模型
**对应代码**：填 `tiny-rag/indexer.py` 的三个 stub

---

## 🎯 今日目标

- 读完 #013 #014 #015 三道选型题，把 `tiny-rag/indexer.py` 三个 stub 全部填完
- 跑通「建索引」全流程：文档 → 分块 → embedding → 存 FAISS
- v0.1 的离线侧完工

## 🛠️ 产出（预期）

- 精读 3 篇，完成 read_feelings
- `indexer.py` 三个函数从 `NotImplementedError` → 能跑
- 用 `docs/` 的 10 篇 RAG 文章建好第一个 FAISS 索引 `tiny.index`

---

## 📚 今日阅读

| 顺序 | 编号 | 标题               | 文件                                  | 配套代码                      | 状态 |
| ---- | ---- | ------------------ | ------------------------------------- | ----------------------------- | ---- |
| 1    | #013 | 分块策略           | `./013-chunking-strategies.md`        | `chunk_text()`                | ⏳   |
| 2    | #014 | 向量数据库选型     | `./014-vector-database-comparison.md` | `build_index()` 里 FAISS 存取 | ⏳   |
| 3    | #015 | Embedding 模型选择 | `./015-embedding-model-selection.md`  | `embed_text()`                | ⏳   |

> **原则**：读完一篇立刻填对应函数。**不要三篇读完再一起写**。

---

## 🔍 阅读引导

### 1. `./013-chunking-strategies.md` — 分块策略

**带着这个问题去读：**

- 分块太大 → LLM 吃到的切片包含无关信息（噪音）。分块太小 → 上下文被割裂，语义碎片化。**什么长度是"刚好"？**你能想出一种不需要硬调 chunk_size 的自适应分法吗？
- **固定长度**（多长算合适？500/1000 token？） vs **递归分块**（按 markdown 标题层级切） vs **语义分块**（句子边界 + embedding 相似度）——v0.1 用哪种？读完能列出各自的适合场景吗？

**读完应该能决定**：

> 我的 tiny-rag v0.1 选 **_ 分块策略，参数 chunk_size=_** / overlap=**_，因为 _**

**读完立刻填**：打开 `tiny-rag/indexer.py`，把 `chunk_text()` 从 stub 变成能跑的代码（~10 行）。

---

### 2. `./014-vector-database-comparison.md` — 向量数据库选型

**带着这个问题去读：**

- FAISS / ChromaDB / Qdrant / Milvus —— 名字挂出来。**v0.1 选哪个？**除了"简单能跑"，还要想什么？比如：单机能 hold 住吗？索引能持久化到磁盘吗？部署复杂度？
- 你大数据/K8s 背景特别能理解的一个权衡：**FAISS 像内存里的 HashMap，重跑就丢**；ChromaDB 像有持久化层的 SQLite。什么时候必须上"真向量数据工厂"？

**读完应该能决定**：

> 我的 tiny-rag v0.1 选 **_，理由是 _**；v0.2 要不要换 \_\_\_？

**读完立刻填**：`build_index()` 的 FAISS 存储逻辑，挑个合适的 API（flat L2 index / IVF / HNSW 哪个适合 demo）。

---

### 3. `./015-embedding-model-selection.md` — Embedding 模型选择

**带着这个问题去读：**

- **Day 6 你写过一条 hard truth**：「提问 embedding 模型必须跟存储时一致」。这意味着一开始选模型就要定下来——**deepseek-chat 能同时做 embedding 吗？还是必须用独立模型？**（提示：查一下 DeepSeek 有没有 embedding API，openai 呢？）
- **中英文差异**：你 tiny-rag 的语料是全中文的 RAG 文档——用 OpenAI `text-embedding-3-small`（多语言）还是 `bge-m3`（本地开源、中文好）？v0.1 快速跑通选 API，v0.2 追求可控选本地，各自代价是什么？
- **向量维度 vs 准确率 vs 速度**的三角权衡——这篇里有没有具体数字？

**读完应该能决定**：

> 我的 tiny-rag v0.1 用 **_ embedding 模型，维度 _**，原因 \_\_\_

**读完立刻填**：`embed_text()` 把 API 调用写通（~15 行）。

---

## 🧪 验收标准（三篇全读完后跑一次）

```bash
cd tiny-rag
python indexer.py
```

预期输出类似：

```
📂 加载 10 个文档
✂️  切成 85 个 chunks
🧮 正在嵌入 85 个 chunks（model: text-embedding-3-small）...
💾 索引已保存到 tiny.index（85 条向量，维度 1536）
```

---

## ❓ 困惑/待消化

- TBD（边读边记）

## 🚀 下一步

- 今天跑通 indexer.py
- 明天 Day 8 读 #016 #017，填 `retriever.py`，跑通第一条完整 RAG 问答

## 📝 备忘

- **读一篇填一个函数**，别一口气全读完——手感必须在第一行代码产生
- 选型不要追求最优（**过度选择耗散精力**），v0.1 就是"能跑就行"版本，复杂选型留给 v0.2
