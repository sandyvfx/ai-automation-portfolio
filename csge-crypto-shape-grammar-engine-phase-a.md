# CSGE — Crypto Shape Grammar Engine (Phase A)

Deterministic pipeline that converts historical OHLCV+Volume data into a symbolic market
grammar corpus. This repository implements **Phase A** only: CSV → canonical bars → geometry →
candidate shapes → global shape dictionary → per-asset corpus.

- Frozen design: [`ARCHITECTURE_FREEZE.md`](ARCHITECTURE_FREEZE.md)
- Implementation decisions (the §9 open items): [`DECISIONS.md`](DECISIONS.md)

Phase A ends at the Canonical Shape Dictionary and encoded Corpus. Grammar discovery,
analytics, visualization, and prediction are **Phase B** and are intentionally not built here.

## Pipeline (frozen module order)

```
CSV
 └─ M1 Loader ─ M2 Canonical ─ M3 Geometry ─ M4 Segmentation ─ M5 Candidate Descriptor
       (per asset)                                              └─► candidates/<asset>.parquet  [immutable]
    pool candidates ─ M6 Vocabulary Discovery (pluggable) ─ M7 Canonicalization
                                                              └─► dictionary/dictionary.parquet [global ids]
    (per asset) M8 Encode ─► corpus/<asset>.parquet  ─ M9 ─► corpus.duckdb (views over Parquet)
```

Two frozen architectural decisions shape this:
- **FB#2 (fit/encode split):** the pre-symbolic **Candidate Shape Objects** are materialized;
  discovery and encoding run over them. Discovery is pluggable behind one *Assignment Contract*.
- **FB#4 (hybrid scope):** one **global vocabulary** with **shared shape ids**, but
  **per-asset corpora** (and per-asset grammar in Phase B).

## Requirements

Python ≥ 3.14, `polars>=1.40`, `duckdb>=1.5`, `pyarrow>=24`. (All already present in this env.)

## Quick start

```bash
# generate synthetic sample data, run the full pipeline, and verify byte-level reproducibility
PYTHONHASHSEED=0 PYTHONPATH=src python -m csge.run --sample --out runs/demo --check-determinism

# run on your own CSVs (columns: timestamp, open, high, low, close, volume)
PYTHONHASHSEED=0 PYTHONPATH=src python -m csge.run \
    --input BTCUSD=data/btc.csv --input ETHUSD=data/eth.csv --out runs/live
```

Query the corpus (DuckDB is a query layer over Parquet — the corpus of record stays in Parquet):

```sql
SELECT * FROM corpus_summary;                       -- windows / distinct shapes / OOV per asset
SELECT * FROM shape_frequency WHERE asset='BTCUSD'; -- shape histogram
SELECT * FROM dictionary ORDER BY count DESC;        -- the global alphabet
SELECT * FROM lineage;                               -- content-addressed provenance chain
```

## Configuration

Defaults live in `PipelineConfig` (see `DECISIONS.md` for rationale). Common overrides:

| Flag | Meaning | Default |
|------|---------|---------|
| `--scales 16,32,64` | window sizes (multi-scale) | `32` |
| `--paa-segments N` | descriptor length | `8` |
| `--alphabet-size K` | SAX alphabet | `5` |
| `--stride-ratio R` | window stride = round(W·R) | `1.0` (non-overlapping) |
| `--strategy ID` | discovery strategy | `sax_v1` |
| `--epoch-unit U` | numeric timestamp unit | `ms` |

Any parameter change alters `config_hash`, which re-versions all downstream artifacts.

## Determinism

- Total ordering + stable sorts at every boundary; canonical rounding at the identity boundary.
- Content-addressed `dataset_id`s and sidecar manifests make artifacts logically write-once.
- `--check-determinism` rebuilds and asserts the corpus is byte-identical.
- Numeric output is guaranteed only within the library versions captured in each manifest.

## Layout

```
src/csge/
  contracts/     X0  frozen schemas / enums / order keys
  config/        X1  PipelineConfig + hashing
  lineage/       X2  manifests, dataset ids, determinism harness
  storage.py         Parquet I/O with schema validation + manifests
  pipeline/
    m1_loader.py  m2_canonical.py  m3_geometry.py  m4_segmentation.py
    m5_descriptor.py  descriptor_math.py
    m6_discovery/  base.py (Assignment Contract) · sax.py (default) · registry.py
    m7_canonicalize.py  m8_encode.py  m9_storage.py
  run.py             orchestrator / CLI
tests/               determinism, invariance, gap-policy, shared-id, e2e reproduction
```

## Extending discovery (the one sanctioned plug point)

Add a strategy by implementing `DiscoveryStrategy` + `Vocabulary`
(`src/csge/pipeline/m6_discovery/base.py`) and registering it in `registry.py`. Nothing else in
the pipeline changes — M5 candidates and M7/M8 are untouched. Clustering, canonical hashing,
graph matching, and symbolic induction all fit this interface.

## Status

Phase A is complete and runnable. Deliberately deferred, documented decisions (replaceable
behind stable interfaces) are in `DECISIONS.md`; the remaining architectural open items that do
not block Phase A are tracked in `ARCHITECTURE_FREEZE.md §9`.
