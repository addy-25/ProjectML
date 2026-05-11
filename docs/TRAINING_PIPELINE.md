# ProjectML — Training Pipeline

This document explains the process of taking raw data and producing a trained model.

## Where the pipeline lives

- The orchestration logic is in [src/pipeline/train_pipeline.py](src/pipeline/train_pipeline.py) (currently empty/placeholder).
- The actual components that do the work are in `src/componenets/`:
  - [src/componenets/data_ingestion.py](src/componenets/data_ingestion.py)
  - [src/componenets/data_transformation.py](src/componenets/data_transformation.py)
  - [src/componenets/model_trainer.py](src/componenets/model_trainer.py)
- Interactive examples are in [notebook/2. MODEL TRAINING.ipynb](notebook/2.%20MODEL%20TRAINING.ipynb).

## What the training pipeline does

The training pipeline orchestrates three main steps:

1. **Data Ingestion:** read raw CSV, split into train/test, save to disk.
2. **Data Transformation:** fit encoders and scalers on training data, produce transformed arrays, save the preprocessing pipeline.
3. **Model Training:** train multiple candidate models, evaluate them, select the best one, save it to disk.

The output is a trained model and a preprocessing pipeline, both pickled and saved to `artifacts/`.

## Step 1: Data Ingestion

File: [src/componenets/data_ingestion.py](src/componenets/data_ingestion.py)

- Reads a raw CSV from `notebook/data/stud.csv`.
- Creates the `artifacts/` directory if it doesn't exist.
- Saves the raw data as `artifacts/data.csv`.
- Splits the data into train (80%) and test (20%) using `random_state=42` for reproducibility.
- Outputs: `artifacts/train.csv` and `artifacts/test.csv`.

## Step 2: Data Transformation

File: [src/componenets/data_transformation.py](src/componenets/data_transformation.py)

This step builds a preprocessing pipeline and applies it to the training and test data.

### Feature schema

**Numerical columns:**

- `reading_score`
- `writing_score`

**Categorical columns:**

- `gender`
- `race_ethnicity`
- `parental_level_of_education`
- `lunch`
- `test_preparation_course`

**Target column:**

- `math_score`

### Preprocessing logic

For numerical columns:

1. Impute missing values with the median.
2. Scale with `StandardScaler`.

For categorical columns:

1. Impute missing values with the most frequent value.
2. One-hot encode.
3. Scale with `StandardScaler`.

Both pipelines are combined into a single `ColumnTransformer`.

### Outputs

- **Transformed training array:** `input_feature_train_arr`, plus the target column concatenated.
- **Transformed test array:** `input_feature_test_arr`, plus the target column concatenated.
- **Saved preprocessor:** `artifacts/preprocessor.pkl` — the fitted preprocessing pipeline, reused during prediction.

## Step 3: Model Training

File: [src/componenets/model_trainer.py](src/componenets/model_trainer.py)

This step trains multiple regressor models, selects the best one, and saves it.

### Models trained

The code trains these models:

- Linear Regression
- Decision Tree
- Random Forest
- Gradient Boosting
- XGBoost
- CatBoost
- AdaBoost

### Model selection

1. Each model is trained with its own hyperparameter grid.
2. The `evaluate_models()` function (from `src/utils.py`) runs hyperparameter tuning or cross-validation for each.
3. Models are scored on the test set (likely using R² or a similar metric).
4. The model with the best test score is selected.
5. If the best score is below 0.6, training fails with an error.

### Output

- **Saved model:** `artifacts/model.pkl` — the trained regressor object.

## Common hyperparameters tuned

Some examples from the code:

- **Decision Tree:** criterion (squared_error, friedman_mse, absolute_error, poisson)
- **Random Forest:** n_estimators (8, 16, 32, 64, 128, 256)
- **Gradient Boosting:** learning_rate, subsample, n_estimators
- **XGBoost:** learning_rate, n_estimators
- **CatBoost:** depth, learning_rate, iterations
- **AdaBoost:** learning_rate, n_estimators

The exact tuning strategy is inside `evaluate_models()` in `src/utils.py`.

## Full pipeline execution order

1. `DataIngestion.initiate_data_ingestion()` → returns train_path, test_path
2. `DataTransformation.initiate_data_transformation(train_path, test_path)` → returns transformed train array, test array, preprocessor
3. `ModelTrainer.initiate_model_trainer(train_array, test_array)` → returns trained model

These steps should be orchestrated by `train_pipeline.py`, though it is currently a placeholder. The notebooks show the full flow in practice.

## Important notes

- **Reproducibility:** `random_state=42` is used in train/test split and likely in model training.
- **Target variable:** `math_score` is the variable being predicted. All other columns are features.
- **Data leakage:** the preprocessing is fitted on the training data only, then applied to test data (correct practice).
- **Artifacts are persistent:** once saved, they are reused by the prediction pipeline and web app.

## If you want to retrain

1. Ensure the raw CSV exists at `notebook/data/stud.csv`.
2. Run the training notebook or call the components in order:

   ```python
   from src.componenets.data_ingestion import DataIngestion
   from src.componenets.data_transformation import DataTransformation
   from src.componenets.model_trainer import ModelTrainer

   ingestion = DataIngestion()
   train_path, test_path = ingestion.initiate_data_ingestion()

   transformation = DataTransformation()
   train_arr, test_arr, preprocessor = transformation.initiate_data_transformation(train_path, test_path)

   trainer = ModelTrainer()
   trainer.initiate_model_trainer(train_arr, test_arr)
   ```

3. New artifacts will overwrite the old ones.

## Reading order if you are learning the project

1. [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)
2. [docs/TRAINING_PIPELINE.md](docs/TRAINING_PIPELINE.md) (this file)
3. [docs/COMPONENTS.md](docs/COMPONENTS.md)
4. [src/pipeline/train_pipeline.py](src/pipeline/train_pipeline.py)
5. [src/componenets/data_ingestion.py](src/componenets/data_ingestion.py)
6. [src/componenets/data_transformation.py](src/componenets/data_transformation.py)
7. [src/componenets/model_trainer.py](src/componenets/model_trainer.py)
