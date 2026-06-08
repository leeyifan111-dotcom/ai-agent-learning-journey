# 2026-06-08 · Day 2

**阶段**：1（Agent 全景认知）
**主题**：把 Day 1 写的 ReAct 循环放进更大的地图——核心组件 / 架构模式 / Agent vs Workflow

---

## 🎯 今日目标

- 把 Day 1 写的 ReAct 循环放进**更大的地图**——理解感知/推理/行动/记忆四件套对应 `agent.py` 哪一块
- 弄清楚自己写的是 **ReAct**，Plan-and-Execute / LATS / Proactive 各自适用什么场景
- 想清楚 **tiny-agent-loop 当前是 Agent 还是 Workflow**——边界到底在哪

## 🛠️ 产出

- ✅ 精读 3 篇文档（#002 / #003 / #007），完成 read_feelings
- ✅ 把 Day 1 的 `agent.py` 在四件套坐标系里完整对号入座
- ✅ 推导出 Agent / Workflow / Agentic Workflow 三种生产形态

## 💡 核心收获

1. **四件套不是孤岛，靠 messages 串联**：
   - 行动 = `dispatch(call.name, call.args)` 真调工具
   - 感知 = `{"role":"tool", "content": result}` 这个消息**对象本身**（外界反馈的载体）
   - 记忆 = `messages` 列表本身（容器），`.append()` 是写入动作
   - 推理 = `client.chat.completions.create(...)` LLM 思考
2. **短期 vs 长期记忆的本质差异是"喂入方式"**：短期=全量喂，长期=检索后喂——这就是 RAG 存在的理由（阶段 2 伏笔）
3. **ReAct vs P&E 的关键差异是"决策粒度"**：每步反思 vs 整体规划+按需 replan；ReAct 失败会爆 token，P&E 失败会硬停（反而便宜）
4. **HITL 是跨架构的生产救场术**：工具异常 → 强制人工卡点，把"自动驾驶"降成"L2 辅助"，安全等级两个量级提升
5. **Agent vs Workflow 真正的分界点**：**LLM 是否控制流程**，不是"是否使用 LLM"。生产最常见的是 **Agentic Workflow**（大流程 Workflow + 关键节点嵌 Agent）

## 📚 今日阅读（文档副本已放入本目录）

| 顺序 | 编号 | 标题                                      | 文件                                   | 状态    |
| ---- | ---- | ----------------------------------------- | -------------------------------------- | ------- |
| 1    | #002 | 核心组件：感知 / 推理 / 行动 / 记忆       | `./002-agent-core-components.md`       | ✅ 已读 |
| 2    | #003 | 架构模式：ReAct / Plan-and-Execute / LATS | `./003-agent-architecture-patterns.md` | ✅ 已读 |
| 3    | #007 | Workflow vs Agent                         | `./007-workflow-vs-agent.md`           | ✅ 已读 |

> 原则：「做中读」—— 带着 Day 1 写代码时的亲身困惑去印证，不追求记全

## 📝 读后感（详见 [read_feelings](./read_feelings)）

- **#002 — 核心组件**：Agent 工作流 = 用户输入 → 初始化记忆容器 → 推理 → 行动 → 把工具结果构造为感知回填记忆 → 循环至 LLM/程序终止。记忆三层：工作（单任务）/ 短期（单 session，全量喂）/ 长期（向量库，检索后喂）。
- **#003 — 架构模式**：ReAct 动态灵活但易爆 token；P&E 结构性最强但出错就停；LATS 复杂任务最强但烧钱；Proactive 主动预测但难以拿捏度。**ReAct vs P&E+Replan 核心差异 = 决策粒度**（每步思考 vs 闷头执行+按需重规划）。通用调优：**HITL**。
- **#007 — Workflow vs Agent**：分界点是 **LLM 是否决定流程**，不是是否用 LLM。三种形态：纯 Workflow（路径固定可审计）/ 纯 Agent（探索性任务）/ Agentic Workflow（生产最常见）。我的 tiny-agent-loop 是纯 Agent，生产可嵌进 Workflow 的某个节点。

## ❓ 困惑/待消化

### Q1：messages 列表里，"感知"到底是动作还是数据？

**答**：是**数据**（外界反馈的载体），不是动作。

- `dispatch(...)` 调工具 = **行动**（Act）
- 工具返回的 `result` 包装成 `{"role":"tool", "content": result}` = **感知**（这个对象本身）
- `messages.append(...)` = **记忆写入**（把感知存进短期记忆）
- 这一刀切清楚后，四件套就不再混淆。

### Q2：多轮对话怎么实现？短期记忆和长期记忆的连接点在哪？

**答**：多轮对话 = 长期进程 + messages 持续累积（短期记忆容器跨多个 user_input 保留）。但短期记忆有 token 上限，所以引入**长期记忆**——存进向量库 → 下一轮先 `vector_db.search(user_input)` 检索相关条目 → 拼进 messages 再喂 LLM。

**关键差异**：

- 短期记忆 = **全量喂**（messages 全部塞进 LLM）
- 长期记忆 = **检索后喂**（只塞 top-K 相关条目）

→ 这正是阶段 2 RAG 的核心思想。**RAG 本质就是给 LLM 装长期记忆**。

### Q3：P&E + Replanning 跟 ReAct 是不是变得很像？

**答**：像但不一样，差在**决策粒度**：

| 维度       | ReAct                         | P&E + Replan                |
| ---------- | ----------------------------- | --------------------------- |
| 决策频率   | **每一步**问 LLM "下一步干啥" | **只在出错/完成**时重新规划 |
| LLM 调用数 | N 步 = N 次                   | 1 plan + M replan（M << N） |
| 失败模式   | 容易爆 token（反复横跳）      | 硬停（反而便宜）            |
| 比喻       | 走一步看一步的徒步            | 先画路线，发现路断了才重画  |

→ Day 1 那次"工具改坏反复横跳 4 次"就是 ReAct 的典型失败模式。

### Q4：Agent vs Workflow 的分界点

**答**：不是"是否使用 LLM"——Workflow 也大量用 LLM（提取字段/分类/摘要）。真正的分界是**谁决定下一步**：

- **代码决定** → Workflow（写死顺序 + LLM 只填工具，仍是 Workflow）
- **LLM 决定** → Agent（LLM 自己控制循环、自主选工具）
- **大流程 Workflow + 关键节点 Agent** → **Agentic Workflow**（生产最常见）

我的 tiny-agent-loop 属于纯 Agent；生产中可能被嵌进某个 Workflow 节点变成混合形态。

## 🚀 下一步

- 给 tiny-agent-loop 加 HITL 兜底：工具 dispatch 出错时强制 `input()` 等人工确认（v0.4 候选）
- 继续 Phase 1：#027 MCP / #059 System Prompt / #061 结构化输出 / #062 Agentic Prompting
- 阶段 2 RAG 的前置认知已经埋好：短期/长期记忆的"喂入方式"差异 + Agentic Workflow 思想

## 📝 备忘

- 跨篇章串联首次出现：**HITL** 在 #003 和 #007 都用上了；**长期记忆 ↔ RAG** 伏笔已埋；**Agentic Workflow** 跟阶段 2 后期的多 Agent 协作呼应
- 写读后感的关键教训：聊天时聊出的"原来如此"必须立刻落到笔记，三个月后你只看得到笔记看不到聊天
