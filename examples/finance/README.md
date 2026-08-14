# Example: Quantitative Finance Decision Pipeline — Decision Distillation in Action

> **Keywords**: quantitative finance · trading agent · LLM decision pipeline · token-efficient trading · regime model · risk screening · DecisionState · operator catalog

This directory shows the **expected layout** of the Decision Distillation Framework applied to **quantitative finance / algorithmic trading** — a real-world, token-efficient LLM decision pipeline.

## What You'd Find Here (desensitized project layout)

```
examples/finance/
├── rules/
│   ├── register.md          ← Thin rules index from production (loaded by default)
│   ├── decision_schema.json ← Finance-specific DecisionState schema
│   └── operator_catalog.md  ← Trading operator dictionary (POS_ADD / RISK_STOP / SIG_ENTRY …)
├── scripts/
│   ├── state_compiler.py    ← Market data → DecisionState (Python pre-computes indicators)
│   └── output_validator.py  ← 4-layer LLM output validator
└── demo_data.json           ← Synthetic market data for testing
```

## Why The Full Code Isn't Included

The production implementation contains **proprietary trading strategies** (G3 regime model, QS-7 quality screening, Loose safety belt) and **real position data** that cannot be open-sourced.

The templates in [`/templates/`](../templates/) give you everything needed to build your own — the **methodology is 100% transferable** and cuts token cost ~87%.

## Adapting to Your Domain

1. Replace `TICKER_A`, `TICKER_B` with your actual symbols.
2. Implement `_compute_regime()` based on your strategy's market regime model.
3. Define operators matching your decision space (`POS_HOLD` / `RISK_STOP` / `SIG_ENTRY`).
4. Configure validation rules for your domain's constraints.

See the main [`README.md`](../README.md) for the full guide, and [`skills/quant-token-opt/SKILL.md`](../../skills/quant-token-opt/SKILL.md) for 10 additional token-saving strategies for trading agents.
