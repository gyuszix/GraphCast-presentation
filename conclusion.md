---
layout: default
transition: fade-out
---

# Verification Methods: How Do We Know It's Good?

GraphCast is evaluated against **HRES** across a massive verification space:

- **69 variable-level combinations** × **20 lead times** (12h to 10 days) = **1,380 verification targets**
- Two primary metrics: **RMSE** (root mean square error) and **ACC** (anomaly correlation coefficient)

### Fair Comparison Protocol

- GraphCast evaluated against **ERA5** ground truth (what it was trained on)
- HRES evaluated against **HRES-fc0** ground truth (its own initial conditions)
- Only evaluated at initialization times where both systems have the **same ±3h observational lookahead** — prevents either model from having privileged future information
- Total precipitation excluded due to known biases in ERA5 precipitation data

---
layout: two-cols
transition: slide-left
---

# RMSE & ACC: The Core Metrics

### RMSE (Root Mean Square Error)

Measures average magnitude of forecast errors:

$$\text{RMSE} = \sqrt{\frac{1}{N}\sum_{i=1}^{N}(\hat{y}_i - y_i)^2}$$

Lower is better. Area-weighted by grid cell size to account for latitude distortion.

::right::

<div class="ml-4 mt-12">

### ACC (Anomaly Correlation Coefficient)

Measures how well the forecast captures **deviations from climatology**:

$$\text{ACC} = \frac{\sum (f_i - c_i)(a_i - c_i)}{\sqrt{\sum (f_i - c_i)^2 \sum (a_i - c_i)^2}}$$

Where $f$ = forecast, $a$ = actual, $c$ = climatology.

Higher is better (1.0 = perfect). Crucial because simply predicting the average climate would score low.

</div>

---
layout: default
transition: fade-out
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
transition: slide-left
---

# Disaggregated & Spectral Analysis

The authors didn't just report global averages — they stress-tested the results:

### Regional Breakdowns
- Performance evaluated across **Northern Hemisphere, Tropics, Europe**, and by latitude
- RMSE mapped by **latitude × pressure level** — important for understanding tropopause/stratosphere behavior
- Results **generally hold across all regions**

### Spectral Decomposition
- MSE decomposed across **spherical harmonic wavelengths** — large-scale vs. fine-scale patterns
- RMSE evaluated at different **horizontal resolutions** (truncating at different wavenumbers)
- GraphCast maintains advantage across spatial scales

### Bias Analysis
- Per-location **Mean Bias Error (MBE)** mapped geographically
- Bias correlation measured between GraphCast and HRES — do they make similar systematic errors?

---
layout: default
transition: fade-out
---

# The Blurring Problem & Optimal Filtering

A critical question: **Is GraphCast only winning because it blurs its forecasts?**

MSE-trained models naturally learn to smooth predictions at longer lead times to minimize error under uncertainty. HRES, based on physical equations, does not blur.

### The Test
- Applied **optimal linear isotropic filters** to both GraphCast and HRES forecasts
- This gives HRES the same statistical blurring advantage
- Result: optimally blurred GraphCast still beats optimally blurred HRES on **88.0%** of 1,380 targets

> GraphCast's advantage is real — not just a blurring artifact. Though down from 90.3% to 88.0%, the conclusion holds.

This is important because blurring can limit practical value — a forecast that says "it'll rain somewhere in this region" is less useful than a sharp prediction of where exactly.

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

---
layout: default
transition: fade-out
---

# Limitations & What GraphCast Is Not

### Uncertainty Handling
- GraphCast produces **deterministic forecasts** — one prediction, not a distribution
- ECMWF's ensemble system (**ENS**) generates multiple stochastic forecasts for uncertainty
- GraphCast expresses uncertainty through **spatial blurring**, not explicit probability distributions
- Building models that handle uncertainty more explicitly is a crucial next step

### Resolution & Scale Constraints
- Operates at **0.25°, 37 levels, 6h steps** (limited by ERA5's native resolution)
- HRES runs at **0.1°, 137 levels, up to 1h steps**
- Only **36.7M parameters** — small by modern ML standards, constrained by hardware memory

### Not a Replacement
- MLWP depends critically on **NWP-generated reanalysis data** like ERA5
- Should be viewed as a **complement** to traditional methods, not a successor

---
layout: default
transition: slide-left
---

# Conclusions: Why This Paper Matters

### For Weather Forecasting
- First ML model to **decisively surpass** the best operational NWP system across a broad evaluation
- Demonstrates that **learned simulators** can compete with decades of physics-based modeling
- 10-day global forecasts in **< 1 minute on a single TPU** vs. **~1 hour on 11,664 cores**

### For the Field of Deep Learning
- Validates **graph neural networks** for complex physical simulation at scale
- Encode-Process-Decode architecture with multi-mesh is a powerful paradigm for spatiotemporal problems
- Autoregressive curriculum training enables stable long-horizon rollouts

### Beyond Weather
- Opens directions for **climate, ecology, energy, agriculture**, and other geo-spatiotemporal forecasting
- The principle — train learned simulators on rich real-world data — generalizes to many complex dynamical systems

> *"We believe this marks a turning point in weather forecasting."*
