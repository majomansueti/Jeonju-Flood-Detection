# Jeonju Flood Detection

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/majomansueti/Jeonju-Flood-Detection/blob/main/Jeonju_Flood_Map.ipynb)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/)

Low-cost urban flood detection pipeline for mid-sized Korean cities using
Sentinel-1 SAR, NASA IMERG rainfall, and Google Earth Engine.
Validated on the July 2023 Jeonju flood event.

## Results

Trained on stratified samples of 1500 flood + 1500 non-flood pixels:

| Metric | Value |
|---|---|
| F1 score (optimal) | **0.69** |
| Precision | 0.55 |
| Recall | 0.91 |
| Probability threshold | 0.30 |

Slope-stratified API alert thresholds:

| Slope class | Yellow API (mm) | Red API (mm) |
|---|---|---|
| 0–5° (flat) | **14.7** | **30.4** |
| 5–15° (gentle) | 25.9 | 36.6 |
| 15–30° (moderate) | 25.9* | 36.6* |
| ≥30° (steep) | insufficient samples | — |

*neighbor-borrowed*

## How to run

**Option 1 — Colab (recommended)**

Click the **Open in Colab** badge above. Replace `EE_PROJECT_ID` in Cell 1 with your
Earth Engine project ID, then **Runtime → Run all**. Total runtime ≈ 8 minutes.

**Option 2 — local**

```bash
git clone https://github.com/majomansueti/Jeonju-Flood-Detection.git
cd Jeonju-Flood-Detection
pip install -r requirements.txt
jupyter notebook Jeonju_Flood_Map.ipynb
```

You need a free [Google Earth Engine](https://earthengine.google.com/) account.

## What the pipeline does

1. **Load features** — IMERG V07 rainfall (6h, 24h, 72h accumulations + weighted API),
   SRTM 30 m slope and aspect, Sentinel-1 ΔdB
2. **Label** — pixels where ΔdB < −1.5 dB are tagged as flooded
3. **Sample** — stratified 1500 + 1500 balanced pixels
4. **Train** — XGBoost classifier on auxiliary features (API, rainfall, terrain)
5. **Evaluate** — pixel-level precision / recall / F1 across probability thresholds
6. **Derive thresholds** — slope-stratified API thresholds via logistic regression

## Data sources

All datasets are public and accessed through Google Earth Engine — no external downloads required.

| Dataset | GEE Asset ID | Provider |
|---|---|---|
| Sentinel-1 GRD | `COPERNICUS/S1_GRD` | ESA Copernicus |
| IMERG V07 | `NASA/GPM_L3/IMERG_V07` | NASA GPM |
| SRTM 30 m DEM | `USGS/SRTMGL1_003` | USGS / NASA |
| Heritage sites | `data/jeonju_heritage_points.csv` | Cultural Heritage Administration (CHA) |

## Repository structure

```
.
├── Jeonju_Flood_Map.ipynb        # main notebook — run this
├── requirements.txt              # pinned Python dependencies
├── LICENSE                       # MIT
├── data/
│   ├── jeonju_heritage_points.csv     # 10 cultural heritage sites
│   ├── pixel_metrics.csv               # threshold sweep results
│   └── policy_thresholds.csv           # slope-stratified API thresholds
└── figures/                      # output map screenshots
```

## Methodology notes

The XGBoost classifier is trained on **SAR-derived labels**
(`ΔdB < −1.5 dB`) using rainfall + terrain auxiliary features. This is a
self-supervised approach — the model learns to predict the per-pixel SAR
flood signature from features available before SAR acquisition. The
reported F1 reflects how well rainfall and terrain features can reproduce
the SAR flood mask.

For applications requiring independent ground-truth validation
(e.g. against field observations or official flood-trace records),
the pipeline outputs (`risk_xgb`, `flood_polygons`) should be
cross-referenced against authoritative sources.

## Citation

If you use this code, please cite:

> López Mansueti, M. J. (2026). *Jeonju Flood Detection — Sentinel-1 SAR
> and Google Earth Engine pipeline.* GitHub repository.
> https://github.com/majomansueti/Jeonju-Flood-Detection

## License

MIT — see [LICENSE](LICENSE).

## Contact

María José López Mansueti — majolopezmansueti@gmail.com
