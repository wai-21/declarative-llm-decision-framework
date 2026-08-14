---
name: quant-token-opt
description: >
  Token-cost optimization playbook for local OpenClaw quant / trading / signal agents. USE when the user
  wants to reduce tokens / cut cost / slim the prompt / control the context window / reduce LLM call
  count / optimize a quant agent's context / apply model layering — especially for agents running
  quotes, indicators, and signal decisions. Ten strategies ranked by cost-performance; the fastest
  combo (externalize computation → model layering → structured output → limit window → cap output)
  typically cuts 60–80% of tokens. Reference only — no code, no config edits unless the user explicitly
  asks to deploy into openclaw.json.
version: 1.0.0
metadata:
  agent_created: true
  root: ~/.qclaw
  installed: 2026-08-14
---

# Quant Agent Token Cost Optimization

For: local OpenClaw quant / trading / signal agents. Goal: minimum token consumption at identical decision quality.

## Three token sources (locate the biggest leak first)

1. Large market/indicator data stuffed into the prompt (**input, most controllable**)
2. Accumulated chat history + tool results (**grows linearly per turn**)
3. System prompt, tool descriptions, overly long outputs (**re-billed every turn + output is the priciest**)

First decide which bucket your agent is stuck in, then treat that one. Do not apply all ten at once.

## Ten strategies (ranked by cost-performance)

### 1. Externalize computation — LLM reads only the conclusion (biggest input saving)
- Never let the LLM read raw K-line / tick / full indicator series.
- Pre-compute RSI, MACD, moving averages, volatility in Python/shell; return only the final signal + a short reason.
- Tool return template (single entry ≤ 80 tokens):
```json
{"symbol":"BTCUSDT","timeframe":"1h","signal":"long","confidence":0.72,"reason":"RSI 35 rebound, MACD golden cross"}
```

### 2. Model layering — small quant model for high frequency
- High-frequency simple signals → `qwen2.5:7b-instruct-q4_K_M` or a smaller quant model.
- Complex review / news analysis / strategy explanation → large model.
- Pure rule triggers → straight Python, **no LLM at all**.
- A small quant model with a low context window is enough for fixed-format decisions.

### 3. Slim the system prompt and tool descriptions
- Every extra paragraph in the system prompt is re-billed every turn; tool descriptions enter the schema of every request.
- Cut: repeated role statements, long examples, filler.
- Strategy rules as short sentences/lists; tool descriptions only state "purpose, params, return format".

### 4. Limit the context window — sliding window
- Shrink `contextWindow` (e.g. 8k–16k); keep only the most recent N turns / M messages.
- A quant agent does not need to remember every single signal of the whole day.
- Verified OpenClaw landing points (checked on this machine):
  - Model level: `models.providers.<provider>.models[].contextWindow` / `.maxTokens`
  - Agent level: `agents.list[].model` points to a specific model; `agents.defaults` has `maxConcurrent`, `timeoutSeconds`

### 5. Memory summarization — drop raw history
- Once over a threshold, compress old content into a summary; do not keep full indicator returns, full press releases, full logs.
- Summary template:
```
Decision made: BTC 1h long @65000 | Current position: 1 contract | Last result: +0.8%
```

### 6. Prompt caching (fixed prefix hits KV cache)
- Keep system prompt, tool schema, strategy notes as stable as possible.
- Locally use Ollama / llama.cpp (cache supported); confirm whether OpenClaw exposes the setting.
- Effect: fixed content hits cache, repeated-input cost drops sharply.

### 7. Trim tool results + force structured output
- Each tool returns only necessary fields, not a full analysis report.
- `maxTokens` 512–1024, output only four fields: `signal / confidence / reason / risk_level`.
- Use JSON mode or regex to constrain format; ban long prose.

### 8. Batch tool calls — fewer round-trips
- Anti-pattern (3 calls): query BTC RSI → query BTC MACD → query ETH RSI.
- Good pattern (1 call):
```json
{"symbols":["BTCUSDT","ETHUSDT"],"indicators":["rsi","macd","ema"],"timeframe":"1h"}
```
- One call returns multi-symbol, multi-indicator summaries; the LLM decides once, completely.

### 9. Lower frequency — event-driven / batch processing
- Do not call the LLM every tick / every minute. Instead: once per 15min–1h; trigger only on price breakout / indicator flip; batch a set of signals for one analysis.
- Batching compresses multiple events into one summary — saves scheduling + repeated prompts.

### 10. Cap output — low temperature + maxTokens + stop sequence
- `temperature = 0~0.3`; `maxTokens = 512~1024`; stop sequence e.g. `\n---` or `</output>`.
- A fixed template lets the model fill only a few fields; short JSON is the cheapest.
- Output is the priciest and most overlooked item.

## Fast-win priority

Just these five usually cut 60–80%:

```
1 → 2 → 7 → 4 → 10
```

The rest (3/5/6/8/9) are rolled in gradually based on OpenClaw version and inference-service support.

## Deployment checklist (review each item before editing config)

| # | Check | Action |
|---|---|---|
| 1 | Is the LLM still reading raw market series? | Externalize — return only signal + reason |
| 2 | Does high-frequency work go through a large model? | Switch to small quant model or pure Python |
| 7 | Does the tool return a full report? | Trim to 4-field JSON |
| 4 | Is `contextWindow` > 16k? | Shrink + sliding window |
| 10 | Are `temperature`/`maxTokens`/stop set? | 0~0.3 / 512~1024 / set stop |

> Config key names drift across OpenClaw versions. Before deploying, verify the actual fields with:
> `python3 -c "import json;print(list(json.load(open('$HOME/.qclaw/openclaw.json'))['models']['providers'].keys()))"`
> Do not copy from docs blindly.
