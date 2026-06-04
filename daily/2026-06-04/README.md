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
| 2    | #021 | Function Calling 基础     | `./021-function-calling-basics.md`       | 📖 在读 |
| 3    | #006 | Agent Loop 设计与错误恢复 | `./006-agent-loop-and-error-recovery.md` | ⏳ 待读 |
| 4    | #022 | Tool Schema 设计          | `./022-tool-schema-design.md`            | ⏳ 待读 |

> 原则：「做中读」—— 带着 Day 1 写代码时的亲身困惑去印证，不追求记全

## ❓ 困惑/待消化

- （暂无 — 进入阅读阶段后再补充）

## 🚀 下一步

- 精读上述 4 篇文档，对照 `agent.py` 代码"对号入座"
- 读完后决定：进 Day 3（加错误处理 + 真实天气 API） vs 直接进 tiny-rag

## 📝 备忘

- 解决了一个工程问题：socks5 代理 + GitHub 密码认证失效 → 用 Personal Access Token (classic, scope=repo) push 成功
- DeepSeek `deepseek-chat` 用的是 OpenAI 兼容协议，参数是 JSON 字符串需 `json.loads()`，结果回填用 `role:"tool"` + `tool_call_id`
