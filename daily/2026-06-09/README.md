# 2026-06-09 · Day 3

**阶段**：1（Agent 全景认知 · 收官）
**主题**：从「会写工具」到「会让工具协议化、让模型可控」——MCP + Prompt 工程

---

## 🎯 今日目标

- 搞清楚 **MCP** 到底解决了什么问题——为什么有 Function Calling 还需要它
- 弄明白 **System Prompt** 怎么影响 Agent 的行为边界（对照 Day 1 工具改坏反复横跳的事）
- 学会 **结构化输出**（JSON/XML）的可靠拿法——为后续 RAG / Eval 打底
- 理解 **Agentic Prompting** 跟普通 prompt 的差异——给 Agent 写指令的特殊技巧

## 🛠️ 产出

- ✅ 精读 4 篇文档（#027 / #059 / #061 / #062），完成 read_feelings
- ✅ 把 Day 1 反复横跳实验在三档修复力度（普通 prompt → Agentic prompt → Agentic + JSON Mode）下重新推演
- ✅ 推导出生产 system prompt 六件套 + Agentic prompt 三块特有维度
- ✅ Phase 1 收官，跨篇章心智地图基本成型（MCP / Prompt / 软硬约束 / Agent vs Workflow 互通）

## 💡 核心收获

1. **MCP vs Function Calling 是"分层"不是"替代"**：FC 管 model↔host（私有协议），MCP 管 host↔tool server（公共协议）；类比 USB-C——工具厂商造一次，所有 host 都能用
2. **System Prompt 是"软约束"不是"硬护栏"**：关键路径必须代码层兜底（input() / interrupt()）；结合 Day 2 的 HITL 思想 = 教育（模型）+ 法律（代码）双轨
3. **Lost in the Middle 现象**：关键约束放开头+结尾重复——跟 #006 的 detect_loop 思路同源（关键信号必须重复，不能依赖一次性传递）
4. **结构化输出三档力度**：Function Calling（解码层 constrained decoding + 训练双重保证）> JSON Mode（response_format 强制）> prompt 求 LLM 自觉（看脸）
5. **JSON Schema 是 API 和 LLM 共享的契约语言**：两边都要看——Day 1 写错 `"list(string)"` 两边都崩，不是单边问题
6. **Agentic Prompting 核心心法：显式优于隐式（What, not How）**——告诉模型做什么有边界，怎么做留自由度
7. **Agent prompt 特有三块**：循环意识 / 反思触发 / 终止判断显式声明——这是 #059 普通 prompt 没有的维度
8. **今日最大洞察（跨 #059/#061/#062 闭环）**：生产 Agent = Agentic prompt（软约束教思维流程）+ JSON Mode（硬约束保格式）+ 代码 HITL（兜底护栏），三层叠加才稳

## 📚 今日阅读（文档副本已放入本目录）

| 顺序 | 编号 | 标题                          | 文件                                | 状态    |
| ---- | ---- | ----------------------------- | ----------------------------------- | ------- |
| 1    | #027 | MCP（Model Context Protocol） | `./027-model-context-protocol.md`   | ✅ 已读 |
| 2    | #059 | System Prompt 设计核心原则    | `./059-system-prompt-principles.md` | ✅ 已读 |
| 3    | #061 | 结构化输出（JSON / XML）      | `./061-structured-output.md`        | ✅ 已读 |
| 4    | #062 | Agentic Prompting             | `./062-agentic-prompting.md`        | ✅ 已读 |

> 原则：「做中读」—— 带着 Day 1/2 写代码时的亲身困惑去印证，不追求记全

## 📝 读后感（详见 [read_feelings](./read_feelings)）

- **#027 — MCP**：FC=私有协议（model↔host），MCP=公共协议（host↔tool server）；MCP 把工具从应用进程解耦为独立服务，单 host 单工具时强行 MCP 化是过度设计
- **#059 — System Prompt**：软约束本质（教育模型），不是硬护栏；关键路径必须代码兜底；开头+结尾重复对抗 Lost in the Middle；生产六件套（身份/工具规则/不确定性/思维触发/格式/结尾重申）
- **#061 — 结构化输出**：强制（API 层）vs 信任（LLM 自觉）可靠性差一个数量级；FC > JSON Mode > prompt 自觉；JSON Schema 是 API+LLM 共享的契约语言
- **#062 — Agentic Prompting**：显式优于隐式（What, not How）；Agent prompt 多三块（循环意识/反思触发/终止判断）；跟 #061 串联=软硬约束双轨（教育+法律）

## ❓ 困惑/待消化

### Q1：MCP 跟 Function Calling 到底是替代还是叠加？

**答**：**叠加**——它们管不同的链路段：

```
LLM (DeepSeek/Claude/...)
   ↑ Function Calling 协议（model ↔ host）
Host 应用 (Claude Desktop / Cursor / 你的 agent.py)
   ↑ MCP 协议（host ↔ tool server）
MCP Server (独立工具进程)
```

**模型从头到尾不知道 MCP 存在**——host 偷偷把 MCP server 暴露的工具翻译成 FC schema 喂给 LLM。

**类比（USB-C 模型）**：MCP = USB-C 接口标准（工具厂商造一次，所有 host 都能用）；FC = 设备 USB 驱动（OS 怎么跟接口里的设备通信）。

### Q2：System Prompt 能不能当强制护栏？

**答**：**不能，永远是软约束**。

| 失效模式 | 原因                               | 对策                              |
| -------- | ---------------------------------- | --------------------------------- |
| **遗忘** | 上下文长，开头约束被冲淡           | 关键约束开头+结尾重复             |
| **越狱** | 用户消息伪装系统指令、多轮诱导     | 代码层过滤 + 输出校验             |
| **歧义** | "高危操作要确认"——什么算高危没说清 | 列白名单/黑名单，别让模型自己判断 |

**生产真相：教育（system prompt）+ 法律（代码 input()/interrupt()）双轨，缺一不可**。

### Q3：Function Calling 是怎么"保证"返回合法 JSON 的？

**答**：**API 双重机制**：

| 路径                             | 谁干的                            | 可靠性           |
| -------------------------------- | --------------------------------- | ---------------- |
| **训练时**学会吐 JSON            | 厂商在模型微调阶段教过它          | 90-95%（会出错） |
| **解码时**强制只能输出合法 token | API 在背后做 constrained decoding | 99.9%+（强制）   |

→ OpenAI 早期靠训练，后来推 "strict mode" 也走上 constrained decoding；Anthropic 一直在做这条路径。

### Q4：Day 1 反复横跳，三档修复方案差在哪？

**答**：力度递增，失效空间递减：

| 方案                       | 修复机制                                                           | 失效场景                                   |
| -------------------------- | ------------------------------------------------------------------ | ------------------------------------------ |
| 普通 system prompt         | "工具失败必须停手" → 模型可能听话也可能换花样                      | 模型把"失败"理解成"参数不对，换个参数再试" |
| Agentic prompt             | "工具失败 → **先反思**是参数错还是工具坏 → 工具坏就 STOP"          | 强制走反思流程，瞎试空间少很多             |
| Agentic prompt + JSON Mode | 上面那段 + 输出强制 `{"reflection":..., "decision":"stop\|retry"}` | 几乎不失效——必须在枚举值里选               |

**一句话差别**：#059 告诉模型"边界"（防做错事），#062 告诉模型"怎么思考"（防做傻事）。

## 🚀 下一步

- **Phase 1 收官** —— 11 篇文档全部读完，tiny-agent-loop 心智地图成型
- **tiny-agent-loop v0.4 候选**（不一定立刻做）：
  - 加 system prompt（按今天六件套写）
  - dispatch 出错时加 input() HITL 兜底
  - 关键决策用 JSON Mode + Agentic prompt 写一次试试
- **进入 Phase 2 RAG**（推荐路径）：tiny-rag 项目，#011→#020 共 10 篇；前置认知已埋好（短期/长期记忆喂入方式差异 + JSON Mode 做 eval 打分）

## 📝 备忘

- **Phase 1 收官的最大收获**：知识开始**横向结网**——HITL 跨 #003/#007/#059/#062 反复出现；Lost in Middle ↔ detect_loop 同源；软硬约束在 prompt/输出/HITL 三层都成立
- **写读后感的最佳姿势**（Day 1→3 的进化）：从"罗列知识点"→"建立心智地图"→"主动跨篇章串联"。**笔记是给三个月后的自己看的，跨篇章的钩子比单篇要点值钱 10 倍**
