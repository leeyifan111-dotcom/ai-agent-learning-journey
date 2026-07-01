# 2026-07-01 · Day 9 · Phase 2 收官

**阶段**：2（RAG 猛攻 · 最后一天）
**主题**：Agentic RAG + 高级 RAG 变体 + 评估指标
**对应代码**：填 `rag.py`（串通 retriever + LLM） + `evaluate()`

---

## 🎯 今日目标

- 读完 #018 #019 #020，RAG 全栈认知闭环
- 把 `rag.py` 从 stub 填到能跑：检索 → 拼 prompt → LLM 生成 → JSON Mode 评估
- Phase 2 10 篇全部读完，tiny-rag 最小完整版本完工

---

## 📚 今日阅读

| 顺序 | 编号 | 标题          | 配套代码                    | 状态 |
| ---- | ---- | ------------- | --------------------------- | ---- |
| 1    | #018 | Agentic RAG   | `rag.py` ask() 接 retriever | ⏳   |
| 2    | #019 | 高级 RAG 变体 | 无（认知储备）              | ⏳   |
| 3    | #020 | RAG 评估指标  | `rag.py` evaluate()         | ⏳   |

---

## 🔍 阅读引导

### 1. `./018-agentic-rag.md` — Agentic RAG

**带着这个问题去读：**

- 你的 tiny-agent-loop 有个 Agent 循环（LLM 自主决定调哪个工具），tiny-rag 有个检索 pipeline。**Agentic RAG 就是把这两个拼起来——LLM 自己决定"要不要检索"、"检索什么"、"搜到的东西够不够"**。这不是一个项目合并，而是一个架构演进。
- **检索的"粒度决策"**：普通 RAG = 每次必检索（workflow）。Agentic RAG = LLM 判断需要时才检索（agent）。这跟你 Day 2 #007 学的 Workflow vs Agent 是什么关系？
- 你的 `rag.py` 当前是固定的 ask() 流程——**如果要改成 Agentic 版，哪一个组件会变成一个"工具"被 tiny-agent-loop 调用？**

**读完应该能**：

> Agentic RAG = **_ + _**。我的 tiny-rag 当前是 **_（Agentic / 非 Agentic），要把 _** 改成一个工具函数传给 tiny-agent-loop 才算 Agentic。

---

### 2. `./019-advanced-rag-variants.md` — 高级 RAG 变体

**带着这个问题去读：**

- **你 Day 6 读 #012 时就提前命中了三个查询优化技巧**：Query Rewriting、Query Decomposition、HyDE。这篇是对它们的正式展开。
- **HyDE（生成假设性回答）** 是最反直觉的：用 LLM 编一个答案 → 用答案去检索 → 答案的向量比问题更接近文档。这跟你 #022 学的「name/description/input_schema 三要素相辅相成」是同一种设计哲学——**用冗余换鲁棒性**。
- **Parent Document Retrieval**：你昨天提到"能不能根据 chunk 的 metadata 把整篇文档喂给 LLM"——这就是这个技术的正式名字。

**读完应该能**：

> 高级 RAG 变体中，**_ 适合我的场景（全中文 markdown 文档，单篇 5000-8000 字符），理由 _**

---

### 3. `./020-rag-evaluation-metrics.md` — RAG 评估指标

**带着这个问题去读：**

- **Faithfulness vs Relevance** 是两个维度：前者问"答案是不是基于检索到的 chunk 写的"，后者问"检索到的 chunk 跟问题有没有关系"。前者只能靠 LLM-as-Judge 判，后者可以部分自动化。
- **你 Day 3 #061 学的 JSON Mode**——现在用在 RAG 评估上。让 LLM 对每个答案打出 `{"faithfulness": 1-5, "relevance": 1-5, "reason": "..."}`——这就是 `evaluate()` 函数的核心。
- **RAGAS 框架** 是一个专门做评估的工具——你的 tiny-eval（Phase 4）可能会用到，现在先知道名字。

**读完应该能**：把 `rag.py` 的 `evaluate()` 从 stub 变成能用 JSON Mode 打分

---

## 🧪 验收标准

```bash
cd tiny-rag
python rag.py "什么是 RAG？"
```

预期完整的 RAG 闭环：检索 → 拼 prompt → LLM 生成 → 打印答案。

---

## ❓ 困惑/待消化

- TBD

## 🚀 下一步

- Phase 2 收官，整理 10 篇 read_feelings
- v0.1 完整版 commit + push
- 回顾 Phase 2 全程，决定是否进 Phase 3 还是回头打磨 tiny-agent-loop
