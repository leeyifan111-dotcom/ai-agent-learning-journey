# 2026-06-26 · Phase 2 启动日

**阶段**：2（RAG 猛攻 · 最佳切入点）
**主题**：从「会写 Agent」到「能给 Agent 装长期记忆」——RAG 全栈认知 + tiny-rag 项目

---

## 🎯 Phase 2 整体目标

- 把 Day 2 #002 学过的「长期记忆 = 检索后喂」从概念变成代码
- 跑通一个最小可用的 RAG pipeline：文档→分块→Embedding→检索→生成
- 学会用 #061 JSON Mode 给 RAG 结果打分（接 #020 评估指标）
- **生产作品**：tiny-rag（个人项目 + 简历素材）

## 🗺️ Phase 2 学习路线（2 周节奏建议）

| 周      | 阅读                                            | 项目里程碑                                               |
| ------- | ----------------------------------------------- | -------------------------------------------------------- |
| 第 1 周 | #011 → #015（基础认知 + 选型）                  | tiny-rag v0.1：跑通 indexer + retriever + 生成的最小链路 |
| 第 2 周 | #016 → #020（混合检索 / 重排 / Agentic / 评估） | tiny-rag v0.2：混合检索 + 重排 + JSON Mode eval          |

---

## 📚 今日阅读（先读前 2 篇打基础）

| 顺序 | 编号 | 标题               | 文件                                  | 状态 |
| ---- | ---- | ------------------ | ------------------------------------- | ---- |
| 1    | #011 | 什么是 RAG         | `./011-what-is-rag.md`                | ⏳   |
| 2    | #012 | RAG Pipeline 组件  | `./012-rag-pipeline-components.md`    | ⏳   |
| 3    | #013 | 分块策略           | `./013-chunking-strategies.md`        | ⏳   |
| 4    | #014 | 向量数据库选型     | `./014-vector-database-comparison.md` | ⏳   |
| 5    | #015 | Embedding 模型选择 | `./015-embedding-model-selection.md`  | ⏳   |
| 6    | #016 | 混合检索           | `./016-hybrid-retrieval.md`           | ⏳   |
| 7    | #017 | Re-ranking         | `./017-reranking-strategies.md`       | ⏳   |
| 8    | #018 | Agentic RAG        | `./018-agentic-rag.md`                | ⏳   |
| 9    | #019 | 高级 RAG 变体      | `./019-advanced-rag-variants.md`      | ⏳   |
| 10   | #020 | RAG 评估指标       | `./020-rag-evaluation-metrics.md`     | ⏳   |

> 原则：「做中读」—— 每读完 2-3 篇就回 tiny-rag 写一段代码，**不要一口气读完再开始写**，否则跟 Phase 1 #061/#062 一样会"概念已懂但没手感"

---

## 🔍 阅读引导 · 今日先开 #011 + #012

### 1. `./011-what-is-rag.md` — 什么是 RAG

**带着这个问题去读：**

- 你 Day 2 #002 推导过：**短期记忆 = 全量喂；长期记忆 = 检索后喂**。RAG 在这条思路里到底是「长期记忆的实现」还是别的什么？
- **RAG vs Fine-tuning** 是常见的面试比较——别只记"一个改提示一个改权重"。从你大数据背景看：**fine-tune 像离线计算（成本大、改一次留长久），RAG 像流式查询（实时灵活）**——这个类比对吗？哪里不对？
- 为什么不直接把所有文档塞 system prompt 完事？想清楚 token 上限 + 成本 + 时效三个维度。

**读完应该能填空**：

> RAG 解决的核心问题是 **_（提示：LLM 训练数据 + 时效 + 私域）。它本质上是给 LLM 装 _**（你 Day 2 推导的那个概念）。相比 fine-tuning 的优势是 **_，劣势是 _**。

---

### 2. `./012-rag-pipeline-components.md` — RAG Pipeline 组件

**带着这个问题去读：**

- 一个完整 RAG 系统拆成几个组件？哪些是**离线**做的，哪些是**在线**做的？（这正是大数据工程师的拿手活——批 vs 流的分工）
- **你 Day 1 写 tiny-agent-loop 时已经踩过的坑会重现**——派发表、错误恢复、检测循环这些 Agent loop 的护栏，**在 RAG pipeline 里有没有同源问题**？比如：检索返回 0 条结果时怎么办？Embedding API 超时怎么办？

**读完应该能：**

- 画一张 RAG pipeline 全景图（离线索引 + 在线查询两条线）
- 对照 tiny-rag 待办：每个组件你打算用什么实现（开源库 / 自己写 / API）

---

## 🛠️ tiny-rag 项目骨架建议

读完 #011 + #012 立刻开仓库（别等读完再写）：

```bash
mkdir tiny-rag && cd tiny-rag
git init
```

**v0.1 最小可用版本（读完 #011-#015 就能写）**：

```
tiny-rag/
├── docs/                    # 测试文档目录（放几个 markdown / txt）
├── indexer.py              # 离线：文档 → 分块 → embedding → 存向量库
├── retriever.py            # 在线：query → embedding → top-K 检索
├── rag.py                  # 主入口：retriever + LLM 生成
├── requirements.txt
└── README.md
```

**技术选型建议（先用最简单的，跑通再优化）**：

| 组件      | v0.1 建议                                | v0.2 升级方向                       |
| --------- | ---------------------------------------- | ----------------------------------- |
| Embedding | DeepSeek / OpenAI text-embedding-3-small | 本地 bge-small-zh                   |
| 向量库    | **FAISS（内存版）** 或 ChromaDB          | Qdrant / Milvus                     |
| 分块      | 固定长度（500 token + 50 overlap）       | 语义分块 / 递归分块                 |
| LLM       | 复用 Day 1 的 DeepSeek client            | —                                   |
| 评估      | 先不做                                   | JSON Mode + 5 分量表（#020 + #061） |

> **建议**：v0.1 故意选 FAISS 内存版——简单到 30 行代码跑通整个 pipeline。等 v0.2 再换 ChromaDB / Qdrant，体会"为什么需要持久化向量库"。

---

## ❓ 困惑/待消化

- TBD（边读边记）

## 🚀 下一步

- 开始读 #011，回答上面两个填空
- 跑通 `indexer.py + retriever.py` 最小版（用 5 个 markdown 文档当语料）
- Phase 2 前置认知（已就绪）：短期/长期记忆喂入差异 ✅ / JSON Mode eval ✅ / 软硬约束 ✅ / MCP 解耦动机 ✅

## 📝 备忘

- **做中读节奏**：建议每 2-3 篇文档就动一次手，避免重蹈 Phase 1 后期"读完才上手"的覆辙
- **Phase 2 真正的考验**：能不能把 tiny-agent-loop 和 tiny-rag **串起来**——tiny-agent-loop 调 tiny-rag 当工具，就是「Agentic RAG」（#018），下次 Phase 旗舰项目「企业知识库 Agent」就长这样
