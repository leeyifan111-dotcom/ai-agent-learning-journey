# 2026-07-07 · Phase 3 · 推理与规划

**阶段**：3（补认知短板）
**主题**：CoT/ToT、任务分解、Plan-and-Solve——把推理能力从 prompt 技巧升级到系统化

---

## 🎯 今日目标

- 读懂三种推理增强手法，跟 Phase 1 #003 的 ReAct/P&E 接上
- 理解你 Day 4 写的 `llm_reflect` 和 ToT 的关系

---

## 📚 今日阅读

| 顺序 | 编号 | 标题           | 状态 |
| ---- | ---- | -------------- | ---- |
| 1    | #049 | CoT / ToT      | ⏳   |
| 2    | #050 | 任务分解       | ⏳   |
| 3    | #052 | Plan-and-Solve | ⏳   |

---

## 🔍 阅读引导

### 1. `./049-cot-and-tot.md` — CoT / ToT

- CoT（Chain of Thought）你在 Day 4 v0.4 的 system prompt 已经用了："先 thought 再 action"。这篇讲的是这个技巧的原理。
- ToT（Tree of Thoughts）不是一条链，是多条推理路径同时走，中间评估哪条有希望，差的剪掉。**这跟你 `llm_reflect` 里"retry or stop"的决策有没有结构上的相似？**

---

### 2. `./050-task-decomposition.md` — 任务分解

- 你 Day 2 #003 学的 Plan-and-Execute 第一步就是"先列计划"。**任务分解 = 那个计划的生成过程**。文档讲了几种分解方法？
- 你在 #012 read_feelings 里写过"查询优化三件套"中的 Query Decomposition——那是检索层的分解。这篇是任务层的分解。两种分解本质上是一件事吗？

---

### 3. `./052-plan-and-solve-replanning.md` — Plan-and-Solve

- 你 Day 2 #003 已经学过 P&E + replanning，这篇是对 replanning 机制的展开。
- **触发 replanning 的条件**除了"某步执行失败"，还有什么？你 v0.4 的 `detect_loop` 算不算触发条件？

---

## 🚀 下一步

- 下一批：多 Agent（#031 #033 #037）
