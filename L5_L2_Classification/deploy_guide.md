# Deploy Guide — KANTOR_K5
## Prerequisites
- Python 3.11+, SQLite (stdlib), jsonld 1.3+

## Environment
- CPU-only. 512MB RAM. 1GB disk for full Anticloud KB.

## Install
```bash
pip install anticloud-kantor jsonld
```

## Initialize KB
```bash
python -m kantor_k5 init --output ./kantor_k5.db
python -m kantor_k5 import --source ./benchmark_results/ --type benchmarks
```

## Air-Gap
Single SQLite file — copy to target machine. No network required.

## AIOSS Integration
```bash
aioss init --module KANTOR_K5 --output ./kantor.aioss
```

## Verification
```bash
python -m kantor_k5 verify --db ./kantor_k5.db
```
