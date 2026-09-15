# Landslide Slope Stability Digital Twin
*Rainfall-Coupled Risk Assessment & Decision-Support Dashboard*

## Abstract

Rainfall-triggered landslides are among the most sudden and destructive natural hazards in hilly terrain, yet most slope stability assessments are static, one-time calculations that do not capture how a hillside's condition changes as a storm unfolds. This project builds a MATLAB-based digital twin that models this evolution directly — continuously simulating slope stability, hour by hour, as rainfall infiltrates the soil, raises pore water pressure, and erodes the slope's margin of safety.

The twin layers three physically grounded models on a gridded terrain: a geotechnical layer that applies the infinite slope stability method (Mohr-Coulomb failure criterion) to compute a Factor of Safety (FoS) at every point on the slope; a hydrological layer that routes an incoming rainfall event through infiltration and soil-moisture storage to generate time-varying saturation; and a pore-pressure layer that feeds that saturation back into the FoS calculation via the effective stress principle — the same mechanism behind most real-world rainfall-induced landslides.

The result is a live, interactive dashboard: a time slider lets the user scrub through a simulated storm and watch a colour-coded risk map, a four-tier risk classification (Low / Moderate / High / Extreme), affected area, and exposed houses and roads update in real time, alongside an automatically generated recommended action ranging from "Monitor Region" to "Immediate Evacuation." It turns a one-off engineering calculation into a continuously evolving, decision-ready risk picture — the core purpose of a digital twin.

## Problem Statement Addressed

> "Develop a digital twin model that simulates slope stability or flood-prone regions to assess disaster risks under different environmental conditions."

This project addresses the slope-stability half of the statement — modelling rainfall-induced landslide risk on a hillslope. The pipeline is deliberately modular so it can later be extended to flood-prone-region modelling (see *Future Scope*).

## Objectives

- Simulate spatial slope stability across a terrain grid using elevation-derived slope angles and the infinite slope stability method.
- Couple a rainfall-infiltration-saturation model with pore-water-pressure generation, so the model reproduces the actual physical trigger behind rainfall-induced landslides rather than an arbitrary risk score.
- Track how the Factor of Safety evolves *in time* over a rainfall event, not just as a single static number — the defining feature of a digital twin.
- Convert raw stability numbers into a tiered, human-readable risk classification and a specific recommended action.
- Estimate concrete infrastructure exposure (affected area, houses, roads) so risk is communicated in relatable terms.

## Proposed Methodology

1. **Terrain & slope modelling** — A gridded elevation surface is used to compute terrain gradients and per-cell slope angle.
2. **Baseline (dry) stability** — The infinite slope equation is applied with assigned soil parameters to generate a baseline Factor of Safety map, cross-checked against elevation, slope, and contour visualisations.
3. **Rainfall-hydrology coupling** — An hourly rainfall hyetograph is passed through an infiltration fraction and a capacity-limited soil-moisture "bucket" model to compute time-varying saturation.
4. **Pore pressure generation** — Saturation at each time step is converted into pore water pressure, reducing the slope's effective normal stress.
5. **Time-evolving wet FoS** — The effective-stress form of the infinite slope equation is recomputed for every grid cell at every time step, producing a full spatiotemporal stability field across the storm event.
6. **Risk classification & action logic** — Each cell is tagged Low / Moderate / High / Extreme by FoS threshold; the proportion of high-risk area drives an overall regional risk rating and a matched recommended action.
7. **Infrastructure exposure** — Prototype building locations and a representative road corridor are checked against local FoS values to estimate affected structures, road risk, and total at-risk area.
8. **Interactive dashboard** — A MATLAB-based live dashboard ties all of the above into one view: a colour-coded risk map, real-time status readouts, an infrastructure-impact panel, and dual trend charts (FoS and rainfall), all driven by a time slider.

**Representative parameters used (demo values):** cohesion 15 kPa · friction angle 30° · soil density 1800 kg/m³ · soil depth 5 m · infiltration fraction 60% · soil-water capacity 150 mm · 8-hour storm scenario with hourly rainfall rising from 0 to 100 mm.

## Key Features & Innovation

- **Physics-based, not black-box** — grounded in established Mohr-Coulomb / infinite-slope geotechnical theory and standard infiltration hydrology, so every output traces back to an interpretable physical cause.
- **A genuine time-evolving twin** — risk is tracked continuously through a simulated storm, not computed once and left static.
- **Actionable by design** — automatically converts numeric FoS into a tiered action (Monitor → Issue Warning → Evacuate → Immediate Evacuation), cutting the interpretation burden on responders.
- **Infrastructure-aware** — links abstract risk scores to tangible exposure: number of houses, km² affected, road segments at risk.
- **Modular & extensible** — terrain, soil, rainfall, and infrastructure are all swappable inputs, so the same stability engine can be re-pointed at real site data without a redesign.

## Feasibility & Viability

- **Already working, not just conceptual** — the full pipeline (terrain → dry FoS → rainfall coupling → wet FoS → risk classification) and the live dashboard are implemented and running.
- **Low-cost, standard tooling** — built entirely on MATLAB's numerical and UI capabilities, with no external paid services or specialised hardware needed to run the twin itself.
- **Well-understood physics** — relies on decades-old, widely validated geotechnical and hydrological models, keeping the risk of the underlying science being wrong low.
- **Remaining gap is data, not modelling** — sourcing real terrain, soil, and rainfall data for a specific site is the main step before field deployment, not the modelling approach itself.

## Tools & Technologies

- **MATLAB** for grid-based numerical simulation, gradient/slope computation, and the geotechnical and hydrological modelling.
- **MATLAB App Designer-style UI** (`uifigure`, `uigridlayout`, `uiaxes`, `uislider`) for the real-time interactive dashboard.
- **Simulink** is earmarked for the next phase — closed-loop alert automation and sensor/hardware-in-the-loop extensions — since the current workload (grid-based spatial modelling, custom dashboard UI) is naturally script-oriented rather than block-diagram/control-loop-oriented.

## Expected Outcome & Impact

A working prototype that visibly demonstrates, hour by hour, how a rain event pushes a hillside from stable toward failure — and what that means for the people and infrastructure in its path. Because the underlying physics is standard and every input is parameterised, the same digital twin can be re-pointed at a real hillside simply by swapping in real terrain, soil, and rainfall data, making it a reusable decision-support tool for disaster-management authorities, municipal planners, and early-warning-system operators rather than a one-off demo.

## Future Scope

- Replace synthetic terrain with real DEM/GIS data for an actual site.
- Integrate live or forecast rainfall via weather APIs and real soil-moisture/piezometer sensor feeds, so the twin reflects actual rather than simulated conditions.
- Introduce spatially varying soil properties from field/geotechnical survey data instead of uniform values.
- Extend the hydrology model to cover the flood-prone-region half of the problem statement, so one twin can assess both landslide and flood risk.
- Use Simulink to automate real-world alerts (SMS/siren triggers) and to prototype instrumented early-warning hardware.
- Validate FoS thresholds against historical rainfall-landslide records, and explore ML-based risk calibration once such data is available.
