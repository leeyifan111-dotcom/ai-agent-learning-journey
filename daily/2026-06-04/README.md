# 2026-06-04 · 启动日

**阶段**：1（Agent 全景认知）
**主题**：tiny-agent-loop 从 0 跑通

---

## 🎯 今日目标

- 跑通一个最小可用的 Agent 循环
- 理解 Function Calling 的核心机制
- 把代码做成第一个 GitHub 作品

## 🛠️ 产出

- ✅ 写完 `agent.py`（~60 行），基于 DeepSeek API（OpenAI 兼容协议）
- ✅ 完整跑通 Agent Loop：LLM 决策 → 工具调用 → 结果回填 → 再决策 → 结束
- ✅ 配齐工程基础：`.env` 隔离密钥、`.gitignore`、README、独立 git 仓库
- 🆕 GitHub 仓库 [tiny-agent-loop](https://github.com/leeyifan111-dotcom/tiny-agent-loop) 创建并 push 成功
- 🧪 做了 2 次破坏性实验（改工具名/描述），验证模型选工具的真实机制

## 💡 核心收获

1. **Agent ≠ 单次问答**：本质是一个由 `messages` 不断增长驱动的循环
2. **LLM 不执行工具**：它只"开工单"返回 `tool_calls`，本地代码负责真正执行 + 回填
3. **工具选择是模型推理**，不是代码逻辑：靠 `name + description + parameters` 综合语义判断，三者**互相印证、互相推导**
4. **Schema 三部分的分工**：
   - `name + description` → 主要决定**选哪个工具**
   - `parameters` → 决定**填什么参数**（格式契约，必须严格遵守）
   - 语义上三者冗余兜底；格式上 parameters 不容偏差
5. **破坏性实验观察**：工具失败时，模型不会崩，而是**反复试错 → 最终用内置知识糊弄过去**（幻觉风险，对应 #082）

## 📚 今日阅读（文档副本已放入本目录）

| 顺序 | 编号 | 标题                      | 文件                                     | 状态    |
| ---- | ---- | ------------------------- | ---------------------------------------- | ------- |
| 1    | #001 | 什么是 LLM Agent          | `./001-what-is-llm-agent.md`             | ✅ 已读 |
| 2    | #021 | Function Calling 基础     | `./021-function-calling-basics.md`       | ✅ 已读 |
| 3    | #006 | Agent Loop 设计与错误恢复 | `./006-agent-loop-and-error-recovery.md` | ✅ 已读 |
| 4    | #022 | Tool Schema 设计          | `./022-tool-schema-design.md`            | ✅ 已读 |

> 原则：「做中读」—— 带着 Day 1 写代码时的亲身困惑去印证，不追求记全

## 📝 读后感（一句话自我总结，见 [read_fellings](read_feelings)）

- **#001 — 什么是 LLM Agent**：Agent 的本质是**在环境中持续用工具推理决策直到目标达成**——有状态的、感知外界的、多步推理的、自主的、自适应的，而不是无状态的、单一问答的、被动响应的。
- **#021 — Function Calling 基础**：Agent 通过工具定义让 LLM 生成 JSON 请求调用工具。工具组成为 `name / description / input_schema`；除此之外还要告诉 LLM 有哪些工具（schema），并在本地维护"工具名 → 函数"的派发表，调用结果回填后 LLM 再根据返回继续推理。
- **#006 — Agent Loop 与错误恢复**：Agent 总有 loop 循环，为防无限循环需要**限制条件**（最大重试次数、单次任务最大时间、模型降级、网络抖动重试）和**确定的终止条件**（超过最大重试 / 单次最大时间 / 模型结束信号 / 重复任务 / 最大资源）。
- **#022 — Tool Schema 设计**：Tools 定义三要素 `name / description / input_schema` 相辅相成，LLM 据此生成 JSON 调用参数。高质量的定义往往能减少 LLM 试错成本。

## ❓ 困惑/待消化

### Q1：`detect_loop` 这段代码看不懂（来自 #006）

```python
def detect_loop(history: list[str], window: int = 3) -> bool:
    recent = history[-window:]
    return len(set(recent)) < len(recent)
```

**答**：用「集合自动去重」的特性判断最近 N 步是否有重复操作。

- `history[-window:]` → 取最近 3 步
- `set(recent)` → Python 集合**自动去重**
- 比较长度：set 比 list 短 → 说明有重复 → 在打转 → 返回 `True`

**举例**：

- `["搜索", "查天气", "查天气"]` → set 长度 2 < list 长度 3 → 检测到循环 ✅
- `["搜索", "查天气", "回答"]` → set 长度 3 = list 长度 3 → 正常 ❌

**联系自己**：Day 1 那次「工具改坏，模型反复横跳 4 次」如果加了 `detect_loop`，第 3 次重复时就能刹车。**这是 Agent Loop 防失控的标准防线之一**，v0.2 可以考虑加。

---

### Q2：`request_heartbeat` 是什么？（来自 #006 表格里的 MemGPT 行）

**答**：MemGPT 框架里 LLM 调用工具时多带的一个布尔参数，控制「工具执行完后是否要再喂一轮给 LLM 继续推理」。

**与 ReAct 的本质区别 —— 默认行为相反**：

| 架构            | 默认行为         | 想要相反时怎么做                                   |
| --------------- | ---------------- | -------------------------------------------------- |
| ReAct（我写的） | 默认**继续循环** | LLM 主动给 "Final Answer" 才停                     |
| MemGPT          | 默认**终止**     | 工具调用必须显式带 `request_heartbeat=True` 才继续 |

**为什么 MemGPT 这么设计**：默认终止更安全 → 防止模型陷入循环烧 token（正好对应我 Day 1 观察到的"反复横跳"问题）。代价：对模型的指令遵循能力要求更高（必须明确表达"我还要继续"）。

**启发**：`request_heartbeat` 本质是把「继续执行」从隐式假设变成**显式信号**。v0.2 加错误处理时可以借鉴这种思路 —— 让 LLM 主动表态是否继续，比纯靠 `max_turns` 兜底更优雅。

---

### Q3：多 Agent 的"乒乓球死循环" —— A 和 B 反复 handoff 怎么办？（#006 没讲，自己延伸）

**场景**：客服系统中前台 A 把"账号登不上"丢给技术 B，B 觉得信息不够丢回 A，A 又觉得这是技术问题再丢给 B …… 死循环。

**根因**：跟单 Agent 死循环本质同源 —— **没有"必须自己解决"的兜底路径**。

- 单 Agent 版：工具坏了没 Plan B → 反复试同一个工具
- 多 Agent 版：职责边界模糊 + handoff 是合法选项 → 互相踢皮球

**4 种修复手段（由表入里）**：

1. **全局 handoff 上限** —— 类似 `max_turns` 但作用域是整段对话，超限转人工（治标）
2. **任务签名去重** —— `detect_loop` 思路升级：检测 `(from, to, task_hash)` 元组是否重复
3. **中心调度（Orchestrator）** —— 取消对等 handoff，所有任务流转过仲裁者；LangGraph / AutoGen 的核心设计
4. **system prompt 写死边界**（根治）—— 给每个 Agent 强制规定"信息不足时必须自己反问/给可执行建议，禁止退回"

**伏笔**：阶段 3（#031 / #033 / #037）会正式学多 Agent 协作，回头印证这个推断对不对。

## 🚀 下一步

- 精读上述 4 篇文档，对照 `agent.py` 代码"对号入座"
- 读完后决定：进 Day 3（加错误处理 + 真实天气 API） vs 直接进 tiny-rag

## 📝 备忘

- 解决了一个工程问题：socks5 代理 + GitHub 密码认证失效 → 用 Personal Access Token (classic, scope=repo) push 成功
- DeepSeek `deepseek-chat` 用的是 OpenAI 兼容协议，参数是 JSON 字符串需 `json.loads()`，结果回填用 `role:"tool"` + `tool_call_id`
