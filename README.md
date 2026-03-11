# Indian Power Grid Analysis

Exploratory data analysis and demand modeling on India power grid generation and demand data (2021-2023). The workflow covers data consolidation, cleaning, feature engineering, and linear regression baselines with dimensionality reduction (SVD, PCA).

## Dataset
- Source: Mukherjee, Debanjan; Kalita, Karuna; Kumar, Subhash (2024), "Electricity Demand, Solar and Wind Generation Data (2021-2023) of India at 1-hour interval", Mendeley Data, V1, DOI: 10.17632/y58jknpgs8.1
- Raw files: monthly Excel sheets under `Datasets/` (Sep 2021 to Dec 2023).
- Combined CSV: `energy_combined.csv` (created by the conversion notebook).
- After cleaning/resampling: DatetimeIndex from 2021-09-01 00:00:00 to 2023-12-31 23:55:00 at 5-minute frequency (245,376 rows).

## Project Structure
- `Datasets/`: raw monthly Excel files.
- `Data Conversion to csv.ipynb`: merges Excel sheets into `energy_combined.csv`.
- `EDA.ipynb`: cleaning, EDA, feature engineering, and modeling.
- `energy_combined.csv`: full combined dataset (large).
- `sample_data.csv`: small sample for quick inspection.
- `pyproject.toml`, `uv.lock`: dependency and environment metadata.

## Data Preparation (EDA.ipynb)
- Parse `Date&Time`, sort, set as index.
- Resample to 5-minute intervals using mean.
- Merge `Solar+Wind` with `Wind+Solar`, then drop `Wind+Solar`.
- Drop duplicate solar column (`Solar`) after correlation check with clipped `ALL_IND_SOLAR|P`.
- Rename columns to snake_case.
- Clip negative values for `solar_mw` and `gas_mw` to zero.

Renamed columns:
| Original | New |
| --- | --- |
| `TOTAL_THM_ONLY|P` | `thermal_mw` |
| `TOTAL_NLDC_HYD|P` | `hydro_mw` |
| `TOTAL_NLDC_GAS|P` | `gas_mw` |
| `TOTAL_NLDC_NPC|P` | `nuclear_mw` |
| `ALL_INDIA_WIND|P` | `wind_mw` |
| `ALL_IND_SOLAR|P` | `solar_mw` |
| `Solar+Wind` | `solar_wind_mw` |
| `Total` | `total_generation_mw` |
| `NLDC_DEMAND|P` | `nldc_demand_mw` |
| `INTRA_SYSTEM_INTERTIA|P` | `intra_system_inertia` |
| `ER_Total_Inertia|P` | `er_total_inertia` |
| `SYS_IN|P` | `sys_in` |
| `SPARE 05` | `spare_05` |

## Feature Engineering
- Time features: `hour`, `day_of_week`, `month`.
- Peak-hour flag: `is_peak_hour` for 18:00-21:59.
- Season categories: summer (Mar-May), monsoon (Jun-Sep), post_monsoon (Oct-Nov), winter (Dec-Feb).
- One-hot encoding for seasons with monsoon as the baseline.
- Target: `nldc_demand_mw`.

Model features used:
- `thermal_mw`, `hydro_mw`, `gas_mw`, `nuclear_mw`, `wind_mw`, `solar_mw`
- `is_peak_hour`, `hour`, `month`, `day_of_week`
- `season_post_monsoon`, `season_summer`, `season_winter`

Train/test split: temporal 80/20 (first 80% train, last 20% test).

## Modeling Results (from EDA.ipynb)
- Linear Regression (baseline): Train R2 0.9920 (RMSE 1917.36), Test R2 0.9931 (RMSE 1824.24).
- Truncated SVD + Linear Regression (11 components): Test R2 0.9174 (RMSE 6306.89), Reconstruction MSE (scaled) 0.0346.
- PCA (95% variance, 11 components) + Linear Regression: Test R2 0.9174.

Notes:
- Results are taken from the notebook outputs and may change if the notebook is re-run.
- The dataset is large; consider using `sample_data.csv` for quick tests.

## Setup
- Python >= 3.14 (as specified in `pyproject.toml`).
- Install dependencies using one of the following:

```bash
uv sync
```

```bash
pip install .
```

Optional for Jupyter:
```bash
python -m ipykernel install --user --name 1stproject
```

## How to Run
1. Convert raw Excel files to CSV by running `Data Conversion to csv.ipynb` (writes `energy_combined.csv`).
2. Run analysis and modeling in `EDA.ipynb`.
