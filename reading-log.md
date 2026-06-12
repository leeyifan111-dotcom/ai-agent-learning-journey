# 📚 阅读追踪

> 状态说明：⏳ 待读 · 〰️ 扫读 · ✅ 精读完成
> 关联代码 = 这篇对应你写过/即将写的代码段

## 阶段 1 · Agent 全景认知

| #   | 标题                                                        | 状态 | 关联代码                           | 读后心得 / 困惑                                                     |
| --- | ----------------------------------------------------------- | ---- | ---------------------------------- | ------------------------------------------------------------------- |
| 001 | 什么是 LLM Agent？与传统 LLM 应用有何区别？                 | ✅   | agent.py 整体                      | 本质=在环境中持续用工具推理决策直到目标达成；五特征非单问答         |
| 002 | 解释 Agent 的核心组件：感知、推理、行动、记忆               | ✅   | `messages` + `dispatch` + `create` | 四件套靠 messages 串联；短期=全量喂，长期=检索喂（RAG 伏笔）        |
| 003 | Agent 架构模式：ReAct / Plan-and-Execute / LATS / Proactive | ✅   | tiny-agent-loop = ReAct            | ReAct vs P&E 核心差异=决策粒度；HITL 是跨架构生产救场术             |
| 006 | Agent Loop 设计：循环控制、终止条件与错误恢复               | ✅   | `for turn in range(max_turns)`     | 限制条件 + 确定终止条件双保险；衍生多 Agent 死循环延伸思考          |
| 007 | Workflow vs Agent                                           | ✅   | tiny-agent-loop = 纯 Agent         | 分界点=LLM 是否决定流程；生产最常见是 Agentic Workflow 混合         |
| 021 | Function Calling 基础                                       | ✅   | `tools=tools` & `tool_calls` 处理  | LLM 生成 JSON 开工单，本地执行 + 回填；批量 vs 并行实验             |
| 022 | Tool Schema 设计                                            | ✅   | `tools` 列表                       | 三要素 name/description/input_schema 相辅相成减少试错               |
| 027 | MCP（Model Context Protocol）                               | ✅   | tools 协议化（未来 v0.5）          | FC=私有协议(model↔host)，MCP=公共协议(host↔tool server)；叠加非替代 |
| 059 | System Prompt 设计核心原则                                  | ✅   | agent.py 待补 system prompt        | 软约束本质；开头+结尾对抗 Lost in Middle；生产六件套                |
| 061 | 结构化输出（JSON/XML）                                      | ✅   | Phase 2 RAG eval 打分              | 强制(API) vs 信任(LLM) 差一个数量级；JSON Schema 是契约语言         |
| 062 | Agentic Prompting                                           | ✅   | Day 1 反复横跳三档修复推演         | 显式优于隐式；Agent prompt 三块（循环/反思/终止）+ JSON Mode 双轨   |

## 阶段 2 · RAG（10 篇，待启动）

| #   | 标题               | 状态 |
| --- | ------------------ | ---- |
| 011 | 什么是 RAG         | ⚪   |
| 012 | RAG Pipeline 组件  | ⚪   |
| 013 | 分块策略           | ⚪   |
| 014 | 向量数据库选型     | ⚪   |
| 015 | Embedding 模型选择 | ⚪   |
| 016 | 混合检索           | ⚪   |
| 017 | Re-ranking         | ⚪   |
| 018 | Agentic RAG        | ⚪   |
| 019 | 高级 RAG 变体      | ⚪   |
| 020 | RAG 评估指标       | ⚪   |

## 阶段 3 · 补认知短板（13 篇，待启动）

040 · 041 · 044 · 049 · 050 · 052 · 064 · 078 · 079 · 082 · 031 · 033 · 037

## 阶段 4 · 数据工程优势（12 篇，待启动）

086 · 087 · 088 · 090 · 092 · 094 · 095 · 104 · 107 · 108 · 089 · 102
