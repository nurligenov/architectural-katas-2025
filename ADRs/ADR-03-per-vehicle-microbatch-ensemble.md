# ADR-03: Per-Vehicle Micro-Batch + Ensemble Inference for 2–3h Maintenance Risk

## Status
- **Accepted**

## Context
We collect real-time telemetry for each electric vehicle (EV): battery (SoC, temperature, voltage), GPS/velocity, diagnostic error codes, usage patterns, and recent maintenance events.  
Our goal is to predict whether a vehicle will likely require maintenance in the **next 2–3 hours**, enabling proactive dispatch and reducing unplanned downtime.

**Key drivers and constraints:**
- Decisions must be refreshed frequently (sub-5-minute staleness) but can tolerate seconds of latency — strict real-time scoring is not required.  
- Feature engineering benefits from short-window aggregates (e.g., moving averages, rate-of-change metrics).  
- Model specialization is preferred (battery, thermal, contextual), since a single model underperforms across all failure modes.  
- Interpretability and auditability are required for fleet operations and compliance.

## Decision
Adopt a **per-vehicle micro-batch inference pipeline** that:
1. Aggregates the last N minutes of telemetry per vehicle into a feature vector (1–3 minute windows).  
2. Executes multiple **specialized predictive models** (battery degradation, thermal risk, anomaly detection, contextual usage).  
3. Combines model outputs using a **calibrated weighted ensemble** (meta-learner or constrained linear combiner) to produce a single probability of “maintenance needed in the next 2–3 hours.”  
4. Emits a scored decision and explanatory metadata to Fleet Operations within an SLA of under 5 minutes from data arrival.

This design supports near-real-time decisioning with interpretable, model-driven predictions that balance accuracy, latency, and operational usability.

## Alternatives Considered
1. **Pure streaming (event-by-event) model**  
   - Pros: Lowest latency; simplest pipeline.  
   - Cons: Poor feature quality, higher noise, reduced precision.  
2. **Hourly/daily batch predictions**  
   - Pros: Easy to manage and scale.  
   - Cons: Too coarse for 2–3h prediction horizon.  
3. **Single global model**  
   - Pros: Simplifies governance and deployment.  
   - Cons: Underfits diverse failure modes across vehicle conditions.

## Rationale
- **Micro-batching** provides stability and reduces computational overhead while preserving timeliness.  
- **Model specialization** captures domain-specific risk patterns (battery, usage, environment).  
- **Ensemble inference** yields improved overall precision/recall and model stability across contexts.  
- **Per-vehicle aggregation** naturally aligns with how maintenance actions are executed.

## Risks & Mitigations
| Risk | Mitigation |
|------|-------------|
| Telemetry gaps | Impute missing data and surface confidence levels |
| Concept drift (weather, usage shifts) | Scheduled retraining; monitoring PSI on features |
| Alert fatigue | Use hysteresis, thresholds, and rate limits |
| Label leakage | Enforce time-forward validation during training |

## Acceptance Criteria
- End-to-end pipeline delivers decisions within 5 minutes for ≥95% of vehicles.  
- Pilot achieves **≥0.7 precision** and **≥0.5 recall**, with median lead time ≥60 minutes.  
- Demonstrated 15%+ reduction in unplanned outages during pilot phase.  
