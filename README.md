Fire-Season Shift in Central Brazil
Decadal analysis of the fire-season peak displacement across latitudinal bands of Central
Brazil (Cerrado + Pantanal + transition zones), with a formal test of robustness against the
El Niño–Southern Oscillation (ENSO).
The central question: over 1985–2024, has the peak of the burning season shifted to a later
point in the year, and is that shift a genuine secular (climatic) trend rather than an artefact of
El Niño events being unevenly distributed across decades?
---
Key result
For natural vegetation, the burning-window peak shifts significantly later in the central
latitudinal bands (≈5–7 days/decade), with the signal strengthening southward. The shift is:
absent in anthropic-use fire (which follows a management calendar, not climate);
predominantly a translation of the season, not an asymmetric broadening (BWS moves while
Extent stays largely stable);
robust to ENSO — it survives controlling for ONI phase, and where ENSO does modulate fire
(the northernmost band, Amazonian arc) there is no secular trend.
This produces a clean spatial dissociation between the secular trend (central bands) and ENSO
variability (northern band).
> Numbers above are reproduced by the notebook from the included data and may be refined as the
> manuscript is finalised.
---
Repository structure
```
fire-season-shift-central-brazil/
├── notebooks/
│   └── fire_season_analysis.ipynb   # main analysis, runs end-to-end
├── data/
│   ├── burn_monthly_by_latband_new.csv   # aggregated burned area (month × band × type)
│   └── ElNino.txt                        # NOAA ONI index (1950–2026)
├── docs/
│   └── METHODOLOGY.md               # full methodological rationale and limitations
├── outputs/                         # tables and figures produced by the notebook
├── requirements.txt
├── LICENSE
└── README.md
```
---
How to run
Google Colab (recommended)
Upload this repository to your Google Drive (or clone it).
Open `notebooks/fire_season_analysis.ipynb` in Colab.
In Section 0, uncomment the `pip install pymannkendall` line on first run.
In Section 1 and Section 6, set `CSV_PATH` and `ONI_PATH` to your Drive locations (look for the
`>>> ADJUST THE PATH <<<` markers).
Run all cells.
Local
```bash
pip install -r requirements.txt
jupyter notebook notebooks/fire_season_analysis.ipynb
```
With the default relative paths (`data/...`), launch Jupyter from the repository root.
---
Data provenance
Burned area: MapBiomas Fire Collection 4, monthly product, separated per-pixel into natural
vegetation vs. anthropic use against each year's MapBiomas LULC (Collection 9), clipped to the
Central Brazil ROI and reprojected to Albers equal-area (EPSG:5880). The CSV here is the
band-aggregated product of that pipeline (not the raw rasters).
ENSO: NOAA Oceanic Niño Index (ONI), 3-month running means of the Niño 3.4 SST anomaly.
The upstream geoprocessing (raster reclassification and tabulation) is documented in
`docs/METHODOLOGY.md`.
---
Method in one paragraph
The fire season of each year/band/type is summarised with circular statistics: the burned
fraction across the 12 months is treated as vectors on a circle, yielding the BWS (weighted
mean month = peak timing) and Extent (1 − R = season spread). The displacement is measured as
the Theil–Sen slope (days/decade) with Mann–Kendall significance over the full 40-year
series — deliberately not a difference between two selected decades, to avoid a multiple-
comparisons bias. ENSO robustness is tested by correlating BWS with each ONI trimester (JJA/JAS/
ASO/SON) individually, and by checking that the BWS trend persists in the residual after removing
the ONI effect.
---
Status
Analysis code accompanying a manuscript in preparation. Repository currently private pending
submission.
Authors
Mario Barroso · with Gabriel Hofmann (Cerrado climate change analysis).
License
See LICENSE.
