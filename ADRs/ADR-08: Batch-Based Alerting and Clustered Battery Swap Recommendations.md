# ADR-08: Batch-Based Alerting and Clustered Battery Swap Recommendations

**Status:** Accepted  
**Date:** 2025-10-19  

## Context
Scooters and bikes frequently deplete their batteries during high-demand periods, impacting service availability and customer satisfaction.  
While real-time alerts are valuable, continuous event-driven triggers risk alert fatigue and operational noise.  
Instead, the operations team needs **batch-based recommendations** — concise, periodic summaries that identify *which areas need battery swaps*, *how urgent they are*, and *optimal routes* for staff.  

## Decision
Shift from **event-driven alerts** to a **batch-oriented recommendation pipeline** that runs every 15–30 minutes.  
Each batch process will:
1. **Aggregate telemetry and prediction data** (SoC, location, demand forecast).  
2. Identify **low-SoC vehicles** (e.g., SoC < 25%) in **high-demand bays**.  
3. **Cluster nearby vehicles/bays** using **K-Means** (or DBSCAN for density-based grouping) to form logical swap zones.  
4. Generate optimal routes per cluster using a **CVRPTW solver** (Capacitated Vehicle Routing Problem with Time Windows).  
5. Pass results to the **LLM Summarizer**, which converts raw metrics into actionable text for dispatchers, e.g.:

> “⚡ You should charge 5 scooters in the Downtown cluster and 3 in Midtown.  
> Estimated completion: 2.5 hours. Route 1 covers Bays 14, 15, and 18.”

## Model Choices
| Purpose | Model | Description |
|----------|--------|-------------|
| **Clustering** | *K-Means* | Groups low-SoC vehicles geographically into manageable clusters (e.g., per 2–3 km zone). |
| **Density-based alternative** | *DBSCAN* | Automatically detects natural clusters without specifying K, good for uneven spatial data. |
| **Nearest-bay lookup** | *KNN (supporting)* | Finds the nearest available bay with charged vehicles or chargers. |
| **Optimization** | *CVRPTW Solver (OR-Tools)* | Generates optimal routes per cluster for staff vans. |

## Consequences
**Positive:**
- Reduces alert noise and operational fatigue.  
- Provides structured, scheduled insights instead of constant event streams.  
- Optimizes staff routes by clustering geographically close tasks.  
- Enables the LLM Co-pilot to give clear, high-level summaries (“Top 3 clusters to handle next”).  

**Negative:**
- Alerts are less immediate (15–30 min latency).  
- Requires careful scheduling to balance freshness vs. computational cost.  
- Cluster tuning (K size, distance thresholds) must adapt to city density and fleet size.  

## Alternatives Considered
- **Real-time event streaming (Kafka)** — rejected due to excessive alert volume and limited route-level context.  
- **Manual dispatcher monitoring** — rejected as non-scalable and error-prone.  
- **Static region assignments** — rejected as inefficient under dynamic demand and shifting vehicle distribution.  

## Implementation Notes
- Use **batch orchestrator (Airflow / Dagster)** to trigger analysis jobs.  
- Store intermediate cluster results in **Redis or Postgres** for fast retrieval.  
- The **LLM Summarizer** reads aggregated cluster data and composes daily and hourly recommendations.  
- Future enhancement: integrate **reinforcement learning** to adapt cluster radius dynamically based on operational feedback.  
