# ADR-07: Event-Driven Alerting for Low SoC + High Demand

**Status:** Declined
**Date:** 2025-10-19

## Context
Scooters and bikes often run out of battery during high-demand periods, leading to customer dissatisfaction.
Dispatchers need early alerts when vehicles in high-demand areas are approaching low SoC thresholds.

## Decision
Implement an **event-driven alerting system** built on **Kafka or Redis Streams**.
Each telemetry message includes `vehicle_id`, `bay_id`, `SoC`, and `predicted_demand`.
An alert is triggered when:
```SoC < 25% AND Predicted_Demand_next_2h > threshold```

Alerts are summarized by an **LLM Alert Summarizer** and pushed to the Dispatcher Dashboard and mobile apps.

## Consequences
**Positive:**
- Enables proactive response before battery depletion.
- Integrates with natural-language alert summaries.
- Scales horizontally with telemetry volume.

**Negative:**
- Risk of false positives under noisy conditions.
- Requires tuning to prevent “alert fatigue.”
- There’s no real-time impact, as the dispatcher team may take several hours to act on the requests.

## Alternatives Considered
- Batch polling every n minutes (more practical approach).
- Manual dispatcher monitoring (rejected: not scalable).
