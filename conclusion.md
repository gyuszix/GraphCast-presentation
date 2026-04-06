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

# Headline Result: GraphCast Beats HRES on 90.3% of Targets

<div class="flex justify-center">
  <img src="./images/figure 2d.png" class="w-full max-h-56" />
</div>

<p class="text-center text-sm -mt-1 mb-1">RMSE skill scorecard — each cell is one variable × level × lead time. <strong>Blue = GraphCast wins</strong>, red = HRES wins.</p>

<div class="flex items-center gap-6 mt-1">
  <img src="./images/figure 2b.png" class="h-44 shrink-0" />
  <div class="text-sm">

  **z500** (geopotential at 500 hPa) — the headline variable for medium-range forecasting. GraphCast shows **7%–14% improvement** across all lead times. The **dashed vertical line** at 3.5 days marks where the HRES baseline switches from its short-range runs (06z/18z) to its full 10-day runs (00z/12z) — GraphCast wins on both sides.

  </div>
</div>

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

# Severe Weather Events: Cyclones

GraphCast was **never trained to track cyclones** — it only learned to predict raw atmospheric variables one step ahead. But a separate tracking algorithm applied to its outputs produces cyclone paths that are more accurate than HRES.

<div class="flex gap-4 mt-2">
  <div class="flex-1">
    <img src="./images/figure 3a.png" class="h-48 mx-auto" />
    <p class="text-center text-xs mt-1">Median track error (km) — lower is better</p>
  </div>
  <div class="flex-1">
    <img src="./images/figure 3b.png" class="h-48 mx-auto" />
    <p class="text-center text-xs mt-1">Paired difference — negative means GraphCast is closer</p>
  </div>
</div>

This is emergent behavior — the model learns enough about atmospheric dynamics from MSE on basic variables that complex downstream tasks just work. No task-specific fine-tuning, no cyclone labels in the training data.

---
layout: default
transition: fade-out
---

# Severe Weather Events: Rare Events

The same pattern holds for two more downstream tasks the model was never trained on:

<div class="flex gap-4 mt-1">
  <div class="flex-1">
    <img src="./images/figure 3c.png" class="h-40 mx-auto" />
    <p class="text-center text-xs mt-1">Predicting moisture transport (a derived quantity)</p>
  </div>
  <div class="flex-1">
    <img src="./images/figure 3d.png" class="h-40 mx-auto" />
    <p class="text-center text-xs mt-1">Detecting extreme heat (a rare-event classification task)</p>
  </div>
</div>

**Derived quantities** — moisture transport is computed from wind × humidity. GraphCast only predicts wind and humidity separately, yet the nonlinear combination is **25% more accurate** than HRES at short lead times.

**Rare event detection** — "will temperature exceed the 98th percentile?" is a heavily imbalanced binary classification problem. Evaluated with **precision-recall curves** (not accuracy — class imbalance makes accuracy meaningless). GraphCast is better at 5- and 10-day horizons.

> The takeaway: a model trained on simple per-variable MSE learns representations rich enough to generalize to complex, unseen tasks.

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
- Suggests the model captures **evolving weather patterns** — ocean-atmosphere cycles, shifting climate trends

> Key advantage over static NWP: GraphCast can adapt to a changing climate through periodic re-training.

---
layout: default
class: flex flex-col justify-center items-center
transition: fade-out
---

# Key Takeaways

**Training** — curriculum learning enables stable 3-day rollouts; careful loss weighting matters more than model size

**Verification** — rigorous fair comparison protocol; wins on 90.3% of 1,380 targets even after controlling for blurring

**Generalization** — emergent performance on cyclone tracking, moisture transport, and extreme heat detection without task-specific training

**What came next** — GenCast (2024) added probabilistic ensembles via diffusion; WeatherNext 2 (2025) scaled to 180M params with 8× faster inference

<div class="mt-8 text-2xl">

**Questions?**

</div>
