# ArduPilot Log Diagnosis — v2.0 Platform Roadmap

**Author:** Agastya Pandey (BeastAyyG)
**Status:** Active — v1.0 complete, v2.0 in progress
**Last Updated:** May 2026

---

## Vision

The v1.0 engine (XGBoost + IsolationForest + CITA + 3D replay) is **production-ready and fully tested**
(176 passing tests, 1.00 Macro F1). The v2.0 goal is to evolve this into the **definitive,
open-source ArduPilot diagnostic platform** — containerized, explainable, modular, and capable
of multi-flight trend analysis.

The core design philosophy does not change:
- **Physics and ML make the decisions.** LLMs only explain and orchestrate.
- **Rules + causality before statistics.** The CITA policy is never bypassed.
- **Honest metrics only.** No inflation, no stale numbers.

---

## v2.0 Target Architecture

```
ardupilot-diagnosis-platform/
├── docker-compose.yml
├── core-engine/          ← XGBoost + IsolationForest + CITA (current engine)
├── temporal-layer/       ← HMM + Kalman filter for noise filtering & sequence detection
├── causal-arbitrator/    ← Enhanced CITA + modular rule engine
├── llm-orchestrator/     ← LangGraph + local/API LLM (explanation, chat, workflow only)
├── feature-store/        ← Shared feature engineering (94+ features, Parquet, DuckDB)
├── report-service/       ← PDF/HTML/JSON generation + rich visuals
├── tuning-advisor/       ← PID ratings, vibration attribution, filter suggestions
├── multi-flight-analyzer/← Trend & degradation analysis across multiple logs
├── web-gateway/          ← FastAPI + modern frontend (unified UI)
└── data-pipeline/        ← Log ingestion, Parquet storage, DuckDB queries
```

**Why containerized microservices:**
- Any service can fail independently without killing the whole system.
- Each component can be upgraded (or replaced with a better model) without risk.
- Easy to open-source individual services separately if needed.
- LLM and ML concerns remain physically separated in code and runtime.

---

## Milestone 0 — Foundation & Containerization

**Goal:** Turn the existing monorepo into a clean, containerized platform.

**Status:** 🟩 Complete

### Tasks

- [x] Write `docker-compose.yml` that starts all services.
- [x] Create one folder per service, each with its own `Dockerfile`.
- [x] Port the current `core-engine` into its own container.
- [x] Create `feature-store` service that computes and caches features from `.BIN` files.
- [x] Set up `data-pipeline` service with Parquet storage + DuckDB querying.
- [x] Create `web-gateway` FastAPI service that routes requests to other services.
- [x] Update `.github/workflows/ci.yml` to build and test each service independently.

### Deliverable

`docker compose up` starts the full platform. The current v1.0 `analyze` endpoint works
through the gateway and returns the same results as before.

### Done when

- [x] Fresh clone + `docker compose up` works with no manual steps.
- [x] All 176 existing tests still pass inside the `core-engine` container.

---

### Deliverable

`temporal-layer` container available. `core-engine` can optionally call it before running XGBoost.
Benchmark shows improved precision on logs with known transient noise.

### Done when

- [ ] Temporal filter reduces false positives on at least 3 known noisy test logs.
- [ ] HMM training script is documented and reproducible.

---

## Milestone 2 — LLM Orchestration Layer

**Goal:** Add a natural language explanation and chat layer on top of the diagnostic engine.

**Status:** ⬜ Not started

### Philosophy

**The LLM never makes diagnostic decisions.** It only:
1. Converts structured diagnosis output into human-readable reports.
2. Answers user questions like "Why did EKF spike at 47s?" using the diagnosis context.
3. Orchestrates multi-step analysis workflows (e.g., "analyze, then generate PDF, then summarize").
4. Generates hypotheses for human review — clearly labelled as unverified.

This is explicitly **not** LLM-based fault detection. Physics and ML stay in control.

### Tasks

- [ ] Create `llm-orchestrator/` service using LangGraph for structured workflow management.
- [ ] Support two LLM backends: local (Ollama) and API (Groq / OpenAI).
- [ ] Build prompt templates that inject structured diagnosis JSON and ask for explanation only.
- [ ] Expose a `/explain` endpoint: takes a `DiagnosisResult`, returns natural language report.
- [ ] Expose a `/chat` endpoint: stateful Q&A about the current diagnosis.
- [ ] Add a "hypothesis mode" that generates alternative explanations — clearly marked as LLM-generated.
- [ ] Integrate with `report-service` to produce PDF/HTML reports with both technical + narrative sections.

### Deliverable

A chat interface in the web UI where users can upload a `.BIN` file, get a diagnosis,
then ask follow-up questions in plain English.

### Done when

- [ ] `/explain` endpoint produces a correct, grounded explanation for `vibration_high` on `sample.bin`.
- [ ] LLM output never contradicts the structured diagnosis from `core-engine`.
- [ ] Local (Ollama) path works without any external API keys.

---

## Milestone 3 — Unified Diagnostic Engine

**Goal:** Merge the core engine + temporal layer + causal arbitrator into a single, coherent pipeline.

**Status:** ⬜ Not started

### Tasks

**Rule Engine Refactoring (from open issues):**
- [ ] Break `src/diagnosis/rule_engine.py` into `src/diagnosis/rules/` modules:
  - `vibration.py`, `compass.py`, `power.py`, `gps.py`, `motors.py`
  - `ekf.py`, `mechanical_failure.py`, `pid_tuning.py`
  - `rc_failsafe.py`, `thrust_loss.py`, `brownout.py`, `crash_unknown.py`
- [ ] Add `tests/test_diagnosis_rules.py` with threshold boundary tests for every rule.

**Dead Label Remediation:**
- [ ] Add ML + rule coverage for: `power_instability`, `pid_tuning_issue`, `motor_imbalance`,
  `thrust_loss`, `gps_glitch`, `battery_failsafe`, `rc_failsafe`, `brownout`.
- [ ] Verify and fix `check_compass` rule — reduce reliance on ML fallback.

**Scaler Alignment:**
- [ ] Align the IsolationForest "healthy-only" scaler with the XGBoost "full-dataset" scaler.
  Document the decision or unify them.

**ML Artifacts:**
- [ ] Write `models/manifest.json` with: model version, feature schema hash, label schema hash,
  training dataset id, calibration date, threshold config hash.
- [ ] Add missing-artifact, schema-mismatch, and corrupted-model fallback tests.

### Deliverable

Single `/analyze` endpoint returns richer results with temporal smoothing, modular rule output,
and full 14-label coverage. All results traceable to physics evidence.

### Done when

- [ ] All 14 `VALID_LABELS` have at least one rule or ML path that can trigger them.
- [ ] No rule change requires editing a file longer than 200 lines.

---

## Milestone 4 — Advanced Capabilities

**Goal:** Build the features that make this platform irreplaceable.

**Status:** ⬜ Not started

### Tasks

**Multi-Flight Trend Analysis:**
- [ ] Create `multi-flight-analyzer/` service.
- [ ] Accept multiple `.BIN` files or a folder; detect degradation trends across flights.
- [ ] Output: trend plots (vibration over 10 flights, battery health curve, motor current drift).

**Tuning Advisor:**
- [ ] Create `tuning-advisor/` service.
- [ ] Rate PID parameters against the BASiC dataset baseline.
- [ ] Attribute vibration sources (motor, prop, frame resonance).
- [ ] Suggest notch filter center frequencies from FFT peaks.

**Report Service:**
- [ ] Create `report-service/` service.
- [ ] Generate PDF reports with: diagnosis summary, causal timeline, 3D trajectory, evidence panels.
- [ ] Generate structured JSON for external consumption (LLM agents, WebTools integration).

**CLI Refactoring (from open issues):**
- [ ] Break `src/cli/main.py` into `src/cli/commands/` modules:
  - `analyze.py`, `features.py`, `benchmark.py`, `batch_analyze.py`, `demo.py`
- [ ] `main.py` becomes a thin dispatcher only.

**Bad Input Handling (from open issues):**
- [ ] All batch, benchmark, and API endpoints handle empty/corrupt/partial `.BIN` files explicitly.
- [ ] Exit codes and error messages are consistent across all paths.

### Deliverable

Full platform — analyze, trend, tune, explain, export — all working end-to-end.

### Done when

- [ ] A user can upload 10 flights, get a degradation trend report, and receive a tuning recommendation.

---

## Milestone 5 — Production Hardening & v2.0 Release

**Goal:** Make the v2.0 platform release-ready for public use.

**Status:** ⬜ Not started

### Tasks

- [ ] Comprehensive test coverage: unit + integration + real crash log regression tests.
- [ ] Performance optimization: keep `/analyze` under 500ms on a standard log.
- [ ] All Docker images published to GitHub Container Registry.
- [ ] `docker compose up` documented as the one-line setup path.
- [ ] Updated model card, architecture doc, output formats doc.
- [ ] CHANGELOG updated to v2.0.
- [ ] README updated to reflect v2.0 capabilities.
- [ ] Release tag `v2.0.0` created on GitHub.

### Deliverable

Public v2.0 release. Anyone can run `docker compose up` and get a working instance.

### Done when

- [ ] All release gates from v1.0 still pass.
- [ ] Docker image size is reasonable (< 2GB).
- [ ] `README.md` accurately describes the v2.0 platform with no contradictions.

---

## Milestone 6 — Community Integration & Upstream Adoption

**Goal:** Get this adopted as a recognized ArduPilot community tool.

**Status:** ⬜ Not started

### Tasks

- [ ] Write a technical blog series covering: architecture decisions, CITA policy, temporal layer,
  LLM-as-orchestrator philosophy.
- [ ] Post the v2.0 release to the ArduPilot Discuss forum with a demo video.
- [ ] Investigate integration with MAVProxy and ArduPilot WebTools.
- [ ] Create a contribution guide for adding new rules, labels, and crash log datasets.
- [ ] Publish Docker images to Docker Hub for easy discoverability.
- [ ] Submit as a candidate for official ArduPilot tooling.

### Deliverable

ArduPilot community recognizes this as the best open-source log diagnosis platform.

---

## Current v1.0 Status (Baseline)

Before starting v2.0, the baseline is solid:

| Component | Status |
|---|---|
| XGBoost + IsolationForest classifier | ✅ Production-ready |
| CITA temporal arbitration | ✅ Production-ready |
| 3D interactive flight replay | ✅ Working |
| Pre-flight parameter validation | ✅ Working |
| FastAPI web endpoint | ✅ Working |
| CLI | ✅ Working |
| Test suite | ✅ 176/176 passing |
| Macro F1 score | ✅ 1.00 on holdout set |
| Rule coverage | ⚠️ 6 of 14 labels |
| Compass rule | ⚠️ Relies heavily on ML fallback |
| Docker / containerization | ❌ Not yet |
| LLM explanation layer | ❌ Not yet |
| Temporal HMM layer | ❌ Not yet |
| Multi-flight analysis | ❌ Not yet |
| Tuning advisor | ❌ Not yet |

---

## Recommended Execution Order

1. **Milestone 0** — Containerize first. Safety net before touching anything.
2. **Milestone 1** — Temporal layer. Fastest improvement to diagnostic quality.
3. **Milestone 3 (Rule Engine only)** — Clean up the rule engine in parallel.
4. **Milestone 2** — LLM layer. High user-facing value, low risk (rules still decide).
5. **Milestone 3 (Dead Labels + Scalers)** — Complete after LLM layer is working.
6. **Milestone 4** — Advanced features once the core is solid.
7. **Milestone 5** — Harden and release.
8. **Milestone 6** — Community adoption.

---

## One-Line Rule

Do not add flashy new features on top of a broken foundation.
Earn the reputation by making what exists clean, reproducible, honest, and hard to break.
Then build upward from a position of strength.
