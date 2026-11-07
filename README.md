# Depth-Resolved Marine Heatwave Prediction in the Levantine Basin

Code and results for the paper:

> D. N. H. Weerasinghe and S. Viswakula, "Depth-Resolved Marine Heatwave Prediction in the Levantine Basin Using Machine Learning and Explainable Artificial Intelligence," in *Proc. IEEE International Conference on Advancements in Computing (ICAC)*, 2026.

The study predicts marine heatwave (MHW) occurrence at four depth configurations (sea surface, 0–50 m, 50–100 m, 100–150 m) in the Levantine Basin (Eastern Mediterranean) with LightGBM and LSTM models, for lead times of 3, 5, 7, 9, 12 and 14 days (48 configurations). It evaluates them against a persistence baseline, on onset days, and on a common set of cells, and explains them with SHAP.

## Repository layout

```
notebooks/
  01_data/
    atmospheoric-dataset-creation.ipynb      ERA5 download (CDS API), daily means, wind speed
    subsurface-dataset-creation.ipynb        CMEMS download, thickness-weighted layer means
    integration-subsurfaceatmos.ipynb        merge ocean and atmosphere onto one grid
    Integrated_data_valiadation.ipynb        coverage, range, alignment and continuity checks
    label_dataset.ipynb                      Hobday et al. (2016) MHW labels per layer
    Exploratory-Data-Analysis-EDA.ipynb      exploratory analysis
  02_training/
    LightGBM_SST_Subsurface_Combined_SHAP.ipynb   LightGBM, sea surface (SST)
    LightGBM_Subsurface_Layer01_SHAP.ipynb        LightGBM, 0-50 m
    LightGBM_Subsurface_Layer02_SHAP.ipynb        LightGBM, 50-100 m
    LightGBM_Subsurface_Layer03_SHAP.ipynb        LightGBM, 100-150 m
    LSTM_Subsurface_Combined_SST_SHAP.ipynb       LSTM, sea surface (SST)
    LSTM_Subsurface_Layer01_SHAP.ipynb            LSTM, 0-50 m
    LSTM_Subsurface_Layer02_SHAP.ipynb            LSTM, 50-100 m
    LSTM_Subsurface_Layer03_SHAP.ipynb            LSTM, 100-150 m
  03_evaluation/
    EvaluationWithCalibration_v3.ipynb            calibration, variant selection, thresholds,
                                                  test evaluation, persistence, common cells
    OnsetConditional.ipynb                        onset-day evaluation (no MHW on day t)
    Model_EvaluationFeature_Importance_Plots_Generation.ipynb   tables, plots, SHAP summaries
results/                                    CSV outputs used in the paper
```

Each training notebook trains one layer for all six lead times and computes SHAP values.

**Note on the training notebooks.** They also fit an isotonic calibrator and print test metrics. Those were preliminary checks during development. All results reported in the paper come from `03_evaluation/EvaluationWithCalibration_v3.ipynb`, which reloads the trained models and applies the calibration and selection described below.

## Data splits

| Split | Period | Used for |
|---|---|---|
| Train | 1993-01-01 to 2019-12-31 | model fitting |
| Calibration (CAL) | 2020-01-15 to 2021-01-30 | fitting Beta / temperature-scaling calibrators |
| Threshold tuning (THR) | 2021-01-31 to 2022-12-31 | variant selection and decision thresholds |
| Test | 2023-01-15 to 2025-12-31 | final evaluation only |

The test set is never used for fitting, calibration, variant selection or threshold tuning:

- **Variant selection:** for each configuration, the calibration variant (raw, calibrated, or calibrated with prior correction) is chosen on the THR split by the summed ranks of Brier Score and ECE (`brier_thr`, `ece_thr`). Ties go to the simpler variant.
- **Prior correction:** the Saerens prior correction uses the THR-split MHW rate as its target.
- **Test metrics:** they are computed afterwards, and all variants are reported to show how calibration transfers under distribution shift.

## Results files

| File | Contents |
|---|---|
| `best_per_model.csv` | selected variant per configuration (Table II of the paper) |
| `all_evaluation_results.csv` | every variant × threshold strategy |
| `model_vs_persistence.csv` | models vs. persistence on the same test rows |
| `persistence_baseline.csv` | persistence forecast scored per configuration and lead |
| `brier_comparison.csv` | test-set Brier Score of every calibration variant, for comparison only (not used for selection) |
| `common_cells_chosen.csv` | selected variants and persistence on common cells (valid 100–150 m data) |
| `common_cells_results.csv` | all variants on common cells |
| `onset_conditional.csv` | onset-day evaluation (no MHW on day *t*); `chosen` marks the selected variant |
| `shap_combined.csv` | mean SHAP importance per feature, configuration and lead |


## Running the code

The notebooks were run on two platforms:

- **WEkEO JupyterHub** (Copernicus Data and Information Access Service): data extraction, preprocessing, integration, validation and MHW labelling. These are the notebooks in `01_data/`, except the EDA notebook.
- **Google Colab** with Google Drive mounted: exploratory data analysis (`Exploratory-Data-Analysis-EDA.ipynb`), model training (`02_training/`) and evaluation (`03_evaluation/`). The labelled dataset `ocean_atmos_DAILY_WITH_LABELS_1993_2025.nc` was copied from WEkEO to Google Drive for these steps.

To reproduce:

1. Register for the Copernicus Climate Data Store and Copernicus Marine, and enter your own credentials where the data notebooks ask for them. No credentials are stored in this repository.
2. Set the data paths at the top of each notebook to your own folders.
3. Install the requirements: `pip install -r requirements.txt`.
4. Run `01_data`, then `02_training`, then `03_evaluation`.

The training notebooks save each model folder directly under `trained_models/`. The evaluation notebook expects them grouped as `trained_models/lightgbm/<folder>` and `trained_models/lstm/<folder>`. Move the folders, or change `LGBM_BASE` and `LSTM_BASE` at the top of `EvaluationWithCalibration_v3.ipynb`.

## Data

Inputs come from:

- E.U. Copernicus Marine Service, Mediterranean Sea Physical Reanalysis (MEDSEA_MULTIYEAR_PHY_006_004), doi:10.25423/CMCC/MEDSEA_MULTIYEAR_PHY_006_004_E3R1;
- Copernicus Climate Change Service ERA5 hourly data on single levels, doi:10.24381/cds.adbb2d47.

The raw data are not redistributed here. Download them from Copernicus under their licences.

**Copernicus attribution.** The result files in `results/` are derived from Copernicus data:

- Generated using E.U. Copernicus Marine Service Information.
- Contains modified Copernicus Climate Change Service information (2026).
- Neither the European Commission nor ECMWF is responsible for any use that may be made of the Copernicus information or data it contains.



## Licence

MIT, see `LICENSE`.
