# Methodology

Full methodological rationale for the fire-season displacement analysis in Central Brazil. This
document records not only *what* was done but *why* each design decision was made, including the
alternatives that were considered and rejected, so the analysis is reproducible and defensible.

---

## 1. Study question and framing

We test whether the **peak of the fire season** has shifted to a **later** point in the year over
1985–2024 in Central Brazil, whether that shift differs between **natural vegetation** and
**anthropic use**, how it varies **with latitude**, and whether it is a **secular climatic trend**
or an artefact of **ENSO** variability.

The work sits alongside a broader investigation (with G. Hofmann) into a possible **widening and
late-shifting of the dry season** in the Cerrado, probed through several independent proxies (fire
peak, maximum NDVI drop, river-flow minima). This repository covers the **fire** proxy.

A deliberate framing choice: the analysis is organised by **latitudinal bands and decades**, not by
biome and not by calendar year. Latitude is the axis along which a large-scale climatic forcing is
expected to act coherently, and the decadal lens is a low-pass filter that isolates the secular
component from interannual noise.

---

## 2. Study area (ROI)

A **rectangular domain over Central Brazil** is used as the geometric reference frame, rather than a
biome mask. Rationale:

- A climatic forcing of broad spatial extent does not respect biome boundaries; a biome mask imposes
  a contour the process does not see.
- The centroid and latitudinal-profile statistics are sensitive to **domain shape**. A biome mask
  (narrow and dispersed in the south, broad in the centre, tapering north) would bias any
  displacement metric through the silhouette of the container. A rectangle holds shape constant.
- Bordering portions of adjacent biomes are included deliberately, so that any **leakage** of
  fire-prone conditions beyond the Cerrado envelope (into transition zones north and south) is
  observable rather than masked.

Biome and land-use class are retained as **per-pixel attributes**, not as the clipping boundary, so
that the contribution of each can be decomposed after the fact without re-running the spatial
processing.

The northern hemisphere portion (e.g. Roraima, with its counter-seasonal fire regime) is **excluded
by design**: this study concerns the south→north gradient of Central Brazil. The southwestern
Pantanal/Chiquitania cluster (a distinct wetland regime) is likewise treated as outside the target
process.

The ROI is projected to **Albers equal-area (EPSG:5880)** so that burned areas can be summed and
compared across latitudes without projection-induced area distortion.

---

## 3. Source data and upstream processing

### 3.1 Burned area
- **MapBiomas Fire Collection 4**, monthly burned-area product, 1985–2024.

### 3.2 Land use / land cover
- **MapBiomas LULC Collection 9**, reclassified each year into a binary mask:
  natural vegetation (1) vs. everything else (0).
  Natural-vegetation classes: `{3, 4, 5, 6, 11, 12, 13, 49, 50}`.

### 3.3 Per-pixel natural/use separation
For every year, the monthly burned raster is split into `burn_nat` and `burn_uso` by intersecting it
with **that same year's** natural-vegetation mask. The natural/use frontier is therefore recomputed
annually — a pixel that was natural vegetation in 1990 and pasture in 2015 is attributed correctly
in each year. This avoids assigning to the natural class any fire that occurred on already-converted
land.

Processing is windowed (block-wise with `rasterio`) to handle the full-resolution national rasters;
where grids differ, the LULC mask is warped to the burned-area grid via nearest-neighbour
(`WarpedVRT`) to preserve class integrity.

### 3.4 Tabulation
Burned area is summed by **month × 5° latitudinal band × type (nat/use)** within the ROI, and joined
with the **available area** of each class per band per year. The result is
`data/burn_monthly_by_latband_new.csv`, the input to this repository's notebook.

---

## 4. Normalisation

The analysis variable is the **burned fraction**, not raw burned area:

```
burn_rate = burned_area_km2 / available_area_km2   (per band, per year, per class)
```

The denominator (`uso_km2 + nat_km2`) changes through time — converted area grows, natural area
shrinks. Without normalisation, a rising trend in use fire could be a mere consequence of more
converted area existing, mimicking a regime change that is not there; for natural fire, the
shrinking denominator creates the mirror problem. Normalisation separates **frontier change** from
**regime change**.

Because the displacement metric targets the **timing** of the peak rather than its magnitude,
normalisation is expected to affect amplitude more than timing — but this is verified, not assumed.

---

## 5. Circular metrics

The fire season is cyclic (December and January are adjacent), so linear monthly averages are
inappropriate. Each month *m* is mapped to an angle θ = 2π(m−1)/12, and the monthly burned fractions
are treated as vector magnitudes.

### 5.1 BWS — Burning Window Shift
The **weighted mean month**, i.e. the angular mean of the monthly distribution converted back to a
1–12 month scale. It is the **timing of the season's centre of mass** (the peak). A larger BWS means
a later peak.

### 5.2 Extent (1 − R)
`R` is the length of the resultant vector (0–1): R≈1 = burning concentrated in one month (short,
peaked season); R≈0 = burning spread across the year (long, diffuse season). **Extent = 1 − R**
increases with dispersion.

### 5.3 Why both
BWS alone is **structurally ambiguous**: an increase can mean either the peak genuinely moved
(translation) or the season grew a late tail that drags the mean while the peak stays put
(asymmetric broadening). The two are physically different. Reading BWS jointly with Extent
disambiguates them (Section 7).

This is the temporal analogue of the spatial centroid-plus-dispersion approach used elsewhere in the
group's fire-regime work: BWS is the temporal centroid, Extent its dispersion.

---

## 6. Displacement measure (main result)

The official displacement measure is the **trend over the full 40-year series**, by band × type:

- **Theil–Sen slope** — the median of pairwise slopes; robust to outliers. Reported in
  **days/decade** (slope in month/year × 10 × 30.4). El Niño years, the natural outliers, do not
  distort it as they would an ordinary-least-squares fit.
- **Mann–Kendall test** — non-parametric significance of a monotonic trend; the standard in
  climatology and hydrology; makes no linearity assumption.

### 6.1 Why not a decade-to-decade delta
Comparing a "base" decade with a "recent" decade requires choosing which decades. Testing several
intervals and keeping those that reach significance is a **multiple-comparisons / selection bias**
that inflates confidence even when the underlying signal is real, and invites the reviewer question
"why these decades?". The full-series trend removes the choice entirely: the reported number is a
property of all 40 years.

A fixed end-to-end delta (first decade 1985–1994 vs. last decade 2015–2024, **fixed a priori**) is
retained **only as an illustrative figure**, never as the headline number.

---

## 7. Checks

1. **Translation vs. broadening** — the Extent trend is tested with the same Theil–Sen/Mann–Kendall
   machinery. Where BWS is significant but Extent shows no trend, the displacement is **translation**
   (the whole season moved). Where both rise, there is a **broadening** component to report rather
   than smooth over.
2. **Completeness** — every year × band × type combination is checked for presence; gaps bias
   trends.
3. **Illustrative decadal delta** — fixed end decades, for visualisation only.

---

## 8. ENSO robustness

The decisive validity test for the main result.

### 8.1 The threat
If El Niño years tend to have a later fire peak, and El Niño events are concentrated in recent
decades (e.g. the strong 2015–16 and 2023–24 events), then part of the measured BWS trend could be
**sample composition** rather than a secular climatic signal. A reviewer will ask exactly this.

### 8.2 ENSO classifier
The **NOAA ONI** (3-month running mean of the Niño 3.4 SST anomaly). Rather than collapsing the dry
season into a single mean ONI value, the four trimesters spanning ~80 % of the Central Brazil fire
season — **JJA, JAS, ASO, SON** — are treated **individually**. This preserves information about
*when* within the season the ENSO state matters (an early- vs. late-season effect), and avoids
washing out a timing signal by averaging. Because the four trimesters are strongly autocorrelated,
they are tested **one at a time** (not jointly in one model), which sidesteps multicollinearity and
keeps the result legible.

### 8.3 Tests
1. **Correlation** — Spearman ρ between the annual BWS series and each ONI trimester, per band, for
   natural vegetation. Where a trend exists, is there also ENSO correlation? If not, the trend is
   ENSO-independent.
2. **Residual trend** — the ONI effect is regressed out of BWS and the Mann–Kendall trend is
   recomputed on the residual. If the residual trend stays significant where the raw trend was
   significant, the displacement does not depend on ENSO.

### 8.4 Autocorrelation caveat
The Spearman test does not correct for temporal autocorrelation, which inflates significance. In the
present results the robustness correlations are **non-significant**, which *supports* the
independence conclusion, so autocorrelation does not change the reading. **If** a future analysis
yields a significant ENSO correlation, its p-value must be recomputed with an effective
degrees-of-freedom correction (e.g. the modified Mann–Kendall for autocorrelated series,
`pymannkendall.hamed_rao_modification_test`). This is flagged so it is not overlooked.

---

## 9. Interpretation framework

- A significant, ENSO-robust **later** shift in **natural** fire, strengthening southward, supports
  a secular dry-season change consistent with the broader (NDVI, river-flow) lines of evidence.
- Absence of the shift in **use** fire is expected: anthropic burning follows a management calendar
  weakly coupled to the climatic forcing.
- ENSO modulation confined to the **northern** band (Amazonian arc), where there is no secular
  trend, yields a spatial dissociation between secular trend and interannual ENSO variability —
  strengthening rather than weakening the climatic interpretation.

---

## 10. Known limitations

- **Band coarseness.** 5° bands contain real internal heterogeneity. Adequate for this 1-D temporal
  analysis; a finer spatial treatment is left to subsequent (geometric) phases.
- **Aggregated input.** This repository works from the band-aggregated CSV, not the raw rasters;
  the upstream pipeline (Section 3) is documented but its rasters are external.
- **Single fire product.** Results are conditional on MapBiomas Fire Collection 4; cross-validation
  against an independent burned-area product is a possible extension.
- **ONI as the sole ENSO descriptor.** Other indices (e.g. SOI, ONI flavours by region) could be
  tested; ONI was chosen as the operational standard with native trimestral structure.

---

## 11. Reproducibility

The notebook `notebooks/fire_season_analysis.ipynb` runs end-to-end from the two files in `data/`
and writes all tables and the main figure to `outputs/`. Library versions are pinned in
`requirements.txt`. No step depends on hidden state or manual intervention.
