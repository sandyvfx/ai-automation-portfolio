# 🚀 AI-Augmented Systems Architecture Portfolio

Welcome to my independent R&D portfolio. Over an intensive 9-month sprint, I architected, designed, and validated 25+ production-grade systems spanning workflow automation, AI security, data engineering, and model governance.

## 🧠 My Engineering Philosophy
I am an **AI-Augmented Systems Architect**. I do not write boilerplate syntax manually. Instead, I:
1. **Architect the Logic:** I design the state machines, data flows, edge-case handling, and system boundaries.
2. **Orchestrate the AI:** I break down complex specs into actionable prompts, using AI (LLMs) as a force multiplier to generate the syntax rapidly.
3. **Validate the Output:** I build rigorous, rubric-based testing frameworks (like deterministic replay and hash-chain ledgers) to ensure the AI-generated code actually works in production.

This portfolio proves that in the modern tech landscape, **system design and validation are the true engineering skills**, not manual typing.

---
## 🏆 Highlighted Projects

### 1. 📈 ECG Market Terminal v1.0 (Real-Time Systems)
A production-grade desktop app rendering live Binance Futures data as a real-time ECG waveform. Broke conventional rules by implementing a **lock-free, dual-threaded architecture** (Asyncio WebSocket ingest ↔ 60 FPS PySide6 rendering) connected via a `SimpleQueue` event bus. Features 3-phase no-repaint optimization, 1×–100× deterministic replay, and crash recovery via `sys.excepthook`.
👉 **[Read the Documentation](./ecg-market-terminal-v10.md)**

### 2. 🤖 Resilient AI Lead Router (n8n Automation)
A production-grade B2B automation engine that processes messy webhook payloads, normalizes them via custom JavaScript, scores them using an LLM (Gemini), and routes them dynamically to a database. Features a secondary validation layer to catch AI hallucinations and apply deterministic fallback logic.
👉 **[Read the Documentation](./resilient-ai-lead-router_N8N.md)**

### 3. 🏥 OCT Engine / RetiNet (AI Model Governance)
A privacy-first, review-support workflow for OCT/macula case intake. Focuses on safe AI deployment via Bring-Your-Own-Model (BYOM) integration contracts, model evidence registries, and strict clinical safety boundaries. Achieved 99.79% accuracy on a held-out medical dataset.
👉 **[Read the Documentation](./oct-engine-retinet.md)**

### 4. ️ SMAR v0.3 (Secure Modular Agent Runtime)
A 4-layer security runtime designed to evaluate and restrict autonomous AI agents. Enforces a strict "Data can pass, Authority must be earned" policy layer and features a dedicated benchmarking harness to test system behavior against hostile prompt-injection payloads.
 **[Read the Documentation](./smar-v03-foundation-secure-modular-agent-runtime.md)**

### 5. 📊 CSGE (Crypto Shape Grammar Engine)
A highly deterministic, reproducible data pipeline for financial microstructure research. Built a 9-module ETL pipeline using Polars and DuckDB, featuring content-addressed lineage and a `--check-determinism` harness that asserts byte-level reproducibility across runs.
 **[Read the Documentation](./csge-crypto-shape-grammar-engine-phase-a.md)**

---

## 🛠️ Technical Arsenal
*   **Workflow Orchestration:** n8n, Make.com, Webhooks, REST APIs, JSON Schema Validation, Exponential Backoff Retries.
*   **AI Evaluation & RLHF:** Rubric-Based QA, Prompt Engineering, Model Benchmarking, BYOM Integration Contracts, LLM-as-a-Judge.
*   **Data Engineering:** Deterministic ETL, Polars, DuckDB, Parquet, Content-Addressed Lineage.
*   **System Architecture:** Asyncio, WebSocket Microstructure, Event-Driven Design, Edge AI.

---

##  Contact & Connect
*   Email: SANDEEP.MARNI@OUTLOOK.COM
*   Phone: +91 9493136773
  

*This portfolio was built using AI-augmented workflows. The architecture, logic, and validation are 100% mine; the syntax was accelerated by AI.*
