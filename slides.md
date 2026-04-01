---
theme: seriph
background: https://images.unsplash.com/photo-1534088568595-a066f410bcda?w=1920
class: text-center
highlighter: shiki
drawings:
  persist: false
transition: slide-left
title: "GraphCast: Learning Skillful Medium-Range Global Weather Forecasting"
---

# GraphCast

### Learning Skillful Medium-Range Global Weather Forecasting

**A presentation by Hrishikesh Pradhan, Danny Rollo and Gyula Planky**


---
transition: fade-out
layout: image-right
image: https://images.unsplash.com/photo-1527482797697-8795b05a13fe?w=800
---

# Why Weather Prediction Matters

- Daily decisions — what's the weather gonna be today?
- Agriculture — precipitation patterns for crops
- Natural disasters — hurricanes, floods, heatwaves
- Economic planning — energy, logistics, insurance
- Early warning systems — time to evacuate and prepare

---
layout: default
transition: slide-left
---

# The Gold Standard: Numerical Weather Prediction

The **European Centre for Medium-Range Weather Forecasts (ECMWF)** operates the Integrated Forecasting System (IFS), whose deterministic model **HRES** is the most accurate in the world.

- Solves governing equations of atmospheric dynamics on supercomputers
- Global 10-day forecasts at **0.1° resolution**
- Runs on a cluster of **11,664+ cores**, takes ~1 hour per forecast
- Decades of rigorous research and continuous improvement

> *"The accuracy of weather forecasts have increased year after year, to the point where the surface temperature, or the path of a hurricane, can be predicted many days ahead — a possibility that was unthinkable even a few decades ago."*

---
layout: two-cols
transition: slide-left
---

# NWP's Limitations

- **Computationally expensive** — requires supercomputers, ~1 hour per 10-day forecast
- **Cannot learn from historical data** — vast archives like ECMWF's MARS exist but NWP can't use them directly
- **Improvements are slow** — relies on experts hand-crafting better equations and approximations
- **Weak in certain regimes** — sub-seasonal heat waves, precipitation nowcasting

::right::

<div class="ml-4 mt-12">

```
NWP Scaling:

✅ More compute → better accuracy
❌ More data → no direct improvement
❌ Speed → limited by physics solvers
```

This is where **machine learning** enters the picture.

</div>

---
layout: default
transition: fade-out
---

# Machine Learning-Based Weather Prediction (MLWP)

An alternative approach: **train forecast models directly on historical data**.

- Learns from reanalysis archives like **ERA5** — global weather data from 1959 to present, at 0.25° resolution
- Captures patterns not easily expressed in hand-crafted equations
- Exploits modern **deep learning hardware** (GPUs/TPUs) instead of supercomputers
- Dramatically better **speed-accuracy tradeoff**

---
layout: default
transition: slide-left
---

# MLWP Architectures Approaching HRES

Several deep learning architectures have been applied to medium-range forecasting:

| Architecture | Example Model | Key Idea |
|---|---|---|
| **CNNs** | WeatherBench baselines | Local spatial pattern recognition |
| **Transformers** | ClimaX | Global attention, foundation model approach |
| **Fourier Neural Operators** | FourCastNet | Learn in frequency space for efficient global patterns |
| **Graph Neural Networks** | GraphCast | Message-passing on irregular meshes |

These methods have been steadily closing the gap with HRES at **0.25°** resolution and lead times up to **7 days**.

---
layout: center
class: text-center
background: https://images.unsplash.com/photo-1504608524841-42fe6f032b4b?w=1920
---

# And then came GraphCast.

---
layout: default
transition: slide-left
---

# GraphCast: Key Claims

- **Outperforms HRES on 90.3%** of 1,380 verification targets (variables × pressure levels × lead times)
- Produces a full **10-day global forecast in under 1 minute** on a single Cloud TPU v4
- Operates at **0.25° resolution** (~28km at the equator) with **227 predicted variables**
- Only **36.7 million parameters** — small by modern ML standards
- Strong on **severe events** — tropical cyclone tracking, atmospheric rivers, extreme temperatures — without being specifically trained for them

<br>

> HRES: 11,664-core supercomputer cluster, ~1 hour
>
> GraphCast: 1 TPU, < 1 minute

---
layout: image-right
image: https://images.unsplash.com/photo-1470071459604-3b5ec3a7fe05?w=800
transition: slide-left
---

# What This Paper Represents

- First ML model to **decisively surpass** the best operational NWP system across a broad evaluation
- Not a replacement — MLWP depends critically on NWP-generated reanalysis data like ERA5
- A complement that can **improve and extend** current best methods
- Opens doors beyond weather: climate, ecology, energy, agriculture

**Next: How does GraphCast actually work?** →