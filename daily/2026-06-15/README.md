# 2026-06-15 · Day 4

**阶段**：Phase 1 收官后 · **做中复**实战
**主题**：tiny-agent-loop v0.4「反复横跳治愈包」—— 用代码复习 Phase 1 全部知识

---

## 🎯 今日目标

- 用「做中复」代替痛苦的默写复习（L2 周复习），把 Phase 1 学到的所有治疗手段落到 tiny-agent-loop 代码里
- 按 [v0.4 路线图](../../v0.4-roadmap.md) 六道任务，每做完一道自问「我刚用了哪一篇的什么概念」
- 验证理论是否真的能落地（写代码比读笔记残酷得多——立刻暴露认知漏洞）

## 🛠️ 产出

- ✅ 切分支 `v0.4-reflexive-recovery`，commit `v0.4`，推送到 GitHub
- ✅ Task 1: TOOL_REGISTRY dict 重构派发表 (#021)
- ✅ Task 2: system prompt 六件套（含内置知识禁令 + 自我正当化检测）(#059 + #062)
- ✅ Task 3: 错误分层 dispatch（RecoverableError vs FatalError + max_retry） (#006)
- ✅ Task 4: detect_loop / turn signature 检测（走了三版弯路才到位） (#006 Q1)
- ✅ Task 5: JSON Mode + llm_reflect（含 retry/stop decision 双轨） (#061 + #062)
- ⏳ Task 6: HITL `input()` 兜底 —— 明天补
- ✅ 终极验证跑通：复刻 Day 1 反复横跳实验，确认 v0.4 行为差异

## 💡 核心收获（今天踩出来的真知，全是新的）

### 1. 「按轮检测」vs「按调用检测」——决策粒度的本质差异

最初想法：每次 tool call 都查重 → 用 `sorted(action_history)` hack → 在测试场景能触发但语义错。

**关键顿悟**：循环不是工具在循环，是 **LLM 的脑子在打转**。所以应该按"一轮决策的指纹"对比，而不是按"工具调用流水"对比。

```python
sig = " | ".join(sorted(f"{call.name}({call.args})" for call in msg.tool_calls))
if sig in turn_signatures: ...  # 跨轮判等，干净利落
```

→ 这一步对应 #002「Agent 是有自主决策」的本质——**循环的载体是决策，不是动作**。

### 2. LLM 是概率函数，同一输入也可能不同输出

调用 `llm_reflect()` 对同一个 `TypeError`，**一次返回 retry 一次返回 stop**。痛感比读 #001 强 100 倍。

**根因两条**：

- 模型本质是从概率分布里采样
- reflect prompt 没给"什么时候 retry / 什么时候 stop"的标准 → LLM 凭直觉乱猜

**修复方向**：

- prompt 里写清楚标准（结构性错误→stop / 临时性错误→retry）
- `temperature=0` 减少随机性
- 但永远不可能 100% 稳定 —— **这就是为什么生产必须代码兜底**

### 3. JSON Mode 只锁"格式"，不锁"内容"——绝对不能混淆

| JSON Mode 保证                         | 不保证                       |
| -------------------------------------- | ---------------------------- |
| ✅ 输出是合法 JSON                     | —                            |
| ✅ 有 reflection / decision 这两个 key | —                            |
| —                                      | ❌ decision 字段的值是稳定的 |

→ **结构化输出 ≠ 结构化思考**。拿到合法 JSON ≠ 拿到一致决策。

### 4. 软硬约束双轨——实战版

终极验证里出现了一个绝佳的"软约束兜底"案例：

- `get_city_play` 调用失败 → reflect 给出 `decision: "retry"` （没硬停）
- 但 result 拼接了"反思+建议"喂给 LLM
- LLM 看到上下文 + system prompt 里的"工具失败必须停手"
- **LLM 自己决定不再调工具，主动输出 final answer**

→ 硬约束（`if decision==stop: return`）没触发，但**软约束（system prompt + 反思上下文）救了场**。这才是 #061 "教育+法律"双轨真正的运行方式。

### 5. 「description = name」时 LLM 会装聋作哑

故意把 `get_city_play` 的 description 写成 `"get_city_play"`（零信息量）+ 参数名错位，发现 LLM 倾向于"先尝试调用看看"而不是"我看不懂就拒绝"——印证 #022「三要素相辅相成，一错两对仍能推断；三个都模糊就崩」。

### 6. Python 作用域踩坑：函数内任何赋值都会让变量被当作局部

加了 `action_history = []` 这行在函数内，导致后续 `action_history.append()` 报 UnboundLocalError——因为 Python 编译时把整个函数里的 `action_history` 都视为局部变量。

→ 教训：**全局可变状态是反模式**。直接把状态搬进函数内初始化，比贴 `global` 关键字胶布优雅。

## 📚 今日"复习"了 Phase 1 哪些文章

| Task                  | 复用概念                               | 来源           |
| --------------------- | -------------------------------------- | -------------- |
| 1 TOOL_REGISTRY       | 派发表机制                             | #021           |
| 2 system prompt       | 六件套 + Lost in Middle + Agentic 三块 | #059 + #062    |
| 3 错误分层            | 可恢复 / 不可恢复 / 硬上限             | #006           |
| 4 turn signature      | 决策粒度 + 集合去重                    | #006 Q1 + #002 |
| 5 JSON Mode + reflect | 软硬约束双轨 + 结构化输出              | #061 + #062    |
| 终极验证              | LLM 概率性 + 工具 schema 三要素        | #001 + #022    |

→ **直接复习掉的 L2 项目**：`review-log.md` 里 Day 1 / Day 3 那几条逾期 L2（#001 / #006 / #021 / #022 / #059 / #061 / #062）。这次做中复一次干掉 7 篇。

## ❓ 困惑/待消化

### Q1：为什么"按轮检测"是对的，"按调用检测"是错的？

**答**：因为**循环的本质是"决策模式的重复"，不是"工具调用的重复"**。一个 Agent 在合法工作中可能反复调用同一工具（如多次 search），但每次的"轮决策"是不同的。

把 history 单位从"一次 call"换成"一轮完整决策"，检测语义才匹配 LLM 行为的真实粒度。

### Q2：JSON Mode 拿到结构化输出 ≠ 拿到一致决策

亲身验证：`response_format={"type":"json_object"}` 只保证拿到合法 JSON，**`decision` 字段的内容仍可能每次不同**。

教训：任何关键判断不能仅靠"我有 JSON 字段就行"，必须配 prompt 标准 + 代码兜底。

### Q3：什么时候用硬约束，什么时候用软约束？

| 场景                                                   | 用什么                                        |
| ------------------------------------------------------ | --------------------------------------------- |
| 不可越界的事（不删数据/不超 max_turns/不让程序崩）     | **硬约束**（if/return/raise）                 |
| 引导风格的事（先 thought 再 action / 输出格式 / 礼貌） | **软约束**（system prompt）                   |
| 复杂决策（retry vs stop / 是否高危）                   | **软引导 + 硬兜底**（prompt 教育 + 代码守门） |

## 🚀 下一步

- **Task 6 HITL** —— 给 dispatch 加 `input()` 卡点，让 FatalError 时强制人工确认
- **reflect 加固** —— 给 prompt 加判断标准（结构性 vs 临时性）+ `temperature=0`
- **v0.4 合 main 发布** —— PR: https://github.com/leeyifan111-dotcom/tiny-agent-loop/pull/new/v0.4-reflexive-recovery
- **review-log 同步** —— 把今天"做中复"覆盖的 7 篇文章在排期表里标记为 ✅，写明触发结果
- **进 Phase 2** —— tiny-rag（#011→#020），前置认知齐了：短期/长期记忆喂入方式 + JSON Mode 做 eval 打分 + 软硬双轨设计

## 📝 备忘 / 教训

- **AI 助手两次把测试条款当 bug**：（1）prompt 里"每个工具调用 5 次" （2）`get_city_play` schema 故意写错。**写实验代码时要注释清楚是 setup**，否则跨会话 AI 也会误判
- **做中复 vs 默写复习**：今天 1 个晚上的代码实验，复习深度超过 L2 默写 30 分钟好几倍——**痛感来源是"代码会立刻暴露认知漏洞"**，光看笔记没有这种反馈
- **意外发现**：当 `decision != "stop"`，LLM 自己选择停下来给 final answer——这是"软约束兜底"的活样本，比任何理论都生动
