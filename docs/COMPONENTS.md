# ProjectML — Components

This document lists the main code components, what they do, and where to look when you want to change behavior.

## `src/componenets/data_ingestion.py`

- Responsibility: read raw CSV, save raw copy, create train/test splits.
- Key functions/classes: `DataIngestion`, `DataIngestionConfig`.
- Inputs: raw CSV (notebook/data/stud.csv in current code). Outputs: `artifacts/data.csv`, `artifacts/train.csv`, `artifacts/test.csv`.

## `src/componenets/data_transformation.py`

- Responsibility: build and apply preprocessing pipeline (encoding, scaling, feature assembly).
- Typical outputs: transformed numpy arrays for train/test, and `artifacts/preprocessor.pkl` (pickled pipeline).
- Change here to add/remove features or alter encoders/scalers.

## `src/componenets/model_trainer.py`

- Responsibility: train candidate models, run hyperparameter search/selection, evaluate models and persist the best model as `artifacts/model.pkl`.
- Key class: `ModelTrainer` and `ModelTrainerConfig`.
- Metrics & selection logic live here (uses `evaluate_models` from `src/utils.py`).

## `src/pipeline/train_pipeline.py`

- Responsibility: orchestrates end-to-end training: ingestion → transformation → training.
- Use this file (or the `notebook/2. MODEL TRAINING.ipynb`) to run full training programmatically.
- See also: [docs/TRAINING_PIPELINE.md](docs/TRAINING_PIPELINE.md) for a detailed walkthrough of each training step.

## `src/pipeline/predict_pipeline.py`

- Responsibility: load `preprocessor.pkl` + `model.pkl`, transform incoming tabular rows and return model predictions.
- Exposes `PredictPipeline.predict()` and `CustomData.get_data_as_dataframe()` which the web app uses.
- See also: [docs/PREDICTION_PIPELINE.md](docs/PREDICTION_PIPELINE.md) for a step-by-step explanation of the inference flow.

## `app.py`

- Responsibility: lightweight Flask UI for manual predictions. Reads form fields, converts them via `CustomData`, calls `PredictPipeline`, and renders templates in `templates/`.
- See also: [docs/FLASK.md](docs/FLASK.md) for the route-level explanation.

## `src/utils.py`

- Responsibility: utility helpers used across components (save/load objects, `evaluate_models`, data helpers). Modify carefully — many modules depend on it.

## `src/exception.py` and `src/logger.py`

- `src/exception.py`: project-specific `CustomException` wrapper for richer errors and stack info.
- `src/logger.py`: centralized logging setup used by components to record progress and errors.

## Artifacts & monitoring

- `artifacts/`: produced CSV splits, `preprocessor.pkl`, `model.pkl` — primary inputs for prediction.
- `catboost_info/`: CatBoost training metadata and TensorBoard event files used for deeper training inspection.

## Notebooks & experiments

- `notebook/1 . EDA STUDENT PERFORMANCE .ipynb`: exploratory data analysis.
- `notebook/2. MODEL TRAINING.ipynb`: interactive training and evaluation flow.

## Templates

- `templates/`: `home.html` and `index.html` used by `app.py`.

## Where to add tests

- Unit test ideas: `PredictPipeline.predict()` with a tiny DataFrame, `CustomData.get_data_as_dataframe()` formatting, `evaluate_models()` behavior on toy data.

## Notes for contributors

- Keep data paths (artifacts, notebook data) in mind when running pipelines locally.
- Respect the project’s error/logging helpers (`CustomException`, `logging`) so results are traceable.
