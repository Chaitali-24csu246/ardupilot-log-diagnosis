# 🎯 BeastAyyG Enhancement Roadmap

## Overview
This document tracks the systematic completion of high-value features to make BeastAyyG the "one tool to rule them all" for ArduPilot log analysis.

**Current Strengths:**
- ✅ Hybrid ML (XGBoost + IsolationForest) + calibrated confidence
- ✅ CITA causal timing (biggest differentiator)
- ✅ 94 engineered features + real crash dataset
- ✅ Full pipeline (parse → diagnose → report + 3D)
- ✅ CLI + FastAPI + JSON/HTML output
- ✅ Low latency (<350ms) + strong benchmarks (176 tests passing)

---

## Phase 1: Quick Wins (1–2 weeks) – Highest ROI

### Goal 1: Multi-flight Comparison Mode
**Priority:** HIGH  
**Effort:** Medium  
**Status:** ⬜ Not Started

**Requirements:**
- [ ] New CLI command: `analyze --compare flight1.bin flight2.bin flight3.bin`
- [ ] Dashboard tab: "Trend Analysis" with line charts for key metrics
- [ ] Track degradation % for: vibration RMS, EKF variance, battery sag, etc.
- [ ] Output actionable insights: "Vibration on Motor 3 increased 47% since last flight"
- [ ] Store analysis results in JSON cache for instant comparison

**Implementation Plan:**
1. Create `src/comparison/` module
2. Add `compare.py` with trend analysis logic
3. Extend CLI with `--compare` flag
4. Add SQLite/JSON caching layer
5. Update dashboard with trend visualization

**Files to Create/Modify:**
- `src/comparison/__init__.py`
- `src/comparison/trend_analyzer.py`
- `src/cli/commands/compare.py`
- `src/web/app.py` (add /api/compare endpoint)
- Update dashboard HTML with trend tab

---

### Goal 2: Extended Failure Classes
**Priority:** HIGH  
**Effort:** Low-Medium  
**Status:** ⬜ Not Started

**Requirements:**
- [ ] Add new failure types to XGBoost classifier:
  - `motor_imbalance` (already defined, needs training data)
  - `thrust_loss` (already defined, needs training data)
  - `power_issue` (new - combine power_instability + brownout)
  - `gps_glitch` (new - transient GPS failures)
- [ ] Mine more labeled examples from discuss.ardupilot.org
- [ ] Update feature engineering to support new classes
- [ ] Retrain model with expanded labels

**Implementation Plan:**
1. Review existing failure_types.py (already has most types)
2. Add detection rules for new failure classes
3. Expand training dataset with new labels
4. Retrain XGBoost model
5. Update benchmark tests

**Files to Modify:**
- `src/diagnosis/failure_types.py` (add POWER_ISSUE, GPS_GLITCH)
- `src/diagnosis/rule_engine.py` (add detection rules)
- `training/mine_expert_labeled_logs.py` (expand queries)
- `models/known_failures.json` (update label mapping)

---

### Goal 3: Better Report Exports
**Priority:** HIGH  
**Effort:** Low  
**Status:** ⬜ Not Started

**Requirements:**
- [ ] Add `--format pdf` export using ReportLab or WeasyPrint
- [ ] Include subsystem radar + timeline + trend charts
- [ ] Improve HTML report styling (Tailwind or professional CSS)
- [ ] Make reports shareable and professional

**Implementation Plan:**
1. Add ReportLab/WeasyPrint to requirements
2. Create `src/tools/pdf_exporter.py`
3. Enhance HTML template with better CSS
4. Add PDF generation to CLI formatter

**Files to Create/Modify:**
- `src/tools/pdf_exporter.py`
- `src/cli/formatter.py` (add PDF format option)
- `pyproject.toml` (add reportlab dependency)
- `src/web/index.html` (improve styling)

---

## Phase 2: Differentiators (2–4 weeks)

### Goal 4: Local AI Chat Assistant
**Priority:** HIGH  
**Effort:** Medium-High  
**Status:** ⬜ Not Started

**Requirements:**
- [ ] Add conversational AI chat over log analysis
- [ ] Support questions like: "Is motor 3 vibration normal?" or "Why did EKF spike at 47s?"
- [ ] Start with rule-based + structured JSON (lightweight)
- [ ] Optional: Integrate local LLM (Ollama + Llama-3.1-8B) later
- [ ] Add endpoint: `POST /api/chat`

**Implementation Plan:**
1. Create `src/chat/` module
2. Implement rule-based Q&A engine first
3. Add keyword matching + confidence score lookup
4. Create `/api/chat` endpoint in FastAPI
5. (Optional) Add LLM integration with RAG over analysis JSON

**Files to Create:**
- `src/chat/__init__.py`
- `src/chat/assistant.py` (rule-based engine)
- `src/chat/llm_integration.py` (optional LLM wrapper)
- Update `src/web/app.py` with chat endpoint

---

### Goal 5: Tuning Advisor Module
**Priority:** HIGH  
**Effort:** Medium  
**Status:** ⬜ Not Started

**Requirements:**
- [ ] PID step-response analyzer (rise-time/overshoot)
- [ ] FFT vibration source attribution (prop vs motor vs frame)
- [ ] Basic magfit + filter suggestions
- [ ] Output as "Tuning Recommendations" section with confidence scores
- [ ] Make it toggleable (`--tuning` flag)

**Implementation Plan:**
1. Create `src/tuning/` module
2. Implement PID analyzer
3. Add FFT vibration source detection
4. Create magfit recommendation engine
5. Integrate into diagnosis pipeline as optional module

**Files to Create:**
- `src/tuning/__init__.py`
- `src/tuning/pid_analyzer.py`
- `src/tuning/vibration_fft.py`
- `src/tuning/recommendations.py`
- Update `src/cli/commands/analyze.py` with `--tuning` flag

---

## Phase 3: Polish & Scope (ongoing)

### Goal 6: Agent/LLM-friendly Structured Output
**Priority:** MEDIUM  
**Effort:** Low  
**Status:** ⬜ Not Started

**Requirements:**
- [ ] Improve JSON schema with clear explanations
- [ ] Add field descriptions for AI copilots
- [ ] Make output future-proof for AI agents
- [ ] Document JSON schema

**Implementation Plan:**
1. Review current JSON output schema
2. Add metadata + field descriptions
3. Create JSON schema documentation
4. Update Pydantic models

**Files to Modify:**
- `src/web/schemas.py` (enhance Pydantic models)
- `docs/OUTPUT_FORMATS.md` (add schema docs)

---

### Goal 7: Improved 3D Replay
**Priority:** MEDIUM  
**Effort:** Medium  
**Status:** ⬜ Not Started

**Requirements:**
- [ ] Add simple drone model or better camera controls
- [ ] Enhance Plotly 3D visualization
- [ ] Add playback controls
- [ ] Show causality markers at exact GPS coordinates

**Implementation Plan:**
1. Review current 3D replay implementation
2. Add drone mesh/model (simple GLTF or Three.js)
3. Improve camera controls
4. Add timeline scrubber

**Files to Modify:**
- `src/web/index.html` (enhance Plotly configuration)
- `src/web/app.py` (add enhanced trajectory data)

---

### Goal 8: Hardware Health Pre-flight Check
**Priority:** LOW  
**Effort:** Low  
**Status:** ⬜ Not Started

**Requirements:**
- [ ] Borrow logic from official Hardware Report tool
- [ ] Add sensor health checks
- [ ] Display "Hardware Health" summary at top of report
- [ ] Link to official tool for complete pre-flight check

**Implementation Plan:**
1. Review official ArduPilot hardware report logic
2. Implement basic sensor health checks
3. Add to report output

**Files to Create/Modify:**
- `src/diagnosis/hardware_health.py`
- Update `src/cli/formatter.py`

---

### Goal 9: Batch Processing + Progress Bars
**Priority:** MEDIUM  
**Effort:** Low  
**Status:** ⬜ Not Started

**Requirements:**
- [ ] Add batch processing for large folders of logs
- [ ] Show progress bars during analysis
- [ ] Generate summary report for batch
- [ ] Cache results for instant re-analysis

**Implementation Plan:**
1. Review existing batch.py command
2. Add tqdm progress bars
3. Generate batch summary statistics
4. Add result caching

**Files to Modify:**
- `src/cli/commands/batch.py` (enhance with progress bars)
- `src/comparison/trend_analyzer.py` (add caching)

---

### Goal 10: Partial Multi-platform Support (v2)
**Priority:** LOW  
**Effort:** High  
**Status:** ⬜ Not Started

**Requirements:**
- [ ] Support Betaflight .bbl logs (optional plugin)
- [ ] Support PX4 .ulg logs (optional plugin)
- [ ] Scope as v2 or optional plugin

**Implementation Plan:**
1. Research Betaflight/PX4 log formats
2. Create plugin architecture
3. Implement parsers as optional modules

**Files to Create:**
- `src/parser/betaflight_parser.py` (v2)
- `src/parser/px4_parser.py` (v2)

---

## Implementation Order

**Week 1-2 (Phase 1):**
1. ✅ Extended Failure Classes (quickest win)
2. ✅ Better Report Exports (PDF + improved HTML)
3. ⬜ Multi-flight Comparison Mode

**Week 3-5 (Phase 2):**
4. ⬜ Local AI Chat Assistant
5. ⬜ Tuning Advisor Module

**Week 6-8 (Phase 3+):**
6. ⬜ Agent/LLM-friendly JSON
7. ⬜ Improved 3D Replay
8. ⬜ Hardware Health Check
9. ⬜ Batch Processing Improvements
10. ⬜ Multi-platform Support (stretch goal)

---

## Success Metrics

After completing all goals:
- **Feature Completeness:** Match or exceed BBA + smarttune-cli feature set
- **User Experience:** Professional PDF reports + conversational AI
- **Diagnostic Power:** 10+ failure classes with calibrated confidence
- **Community Value:** Free, open-source, more powerful than SaaS alternatives
- **Marketing Tagline:** "The most complete open-source ArduPilot log diagnosis + tuning tool"

---

## Risk Mitigation

**To avoid conflicts and maintain stability:**
1. **Branch Strategy:** Create feature branches for each goal
2. **Test Coverage:** Maintain 176+ passing tests throughout
3. **Data Safety:** Never modify existing test data; add new data separately
4. **Backward Compatibility:** Keep existing CLI commands working
5. **Incremental Deployment:** Merge one feature at a time after testing

**Verification Steps Before Each Merge:**
- [ ] Run `pytest` - all tests must pass
- [ ] Test existing CLI commands still work
- [ ] Verify sample.bin analysis unchanged
- [ ] Check JSON output schema backward compatible
- [ ] Update documentation

---

*Last Updated: 2026-05-06*
*Project: ArduPilot AI Log Diagnosis (BeastAyyG)*
*GSoC 2026 Ready*
