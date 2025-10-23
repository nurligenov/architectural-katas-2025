# MobilityCorp | O’Reilly Architectural Katas (Fall 2025)

A structured approach to the **O’Reilly Fall 2025 Architectural Kata Challenge** focused on AI-driven fleet optimization for e-scooters, e-bikes, cars, and vans.

## Table of Contents
- [MobilityCorp | O’Reilly Architectural Katas (Fall 2025)](#mobilitycorp--oreilly-architectural-katas-fall-2025)
  - [Table of Contents](#table-of-contents)
  - [Team](#team)
- [Problem Definition](#problem-definition)
  - [Context](#context)
  - [Operating Model Today](#operating-model-today)
  - [Core Challenges](#core-challenges)
  - [Objectives & Success Criteria](#objectives--success-criteria)
  - [Constraints & Assumptions](#constraints--assumptions)
- [Solution](#solution)
  - [Business Outcomes](#business-outcomes)
  - [AI Use Cases](#ai-use-cases)
  - [Architecture Characteristics](#architecture-characteristics)
  - [Detailed Architecture](#detailed-architecture)
  - [Limitations](#limitations)
  - [Productionizing an AI System](#productionizing-an-ai-system)
- [Our Learnings](#our-learnings)

## Team
- Rahul Kala
- Temirlan Nurligenov
- Rachel Li
- Sebastian Burlacu
- Jonathan Paserman

# Problem Definition

## Context
**MobilityCorp** offers short-term rental of **electric scooters, eBikes, cars, and vans** across multiple city locations. Customers use an NFC-capable app to lock/unlock vehicles; GPS trackers provide continuous location. Cars and vans can be booked in advance; bikes/scooters are bookable within a short window.  

On return, **all vehicles must be parked at designated bays** with **photo proof**; cars/vans must be **plugged into EV chargers**. 

For micro-mobility, **battery packs are swapped by staff in vans**, who also rebalance vehicles toward popular spots. 

## Operating Model Today
- **Bookings:** Cars/vans up to 7 days ahead with fixed duration; bikes/scooters up to 30 minutes ahead, open-ended up to 12 hours. 
- **Payments & Fines:** Per-minute pricing; fines for late/wrong-location returns. 
- **Telematics:** GPS on all vehicles; remote unlock/disable; NFC app interaction. 

## Core Challenges
1. **Right vehicle, right place, right time:** demand often mismatches supply.   
2. **Energy readiness:** vehicles run out of charge; we must **prioritize which bikes/scooters to swap**.   
3. **Inconsistent usage:** customers only use our vehicles on an ad-hoc basis. We would like them to rely on us for daily commute.

## Objectives & Success Criteria
- **Availability Uplift:** Reduce stock-outs at high-demand bays; increase pickup conversion.
- **Ops Efficiency:** **More batteries swapped per staff-hour** and **fewer van-kilometers per swap**.
- **EV Readiness:** Ensure booked cars/vans meet target SoC before pickup.
- **Customer Stickiness:** Grow weekly active commuters and plan adoption.

## Constraints & Assumptions
- **Verification & Safety:** Non-deterministic AI must be monitored and validated in production. 
- **Provider Volatility:** AI vendor/pricing may change; solution must be portable. 
- **Deliverables:** Overview narrative, targeted diagrams, and ADRs with trade-offs.

# Solution

## Business Outcomes

By introducing AI-driven forecasting, route optimization, and natural-language operations support, **MobilityCorp** aims to achieve:

- Increased vehicle availability across high-demand bays.  
- Reduced operational overhead through smarter route planning.  
- Improved staff efficiency and reduced redundant trips.  
- Higher reliability of charged vehicles for reservations.  
- Better customer retention and habitual use (commuter patterns).  

---

## AI Use Cases

| ID | AI Use Case | Description | Model / Technique |
|----|--------------|--------------|-------------------|
| **UC-01** | **Demand Forecasting** | Predict hourly demand per bay using time-series forecasting. | Temporal Fusion Transformer (TFT) |
| **UC-02** | **Battery Depletion Model** | Estimate time-to-empty (TTE) from telemetry and environment data. | Gradient Boosted Decision Trees (GBDT) |
| **UC-03** | **Clustered Battery Swap Routing** | Batch grouping of low-SoC vehicles into clusters and generating optimal routes for staff. | K-Means / DBSCAN + CVRPTW Solver |
| **UC-06** | **AI Trip Assistant** | Plan routes, book vehicles and help users perform daily tasks using our service. | MCP client/server architecture with hooks to LLM models and APIs
| **UC-05** | **Vision-Based Return Verification** | Validate vehicle return photos and charger connections for cars/vans. | CNN-based Image Classification |

---

## Architecture Characteristics

- **Scalability:** Modular microservices (AI Gateway, Alert Engine, Routing Optimizer) scale independently.  
- **Resilience:** Fallback rules ensure continued operation if models or APIs fail.  
- **Observability:** Prometheus and MLflow track forecast error, swap latency, and LLM confidence.  
- **Portability:** AI Gateway abstracts model vendors (OpenAI, Vertex AI, Hugging Face).  
- **Privacy:** Telemetry and user data are anonymized before model training.  
- **Human-in-the-Loop:** Dispatchers approve optimized routes before deployment.

---

## Detailed Architecture

### **Overview**
MobilityCorp’s AI-driven optimization platform revolves around two primary components:
1. **Forecasting Service** — predicts demand and battery depletion using machine learning models.  
2. **Alerting & Notification System** — batches, clusters, and communicates battery swap recommendations to dispatchers and field teams.

Together, these enable proactive fleet management and operational efficiency.

---

### **C3 Diagram — Forecasting and Alerting Engine**
TODO: add C3 diagram
```plaintext
+---------------------------------------------------------------------------------------------+
|                                Forecasting & Alerting System                                |
|---------------------------------------------------------------------------------------------|
|                                                                                             |
|   ┌──────────────────────────────┐                                                          |
|   │   Scheduler (Airflow/Dagster)│                                                          |
|   │  - Runs every 15–30 mins     │                                                          |
|   └─────────────┬────────────────┘                                                          |
|                 │                                                                           |
|                 ▼                                                                           |
|   ┌──────────────────────────────┐   ┌───────────────────────────────┐                      |
|   │  Data Collector              │   │   Forecasting Service         │                      |
|   │  - Pulls telemetry (SoC, GPS)│   │   - Demand Forecast (TFT)     │                      |
|   │  - Gathers user activity     │   │   - Battery Depletion (GBDT)  │                      |
|   │  - Includes weather/events   │   └─────────────┬─────────────────┘                      |
|   └─────────────┬────────────────┘                 │                                        |
|                 │                                 ▼                                         |
|                 │                    ┌───────────────────────────────┐                      |
|                 │                    │   Alert Engine                │                      |
|                 │                    │  - Aggregates forecast + SoC  │                      |
|                 │                    │  - Identifies low-SoC clusters│                      |
|                 │                    │  - Scores urgency per bay     │                      |
|                 │                    └─────────────┬─────────────────┘                      |
|                 │                                  │                                        |
|                 ▼                                  ▼                                        |
|   ┌──────────────────────────────┐   ┌──────────────────────────────┐                       |
|   │  Clustering Engine           │   │  Route Optimizer             │                       |
|   │  - K-Means / DBSCAN          │   │   - Plans swap van routes    │                       |
|   │  - Groups vehicles by area   │   │                              │                       |
|   └─────────────┬────────────────┘   └─────────────┬────────────────┘                       |
|                 │                                  │                                        |
|                 ▼                                  ▼                                        |
|   ┌──────────────────────────────┐   ┌───────────────────────────────┐                      |
|   │  LLM Summarizer              │   │  Alert Dispatcher             │                      |
|   │  - Converts data to text     │   │  - Pushes to Dashboard, App   │                      |
|   │  - Generates ops summaries   │   │  - REST / WebSocket           │                      |
|   └─────────────┬────────────────┘   └─────────────┬─────────────────┘                      |
|                 │                                  │                                        |
|                 ▼                                  ▼                                        |
|   ┌──────────────────────────────┐   ┌───────────────────────────────┐                      |
|   │ Dispatcher Dashboard         │   │ Field Staff App               │                      |
|   │ - Shows alerts & clusters    │   │ - Receives swap routes        │                      |
|   │ - Approves route plans       │   │ - Reports completion          │                      |
|   └─────────────┬────────────────┘   └─────────────┬─────────────────┘                      |
|                 │                                  │                                        |
|                 ▼                                  ▼                                        |
|         ┌──────────────────────────────┐     ┌───────────────────────────────┐              |
|         │  Metrics & Drift Monitor     │     │  Operational Data Store       │              |
|         │  - Logs alert accuracy       │     │  - Stores telemetry + alerts  │              |
|         │  - Retrain trigger thresholds│     │  - Feeds retraining pipeline  │              |
|         └──────────────────────────────┘     └───────────────────────────────┘              |
|                                                                                             |
+---------------------------------------------------------------------------------------------+
```

---

### **Data Flow Overview**

The following flow summarizes how data moves through the **Forecasting and Alerting Engine**, from telemetry ingestion to actionable alerts and feedback loops.
TODO: add diagram with description

---

## Limitations

While the MobilityCorp AI Optimization Platform provides strong predictive and operational capabilities, several limitations and open challenges remain:

### **1. Batch Processing Latency**
- The batch alert pipeline runs every 15–30 minutes, meaning alerts are not instantaneous.  
- Real-time Kafka-based processing could be added in future phases, but current focus is on operational stability.

### **2. Cluster Sensitivity**
- **K-Means** and **DBSCAN** require careful tuning of parameters for each city’s density.  
- Overly tight clustering may cause redundant van trips; loose clustering can reduce responsiveness.

### **3. Data Completeness**
- Forecast accuracy depends on consistent telemetry, user intent, charging status, and weather/event feeds.  
- Missing or delayed data streams can lead to underestimation of demand or missed low-SoC alerts.

### **4. LLM Variability**
- Summarization and recommendations from the LLM Co-pilot may vary slightly between runs.  
- Prompt templates and structured outputs mitigate inconsistency, but dispatcher verification remains essential.

### **5. Model Drift and Retraining**
- Over time, seasonal trends or user behavior changes can affect model accuracy.  
- Automated drift detection and retraining pipelines reduce risk, but periodic human audit is still required.

### **6. Hardware and Infrastructure Dependencies**
- Telemetry accuracy relies on IoT sensors (SoC, GPS) functioning properly across thousands of devices.  
- Connectivity loss or hardware failures can delay updates or generate false low-SoC signals.

### **7. Human-in-the-Loop Bottlenecks**
- Dispatcher review ensures reliability but adds human latency to the process.  
- Long-term goal: semi-automated approvals with confidence thresholds (e.g., 90% auto-approve).

---

## Productionizing an AI System

To ensure scalability, observability, and reliability, the architecture adopts proven **MLOps and AI governance patterns**.

### **1. AI Gateway (ADR-05)**
- Abstracts external AI providers (OpenAI, Gemini).  
- Handles model versioning, cost tracking, and graceful fallback if one provider changes pricing or availability.

### **2. Continuous Learning & Retraining (ADR-06)**
- Telemetry, forecasts, and operational outcomes flow back into the **Feature Store**.  
- Automated Airflow/Dagster jobs retrain and validate models nightly.  
- Retraining triggered early if model drift > 15% or false alert rate > 25%.

### **3. Batch-Based Alerting (ADR-08)**
- Reduces alert noise by grouping low-SoC vehicles in 15–30 minute intervals.  
- Combines predictive and clustering models (TFT + GBDT + K-Means).  
- Optimized for stability and scalability across multiple cities.

### **4. AI Trip Assistant (ADR-09)**
- Help users find the best route using our service
- Book vehicles for them ahead of time on user consent
- Recommend best subscription pack and purchase it for them
- Plan and book vehicles for fun day trips using local events

### **4.1 LLM Ops Co-pilot
- Provides explainability and dispatcher assistance.  
- Generates summaries, route recommendations, and shift briefings.  
- Uses structured prompt templates, guardrails, and confidence-based fallbacks.

### **5. Observability & Monitoring**
- **Prometheus + MLflow + Langwatch** stack for:
  - Forecasting error tracking (MAPE, RMSE).  
  - Alert precision/recall metrics.  
  - LLM confidence and latency.  
- Monitors data pipeline uptime and ingestion latency.

### **6. Governance & Security**
- User IDs, payment references hashed or tokenized before model ingestion.  
- API Gateway enforces role-based access and audit logs.  
- Compliance with GDPR and other data privacy standards.

### **7. Deployment & Scaling**
- Containerized microservices deployed on **Kubernetes** or **GCP Cloud Run**.  
- Auto-scaling triggers on telemetry throughput and model inference load.  
- Canary rollouts for model updates using shadow testing before full deployment.

---

## Our Learnings

Through the design and iteration of the **MobilityCorp AI Optimization Platform**, our team uncovered valuable insights about blending **predictive analytics**, **generative AI**, and **operational systems** in a production environment.

---

### 1. Forecasting Accuracy is Only Half the Battle**
Even highly accurate demand or SoC depletion forecasts do not automatically yield operational impact.  
The key is **translating predictions into actionable workflows** — combining clustering, routing, and human-in-the-loop review to ensure outcomes are both **intelligent and practical**.

---

### 2. Batch vs. Real-Time Trade-Offs**
We discovered that **real-time alerts** introduced too much operational noise, while **batch recommendations** (every 15–30 minutes) offered a better balance between freshness and reliability.  
This hybrid approach keeps operations stable and allows dispatchers to act decisively instead of reactively.

---

### 3. Generative AI as a Communication Bridge**
The **LLM Co-Pilot** proved invaluable not as a decision-maker but as a **translator between data and people**.  
By turning clusters, forecasts, and route data into natural-language recommendations, dispatchers understood system decisions faster and trusted them more — improving adoption and usability.

---

### 4. Clustering > Chaos**
Spatial clustering (K-Means / DBSCAN) simplified what was once an overwhelming task — managing hundreds of low-SoC vehicles at once.  
Grouping nearby tasks gave us a way to reason about operational zones, reduce redundant travel, and make route optimization computationally tractable.

---

### 5. Continuous Feedback Loops are Non-Negotiable**
The system’s effectiveness improves only when **telemetry, outcomes, and human feedback** flow back into retraining pipelines.  
Metrics-driven retraining and drift detection turned the models from static predictors into **living systems that evolve with the city**.
