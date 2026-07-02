# Agentic AI for Dynamic EV Charging Tariff Optimization

> A three-agent system that forecasts EV charging demand, recommends dynamic ₹/kWh tariffs, and monitors the outcomes in simulation — built for Society of Business Open Project 2026.

---

## Overview

Flat ₹/kWh tariffs ignore real-time demand — creating peak-hour congestion and idle off-peak chargers as EV adoption accelerates. This project builds a self-improving **Agentic AI pricing engine** made of three cooperating agents:

```
Demand Prediction Agent → Tariff Pricing Agent → Monitoring & Learning Agent
```

1. **Demand Prediction Agent** — forecasts charger utilization 1 hour ahead using historical session/interval data
2. **Tariff Pricing Agent** — maps predicted demand to a rule-based surge/standard/discount ₹/kWh tariff
3. **Monitoring & Learning Agent** — closes the loop by tracking simulated revenue, queues, and pricing efficiency across episodes

**Headline result:** under base-case demand elasticity, the dynamic pricing policy is associated with **~10.25% higher simulated revenue** vs. a fixed ₹15/kWh baseline, and a **~2.8-unit lower peak queue proxy** per episode.

> ⚠️ All revenue, queue, and elasticity figures are **simulated / proxy outcomes**, not results from a live A/B test. No causal claims are made beyond what the simulation supports.

---

## Objectives

Given real-time and historical charging session data, predict and optimize:

- **Demand Forecast** — how charging demand/utilization varies by time of day, day of week, and station
- **Dynamic Tariff** — the optimal ₹/kWh tariff to maximize revenue while minimizing congestion and wait times
- **Charger Utilization** — which stations are under- or over-utilized, and when
- **Congestion & Wait Time Reduction** — smoother demand distribution across time slots
- **Autonomous Pricing Intelligence** — a self-improving system refined through the Monitoring & Learning Agent's feedback loop

---

## Datasets

| Source | Coverage | Format | Used For |
|---|---|---|---|
| **[ACN-Data](https://ev.caltech.edu/dataset.html)** (Caltech/JPL) | 30k+ real charging sessions | JSON → CSV | Micro-behaviour: energy & dwell-time profiles |
| **[UrbanEV / ST-EVCDP](https://github.com/IntelligentSystemsLab/ST-EVCDP)** (Shenzhen) | 24,798 piles, 5-min interval matrices | CSV | Temporal & spatial demand modelling base |

### Repository Data Files

| File | Description |
|---|---|
| `acndata_sessions_json.xlsx` | Raw ACN-Data charging sessions (converted from JSON) |
| `stations.csv` | UrbanEV station metadata / capacity |
| `occupancy.csv` | 5-min occupancy matrix per pile |
| `duration.csv` | 5-min charging duration matrix |
| `price.csv` | 5-min tariff/price matrix |
| `volume.csv` | 5-min session-volume matrix |
| `adj.csv` | Station adjacency matrix |
| `distance.csv` | Inter-station distance matrix |
| `time.csv` | Time index reference for interval matrices |
| `information.csv` | Additional station/pile metadata |

**Scope:** 2.13M+ modelled interval-station records, time-ordered 80/20 train-test split (no leakage).

Dataset Link - https://drive.google.com/drive/folders/1eOFJTmyd2O3RlnpRdATJoRmCPfsz6nvf?usp=drive_link
---

## Methodology

### 1. Data Preprocessing Pipeline
```
Drop missing keys & duplicates → Parse timestamps, build unified index →
Clip negatives, forward-fill gaps → Melt matrices to long station-time format →
Engineer features & utilization rate
```
Engineered features: **Charger Utilization Rate**, **Occupancy Density**, **Queue-Length Proxy**, **Energy Delivered**, **Revenue per Session**. All preprocessing assumptions are written to a transparency log.

### 2. Exploratory Data Analysis
- Intraday utilization cycles (hour-of-day × weekday/weekend)
- Utilization variance across peak/shoulder/off-peak tariff periods
- Station-level utilization distribution
- ACN session-level dwell time & energy delivered profiles

### 3. Agent 1 — Demand Prediction
- **Target:** utilization rate, shifted −12 intervals (1 hour ahead)
- **Features:** 10 temporal + operational features
- **Split:** time-ordered 80/20 (no leakage)
- **Models compared:** Linear Regression, Random Forest, Gradient Boosting (XGBoost-equivalent)

### 4. Agent 2 — Tariff Pricing
Rule-based pricing engine mapping predicted utilization to tariff:

| Condition | Tariff | Formula |
|---|---|---|
| Utilization ≥ 80% | **Surge** | ₹15 × 1.30 = ₹19.50 |
| 30% – 80% | **Standard** | ₹15.00 (baseline) |
| Utilization ≤ 30% | **Discount** | ₹15 × 0.80 = ₹12.00 |

### 5. Agent 3 — Monitoring & Learning
Tracks simulated outcomes over 5 sequential episodes: average waiting-time reduction, customer response rate (demand-elasticity proxy), and pricing efficiency (₹/kWh delivered) — verifying the policy is stable and non-degrading.

---

## Key EDA Findings

- **Strong intraday cycle** — utilization dips mid-day and rises into evening; weekdays consistently run above weekends.
- **Bimodal station distribution** — a large cluster of fully-saturated stations alongside a long tail of under-used ones.
- **ACN sessions average ~9 kWh over ~5.9 hours** — long dwell time, modest energy draw, consistent with a workplace-charging pattern.
- **Spatial/geo view omitted** as a known data limitation — the station-coordinate merge (`stations.csv` keys) did not resolve. Temporal and station-level findings are unaffected.

---

## Model Performance

### Demand Prediction — 1-Hour-Ahead Utilization

| Model | RMSE | R² |
|---|---|---|
| **Gradient Boosting** ⭐ | **0.079** | **0.942** |
| Random Forest | 0.080 | 0.942 |
| Linear Regression | 0.296 | 0.201 |

Tree ensembles capture the temporal structure well; residuals are tight and centred around zero.

### Tariff Agent — Action Mix (Test Set)

| Action | Share |
|---|---|
| Surge | 67.6% |
| Standard | 17.5% |
| Discount | 14.9% |

> Surge fires frequently because predicted utilization runs high (mean ~77%) — see [Limitations](#-assumptions--limitations) below.

### Monitoring Agent — Simulated Outcomes (5 Episodes)

| Metric | Value |
|---|---|
| Avg peak queue-proxy units reduced/episode | ≈ 2.8 |
| Net session-volume shift (elasticity proxy) | −14.5% |
| Pricing efficiency | ₹19.3+/kWh (vs. ₹15 baseline) |

### Robustness — Revenue Sensitivity to Demand Elasticity

| Elasticity Scenario | Revenue Gain vs. ₹15/kWh Baseline |
|---|---|
| Rigid demand (users ignore price) | **+22.9%** |
| Base assumption (mid-elasticity) | **+10.25%** |
| Highly elastic (users react strongly) | **−8.75%** |

Pricing aggressiveness must be tuned to the *actual measured* elasticity of the user base — under high elasticity, surge pricing backfires and discounting dominates.

---

## Business, Operational & Policy Takeaways

| Lens | Takeaway |
|---|---|
| **Business** | Dynamic pricing is associated with ~10% higher simulated revenue vs. flat ₹15/kWh — a software-only margin lever requiring no new hardware. |
| **Operational** | Surge/discount signals are associated with a smoother load curve — lower peak queues and better use of idle off-peak capacity. |
| **Policy** | Transparent, rule-based tariffs (capped surge, guaranteed off-peak discount) keep pricing explainable and consumer-fair. |

**Recommended next step:** recalibrate the capacity baseline so the surge/standard/discount mix is balanced, then validate elasticity with a small live A/B pilot before full rollout.

---

## Assumptions & Limitations

**Key assumptions:**
- Charger draw fixed at 7.4 kW; baseline tariff ₹15/kWh
- Surge = +30% above 80% utilization; discount = −20% below 30% utilization
- Demand elasticity assumed (surge response −0.15, discount response +0.25), **not estimated from data**
- Forecast horizon fixed at 1 hour (12 × 5-minute intervals)
- Queue-length proxy derived where raw traffic-volume data was unavailable

**Limitations:**
- Mean utilization runs high (~77%) with off-peak ≈ peak — the capacity/occupancy ratio likely needs recalibration, which inflates surge frequency
- Station geo-merge unresolved → spatial/map view is empty
- Revenue and elasticity figures are simulated, not from live A/B tests — directional, not causal
- ACN and UrbanEV datasets are **not fused at session level**; insights are reported per-source

---

## Repository Structure

```
ev-charging-tariff-optimization/
├── data/
│   └── raw/                                  # Place all CSV/XLSX source files here (not tracked)
│       ├── acndata_sessions_json.xlsx
│       ├── stations.csv
│       ├── occupancy.csv
│       ├── duration.csv
│       ├── price.csv
│       ├── volume.csv
│       ├── adj.csv
│       ├── distance.csv
│       ├── time.csv
│       └── information.csv
├── notebooks/
│   └── ev_tariff_optimization.ipynb          # Full pipeline: preprocessing → EDA → 3 agents
├── docs/
│   ├── project_brief.pdf                     # Open Project 2026 problem statement
│   └── presentation_deck.pdf                 # Summary presentation
├── requirements.txt
├── .gitignore
├── LICENSE
└── README.md
```

---

## Tech Stack

- **Language:** Python 3.10+
- **Data Handling:** pandas, numpy, openpyxl
- **Visualization:** matplotlib, seaborn
- **Modeling:** scikit-learn (Linear Regression, Random Forest, Gradient Boosting)
- **Environment:** Jupyter Notebook

---

## Future Work

- Recalibrate the utilization/capacity baseline to rebalance the surge/standard/discount action mix
- Fuse ACN-Data and UrbanEV at a common session/station granularity
- Resolve the `stations.csv` coordinate merge to restore spatial/geo analysis
- Estimate demand elasticity empirically instead of assuming fixed values
- Validate the pricing policy with a small live A/B pilot before full rollout

---

## 👤 Author

**Akanksha Sahni**

---
