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
  - [Validation & Safety](#validation--safety)
  - [Limitations](#limitations)
  - [Productionizing an AI System](#productionizing-an-ai-system)
- [Roadmap](#roadmap)
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
3. **Habit formation:** increase frequency/regularity of customer usage (e.g., commutes). 

## Objectives & Success Criteria
- **Availability Uplift:** Reduce stock-outs at high-demand bays; increase pickup conversion.
- **Ops Efficiency:** **More batteries swapped per staff-hour** and **fewer van-kilometers per swap**.
- **EV Readiness:** Ensure booked cars/vans meet target SoC before pickup.
- **Customer Stickiness:** Grow weekly active commuters and plan adoption.

## Constraints & Assumptions
- **Verification & Safety:** Non-deterministic AI must be monitored and validated in production. 
- **Provider Volatility:** AI vendor/pricing may change; solution must be portable. 
- **Deliverables:** Overview narrative, targeted diagrams, and ADRs with trade-offs. 
