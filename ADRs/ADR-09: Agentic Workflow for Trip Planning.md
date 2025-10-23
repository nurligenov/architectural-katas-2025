# ADR-09: Agentic Workflow for Trip Planning
**Status**: Accepted
**Date**: 2025-10-21

## Context
We are tackling the challenge that most of our customers only use our service on an ad-hoc basis. We would like more of them to rely on us on a regular basis like daily commmute. 

## Current Pain Point
- User uses our app as an ad-hoc to their daily navigation apps(google maps, apple maps etc.). They need to switch over to our app for booking. The booking experience is not seamless.
- They usually only come to us when their navigation app suggests that no public transport alternatives are available or it's too far for walking.
- As a result, their trip planning experience involves a lot of extra thinking.
- The vehicle booking experience can be too length. For first timers, if they are in a rush, first finding the route, then setting up their account and finally locating a bike is usually too much of a hassle. Many users quit when feeling like there are too many steps.

## Decision
We will implement an **agentic workflow** powered by a **personal mobility assistant agent** that manages **both vehicle booking and navigation**.  
This agent will anticipate user intent, reserve vehicles automatically or on confirmation, and guide users through the best route to their destination.

### Core Capabilities
- **Predictive Commute Planning:** Learns from past travel patterns, calendar events, and contextual signals (e.g., time of day, weather, traffic).  
- **Seamless Booking:** Automatically reserves the preferred vehicle near the user’s starting point or destination ahead of commute time.  
- **Integrated Navigation:** Provides step-by-step navigation once the trip starts — switching between walking and e-vehicle routes dynamically.  
- **Adaptive Behavior:** Continuously refines its recommendations based on feedback, cancellations, and trip satisfaction signals.  
- **User Control:** Users can set preferences for automation level (auto-book, suggest only, or manual confirm) and route options.  

### Technical Approach
- **Agent Orchestration Layer:** Built with a modular LLM-driven framework (e.g., LangChain or Semantic Kernel) orchestrating booking and navigation APIs.  
- **Contextual Event Triggers:** Initiated by signals like user proximity, calendar alerts, or regular commute times.  
- **Data Sources:** User trip history, live vehicle availability, maps, traffic, and weather data.  
- **Integration:** The agent will be accessible via both backend and mobile app SDKs for consistent experience across platforms.



