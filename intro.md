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
transition: slide-left
---

# Machine Learning-Based Weather Prediction

An alternative approach: **train forecast models directly on historical data**.

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

# GNN Key Terminology

<div class="flex gap-8">
<div>

- **Nodes** — entities with feature vectors
- **Edges** — connections, can carry features too
- **Message passing** — gather → aggregate → update
- **Pooling** — routing info between graph attributes
- **Permutation invariance** — node order doesn't matter

<img src="/images/GNN image.png" class="mt-4 max-h-56 object-contain" />

</div>
<div>

### Encoder–Processor–Decoder

A common GNN pipeline:

1. **Encode** raw features into embeddings
2. **Process** via repeated message passing
3. **Decode** embeddings into predictions

> This is exactly how GraphCast is structured.

</div>
</div>

---
layout: image-right
image: https://images.unsplash.com/photo-1545987796-200677ee1011?q=80&w=1470
transition: slide-left
---

# How Do Graph Neural Networks Work?

GNNs operate on **graph structures** — nodes connected by edges — and learn through **message-passing**.

1. Each node holds features (e.g. temperature, pressure at a location)
2. In each round, nodes **collect messages** from their neighbors
3. Messages are **aggregated** and used to **update** the node's state
4. After multiple rounds, each node has information from many hops away


> **Why not a regular grid?** Lat-lon grids bunch up at the poles. GNNs can operate on irregular meshes that distribute points evenly across the globe.

---
layout: default
transition: slide-left
---

# GraphCast & What This Paper Represents

- **Outperforms HRES on 90.3%** of 1,380 verification targets (variables × pressure levels × lead times)
- Full **10-day global forecast in under 1 minute** on a single Cloud TPU v4
- **0.25° resolution** (~28km) with **227 predicted variables**, only **36.7M parameters**
- Strong on **severe events** — tropical cyclone tracking, atmospheric rivers, extreme temperatures — without specific training

> HRES: 11,664-core supercomputer cluster, ~1 hour → GraphCast: 1 TPU, < 1 minute

- First ML model to **decisively surpass** the best operational NWP system across a broad evaluation
- Not a replacement — MLWP depends critically on NWP-generated reanalysis data like ERA5
- A complement that can **improve and extend** current methods, opening doors to climate, ecology, energy, and agriculture