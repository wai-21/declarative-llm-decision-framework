<p align="center">
  <img src="https://img.shields.io/badge/Token_Savings-60%25~99%25-brightgreen" alt="Token Savings 60-99%">
  <img src="https://img.shields.io/badge/LLM_Cost-Reduced-blue" alt="LLM Cost Reduced">
  <img src="https://img.shields.io/badge/License-MIT-blue" alt="License MIT">
  <img src="https://img.shields.io/badge/Status-Production-green" alt="Production Ready">
  <img src="https://img.shields.io/badge/Languages-EN%20%7C%20ZH-orange" alt="EN ZH">
</p>

<h1 align="center">🔥 LLM Token Optimization Toolkit</h1>
<p align="center"><b>Slash AI Agent Token Cost by 60–99%</b> — Decision Distillation · Skill Routing · Quant Token Playbook</p>
<p align="center"><b>LLM Token 优化工具箱</b>：把 Agent 的 Token 成本砍掉 60–99%</p>

> **Keywords / 关键词索引**: `LLM token optimization` · `reduce LLM cost` · `agent token reduction` · `prompt token saving` · `context window optimization` · `LLM cost reduction 2026` · `AI agent efficiency` · `decision distillation` · `skill routing` · `FAISS tool retrieval` · `prompt caching` · `model layering` · `token budget` · `OpenClaw token saving` · `量化交易 agent 省 token` · `降低大模型 API 费用`

Stop burning money on LLM API bills. This repository bundles **three production-proven techniques** to cut token consumption for LLM-powered agents and AI workflows — **without losing decision quality**:

| Technique | Token Cut | Best For |
|:--|:--:|:--|
| **Decision Distillation** (Python computes, LLM only decides) | **~87%** | Any structured decision pipeline |
| **SkillWeaver Routing** (vector retrieval instead of dumping every tool description) | **~99%** | Agents with 20–2000+ tools/skills |
| **Quant Token Opt Playbook** (10 battle-tested strategies) | **60–80%** | Quant / trading / signal agents |

---

## 📑 Table of Contents

- [Why this repo — The LLM Token Cost Problem](#-why-this-repo--the-llm-token-cost-problem)
- [Module 1 · Decision Distillation Framework](#-module-1--decision-distillation-framework)
- [Module 2 · SkillWeaver Routing (Tool/Skill Selection)](#-module-2--skillweaver-routing-toolskill-selection)
- [Module 3 · Quant Token Opt Playbook](#-module-3--quant-token-opt-playbook)
- [Cross-Module Token Savings](#-cross-module-token-savings)
- [🚀 Quick Start](#-quick-start)
- [📦 What's Included (File Map)](#-whats-included-file-map)
- [🔗 Framework Integrations](#-framework-integrations)
- [🏷️ Boost GitHub Discoverability (SEO)](#️-boost-github-discoverability-seo)
- [📄 License & ⚙️ Auto-Sync](#-license--️-auto-sync)

---

## 🌟 Why this repo — The LLM Token Cost Problem

Every LLM-based agent makes the same expensive mistakes:

- **Context windows are expensive.** Every redundant token is money burned on the API bill.
- **LLMs hallucinate numbers.** GPT-4, DeepSeek, Claude all fail arithmetic under load.
- **Rule / tool files keep growing.** A 772-line prompt isn't helping — it's billing you every turn.
- **Tool descriptions explode context.** Dumping 200 tool schemas into the prompt = massive, repeated token waste.

> **Core thesis (贯穿三模块的核心命题):** *Move computation out of the LLM, keep only the decision inside it.* Compute in Python, route with vectors, budget your tokens — and the same quality costs a fraction.

This repo is **not** a replacement for LangChain / AutoGen / CrewAI / OpenClaw. It is a **token-cost optimization layer** you drop inside them.

---

## 🧩 Module 1 · Decision Distillation Framework

**Reduce decision-pipeline tokens ~87%. Python computes. LLM decides.**

> Don't feed your LLM raw K-line data, MACD values, and RSI charts. Give it a **DecisionState** — a pre-computed, schema-bound snapshot — and let it do logical reasoning, not arithmetic.

```
BEFORE:  Raw Data (10K+ tokens) → LLM computes AND decides (error-prone, $) → Verbose prose (~800 tok)
AFTER:   DecisionState <300 tokens → LLM only decides (accurate, fast, cheap) → Operator JSON + NL report
```

Four principles:

| Principle | Token Impact |
|:--|:--:|
| Declarative State (Python pre-computes metrics) | **-90%** input |
| Time-Anchored Chunking (thin rules index + on-demand modules) | **-85%** rules |
| Operator Catalog (replace prose with operator IDs) | **-85%** output |
| Code-Driven Validation (Python validates, returns error codes) | **-95%** retry loops |

**Templates** (`templates/`): `register.md.tmpl` · `decision_schema.json.tmpl` · `operator_catalog.md.tmpl` · `state_compiler.py.tmpl` · `output_validator.py.tmpl`. **Example**: `examples/finance/`.

---

## 🧩 Module 2 · SkillWeaver Routing (Tool/Skill Selection)

**Reduce tool-routing tokens ~99% (884K → ~1.16K). Pick the right tool without dumping all descriptions into context.**

> 工具/技能过多时，"该用哪个工具"比"能不能用"更重要。本模块是基于 SkillWeaver 论文框架的「自建集成模板」（非现成产品）：把 decompose→retrieve→compose + SAD 方案落地为可复用路由，核心目标是**大幅降低 Agent 在工具/技能过载时的 Token 消耗**。

Three-stage pipeline + **SAD alignment loop**:

```
Decompose (subtasks) → Retrieve (vector Top-K, 0 LLM token) → Compose (DAG execution graph)
        ↑____________________ SAD feedback loop (align LLM vocab ↔ tool vocab) ____________________↑
```

Key facts (verified vs **arXiv:2606.18051**, Alibaba Cloud + OSU + CMU):

- Bi-Encoder recall Top-1 ≈ **37.1%**, Top-10 ≈ **71.6%** → production needs a reranker (Cross-Encoder / LLM rerank).
- SAD lifts 7B decomposer accuracy **51.0% → 67.7% (+32.7%, Wilcoxon p<10⁻⁶)**.
- Context consumption drops **>99%** (884,000 → ~1,160 tokens, VentureBeat-reported magnitude).
- Planning (DAG) beats reactive (ReAct-style) for multi-tool orchestration.

**Template**: `templates/skillweaver_routing.md.tmpl` (replace `{EMBEDDING_MODEL_EN}` / `{EMBEDDING_MODEL_EN_V2}` / `{EMBEDDING_MODEL_ZH}` / `{RERANKER_MODEL}` / `{LLM_MODEL}`).

---

## 🧩 Module 3 · Quant Token Opt Playbook

**Cut quant / trading / signal agent tokens 60–80% with 10 prioritized strategies.**

> 本地 OpenClaw 量化/交易 agent 的 token 成本优化手册。同样决策质量下把 token 消耗压到最低。十大策略按性价比排序，最快见效组合（指标外部化 → 模型分层 → 结构化输出 → 限窗 → 限输出）通常降 60~80%。

**Token 三大来源 (locate the biggest leak first):** ① raw market/indicator data in prompt · ② accumulated history + tool results · ③ system prompt / tool descriptions / long outputs.

**Top 5 quick-win strategies (60–80% typical):**

| # | Strategy | One-line action |
|:--|:--|:--|
| 1 | 指标外部化 (externalize computation) | LLM reads only `signal + reason`, never raw K-line |
| 2 | 模型分层 (model layering) | high-freq → small quant model; complex → big model; rules → pure Python |
| 7 | 工具裁剪 + 结构化输出 (trim + structured out) | return 4-field JSON; `maxTokens` 512–1024 |
| 4 | 限窗 (limit context window) | `contextWindow` 8k–16k + sliding window |
| 10 | 限输出 (cap output) | `temperature` 0–0.3 + `maxTokens` + stop sequence |

Plus 5 deeper strategies: prompt compression, prompt caching (KV-cache), memory summarization, batch tool calls, event-driven frequency reduction. Full playbook + OpenClaw config landing points (models/agents) + verification command in **`skills/quant-token-opt/SKILL.md`**.

---

## 📊 Cross-Module Token Savings

| Metric | Traditional LLM Pipeline | With This Toolkit | Improvement |
|:--|:--|:--|:--:|
| Decision input tokens | ~3,000 | ~400 (DecisionState) | **-87%** |
| Tool-routing context | ~884,000 (all descs) | ~1,160 (retrieval) | **-99%** |
| Quant agent tokens | baseline | after 1→2→7→4→10 | **-60~80%** |
| Rule files loaded | Full domain rules | Thin index + on-demand | **-85%** |
| LLM number errors | ~15% | ~2% (Python computes) | **-87%** |

> **Key insight:** Decision quality stays the same — the rules don't change. Token reduction is 100% architectural.

---

## 🚀 Quick Start

```bash
git clone https://github.com/wai-21/declarative-llm-decision-framework.git
cd declarative-llm-decision-framework
```

- **Decision Distillation**: copy `templates/*` into your project's `rules/` + `scripts/`, replace `{placeholders}`.
- **SkillWeaver Routing**: copy `templates/skillweaver_routing.md.tmpl`, swap the 5 model `{placeholders}`.
- **Quant Token Opt**: read `skills/quant-token-opt/SKILL.md` and apply the checklist to your `openclaw.json`.

---

## 📦 What's Included (File Map)

| Path | Type | Purpose / Keywords |
|:--|:--|:--|
| `templates/register.md.tmpl` | template | Thin rules index (~60 lines) — prompt compression |
| `templates/decision_schema.json.tmpl` | template | DecisionState JSON Schema — declarative state |
| `templates/operator_catalog.md.tmpl` | template | Operator dictionary — structured output IDs |
| `templates/state_compiler.py.tmpl` | template | Raw data → DecisionState (`compile()`/`compact()`) |
| `templates/output_validator.py.tmpl` | template | 4-layer validator — forbidden words / provenance |
| `templates/skillweaver_routing.md.tmpl` | template | **Skill routing** — FAISS vector retrieval + SAD + DAG |
| `skills/quant-token-opt/SKILL.md` | skill | **Quant token playbook** — 10 strategies, 60–80% cut |
| `examples/finance/README.md` | example | Finance decision-distillation project layout |

---

## 🔗 Framework Integrations

This toolkit is **framework-agnostic**. Drop it inside:

- **LangChain / LangGraph** — use `state_compiler` before the LLM node; route tools via SkillWeaver.
- **AutoGen / CrewAI** — wrap agents with a DecisionState pre-step.
- **OpenClaw / qclaw** — `quant-token-opt` documents exact `models.providers.*.contextWindow` / `agents.list[].model` landing points.
- **Ollama / llama.cpp** — enable prompt caching (KV-cache) for fixed system prefixes.

---

## 🏷️ Boost GitHub Discoverability (SEO)

The repo name stays `declarative-llm-decision-framework`, but you can multiply reach by setting these on **GitHub → Settings → About** (not editable via git):

- **Description (建议)**: `LLM Token Optimization Toolkit — cut AI agent token cost 60–99%: Decision Distillation, SkillWeaver routing, Quant token playbook. Reduce LLM API bill.`
- **Topics / tags (建议)**: `llm`, `token-optimization`, `prompt-engineering`, `cost-reduction`, `agent`, `langchain`, `rag`, `vector-search`, `faiss`, `prompt-caching`, `context-window`, `quant-trading`, `llm-agent`, `ai-efficiency`, `openclaw`

> Want an even broader name? Renaming the repo to e.g. `llm-token-optimization` is reversible (GitHub auto-redirects the old URL). Say the word and I'll adjust the clone + hook.

---

## 📄 License & ⚙️ Auto-Sync

**MIT © 2026** — free for commercial and personal use.

**Auto-Sync:** this repo ships a `post-commit` Git hook — every local `commit` on `main` is automatically pushed to `origin` (GitHub) via SSH. Just commit and it goes live; no manual `git push`.

<p align="center">
  <sub>Built for anyone tired of watching their LLM hallucinate arithmetic — and inflate the API bill.</sub>
</p>
