# Random Forest Land-Cover Transition Report

## Purpose

This report interprets candidate land-cover transitions between **12 August 2023** and **2 September 2023** at an open-pit mining site in Zambia. The observation interval is **21 days**. The focus is the final Random Forest result, while ESD is used only as a supporting spatial screen to restrict interpretation to locations where visible-band change was detected.

![Random Forest transition candidates inside the ESD change footprint](data/processed/random_forest/rf_clean_class_transition_map.png)

## Method

### Random Forest classification

Two Random Forest models were trained from manually digitised polygons on their matching image dates:

- 12 August 2023: `training_samples_2.shp`
- 2 September 2023: `training_samples.shp`

Both models used the B02, B03, and B04 Sentinel-2 bands and the same three classes:

1. Vegetation
2. Water
3. Bare ground / mine surface

Random Forest was selected because it can model non-linear RGB relationships with a modest sample set. It is appropriate for this assignment because the available labelled polygons are sufficient for a three-class model, while a deep-learning workflow would require substantially more representative training data.

Built-up cover was not kept as a fourth class. In 10 m visible-band imagery, built-up surfaces overlap with roads, exposed soil, stockpiles, and mine infrastructure. Keeping it separate increased confusion in the classification. Combining these surfaces as `Bare ground / mine surface` produces a clearer mine-disturbance product.

### Supporting ESD screen

Squared Euclidean Spectral Distance (ESD) was calculated from B02, B03, and B04 to identify locations with a detectable spectral difference between the two dates. Only RF transitions inside this ESD footprint are summarised below. ESD is not used to assign land-cover meaning; Random Forest provides that interpretation.

## Results

The RF transition map shows spatially concentrated candidate transitions rather than site-wide change. This is consistent with a short **21-day** observation interval: the analysis captures short-term surface dynamics, not a long-term land-cover trajectory.

| Candidate transition inside ESD footprint | Area (ha) | Interpretation priority |
|---|---:|---|
| Vegetation to Bare ground / mine surface | 85.99 | Highest priority: possible clearing or expansion of exposed mine surface |
| Water to Bare ground / mine surface | 18.04 | Review around mine-water features, sediment exposure, or drawdown |
| Water to Vegetation | 6.05 | Lower-priority candidate; inspect before/after RGB imagery |
| Vegetation to Water | 1.44 | Local water-change candidate |
| Bare ground / mine surface to Vegetation | 0.44 | Small candidate revegetation or rehabilitation signal |
| Bare ground / mine surface to Water | 0.00 | No mapped candidate within the ESD footprint |

The main result is the **Vegetation to Bare ground / mine surface** category. These candidates occur near the vegetation-disturbance interface and should be reviewed first against mine plans, clearing records, and the two RGB images.

The map also contains narrow linear signals associated with image-seam or classification artefacts. They are not treated as operational findings. The important review targets are compact patches at the vegetation-mine boundary and around disturbed mine surfaces or water features.

## Interpretation

The RF map supports a practical review workflow:

- **Vegetation to Bare ground / mine surface** may represent clearing, disturbance expansion, road work, stockpile development, or newly exposed material.
- **Water to Bare ground / mine surface** may represent water-level change, sediment exposure, drawdown, or classification uncertainty caused by water colour and shadow.
- **Bare ground / mine surface to Vegetation** may indicate possible revegetation, but the small mapped area requires visual and operational confirmation.

These are candidate land-cover transitions. They do not prove a specific activity, environmental impact, reclamation result, or non-compliance. The short observation interval, visible-band-only input, 10 m pixel size, shadows, haze, water turbidity, and residual seam artefacts limit interpretation.

## Related mining-monitoring reference

Prasetya and Tsai (2025) demonstrate how multi-temporal Sentinel-2 analysis can support mining and reclamation monitoring by tracking transitions among vegetation, water, and mining-surface classes over multiple years. Their study uses a U-Net deep-learning model, five classes, permit boundaries, and a longer time series; those components were **not** applied here. It is included as supporting context for why land-cover transitions are relevant to mine monitoring, while this assignment uses a simpler three-class Random Forest workflow that matches the available RGB bands, samples, and short observation interval.
## Supporting ESD outputs and database

The ESD workflow produced the required georeferenced change-intensity raster, binary change raster, and polygon database:

- `data/processed/esd/change_map.tif`
- `data/processed/esd/change_binary.tif`
- `data/processed/esd/processed.gpkg`, layer `change_feature`

The GeoPackage is an SQLite-based spatial database. It stores polygon geometry, dates, area, and mean spectral change-strength attributes. It provides the traceable spatial screen used to focus the RF transition interpretation.

## Conclusion

For this short 21-day interval, the final product is most useful as a targeted mining-change review map. Random Forest identifies the likely direction of change between vegetation, water, and bare mine surface, while ESD ensures that attention is concentrated on locations with measured visible-band change. The highest-priority candidates are compact Vegetation to Bare ground / mine surface patches that should be checked against operational evidence.

## References

- Prasetya, K. D., & Tsai, F. (2025). Deep learning-based multitemporal spatial analytics for assessing reclamation compliance of coal mining permits in Kalimantan with satellite images. *Remote Sensing, 17*, 3477. https://doi.org/10.3390/rs17203477
- Lu, D., Mausel, P., Brondízio, E., & Moran, E. (2004). Change detection techniques. *International Journal of Remote Sensing, 25*(12), 2365-2401. https://doi.org/10.1080/0143116031000139863
- Hussain, M., Chen, D., Cheng, A., Wei, H., & Stanley, D. (2013). Change detection from remotely sensed images: From pixel-based to object-based approaches. *ISPRS Journal of Photogrammetry and Remote Sensing, 80*, 91-106. https://doi.org/10.1016/j.isprsjprs.2013.03.006
