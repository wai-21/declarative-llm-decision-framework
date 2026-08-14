---
name: quant-token-opt
description: >
  本地 OpenClaw 量化/交易 agent 的 token 成本优化手册。MUST USE 当用户要 降 token / 省成本 /
  精简 prompt / 控制 context window / 减少 LLM 调用次数 / 优化量化 agent 上下文 / 模型分层 时，
  尤其针对跑行情、指标、信号决策的 agent。十大策略按性价比排序，最快见效组合（指标外部化 →
  模型分层 → 结构化输出 → 限窗 → 限输出）通常降 60~80% token。
  纯参考，不写代码、不改配置，除非用户明确要求落地到 openclaw.json。
version: 1.0.0
metadata:
  agent_created: true
  root: ~/.qclaw
  installed: 2026-08-14
---

# 量化 Agent Token 成本优化

适用：本地 OpenClaw 跑的量化/交易/信号 agent。目标：同样决策质量下把 token 消耗压到最低。

## token 三大来源（先定位大头）

1. 大量行情/指标数据塞进 prompt（**输入，最可控**）
2. 历史对话 + 工具结果累积（**随轮次线性增长**）
3. system prompt、工具 description、输出过长（**每轮重复计费 + 输出最贵**）

先判断你的 agent 卡在哪一处，再对症下药，不要十条全上。

## 十大策略（按性价比排序）

### 1. 指标计算外部化，LLM 只读结论（省输入最猛）
- 禁止 LLM 直接读 K 线 / tick / 完整指标序列。
- 用 Python/shell 先算好 RSI、MACD、均线、波动率，只回传最终信号 + 短原因。
- 工具回传模板（单条 ≤ 80 token）：
```json
{"symbol":"BTCUSDT","timeframe":"1h","signal":"long","confidence":0.72,"reason":"RSI 35 反弹，MACD 金叉"}
```

### 2. 模型分层，高频率用量化小模型
- 高频率简单信号 → `qwen2.5:7b-instruct-q4_K_M` 或更小量化模型。
- 复杂复盘/新闻分析/策略解释 → 大模型。
- 纯规则触发 → 直接 Python，**不经过 LLM**。
- 量化小模型 + 低 context 足够处理固定格式决策。

### 3. 精简 system prompt 与工具 description
- system prompt 每多一段 = 每轮重复计费；工具 description 会进每次请求 schema。
- 删：重复角色说明、长示例、空话。
- 策略规则用短句/列表；工具 description 只写「用途、参数、回传格式」。

### 4. 限制 context window，滑动窗口
- 把 `contextWindow` 调小（如 8k~16k），只保留最近 N 轮 / M 条消息。
- 量化 agent 不需要记一整天每一笔信号。
- OpenClaw 实际落点（本机已核实）：
  - 模型级：`models.providers.<provider>.models[].contextWindow` / `.maxTokens`
  - agent 级：`agents.list[].model` 指向具体模型；`agents.defaults` 有 `maxConcurrent`、`timeoutSeconds`

### 5. 记忆摘要压缩，不保留原始历史
- 超阈值就把旧内容压成摘要，不要保留完整指标回传、完整新闻稿、完整 log。
- 摘要模板：
```
已做决策：BTC 1h 做多 @65000 ｜ 当前持仓：1 口 ｜ 上次结果：获利 0.8%
```

### 6. Prompt caching（固定前缀命中 KV cache）
- system prompt、工具 schema、策略说明尽量不变。
- 本地用 Ollama / llama.cpp（支持 cache），确认 OpenClaw 是否有对应设置。
- 效果：固定内容命中缓存，重复输入成本大幅下降。

### 7. 工具结果裁剪 + 强制结构化输出
- 每个工具只回传必要字段，不要整份分析报告。
- `maxTokens` 设 512~1024，只输出四字段：`signal / confidence / reason / risk_level`。
- 用 JSON mode 或 regex 限制格式，禁止长篇解释。

### 8. 合并工具调用，减少来回
- 反例（3 次调用）：查 BTC RSI → 查 BTC MACD → 查 ETH RSI。
- 正例（1 次调用）：
```json
{"symbols":["BTCUSDT","ETHUSDT"],"indicators":["rsi","macd","ema"],"timeframe":"1h"}
```
- 一次回传多品种、多指标摘要，LLM 一次做完整决策。

### 9. 降频：事件驱动 / 批处理
- 不要每 tick / 每分钟调 LLM。改成：每 15 分钟~1 小时一次；价格突破/指标翻转才触发；攒一批信号一次分析。
- 批次把多事件压成一段摘要，省排程 + 省重复 prompt。

### 10. 限制输出：低温 + maxTokens + 停止序列
- `temperature = 0~0.3`；`maxTokens = 512~1024`；stop sequence 如 `\n---` 或 `</output>`。
- 固定模板让模型只填少数字段，短 JSON 最省。
- 输出是最贵、最常被忽略的一项。

## 快速见效优先级

只做这五项通常就能降 60~80%：

```
1 → 2 → 7 → 4 → 10
```

其余（3/5/6/8/9）按 OpenClaw 版本与推理服务支持度逐步导入。

## 落地检查清单（改配置前逐项过）

| # | 检查项 | 动作 |
|---|---|---|
| 1 | LLM 是否还在读原始行情序列 | 外部化，只回信号+原因 |
| 2 | 高频率任务是否走了大模型 | 换量化小模型或纯 Python |
| 7 | 工具是否回传整份报告 | 裁剪到 4 字段 JSON |
| 4 | `contextWindow` 是否 > 16k | 调小 + 滑动窗口 |
| 10 | `temperature`/`maxTokens`/stop 是否设好 | 0~0.3 / 512~1024 / 设 stop |

> 配置键名随 OpenClaw 版本变化，落地前先 `python3 -c "import json;print(list(json.load(open('$HOME/.qclaw/openclaw.json'))['models']['providers'].keys()))"` 核验实际字段，别照抄文档。
