# SMAR v0.3 Foundation — Secure Modular Agent Runtime

SMAR means **Secure Modular Agent Runtime**.

This version lays the foundation for the corrected research direction:

1. **Secure Runtime Layer**  
   Handles untrusted payload validation, data-vs-authority separation, policy enforcement, memory permission checks, output control, and audit logging.

2. **Sparse Event Substrate**  
   Receives structured runtime events and converts them into sparse event-node activation telemetry.

3. **Neural-Dynamic Layer**  
   Adds a small dynamic-field style stabilization loop. This is inspired by neural-dynamic cognitive architecture ideas: state evolution, competition/cooperation, attractor-like risk states, and closed-loop behavior.

4. **Benchmark Layer**  
   Compares secure runtime behavior across benign, hostile, and subtle payloads.

## What SMAR v0.3 does not claim

```text
It does not claim consciousness.
It does not claim human-brain simulation.
It does not claim SNNs solve prompt injection.
It does not claim 100k neurons creates intelligence.
```

## Correct claim

```text
The secure runtime handles hostile input validation.
The sparse/event substrate records internal activation telemetry.
The neural-dynamic layer models risk-state stabilization and module response.
```

## Default sparse substrate

```text
100,000 event nodes
4 bytes per node activation/state
50 implicit neighbouring links per node
```

## Run the hostile-payload demo

```bash
python -m runtime.main --payload examples/hostile_payload.txt --policy config/scenario_policy.json
```

## Run benchmark

```bash
python benchmarks/run_benchmark.py
```

## Run tests

```bash
python -m unittest discover -s tests
```

## Core rule set

```text
Data can pass.
Authority must be earned.
Memory access must be revalidated.
Every layer can say no.
```


---

# SMAR v0.3.1 Evidence Layer

Adds:

```text
1. Expanded benchmark payload cases
2. False-positive / false-negative counters
3. Benchmark JSON / CSV / HTML exports
4. Local HTML dashboard
5. Report-freeze script
6. Audit-summary script
7. Faster sparse telemetry backend while preserving 100k 4-byte state array
```

Run complete evidence freeze:

```bash
python scripts/freeze_reports.py
```

Run benchmark only:

```bash
python benchmarks/run_benchmark.py
```

Open dashboard:

```text
reports/benchmark_dashboard.html
```

Summarize audits:

```bash
python scripts/audit_summary.py
```
