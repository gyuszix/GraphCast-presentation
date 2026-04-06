---
layout: default
transition: fade-out
---

# Training Data: ERA5 Reanalysis

GraphCast learns entirely from **ECMWF's ERA5 reanalysis archive** — a dense, global weather dataset.

- **0.25° resolution**, **6-hour intervals**, hundreds of variables from **1979 onward**
- Strict **temporal (causal) split** to prevent any future information leakage:

| Split | Years | Purpose |
|---|---|---|
| **Training** | 1979–2015 | Learn model weights |
| **Validation** | 2016–2017 | Hyperparameter tuning & model selection |
| **Test** | 2018–2021 | Final evaluation (never touched during development) |

- Architecture and training protocol were **fully frozen** before touching test data
- They also tried data back to 1959 but found **no benefit** — pre-satellite-era data may be too noisy

---
layout: default
transition: slide-left
---

# Loss Function: Weighted MSE

The training objective is **mean squared error** between GraphCast's predictions and ERA5 ground truth, but with careful weighting:

$$\mathcal{L} = \frac{1}{|\text{batch}|} \sum \text{area}(i) \cdot \sigma_j^{-1} \cdot w_j \cdot \left\| \hat{X}_{i,j} - X_{i,j} \right\|^2$$

### Three layers of weighting:

- **Area weighting** — grid cells near poles are smaller; weight by cell area to avoid polar bias
- **Inverse variance scaling** ($\sigma_j^{-1}$) — normalize each variable-level pair by the variance of its time differences, so all targets have roughly unit variance
- **Per-variable weights** ($w_j$):
  - Atmospheric variables weighted **proportional to pressure level** (proxy for air density — lower levels matter more)
  - Surface variable weights **hand-tuned** on validation: 2t gets weight **1.0**, while 10u, 10v, msl, and tp each get **0.1**

> This means the model prioritizes getting surface temperature right, while wind/pressure/precip contribute less to the gradient signal.

---
layout: default
transition: fade-out
---

# Autoregressive Curriculum Training

Training happens in **three phases**, progressively teaching the model to handle longer rollouts:

### Phase 1: Warmup (1,000 steps)
- **Single autoregressive step** — predict just one 6h step ahead
- Learning rate linearly ramps up to **1e-3**

### Phase 2: Single-Step Training (299,000 steps)
- Still **single-step** predictions
- Learning rate decays via **half-cosine schedule** back to zero
- This is where the model learns the bulk of its forecasting skill

### Phase 3: Rollout Fine-Tuning (11,000 steps)
- Autoregressive steps **ramp from 2 → 12** (increasing by 1 every 1,000 updates)
- Fixed learning rate of **3e-7** (very small — fine-tuning, not relearning)
- At 12 steps × 6h = the model learns to minimize error over **3-day trajectories**
- Backpropagation through time across all rollout steps

> This curriculum prevents the instability that would come from training on long rollouts from scratch.

---
layout: two-cols
transition: slide-left
---

# Optimization & Compute

### Optimizer: AdamW
- $\beta_1 = 0.9$, $\beta_2 = 0.95$
- Weight decay: **0.1**
- Gradient norm clipping at **32**

### Hardware
- **32 Cloud TPU v4** devices
- Batch size **32** (one sample per device)
- **bfloat16** mixed precision
- **Gradient checkpointing** to fit the 12-step unrolled model into 32 GB per device

::right::

<div class="ml-4 mt-16">

### Training Time
- **~4 weeks** total
- Phases 1+2: ~300K steps of single-step training
- Phase 3: 11K steps of increasingly long rollouts

### Model Size
- Only **36.7M parameters**
- Small by modern standards — constrained by memory needed for multi-step unrolling
- A family of models; this is the largest that fits current hardware

</div>

---
layout: default
transition: fade-out
---

# Verification: Fair Comparison With HRES

Evaluated on **69 variable-level combinations** × **20 lead times** = **1,380 verification targets**, using two metrics:

- **RMSE** — lower is better, area-weighted by grid cell size
- **ACC** (Anomaly Correlation Coefficient) — did you predict the *deviation from normal*, or just the climate average? A model that always predicts "typical weather for this date" scores ACC = 0. Higher is better (1.0 = perfect).

### The ground truth problem

Each model needs to be measured against its own "correct answer." ERA5 and HRES use different data assimilation systems, so they disagree slightly on what the atmosphere looks like at any given moment. If you scored HRES against ERA5, it would start with nonzero error before it even begins forecasting — that's not fair.

- **GraphCast** → measured against **ERA5** (what it was trained on)
- **HRES** → measured against **HRES-fc0** (HRES's own snapshot of the atmosphere at forecast hour zero — its starting state before stepping forward)
- Both evaluated only at times where they had the **same ±3h window of observations** as input

---
layout: default
transition: slide-left
---

# Headline Result: GraphCast vs. HRES Scorecard

GraphCast outperforms HRES on **90.3% of 1,380 verification targets** (RMSE).

<div class="flex gap-4 mt-4">
  <div class="flex-1">
    <img src="./images/figure 2b.png" class="h-60 mx-auto" />
    <p class="text-center text-sm mt-1">RMSE skill score on z500 — 7%–14% improvement across all lead times</p>
  </div>
  <div class="flex-1">
    <img src="./images/figure 2d.png" class="h-60 mx-auto" />
    <p class="text-center text-sm mt-1">Full scorecard — blue = GraphCast wins, red = HRES wins</p>
  </div>
</div>

$$\text{Skill Score} = \frac{\text{RMSE}_A - \text{RMSE}_B}{\text{RMSE}_B}$$

---
layout: default
transition: fade-out
---

# Does GraphCast Only Win Because It Blurs?

MSE-trained models learn to **hedge**: if uncertain whether a storm will be 100km east or west, the MSE-optimal prediction is to smear the storm across both locations. This produces blurry but low-RMSE forecasts. HRES runs physics equations forward and produces sharp (but sometimes misplaced) predictions.

### The skeptic's argument
GraphCast's RMSE advantage might just be a blurring trick — not better forecasting.

### The test: let both models blur
- Find the **best possible spatial blur** for each model's output (the amount that minimizes its RMSE)
- Apply it — GraphCast's barely changes (it's already blurred), HRES gets a significant RMSE boost
- Compare the blurred versions: GraphCast still wins on **88.0%** of 1,380 targets

> Down from 90.3% to 88.0% — so some advantage *was* from blurring, but most of it is real forecasting skill.

This is a known limitation: GraphCast expresses uncertainty through blurring rather than explicit probabilities. Its successor **GenCast** addresses this by generating ensembles of sharp, diverse forecasts using diffusion models.

---
layout: default
transition: slide-left
---

# Severe Event Forecasting: Tropical Cyclones

GraphCast was **not specifically trained** to predict severe events — but its forecasts support downstream severe event prediction.

<div class="flex gap-4 mt-4">
  <div class="flex-1">
    <img src="./images/figure 3a.png" class="h-55 mx-auto" />
    <p class="text-center text-sm mt-1">Median track error — GraphCast vs. HRES (2018–2021)</p>
  </div>
  <div class="flex-1">
    <img src="./images/figure 3b.png" class="h-55 mx-auto" />
    <p class="text-center text-sm mt-1">Paired error difference — significantly better from 18h to 4.75 days</p>
  </div>
</div>

- Tracking algorithm based on **ECMWF's published protocols**, applied to GraphCast's predicted fields
- Ground truth: **IBTrACS** — independent reanalysis dataset of cyclone tracks
- Bootstrapped **95% confidence intervals** confirm significance

---
layout: default
transition: fade-out
---

# Severe Events: Atmospheric Rivers & Extreme Temperatures

<div class="flex gap-4 mt-2">
  <div class="flex-1">
    <img src="./images/figure 3c.png" class="h-50 mx-auto" />
    <p class="text-center text-sm mt-1">Atmospheric river (ivt) RMSE — 25% better at short range, 10% at long range</p>
  </div>
  <div class="flex-1">
    <img src="./images/figure 3d.png" class="h-50 mx-auto" />
    <p class="text-center text-sm mt-1">Extreme heat precision-recall — GraphCast superior at 5- and 10-day lead times</p>
  </div>
</div>

### Atmospheric Rivers
- Responsible for **30%–65% of annual precipitation** on the U.S. West Coast
- Evaluated via **vertically integrated water vapor transport (ivt)** — derived from wind + humidity predictions

### Extreme Heat & Cold
- Binary classification: will temperature exceed the **98th percentile** of historical climatology?
- Assessed using **precision-recall curves** — appropriate for rare, imbalanced events

---
layout: default
transition: slide-left
---

# Training Data Recency Effect

Can GraphCast improve by training on more recent data?

<div class="flex justify-center mt-4">
  <img src="./images/figure 4.png" class="h-55" />
</div>

- Four variants trained 1979→2017, 2018, 2019, 2020 — all tested on **2021 data**
- More recent training data **consistently improves skill scores**
- Suggests the model captures **evolving weather patterns** — ENSO cycles, climate change effects

> Key advantage over static NWP: GraphCast can adapt to a changing climate through periodic re-training.
