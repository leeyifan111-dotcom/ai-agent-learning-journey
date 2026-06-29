# 2026-06-30 · Day 8 · Phase 2 检索侧

**阶段**：2（RAG 猛攻）
**主题**：检索侧的两次升级——混合检索（BM25 + 向量）和重排序
**对应代码**：填 `tiny-rag/retriever.py`

---

## 🎯 今日目标

- 读完 #016 #017，知道「单靠向量检索为什么不够」→「怎么把关键词和语义拼成混合检索」→「怎么用 reranker 对 top-K 重新排序」
- 把 `tiny-rag/retriever.py` 从 stub 填到能跑：加载索引 → 混合检索 → 重排
- 跑通第一条完整 RAG 问答

## 🛠️ 产出（预期）

- 精读 2 篇，完成 read_feelings
- `retriever.py` 的 `search()` 从 stub 变成混合检索
- 给 `rag.py` 接上真实的 retriever，跑通 `python rag.py "什么是 HyDE？"`

---

## 📚 今日阅读

| 顺序 | 编号 | 标题       | 文件                            | 配套代码               | 状态 |
| ---- | ---- | ---------- | ------------------------------- | ---------------------- | ---- |
| 1    | #016 | 混合检索   | `./016-hybrid-retrieval.md`     | `retriever.py` 加 BM25 | ⏳   |
| 2    | #017 | Re-ranking | `./017-reranking-strategies.md` | `retriever.py` 加重排  | ⏳   |

> **原则**：读完一篇，立刻填对应代码。**先读 #016 写完 BM25 再读 #017**。

---

## 🔍 阅读引导

### 1. `./016-hybrid-retrieval.md` — 混合检索

**带着这个问题去读：**

- **向量检索能抓到"天气"和"气候"的语义关联，但对"HyDE 是什么"这种精确术语匹配，BM25 往往更准**——为什么？两种检索在什么场景下各自碾压对方？
- Day 7 你建好了 109 个 chunk 的向量索引——**如果用户问"什么是 RAG？"，向量检索天然擅长这个。但问"#016 讲了什么？"，向量就听不懂文档编号了**——这就是关键词检索的必杀场景。
- 混合检索的两个主流搞法：**结果融合（RRF）和先后叠加（先用 BM25 筛，再向量排）**——v0.1 用哪个最简单？

**读完应该能**：

> 混合检索 = **_ + _**，融合公式是 **_。我 v0.1 的 `search()` 应该是用 _** 做关键词检索 + **_ 做向量检索，融合方式 _**

**读完立刻填**：`retriever.py` 的 `search()`，加 BM25 关键词检索（`rank_bm25` 库，15 行代码）+ Chroma 向量检索 + RRF 融合。

---

### 2. `./017-reranking-strategies.md` — Re-ranking

**带着这个问题去读：**

- 混合检索返回 20 条候选，但 LLM 上下文窗口只能塞 3-5 条——**20 条里怎么挑最重要的 3 条交给 LLM？** 重排序在这一步解决什么问题？
- **精排（cross-encoder）和粗排（bi-encoder/向量检索）的核心区别**：粗排用向量找候选（快，轻量），精排用 cross-encoder 逐对打分（慢，精准）。你的 tiny-rag 里谁干粗排、谁干精排？
- 市面上常用的 reranker 模型：`bge-reranker`、`Cohere Rerank`——v0.1 用哪个最省事？

**读完应该能**：

> 粗排（retrieval，**ms）→ 精排（reranker，**ms/条）→ top-K 喂 LLM。我的 v0.1 用 \_\_\_ 做 reranker。

**读完立刻填**：`retriever.py` 的 `rerank()` 函数。

---

## 🧪 验收标准（两篇全读完后跑一次）

```bash
cd tiny-rag
python rag.py "什么是 HyDE？"
```

预期：retriever 从 Chroma 拉候选 → BM25 融合 → 重排 → LLM 基于 chunk 生成答案 → 控制台打印。

---

## ❓ 困惑/待消化

- TBD（边读边记）

## 🚀 下一步

- 今天跑通 `retriever.py` + 第一条完整 RAG 问答
- 明天 Day 9 读 #018 #019 #020，加 eval + Agentic RAG 思路

## 📝 备忘

- chroma_store/ 已有 109 条 1024 维向量（BAAI/bge-m3），retriever 直接读它
- BM25 不需要嵌模型——跟文档量级无关，纯 TF-IDF 变种，Python 有 `rank_bm25` 一行搞定
