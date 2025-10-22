# ADR-03: Per-Vehicle Micro-Batch + Ensemble Inference for 2–3h Maintenance Risk

**Date:** 2025-10-22  
**Status:** Proposed 
**Decision Type:** Architecture & Modeling

## Context
We collect real-time telemetry for each electric vehicle (EV): battery (SoC, temp, voltage), GPS/velocity, error codes, usage patterns, and recent maintenance events. We want to predict whether a specific vehicle will likely require maintenance in the **next 2–3 hours**, to support proactive dispatch and reduce roadside failures.

### Constraints & drivers
- Decisions must be refreshed frequently (sub-5-minute staleness) but can tolerate seconds of latency; strict real-time per-event scoring is not required.
- Feature engineering benefits from short-window aggregates (e.g., moving averages, rates of change).
- Model diversity (specialists for battery health, thermal issues, location/usage context) is desirable; single models underperform across all failure modes.
- Interpretability and governance are required (explainable alerts, audit trails).

## Decision
Adopt a **per-vehicle micro-batch** pipeline (e.g., 1–3 minute windows) that:
1. **Aggregates** the last N minutes of telemetry per vehicle into a feature vector.
2. **Runs multiple specialized models** (battery degradation, thermal risk, anomaly detection, usage/context risk).
3. **Combines** their outputs via a **calibrated weighted ensemble** (stacked meta-learner or constrained linear combiner) to produce a single probability of “maintenance needed in next 2–3 hours.”
4. **Emits** a decision + explanation to Fleet Ops with an SLA of <5 minutes from data arrival.

## Scope
- In scope: data engineering (micro-batch), feature store, model serving for batch inference, ensemble logic, alerting, monitoring, A/B rollout.
- Out of scope: long-horizon capacity forecasting; spare-parts optimization (future ADR).

## Architecture (high level)
- **Ingestion:** Stream telemetry → durable log (e.g., Kafka).
- **Micro-batching:** Windowed per-vehicle aggregation (1–3 min tumbling windows).
- **Feature Store:** Write/read time-aligned features (online for serving, offline for training).
- **Model Execution:** Parallel scoring of specialized models (containerized).
- **Ensemble Layer:** Calibrated weights or meta-learner (logistic regression) trained on holdout labels; outputs `p(maintenance ≤ 3h)`.
- **Decision Service:** Thresholding with hysteresis, explanations, and routing to Ops.
- **Monitoring:** Data/feature drift, model performance, alert precision/volume.

## Rationale
- **Micro-batch over event-by-event:** Enables robust windowed features (rates/derivatives), reduces noise, and keeps infra costs predictable while meeting freshness needs.
- **Ensemble over single model:** Specialization handles heterogeneous failure modes; calibrated combiner improves overall precision/recall and stability.
- **Per-vehicle batching:** Natural keying aligns with how actions are taken (vehicle-level dispatch).

## Alternatives Considered
1. **Pure streaming + single model per event**
   - Pros: lowest latency; simpler serving.
   - Cons: weaker features (no robust windows), lower accuracy; higher alert noise.
2. **Daily/hourly batch**
   - Pros: cheapest; simplest ops.
   - Cons: stale; misses 2–3h horizon.
3. **One global model**
   - Pros: simpler governance.
   - Cons: underfits rare/long-tail failure modes.

## Data Model & Features (examples)
- **Battery:** SoC level/Δ, voltage stddev, internal resistance proxy, temperature and dT/dt.
- **Motion/Usage:** speed variance, stop/start cycles, elevation gain, utilization in last 1/4/12h.
- **Health Signals:** DTC/error codes, recent repairs, charger session anomalies.
- **Context:** ambient temp/rain (joined), congestion proxies, depot distance.
- **Targets/Labels:** Binary label if maintenance ticket created or verified failure occurred within 2–3h of the prediction timestamp.

## Ensemble Design
- **Base models (parallel):**
  - `M_batt`: gradient boosted trees for battery wear/instability.
  - `M_therm`: thermal excursion classifier.
  - `M_anom`: unsupervised anomaly score (e.g., Isolation Forest/AE) → calibrated to pseudo-prob.
  - `M_usage`: demand/context risk (short-term stress).
- **Combiner:** Logistic regression with non-negative weights + Platt/Isotonic calibration; retrained weekly.
- **Explainability:** Per-model SHAP summaries + ensemble weight contribution in alert payload.

## Decision Logic
- **Primary threshold:** `p ≥ 0.65` → “Maintenance likely within 3h”.
- **Hysteresis:** require persistence across two consecutive micro-batches OR `p ≥ 0.8` once for immediate alert.
- **Rate limit:** Max N alerts/vehicle/12h unless new critical codes appear.
- **Action routing:** nearest capable technician; include ETA, top contributing features, and last service date.

## SLAs & SLOs
- **Freshness:** ≤5 minutes from last event to decision.
- **Availability:** 99.5% decision service.
- **Precision@K:** ≥0.7 precision for top-priority alerts (tunable).
- **Lead time:** median ≥60 minutes before failure/ticket.
- **Coverage:** ≥95% of active fleet scored each cycle.

## Monitoring & Ops
- **Data quality:** missing rate, outliers, timestamp skew; auto-fallback to last good features.
- **Drift:** PSI on key features; auto-retrain suggestions.
- **Performance:** rolling precision/recall, alert acceptance rate by Ops, false-positive cost.
- **Canaries:** 10% vehicles scored by ensemble vs. champion; compare metrics.
- **Audit:** Store features, model versions, scores, thresholds, explanations per decision.

## Security & Privacy
- Pseudonymize vehicle IDs in modeling; restrict GPS granularity in stored features where not needed.
- Encrypt data at rest/in transit; RBAC for Ops vs. Data Science access.

## Rollout Plan
1. **Phase 0 (Sandbox):** Backtest on 60–90 days historical data; finalize thresholds.
2. **Phase 1 (Shadow):** Real-time scoring, no alerts to Ops; validate SLAs.
3. **Phase 2 (Limited Pilot):** 10–20% fleet; measure precision, lead time, Ops feedback.
4. **Phase 3 (GA):** Full fleet; weekly retraining cadence; quarterly model review.

## Risks & Mitigations
- **Telemetry gaps:** Impute and degrade gracefully; surface confidence scores.
- **Concept drift (seasonality/events):** Scheduled retrains; demand/context model uses recent windows.
- **Alert fatigue:** Hysteresis + rate limiting; involve Ops in threshold tuning.
- **Label leakage:** Strict time-forward validation; freeze features at prediction time.

## Acceptance Criteria
- End-to-end pipeline delivers scored decisions within SLA for ≥95% vehicles.
- Pilot achieves **≥0.7 precision** and **≥0.5 recall** on validated labels, median lead time ≥60 min.
- Ops confirms actionable explanations and reduced unplanned outages by ≥15% in pilot group.

## Open Questions
- Final window size (1 vs. 3 minutes) and ensemble threshold per vehicle type.
- Source of external context (weather/events) and licensing.
- Technician capacity constraints integration (to prioritize alerts under load).

## Versioning & Ownership
- Models and ensemble registered in Model Registry with semantic versioning.
- Changes to thresholds/weights require a lightweight RFC and offline validation.
