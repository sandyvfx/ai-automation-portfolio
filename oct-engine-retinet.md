\# OCT Engine / RetiNet: Medical-Grade AI Triage \& Model Governance



\## 🎯 Project Overview

RetiNet is a privacy-first, review-support workflow designed for OCT/macula case intake and AI triage. Rather than just training a model, this project focuses on the \*\*architecture of safe AI deployment\*\*: defining Bring-Your-Own-Model (BYOM) integration contracts, maintaining model evidence registries, and enforcing strict clinical safety boundaries.



\## 🏗️ System Architecture

Designed as a technical handoff layer for backend AI teams and medical officers:

1\. \*\*BYOM Integration Contract:\*\* Defines strict request/response schemas, uncertainty thresholds, safety flags, and review-support boundaries, allowing backend models to be plugged in without disrupting the clinical workflow.

2\. \*\*Model Evidence Registry:\*\* Tracks model versions, validation summaries, metrics, and artifact lineage (e.g., `last\_model.pt`) to ensure full reproducibility and auditability.

3\. \*\*Privacy/PHI Boundary:\*\* The public demo explicitly exposes no Protected Health Information (PHI) or raw vendor files, enforcing a strict separation between the static POC and production-grade secure storage/RBAC requirements.



\## 📊 Evidence Snapshot \& Metrics

\- \*\*Kermany OCT Dataset:\*\* EfficientNet-B0 architecture trained on 83,484 images.

\- \*\*Holdout Validation:\*\* Achieved \*\*966/968 correct classifications\*\*, yielding \*\*99.793% Test Accuracy\*\* and \*\*99.793% Macro F1\*\* on the strictly held-out 968-image test set.

\- \*\*ORIGA Glaucoma Metadata:\*\* Evaluated CDR/ExpCDR triage-support signals, achieving a mean \*\*AUC of 0.8253\*\* (fundus optic-nerve metadata only).



\## 🧠 Engineering Takeaways

This project demonstrates my ability to:

\- Design \*\*safe, governed AI systems\*\* with explicit scope limitations (e.g., "doctor review required," "not a regulated medical device").

\- Architect \*\*model evaluation frameworks\*\* that track lineage, versioning, and validation summaries.

\- Build \*\*integration contracts\*\* that decouple the frontend clinical workflow from the backend inference engine.

