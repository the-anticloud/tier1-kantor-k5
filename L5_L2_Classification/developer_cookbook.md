# Developer Cookbook — KANTOR_K5
**Stack:** Python 3.11, SQLite, JSON-LD

## Query project facts
```python
from kantor_k5 import KantorKB
kb = KantorKB("./kantor_k5.db")
facts = kb.lookup("K_BRAINFLOW", fields=["trl_level", "domain", "regulatory"])
print(facts)  # {"trl_level": "TRL-8", "domain": "biosignals", ...}
```

## Assert a verified fact
```python
kb.assert_fact(subject="K_NANOVLLM", predicate="benchmark_throughput",
               value="97.3 tok/s on Tesla T4", source="kaggle_t4_2026", confidence=1.0)
```

## Query all projects in a tier
```python
for p in kb.query_tier("TIER_7_BIOSIGNALS_NEURO"):
    print(f"{p.name}: TRL={p.trl}, domain={p.domain}")
```

## Export JSON-LD for audit
```python
kb.export_jsonld("./kantor_export.jsonld",
                 context="https://0-1.gg/anticloud/context.jsonld")
```

## Performance
SQLite WAL mode for concurrent reads. `kb.cache_hot_facts(n=1000)` on startup.
Bulk inserts: `kb.batch_assert(facts_list)` — single transaction.

## Integration
Fed by: ANTICODE_AGENT (API facts), benchmark results. Consumed by: PAX_KNOWLEDGE_GRAPH (T2).
