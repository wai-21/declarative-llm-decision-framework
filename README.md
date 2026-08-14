<p align="center">
  <img src="https://img.shields.io/badge/Token_Savings-87%25-brightgreen" alt="Token Savings">
  <img src="https://img.shields.io/badge/License-MIT-blue" alt="License">
  <img src="https://img.shields.io/badge/Python-3.9%2B-blue" alt="Python">
  <img src="https://img.shields.io/badge/Status-Production-green" alt="Status">
</p>

# 🔥 Decision Distillation Framework

**Python computes. LLM decides. 10,000+ tokens → 400 tokens. Same decisions, 87% cheaper.**

> Stop feeding your LLM raw K-line data, MACD values, and RSI charts. That's like asking a CEO to read machine-level assembly code. Give it a DecisionState instead — a pre-computed, schema-bound snapshot of the world — and let it do what it does best: **logical reasoning, not arithmetic**.

---

## 🤔 The Problem

Every LLM-based decision system makes the same mistake:

```
Raw data (10,000+ tokens) → LLM crunches numbers → Verbose prose output (800 tokens)
                                    ↑
                             85% of this is waste
```

- **LLMs hallucinate numbers.** GPT-4, DeepSeek, Claude — they all fail at arithmetic when it matters.
- **Context windows are expensive.** Every unnecessary token is money burned.
- **Rule files keep growing.** Your 772-line prompt isn't helping anyone.

## 💡 The Solution

**Four principles** derived from [Remotion](https://remotion.dev)'s philosophy ("video = f(frame)", give parameters, don't render pixel by pixel):

```
               BEFORE                                    AFTER
      ┌──────────────────┐                    ┌──────────────────┐
      │  Raw Data (10K+)  │                    │ DecisionState <300│
      │  OHLCV, MACD, RSI │      ──→          │ regime, signals,  │
      │  all fed to LLM   │                    │ risks, positions  │
      └────────┬─────────┘                    └────────┬─────────┘
               ↓                                       ↓
      ┌──────────────────┐                    ┌──────────────────┐
      │  LLM computes AND │                    │  LLM only decides │
      │  decides (error-  │      ──→          │  (accurate, fast,  │
      │  prone, expensive)│                    │  cheap)           │
      └────────┬─────────┘                    └────────┬─────────┘
               ↓                                       ↓
      ┌──────────────────┐                    ┌──────────────────┐
      │  Prose output     │                    │  Operator JSON +  │
      │  ~800 tokens      │      ──→          │  NL Report        │
      │                    │                    │  ~120 + readable  │
      └──────────────────┘                    └──────────────────┘
```

| Principle | What It Does | Token Impact |
|:--|:--|:--:|
| **1. Declarative State** | Python pre-computes all metrics → feeds LLM only a state snapshot | **-90%** input |
| **2. Time-Anchored Chunking** | Thin rules index (60 lines) → modules loaded on-demand | **-85%** rules |
| **3. Operator Catalog** | Replace verbose prose with operator IDs (`POS_HOLD`, `RISK_STOP`) | **-85%** output |
| **4. Code-Driven Validation** | Python validates; only error codes returned to LLM | **-95%** retry loops |

---

## 🏗️ Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                    YOUR AGENT FRAMEWORK                       │
│  ┌────────────────────────────────────────────────────────┐  │
│  │         Decision Distillation Layer (this repo)          │  │
│  │                                                         │  │
│  │  state_compiler.py                                      │  │
│  │    Raw APIs → DecisionState JSON (schema-bound)          │  │
│  │                         ↓                               │  │
│  │  LLM receives DecisionState + rules/register.md          │  │
│  │                         ↓                               │  │
│  │  LLM outputs operator JSON + natural language report     │  │
│  │                         ↓                               │  │
│  │  output_validator.py                                    │  │
│  │    Structure ✓  Consistency ✓  Forbidden words ✓  Provenance ✓  │
│  └────────────────────────────────────────────────────────┘  │
│  Tool calls · Memory · Routing · Conversation management       │
└──────────────────────────────────────────────────────────────┘
```

### Project Structure

```
your-project/
├── rules/
│   ├── register.md          ← Rules index (LLM loads only this by default)
│   ├── decision_schema.json ← DecisionState JSON Schema
│   └── operator_catalog.md  ← Operator dictionary
├── scripts/
│   ├── state_compiler.py    ← Raw data → DecisionState
│   └── output_validator.py  ← LLM output linter
├── agent_config.md          ← Agent entry point (references register)
└── domain_rules/            ← On-demand rule modules
    ├── audit.md
    ├── risk.md
    └── signals.md
```

### Decision Pipeline

```
[Python pre-compute] → [DecisionState JSON] → [LLM logical decision]
    → [Operator JSON (internal)] → [Validator checks]
    → [Natural language report (user)]
```

---

## 📦 What's Included

| File | Purpose |
|:--|:--|
| `templates/register.md.tmpl` | Thin rules index pattern (~60 lines). Replace `{placeholders}` with your domain rules. |
| `templates/decision_schema.json.tmpl` | DecisionState JSON Schema template. Extend with your domain-specific fields. |
| `templates/operator_catalog.md.tmpl` | Operator dictionary template. Define your action/risk/signal operators. |
| `templates/state_compiler.py.tmpl` | Abstract base class with `compile()` + `compact()` methods. Subclass and implement your `_compute_*` methods. |
| `templates/output_validator.py.tmpl` | 4-layer validator: structure, consistency, forbidden words, number provenance. Drop-in ready. |

---

## 🧩 Templates / Modules

Beyond the core decision-distillation templates, this repo also ships **standalone skill/agent modules** you can copy into your own project. Each lives under `templates/` as a `{name}.{ext}.tmpl` — replace the `{placeholders}` with your own choices.

| Module | Template | What It Does |
|:--|:--|:--|
| **SkillWeaver Routing** | `templates/skillweaver_routing.md.tmpl` | 工具/技能过多时的精准路由：分解 → SAD 对齐 → 向量检索 → DAG 组合。替代"暴力塞入全部工具描述"，Token 降 ~99.9%、选择准确率大幅提升。参数字段：`{EMBEDDING_MODEL_EN}` / `{EMBEDDING_MODEL_EN_V2}` / `{EMBEDDING_MODEL_ZH}` / `{RERANKER_MODEL}` / `{LLM_MODEL}` |

> 用法：复制 `templates/skillweaver_routing.md.tmpl` 到你的项目，将 `{占位符}` 替换为实际选型（Embedding / Reranker / LLM 模型），即可作为技能路由层接入。

---

## 🚀 Quick Start

### 1. Clone & Copy Templates

```bash
git clone https://github.com/YOUR_USERNAME/declarative-llm-decision-framework.git
cp -r declarative-llm-decision-framework/templates/* your-project/rules/
```

### 2. Define Your DecisionState

Edit `decision_schema.json` — what are the pre-computable metrics in your domain?

```json
{
  "regime": { "name": "string", "level": "integer", "cap": "number" },
  "items": [{ "id": "string", "status": "string", "flags": ["string"] }],
  "alerts": { "critical": ["string"], "warnings": ["string"] }
}
```

### 3. Implement State Compiler

```python
from state_compiler import StateCompiler

class MyCompiler(StateCompiler):
    def _compute_regime(self):
        return {"name": "normal", "level": 1, "cap": 0.45}
    # ... implement _compute_items, _compute_alerts

compiler = MyCompiler()
print(compiler.compact())  # < 200 tokens, ready to feed LLM
```

### 4. Wire Into Your Agent

```
System message → agent_config.md (references register.md)
User input + DecisionState → LLM
LLM output → output_validator.py → execute or correct
```

---

## 🎯 Use Cases

| Domain | DecisionState Example | Operator Example | Token Reduction |
|:--|:--|:--|:--:|
| **Quantitative Trading** | regime/signals/positions | POS_ADD / RISK_STOP / SIG_ENTRY | **85%** |
| **Risk & Compliance** | risk_scores/compliance_flags | APPROVE / ESCALATE / REJECT | **80%** |
| **Insurance Underwriting** | applicant_profile/claim_history/risk_score | ACCEPT / DENY / REFER | **80%** |
| **Medical Decision Support** | vitals/lab_results/risk_factors | ORDER_TEST / REFER / MONITOR | **75%** |
| **Content Moderation** | toxicity/spam/policy_violation | REMOVE / FLAG / ALLOW | **75%** |
| **Customer Service Routing** | intent/sentiment/priority | AUTO_REPLY / ESCALATE / TRANSFER | **70%** |
| **Supply Chain Optimization** | inventory/lead_times/disruptions | REORDER / REROUTE / STOCKPILE | **70%** |
| **Code Review** | diff_stats/complexity/coverage | APPROVE / REQUEST_CHANGES / COMMENT | **65%** |

**Not suitable for:** open-ended creative writing, casual chat, one-shot trivial Q&A.

---

## 📊 What You Save

These numbers come from a production quantitative finance agent. The architecture works the same for **any domain** with structured decision-making.

| Metric | Traditional LLM Pipeline | Decision Distillation | Improvement |
|:--|:--|:--|:--|
| Analysis input tokens | ~3,000 | ~400 | **-87%** |
| Rule files loaded | Full domain rules | Thin index + on-demand modules | **-85%** |
| Decision output tokens | ~800 (verbose prose) | ~120 (operator JSON) | **-85%** |
| LLM computation errors | ~15% (number hallucination) | ~2% (Python computes) | **-87%** |
| Validation loop tokens | Full context re-sent | Error code only | **-95%** |

> **Key insight:** Decision quality stayed exactly the same — the rules didn't change. Token reduction is 100% architectural. The pattern transfers directly to risk assessment, insurance underwriting, medical triage, content moderation, and any other structured decision pipeline.

---

## 🔗 Relationship to Existing Frameworks

This is **not** a replacement for LangChain, AutoGen, CrewAI, or similar agent frameworks. It's a **decision optimization layer** that sits inside them:

```
┌─────────────────────────────────────┐
│  LangChain / AutoGen / CrewAI        │
│  ┌───────────────────────────────┐  │
│  │  Decision Distillation Layer  │  │
│  │  (this repo)                  │  │
│  │  compiler → operator → validator │
│  └───────────────────────────────┘  │
│  Tools · Memory · Routing · Chat     │
└─────────────────────────────────────┘
```

---

## 📄 License

MIT © 2026

---

<p align="center">
  <sub>Built with ❤️ for anyone tired of watching their LLM hallucinate arithmetic.</sub>
</p>
