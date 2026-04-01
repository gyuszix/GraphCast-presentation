---
transition: fade-out
layout: image-right
image: https://plus.unsplash.com/premium_photo-1669048778061-ecd451ecbbed?q=80&w=1470&auto=format&fit=crop&ixlib=rb-4.1.0&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D
---

# Why Weather Prediction Matters

- Daily decisions — what's the weather gonna be today?
- Agriculture — precipitation patterns for crops
- Natural disasters — hurricanes, floods, heatwaves
- Economic planning — energy, logistics, insurance
- Early warning systems — time to evacuate and prepare

---
layout: two-cols
transition: slide-left
---

# NWP: Gold Standard & Its Limits

The **ECMWF**'s Integrated Forecasting System and its model **HRES** is the gold standard in weather prediction.

- Solves governing equations of atmospheric dynamics on supercomputers
- Global 10-day forecasts at **0.1° resolution**
- Runs on **11,664+ cores**, ~1 hour per forecast

::right::

<div class="ml-4 mt-16">

### But it has limits:

- **Computationally expensive** — requires supercomputers
- **Cannot learn from historical data** — vast archives exist but NWP can't use them
- **Improvements are slow** — experts hand-craft better equations
- **Weak in certain regimes** — heat waves, precipitation nowcasting

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
layout: default
transition: slide-left
---

# How Do Graph Neural Networks Work?

GNNs operate on **graph structures** — nodes connected by edges — and learn through **message-passing**.

1. Each node holds features (e.g. temperature, pressure at a location)
2. In each round, nodes **collect messages** from their neighbors
3. Messages are **aggregated** and used to **update** the node's state
4. After multiple rounds, each node has information from many hops away

This is powerful for weather because conditions at one location depend on distant conditions — a pressure system over the Atlantic affects weather in Europe. Message-passing propagates this influence naturally.

> **Why not a regular grid?** Lat-lon grids bunch up at the poles. GNNs can operate on irregular meshes that distribute points evenly across the globe.

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