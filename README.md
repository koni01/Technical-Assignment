# Sentinel-2 Mining Change Analysis

## Purpose

This repository contains a compact geospatial workflow for identifying and interpreting candidate land-cover changes at an open-pit mining site in Zambia. The observation dates are **12 August 2023** and **2 September 2023**, a **21-day interval**.

The final deliverable uses:

- **Squared Euclidean Spectral Distance (ESD)** to identify where the three visible Sentinel-2 bands changed; and
- **Random Forest** to classify each date as Vegetation, Water, or Bare ground / mine surface and describe the likely direction of change.

## Repository guide

| Location | Open this when you need | Contents |
|---|---|---|
| `README.md` | Project overview and run instructions | This guide |
| `report.md` | Final interpretation | Method, RF results, key candidate transitions, and limitations |
| `aoi.geojson` | AOI boundary | Project area of interest |
| `instructions.pdf` | Original assignment brief | Required workflow and deliverables |
| `random_forest.ipynb` | Reproduce the final Python analysis | Training, classification, transition calculation, tables, and maps |
| `change_visualization.ipynb` | View the original ESD visualisation workflow | Supporting ESD visualisation notebook |
| `processed/` | Original ArcGIS ESD raster outputs | `change_map.tif` and `change_binary.tif` |
| `data/sentinel2_20230812/` | First-date source imagery | Sentinel-2 B02, B03, B04 rasters |
| `data/sentinel2_20230902/` | Second-date source imagery | Sentinel-2 B02, B03, B04 rasters |
| `data/training_samples_2.*` | 12 August training data | Shapefile and supporting `.dbf`, `.shx`, `.prj`, `.cpg` files |
| `data/training_samples.*` | 2 September training data | Shapefile and supporting `.dbf`, `.shx`, `.prj`, `.cpg` files |
| `data/processed/esd/processed.gpkg` | Change polygons in GIS | SQLite-based GeoPackage; open the `change_feature` layer in ArcGIS Pro or QGIS |
| `data/processed/esd/change_visualization.png` | Original ESD result map | ESD candidate-change visualisation |
| `data/processed/composite/` | RF input composites | Three-band B02/B03/B04 composite rasters for both dates |
| `data/processed/random_forest/` | Final RF outputs | Classification rasters, transition raster, trained models, CSV tables, and presentation maps |

## Final method

### ESD screening

The B02, B03, and B04 rasters were checked for matching CRS, transform, dimensions, and 10 m resolution. They were stacked for each date. Squared Euclidean spectral distance was calculated as:

```text
D² = (B02_after - B02_before)²
   + (B03_after - B03_before)²
   + (B04_after - B04_before)²
```

The resulting change-intensity raster was thresholded, converted to polygons, and stored in `data/processed/esd/processed.gpkg`.

### Random Forest interpretation

Random Forest is the final interpretation method. Separate models were trained with polygons digitised on the matching acquisition date:

- `training_samples_2.shp` for 12 August 2023;
- `training_samples.shp` for 2 September 2023.

The three final classes are:

| Code | Class |
|---:|---|
| 1 | Vegetation |
| 2 | Water |
| 3 | Bare ground / mine surface |

Random Forest was selected because it handles non-linear RGB relationships and works with a modest set of manually digitised training polygons. A separate built-up class was not retained: at 10 m resolution with B02/B03/B04 only, built-up materials overlap spectrally with roads, stockpiles, exposed soil, and mine infrastructure. Keeping the class separate increased confusion. Combining these materials as `Bare ground / mine surface` produces a clearer and more stable mining-change interpretation.

## How to run the final workflow

1. Open `random_forest.ipynb` in Jupyter Notebook.
2. Select the `Python (s2geo)` kernel.
3. Run the notebook from top to bottom.
4. Check the two RGB/classification panels before using the transition results.
5. Open `data/processed/random_forest/rf_clean_class_transition_map.png` for the final presentation map.
6. Open `rf_esd_transition_summary.csv` for the transition-area table in hectares.

## Main outputs

| File | Purpose |
|---|---|
| `rf_class_20230812.tif` | Random Forest classification for 12 August |
| `rf_class_20230902.tif` | Random Forest classification for 2 September |
| `rf_transition_20230812_20230902.tif` | Pixel-level class-transition raster |
| `rf_esd_transition_summary.csv` | Candidate transition area inside the ESD change footprint |
| `rf_class_area_by_date.csv` | Total mapped area by class and date |
| `rf_transition_area_ha.csv` | Area of every RF class transition |
| `rf_clean_class_transition_map.png` | Final visual map of priority RF transition candidates |

## Important limitation

The 21-day observation interval is short. The results therefore describe short-term surface dynamics rather than a long-term mining-expansion trend. Transition labels are candidate interpretations, not confirmed clearing, water expansion, rehabilitation, or compliance findings. Validate priority areas using before/after RGB imagery, mine plans, operational records, higher-resolution imagery, or field observations.

## Related reference

Prasetya, K. D., and Tsai, F. (2025), *Deep Learning-Based Multitemporal Spatial Analytics for Assessing Reclamation Compliance of Coal Mining Permits in Kalimantan with Satellite Images*, provides relevant context for multi-temporal mine and reclamation monitoring. Its U-Net model, five-class scheme, permit-boundary analysis, and multi-year time series were not used in this assignment. The final workflow remains the three-class Random Forest and ESD approach described above.

https://doi.org/10.3390/rs17203477
