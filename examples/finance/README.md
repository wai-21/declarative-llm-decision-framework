# Example: Quantitative Finance Decision Pipeline

This directory would contain a **desensitized** example of the Decision Distillation Framework applied to quantitative finance.

## What You'd Find Here

```
examples/finance/
├── rules/
│   ├── register.md          ← Actual rules index from production
│   ├── decision_schema.json ← Domain-specific schema
│   └── operator_catalog.md  ← Finance operator dictionary
├── scripts/
│   ├── state_compiler.py    ← Market data → DecisionState (desensitized)
│   └── output_validator.py  ← Validator config (desensitized)
└── demo_data.json           ← Synthetic market data for testing
```

## Why This Isn't Included

The production implementation contains **proprietary trading strategies** (G3 regime model, QS-7 quality screening, Loose safety belt) and **real position data** that cannot be open-sourced.

The templates in `/templates/` give you everything you need to build your own — the methodology is 100% transferable.

## Adapting to Your Domain

1. Replace `TICKER_A`, `TICKER_B` with your actual symbols
2. Implement `_compute_regime()` based on your strategy's market regime model
3. Define operators matching your decision space
4. Configure validation rules for your domain's constraints

See the main [README.md](../README.md) for the full guide.
