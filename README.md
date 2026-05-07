# LakeXG

**XGBoost-based chlorophyll-a prediction across ecologically diverse inland lakes, benchmarked against Sentinel-2 satellite retrievals.**

---

## Overview

Harmful algal blooms are a growing threat to inland water quality worldwide. Timely, accurate chlorophyll-a (Chl-a) estimation is essential for early warning — yet existing satellite retrieval algorithms, calibrated for open-ocean optics, routinely fail in optically complex lakes.

LakeXG addresses this by training per-lake XGBoost models on high-frequency in-situ sensor data from the [LakeBeD-US dataset](https://github.com/FLARE-forecast/LakeBeD-US), covering six ecologically distinct US lakes. Models are evaluated against in-situ measurements and, where available, against Sentinel-2 satellite Chl-a retrievals using the 2BDA band-ratio algorithm.

---

## Key Results

| Lake | Ecosystem | R² (in-situ) | RMSE (µg/L) | R² vs Satellite |
|------|-----------|:------------:|:-----------:|:---------------:|
| SUGG | Subtropical — Florida | **0.953** | 4.97 | −13027 |
| LIRO | Temperate — Wisconsin | 0.709 | 0.66 | N/A |
| CRAM | Temperate — Wisconsin | 0.616 | 0.55 | N/A |
| PRPO | Prairie — North Dakota | 0.608 | 7.63 | N/A |
| BARC | Subtropical — Florida | 0.547 | 0.36 | N/A |
| PRLA | Prairie — North Dakota | 0.445 | 7.53 | N/A |

**XGBoost generalizes across lake types. The satellite 2BDA algorithm fails for inland waters** (negative R², bias of +1.79 to +23.75 µg/L), confirming it is unsuitable for optically complex inland lakes without retraining.

---

## Workflow

![Workflow Diagram](final_workflow_diagram.pdf)

The pipeline proceeds in two tracks:

**In-situ track (all 6 lakes)**
1. Load LakeBeD-US high-frequency Parquet files (106M records, 21 lakes)
2. Filter to surface depth ≤ 0.5 m (to match satellite sensing depth)
3. Aggregate to daily averages per lake
4. Impute missing features with median; log1p-transform Chl-a
5. Engineer features: gap-protected lag-1 Chl-a, month, day-of-year
6. Chronological 80/20 train/test split
7. Train XGBoost with GridSearchCV + TimeSeriesSplit 5-fold CV
8. Back-transform predictions; evaluate R² and RMSE vs in-situ
9. SHAP TreeExplainer for feature importance

**Satellite track (SUGG, BARC, LIRO, CRAM)**
1. Load Sentinel-2 Chl-a retrievals (2BDA algorithm)
2. Filter invalid retrievals (negative Chl-a removed)
3. Match XGBoost predictions to satellite overpass days
4. Compare R², RMSE, and bias

---

## Top SHAP Features

Across all six lakes, the most influential predictors ranked by mean absolute SHAP value:

1. **fDOM** — fluorescent dissolved organic matter (proxy for organic carbon load)
2. **chla\_lag1** — previous day's Chl-a (strong temporal autocorrelation)
3. **day\_of\_year** — seasonal phenology signal
4. **temp** — water temperature drives algal growth rates

---

## Repository Structure

```
LakeXG/
├── project.ipynb              # Main analysis notebook
├── Dataset/
│   ├── LakeBeD-US-CSE/        # NOT included — download separately (see Data section)
│   ├── Chlorophyll_FCR.csv    # Sentinel-2 Chl-a for FCR
│   ├── Chlorophyll_SUGG.csv   # Sentinel-2 Chl-a for SUGG
│   ├── water_potability.csv   # Kaggle water potability (transfer learning pilot)
│   └── AQUAIR_*.csv           # AquAIR indoor air quality data
├── final_workflow_diagram.pdf
├── figures/                   # Output plots (R², SHAP, satellite bias, lake map)
├── source_model.pkl           # Serialized Random Forest (transfer learning pilot)
├── kaggle_imputer.pkl         # Saved median imputer
├── .gitignore
└── README.md
```

---

## Data

### LakeBeD-US (primary dataset)
The high-frequency sensor data is too large to host on GitHub (~106M records).

Download from: [https://github.com/FLARE-forecast/LakeBeD-US](https://github.com/FLARE-forecast/LakeBeD-US)

Place the extracted folder at: `Dataset/LakeBeD-US-CSE/`

### Sentinel-2 Satellite Data
Chl-a retrievals were obtained via the 2BDA algorithm applied to Sentinel-2 L2A tiles accessed through NASA Earthdata. Preprocessed CSV files for SUGG and FCR are included in `Dataset/`.

---

## Installation

```bash
git clone https://github.com/ShuvroSankar/LakeXG.git
cd LakeXG
pip install -r requirements.txt
```

**Core dependencies:**

```
pandas
numpy
xgboost
scikit-learn
shap
matplotlib
cartopy
tqdm
pyarrow
earthaccess
geopandas
```

---

## Usage

Open and run `project.ipynb` in order. The notebook is structured in sections:

- **Section 1** — Transfer learning pilot (water potability → AquAIR)
- **Section 2** — LakeBeD data loading and preprocessing
- **Section 3** — XGBoost training and in-situ evaluation (all 6 lakes)
- **Section 4** — Sentinel-2 satellite comparison (SUGG, BARC, LIRO, CRAM)
- **Section 5** — SHAP feature importance analysis
- **Section 6** — Study lake map visualization

> **Note:** The LakeBeD dataset must be downloaded separately (see Data section above) before running Sections 2–6.

---

## Citation

If you use LakeXG in your work, please cite:

```bibtex
@software{LakeXG2026,
  author  = {ShuvroSankar},
  title   = {LakeXG: XGBoost-Based Chlorophyll-a Prediction Across Ecologically Diverse Inland Lakes},
  year    = {2026},
  url     = {https://github.com/ShuvroSankar/LakeXG}
}
```

---

## License

MIT License. See `LICENSE` for details.
