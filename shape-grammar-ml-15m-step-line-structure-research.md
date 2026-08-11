# Shape Grammar ML — 15m Step-Line Structure Research Engine

Research question: **does the close-only 15m step-line have a repeatable shape grammar
beyond volatility clustering, or is the eye reading GARCH noise as language?**

This is a *shape-discovery* engine, not a trading system. No signals, no PnL.

## Core ideas (frozen framing)

1. **CSV-first.** The step-line is a deterministic function of the close column.
   Screenshots are optional human-inspection material only (`render.py` draws any
   window so eyes and numbers always look at the same data).
2. **Continuous tokenization, windowed detection.** The series is tokenized once,
   globally. Windows only define (a) the context horizon a detector may see and
   (b) normalization scope. Motif occurrences are *events on the timeline*, counted once.
3. **Right edge = "now".** Detection may never see past a window's right edge.
   Future bars are used only afterwards, to classify branch outcomes.
4. **Null class is mandatory.** Most bars should carry no motif. Wall-to-wall labels
   = pareidolia.
5. **Outcome-free names.** No motif may be named by what happened next
   ("top shelf", "trench") — outcomes are measured statistics, never labels.
6. **Motifs are outcome-branches of shared prefixes**, not independent vocabulary.
   The research object is P(branch | prefix), vs. surrogate baselines.

## Language hierarchy

- **Letters** (per 15m step, vol-normalized): `0` quiet · `u`/`d` medium · `U`/`D` large
- **Primitives** (runs of letters): `FLAT`, `DRIFT±`, `IMPULSE±`, `CHOP` (null)
  - plus a causal **state channel**: position-in-trailing-range, extent contraction
- **Structures**: `LADDER(dir)`, `RANGE(position, contraction)`, `IMPULSE`, `SPIKE_REVERSAL(dir)`
- **Branch grammar**: e.g. RANGE → up-IMPULSE → {shelf holds & extends | full retrace | unresolved}

## Pipeline

```
Binance 15m OHLCV CSV
  → close series → log step-line          (stepline.py)
  → causal vol unit, anchored shapes      (normalize.py)
  → letters / primitives / state channel  (tokenize.py)
  → structures & breakout events          (structures.py)
  → branch outcome measurement            (test001.py)
  → compare vs surrogate ladder           (surrogates.py)
```

## Anti-pareidolia protocol (non-negotiable)

- No future leakage anywhere: vol units, extremes, tokens are all causal (shift-1).
- Parameters are fitted on the TRAIN era and **frozen** before touching TEST years
  (see `PREREGISTRATION.md`). Test years are never opened during development.
- Fuzzy-match buffers are calibrated by **false-match rate on surrogates**
  (e.g. radius set so 2% of surrogate windows match), not by eye.
- Vacuousness alarms: a motif matching >10–15% of real windows, or with
  matched-vs-unmatched outcomes indistinguishable, is rejected.
- Every statistic must climb the **surrogate ladder** (weakest → strongest null):
  1. iid-shuffled returns
  2. IAAFT (preserves marginal + spectrum)
  3. block-shuffled returns (preserves vol clustering)
  4. fitted GARCH-t simulation
  5. cross-symbol / time-shifted real data
  Where a statistic *dies* on the ladder tells you what it really was.
- Multiple-comparison correction across motifs × scales × directions.
- Failed lookalikes are included by construction: every prefix match is followed
  to completion-or-invalidation; nothing is discarded.

## Layout

```
config.yaml            all parameters (single source of truth)
PREREGISTRATION.md     frozen spec for Test 001 — fill in BEFORE running test era
src/shape_grammar/
  download.py          Binance klines → CSV (resumable)
  stepline.py          load closes, log step-line
  normalize.py         causal vol unit, step units, anchored window shapes
  tokenize.py          letters, primitive segments, state channel
  structures.py        RANGE detection, upside/downside breakout events
  surrogates.py        iid / block / IAAFT / GARCH surrogate generators
  test001.py           branch classification A/B/C + real-vs-surrogate table
  render.py            draw any window as a step-line PNG (human inspection)
scripts/
  smoke_test.py        end-to-end run on synthetic data (no network needed)
data/
  raw/                 downloaded CSVs (gitignored)
  sample/              small samples, rendered PNGs
  results/             event tables, branch statistics
```

## Quick start

```bash
pip install -r requirements.txt

# end-to-end sanity check on synthetic data (no network)
python scripts/smoke_test.py

# download real data (resumable; ~175k rows per symbol for 5y of 15m)
python -m src.shape_grammar.download --symbol ETHUSDT --start 2020-08-01 --out data/raw

# run Test 001 on real data vs surrogates
python -m src.shape_grammar.test001 --csv data/raw/ETHUSDT_15m.csv --out data/results
```

## Roadmap

1. ✅ Foundation: pipeline skeleton, Test 001, surrogate ladder, renderer.
2. Fit letter thresholds / range-extent thresholds as train-era percentiles; freeze in config.
3. Fill and freeze `PREREGISTRATION.md`; run Test 001 on train era + full surrogate ladder.
4. Add LADDER / SPIKE_REVERSAL detectors; motif transition matrix vs surrogates.
5. Right-edge prefix statistics with volatility/drift-matched real controls
   (the "momentum in costume" check).
6. Only if counting shows non-random structure: sequence models (HMM over tokens).
