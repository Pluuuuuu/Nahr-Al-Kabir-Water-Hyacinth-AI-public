# Nahr Al Kabir water hyacinth — Random Forest

This folder is the delivered data package for mapping water hyacinth along the Nahr Al Kabir river. The completed work is a Random Forest classifier on Sentinel-2 river pixels. The study area is the river polygon in UTM zone 36N (`EPSG:32636`), on a 10 m grid of 881 × 249 pixels.

## What was done

Sentinel-2 scenes were collected from 20 August 2015 through 5 July 2026: **394 unique dates**. Every date has the same river mask, **4,177 valid river pixels**, so the final table has 1,645,738 rows.

Polygon labels exist for **132 dates**. The training target is the field `Water_hyac` (0 = absence, 1 = presence). The field `CLASS` was not used as the target. No labeled date contains both classes:

- Winter (1 Jan–14 Apr) and spring (15 Apr–14 May) dates are absence only.
- Development (15 May–31 Dec) dates are presence only.

Dates were split as whole scenes, inside each phenology period, with 70% kept for training (`random_state` 42):

| Period | Dates | Train | Test |
| --- | ---: | ---: | ---: |
| Winter absence | 30 | 21 | 9 |
| Spring reference | 14 | 10 | 4 |
| Development | 88 | 62 | 26 |
| **Labeled total** | **132** | **93** | **39** |

The other **262** dates have predictors and no polygon labels. A date never appears in both train and test. Training pixels: 35,011 (14,765 class 0, 20,246 class 1). Test pixels: 14,921 (6,392 class 0, 8,529 class 1).

The model uses **36 predictors**: 10 Sentinel-2 bands (B02–B08, B8A, B11, B12), SAVI and FVC2 plus their scene-level max and mean, seven indices (NDVI, NDVIRe2, NDVIRe3, NDWI, NDAVI, FAI, NDMI), 11 converted ERA5 variables, DEM, and slope. Reflectance was scaled by 10,000. There was no scaler, imputation, or oversampling. FVC2 is SAVI clipped to 0.77.

`RandomForestClassifier` was fit with 300 trees, `class_weight="balanced"`, and `random_state=42`. A 5-fold grouped cross-validation on the training dates set the decision threshold to **0.68** (best out-of-fold F1).

## Test result

At threshold 0.68 on the 39 held-out dates:

| Metric | Value |
| --- | ---: |
| Accuracy | 0.940 |
| Precision | 0.954 |
| Recall | 0.942 |
| F1 | 0.948 |
| ROC-AUC | 0.994 |

Confusion counts: true negative 6,000, false positive 392, false negative 498, true positive 8,031. Spring test dates are all class 0 and produce the false positives. Development test dates are all class 1 and produce the false negatives. The headline scores are real on this split, and they also reflect that class and phenology period are the same thing in the labels.

The strongest predictors were FVC2, SAVI, total evaporation, FAI, dewpoint temperature, and NDWI. Spectral indices as a group outweighed ERA5, which outweighed the raw bands. DEM and slope contributed very little.

## What is in this folder

- `Satelitte_Sentinel2_Tiff_Images-...zip` — clipped Sentinel-2 rasters for the 394 dates (bands, indices, SAVI, FVC, color composites).
- `Nahr_Al_Kabir_Shapefile-...zip` — river polygon used as the study area.
- `Training_Data_133_S2_Polygons_Shapefile-...zip` — polygon labels. The archive name says 133; the shapefile covers 132 dates.
- `Predictors_Excels-...001.zip` and `...002.zip` — predictor tables, including the final 394-date Random Forest table and the ERA5 workbook.
- `Random_Forest_ML_Notebook-...zip` — the Random Forest notebook (two copies of the same notebook).
- `RandomForest_Results-...zip` — result figures and the summary slides.
- `Results_Comparison_OldMethodo_Vs_RF-...zip` — comparison of the earlier SAVI/FVC area method with the Random Forest.
- `Notebooks_For_The_Methodology_Of_DrYoussra-...zip` — notebooks for the earlier SAVI and FVC workflow.
- `Splitting_Training_Validation_Prediction_Data.xlsx` — planning notes for the train, validation, and prediction split (two copies).
- `Mnal_Previous_Current_Work.docx` — written note on the previous method and the current Random Forest work.
- `Articles-...zip` — reference articles.

## Reading the results

A file whose name says 132 training dates actually lists 133 dates. The extra date is 10 May 2020: it has predictors and no polygons, so it was not a training date. Unlabeled dates in the executed split are 262, not 261.

Same river locations repeat across dates, and some predictors (max and mean SAVI and FVC2) are one number repeated for every pixel of a date. The model was not given a separate frozen validation set; the 0.68 threshold came from cross-validation on the training dates only.
