# ADR-05: Introduce an AI Gateway for Model Portability

**Status:** Accepted
**Date:** 2025-10-19

## Context
Generative AI and ML model providers frequently change APIs, pricing, and capabilities.
MobilityCorp must remain independent from any single model vendor while retaining flexibility to test new AI technologies.

## Decision
Implement a centralized **AI Gateway Service** that exposes standard task-level APIs:
- `/forecast/demand`
- `/predict/battery_depletion`
- `/generate/route`
- `/summarize/alert`

The gateway will abstract provider details (e.g., OpenAI, HuggingFace, Vertex AI) and handle:
- Authentication and rate limiting
- Cost tracking per call
- Provider health monitoring
- Caching and fallback routing

## Consequences
**Positive:**
- Enables model portability and experimentation.
- Simplifies integration for downstream services.
- Supports hybrid cloud and on-prem deployment.

**Negative:**
- Adds one network hop (small latency overhead).
- Requires governance and monitoring ownership.

## Alternatives Considered
- Direct integration with each provider (rejected: brittle and unscalable).
- On-prem-only models (rejected: limits flexibility and innovation).
