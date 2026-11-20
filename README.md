# Hong Kong Metered Parking — Data-Driven Pricing Policy Simulator (SEEM3650)

Evaluating the Hong Kong Transport Department's **28 September 2025** metered parking fee doubling (`HK$8/h -> HK$16/h`) across **~7.3 million daily occupancy records** (20,559 parking spaces across 18 districts, 1 Apr 2025 – 31 Mar 2026), merged with Hong Kong Observatory daily rainfall and public/CNY holiday calendars.

## Business Questions
1. Did doubling the flat hourly fee from `HK$8` to `HK$16` move metered parking occupancy toward a balanced **90% utilization target**?
2. Is demand price elasticity uniform across Hong Kong's 18 districts, or structurally heterogeneous?
3. Can any single city-wide flat rate achieve balanced utilization, or is a district-differentiated tariff required?

## Methodology
- **Data Engineering & Spatial Aggregation**: Cleaned ~7.3M space-day observations (`Normal` meter status only), mapped individual bays to Hong Kong's 18 administrative districts (`18 x 365 = 6,570` daily panel rows), and joined HKO daily rainfall (`Trace -> 0.025 mm`, `RainyDay`, `HeavyRain`) and Lunar New Year indicators.
- **Stage 1 — Predictive & Causal Modeling (`5-Fold Chronological Time-Series CV`)**:
  - Benchmarked Linear (OLS/Ridge/Lasso), Random Forest, and Gradient Boosting using strict date-grouped chronological splits to prevent regime-change data leakage.
  - **Random Forest** achieved **CV R² = 0.936** (RMSE `0.034`), paired with an auditable **OLS (HC3-robust SE)** model estimating the exact city-wide September 2025 regime shock.
- **Stage 2 — Counterfactual Simulation & Tariff Optimization**:
  - Recovered `HK$8/h` district counterfactual baselines via the trained Random Forest, applied regression-estimated district demand elasticities (`<= 0` clamped), and ran grid search over `HK$4–HK$16/h` to minimize `|E[Occupancy] - 0.90|`.
  - Evaluated sensitivity across `80%`, `85%`, and `90%` targets plus `±0.05` elasticity perturbation stress tests.

## Key Findings
- **Flat fee doubling backfired**: The `HK$8 -> HK$16` hike reduced city-wide occupancy by **2.23 pp**, pushing **15 of 18 districts** further below the 90% target because outer-suburb demand is far more price-elastic (`-0.11` to `-0.18`) than saturated CBD areas (`-0.02`).
- **District-Differentiated 3-Tier Pricing Scheme** (`HK$4–HK$16/h`, mean `HK$6.11/h`):
  - **Tier C (`HK$16/h`)**: Saturated urban core (Yau Tsim Mong).
  - **Tier B (`HK$9–11/h`)**: High-demand inner-urban districts (Sham Shui Po, Wong Tai Sin, Wan Chai).
  - **Tier A (`HK$4–8/h`)**: Remaining 14 districts where rate relief restores balanced turnover.
  - Brings **6 districts within ±1 pp** of the 90% target, with **12 of 18 district recommendations remaining identical** under `±0.05` elasticity shocks; highlights 6 structurally saturated/under-supplied districts where non-price supply interventions are needed alongside tariffs.

## Files & Data Sources
- [`SEEM3650_Project_Code.ipynb`](SEEM3650_Project_Code.ipynb): Full end-to-end Python pipeline, chronological CV benchmarks, statistical tables, counterfactual simulator, and rendered figures.
- **Public Raw Data Sources** *(excluded from Git due to multi-GB CSV size; configure path via `SEEM3650_DATA_DIR=./data`)*:
  - Transport Department metered parking occupancy & bay metadata (`data.gov.hk`)
  - Hong Kong Observatory daily rainfall (`data.gov.hk`)
  - Hong Kong General Holidays Ordinance (`elegislation.gov.hk`)
