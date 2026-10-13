# sqltok

**Your schema, minus the tokens you cannot afford.**

![sqltok](docs/hero.webp)

[![CI](https://github.com/Zion-sewers/sqltok/actions/workflows/ci.yml/badge.svg)](https://github.com/Zion-sewers/sqltok/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

Text-to-SQL agents choke on big schemas. Dumping every table into the prompt burns tokens and buries the signal. sqltok is a schema token budget manager: given a database and a question, it picks the tables and columns that matter inside a hard token budget (measured with tiktoken) and returns a compact, joinable schema string for your prompt.

Under the hood: keyword/BM25 value grounding, MinHash + banded LSH over table/column names and sampled cell values, greedy submodular coverage selection, FK-neighbor expansion, and FK-Steiner-tree bridge tables so the result stays joinable.

On BIRD mini-dev (500 questions) at a 2,000-token budget: **97.4% full-recall** (all gold-query tables kept), 99.0% table recall, with **17.2% fewer total prompt tokens** than the full-schema-dump baseline. Numbers and reproduce commands in [benchmarks/RESULTS.md](benchmarks/RESULTS.md), caveats included.

## 60 seconds

```bash
git clone https://github.com/Zion-sewers/sqltok && cd sqltok
pip install .
python - <<'EOF'
from sqltok import SchemaBudgetManager
mgr = SchemaBudgetManager.from_sqlite("benchmarks/sample_data/dev_databases/retail/retail.sqlite")
ctx = mgr.build_context(question="total order amount for customers in France", token_budget=2000)
print(ctx.text, ctx.tables, ctx.token_count)
EOF
```

A Streamlit demo lives at `demo/app.py`.

## What it does

- Hard token budgets enforced with tiktoken, not vibes.
- Value grounding: MinHash/LSH over names and sampled cell values finds the columns the question actually touches.
- Joinability preserved: FK-Steiner bridges keep the selected tables connected.
- Typed Python API (`py.typed`).

## Honest limits

- Schema-linking only. It does not write SQL and does not call an LLM.
- Evaluation is BIRD mini-dev: small schemas. The headline savings do not transfer 1:1 to large enterprise DBs (the README's own RESULTS.md says this).
- Precision ~40%: spare budget goes to likely join targets via FK-neighbor expansion.
- sqlite entry point documented. Not on PyPI yet; install from source.

---

Part of [Zion](https://github.com/Zion-sewers) — public infrastructure for trust in agents. The eval layer for the long-horizon era.
