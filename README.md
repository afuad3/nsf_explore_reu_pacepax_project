# HARP2 vs. AirHARP2 Polarimetric Comparison (PACE-PAX, 9/26/2024)

Validation of NASA's PACE/HARP2 satellite polarimeter against the airborne AirHARP2 instrument, using coincident measurements collected during the PACE Postlaunch Airborne eXperiment (PACE-PAX) campaign on September 26, 2024.

Completed as part of the **NSF REU EXPLORE** program, under the mentorship of **Dr. McBride**, using instrument data from **ESI** (developers of HARP2 and AirHARP2).

## Motivation

HARP2, flying on NASA's PACE satellite, measures polarized reflectance (radiance, DoLP) across multiple viewing angles and spectral bands to study aerosols and clouds. AirHARP2 is its airborne counterpart, flown on the ER-2 aircraft during PACE-PAX to provide higher-resolution, near-coincident validation data. This project checks how well the two instruments agree — first visually (mapped DoLP), then quantitatively (per-pixel angular curves), and finally after correcting for their very different spatial resolutions (via spatial averaging of AirHARP2 into HARP2-sized "superpixels").

## Workflow

Notebooks are organized in the order they were developed and are meant to be run in this sequence:

1. **`PACE_926.ipynb`** — Downloads and reads a HARP2 L1C granule over the campaign's target site (via `earthaccess` + `nasa-pace-data-reader`). Locates the pixel nearest the site, plots per-pixel radiometric/polarimetric variables vs. view and scattering angle, and maps Degree of Linear Polarization (DoLP) by band (Blue/Green/Red/NIR) in two zoom levels.

2. **`ER2F1_926.ipynb`** and **`ER2F2_926.ipynb`** — Load and process two AirHARP2 flight-line files from the ER-2 (20:29 and 20:34 UTC passes). Reads raw HDF5 fields directly, manually applies the DoLP scale factor and fill-value mask used internally by `nasa-pace-data-reader`, locates the nearest pixel to the target site, and produces the same per-band DoLP maps as the HARP2 notebook for visual comparison.

3. **`Combined926.ipynb`** — Loads the HARP2 granule and both AirHARP2 flight lines together and overlays their DoLP maps on a shared basemap, one figure per AirHARP2 pass, to visually inspect spatial agreement between the satellite and airborne footprints.

4. **`overplot_926.ipynb`** — Performs a quantitative, single-pixel comparison: for a manually matched HARP2/AirHARP2 pixel pair, overlays reflectance (R_i, R_q, R_u) and DoLP curves as a function of view angle and scattering angle, across all four bands. Computes relative RMSE and Spearman correlation between the two instruments' curves and summarizes agreement in a metric heatmap.

5. **`superpix926.ipynb`** — Extends the comparison by averaging a window of AirHARP2 pixels ("superpixel") sized to approximate HARP2's ~5 km footprint, using a haversine-distance-based pixel spacing calculation. Repeats the view-angle/scattering-angle curve comparison and RMSE/Spearman analysis against this superpixel-averaged AirHARP2 signal to test whether resolution-matching improves agreement with HARP2.

## Data

- **HARP2 (PACE):** `PACE_HARP2.20240926T203011.L1C.V3.5km.nc`, downloaded via `earthaccess` from NASA Earthdata.
- **AirHARP2 (ER-2):** `PACEPAX-AH2MAP-L1C_ER2_20240926T202945_R0.nc` and `..._20240926T203445_R0.nc`, ER-2 airborne passes near-coincident with the HARP2 overpass.

Both instruments report intensity (i), Stokes parameters (q, u), and DoLP across multiple view angles per spectral band. Data access requires a NASA Earthdata login for `earthaccess`; the AirHARP2 files are PACE-PAX campaign products.

## Setup

```bash
pip install earthaccess nasa-pace-data-reader cartopy h5py xarray scipy matplotlib numpy
```

An Earthdata account (for `earthaccess.login()`) is required to pull the HARP2 granule. AirHARP2 files are expected locally under `ER2/`.

## Key Results

- Visual DoLP maps show broadly consistent spatial polarization patterns between HARP2 and AirHARP2 across all four bands.
- Single-pixel angular-curve comparisons (`overplot_926`) show good qualitative agreement in shape but quantify a resolution-driven mismatch, since AirHARP2's native pixels are far finer than HARP2's ~5 km footprint.
- Superpixel-averaging AirHARP2 to HARP2's resolution (`superpix926`) improves the agreement metrics (relative RMSE, Spearman correlation) versus the raw single-pixel comparison, supporting the idea that footprint-matching is necessary for a fair cross-instrument validation.

## Repo Structure

```
.
├── PACE_926.ipynb       # 1. Load & visualize HARP2 (satellite)
├── ER2F1_926.ipynb      # 2. Load & visualize AirHARP2 pass 1 (20:29 UTC)
├── ER2F2_926.ipynb      # 3. Load & visualize AirHARP2 pass 2 (20:34 UTC)
├── Combined926.ipynb    # 4. Overlay HARP2 + AirHARP2 DoLP maps
├── overplot_926.ipynb   # 5. Single-pixel HARP2 vs AirHARP2 curve comparison
├── superpix926.ipynb    # 6. Superpixel-averaged comparison
└── README.md
```

## Acknowledgments

This work was conducted as part of the **NSF REU EXPLORE** program, under the mentorship of **Dr. McBride**. HARP2 and AirHARP2 instruments and data products are developed by **ESI**, in support of NASA's PACE mission and the PACE-PAX airborne validation campaign.
