# ADR-002: Adopt a Hybrid Predictive + Generative AI Architecture

**Status:** Accepted
**Date:** 2025-10-19

## Context
MobilityCorp requires an AI-driven system that can both **predict demand and battery depletion** accurately, and **generate flexible route plans** for field staff under uncertainty (traffic, weather, staff availability).

Relying solely on deterministic models would optimize known patterns but fail in dynamic, unpredictable conditions. Conversely, a fully generative approach could lack precision and operational reliability.

## Decision
We will adopt a **hybrid architecture** combining:
- **Predictive models** (Temporal Fusion Transformer, Gradient Boosted Decision Trees) for demand forecasting and SoC depletion.
- **Generative AI models** (Graph Transformer / Diffusion Models) for producing diverse route candidates and what-if scenarios.
- **Classical optimization** (CVRPTW solver) to ensure route feasibility.

## Consequences
**Positive:**
- Balances accuracy and adaptability.
- Enables proactive swap scheduling based on future states.
- Supports scenario testing for uncertain conditions.

**Negative:**
- Requires orchestration between multiple model types.
- Increases compute complexity and monitoring needs.

## Alternatives Considered
- Only predictive models (rejected: less adaptable to real-world variance).
- End-to-end reinforcement learning (rejected: requires long training cycles and costly data).
