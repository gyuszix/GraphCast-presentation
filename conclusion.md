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

### Key findings on z500 (geopotential at 500 hPa):
- **7%–14% RMSE skill score improvement** across all lead times
- Consistent advantage on ACC as well
- Results hold across the full scorecard of variables and pressure levels

### Skill Score Definition

$$\text{Skill Score} = \frac{\text{RMSE}_A - \text{RMSE}_B}{\text{RMSE}_B}$$

Negative = Model A is better. The scorecard visualizes this across all 1,380 targets — blue cells where GraphCast wins, red where HRES wins.

> The scorecard is overwhelmingly blue.

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

### Methodology
- Cyclone tracking algorithm applied to GraphCast's predicted fields (z, 10u/10v, u/v, msl)
- Based on **ECMWF's published tracking protocols**
- Compared against HRES operational tracks from **TIGGE archive**
- Ground truth: **IBTrACS** — independent reanalysis dataset of cyclone tracks

### Results (2018–2021)
- GraphCast has **lower median track error** than HRES
- **Significantly better** from 18 hours to 4.75 days lead time (paired analysis)
- Bootstrapped 95% confidence intervals confirm significance

---
layout: default
transition: fade-out
---

# Severe Events: Atmospheric Rivers & Extreme Temperatures

### Atmospheric Rivers
- Narrow atmospheric regions responsible for **30%–65% of annual precipitation** on the U.S. West Coast
- Characterized by **vertically integrated water vapor transport (ivt)** — derived from wind and humidity predictions
- GraphCast improves ivt prediction over HRES: **~25% at short lead times**, **~10% at longer horizons**
- Evaluated over coastal North America during cold months (Oct–Apr)

### Extreme Heat & Cold
- Evaluated as a **binary classification** problem: will temperature exceed the 98th percentile of historical climatology?
- Assessed using **precision-recall curves** and **SEDI scores** — appropriate for rare, imbalanced events
- GraphCast's precision-recall curves are **above HRES at 5- and 10-day lead times**
- HRES slightly better at 12-hour lead time (consistent with near-zero 2t skill score at short horizons)

---
layout: default
transition: slide-left
---

# Training Data Recency Effect

Can GraphCast improve by training on more recent data?

### Experiment
- Four model variants trained on data starting from 1979 but ending in **2017, 2018, 2019, and 2020**
- All evaluated on **2021 test data**

### Finding
- GraphCast:<2018 is already competitive with HRES
- Training up to 2020 (GraphCast:<2021) **further improves skill scores**
- Suggests the model captures **evolving weather patterns** — ENSO cycles, climate change effects
- Practical implication: periodic re-training with recent data is beneficial

> This is a key advantage over static NWP formulations — GraphCast can adapt to a changing climate through re-training.

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