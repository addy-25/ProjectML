# ProjectML — Troubleshooting

## Error: `SimpleImputer' object has no attribute '_fill_dtype'`

### What it means

This error occurs when loading a saved preprocessing pipeline (`artifacts/preprocessor.pkl`) that was created with an older version of scikit-learn, and the current environment has a newer (incompatible) version.

### Root cause

The `SimpleImputer` class in scikit-learn changed its internal attributes between versions. The pickled object from an older version cannot be unpickled in a newer version because the internal structure is different.

### How to fix it

**Option 1: Retrain the model (recommended)**

Delete the old artifacts and retrain from scratch:

```bash
rm -f artifacts/preprocessor.pkl artifacts/model.pkl
```

Then run the training pipeline to create fresh artifacts compatible with your current sklearn version:

```bash
python -c "
from src.componenets.data_ingestion import DataIngestion
from src.componenets.data_transformation import DataTransformation
from src.componenets.model_trainer import ModelTrainer

ingestion = DataIngestion()
train_path, test_path = ingestion.initiate_data_ingestion()

transformation = DataTransformation()
train_arr, test_arr, preprocessor = transformation.initiate_data_transformation(train_path, test_path)

trainer = ModelTrainer()
trainer.initiate_model_trainer(train_arr, test_arr)
"
```

Or use the notebook at `notebook/2. MODEL TRAINING.ipynb`.

**Option 2: Downgrade scikit-learn (if you know the old version)**

If you know which sklearn version was used to create the artifacts, you can downgrade:

```bash
pip install scikit-learn==X.Y.Z
```

Replace `X.Y.Z` with the version that created the pickle file. However, this is not recommended for production.

### Prevention

- Keep `requirements.txt` pinned to specific versions to ensure reproducibility.
- Document which versions were used to train the model.
- Retrain whenever you update major dependencies.

## Other common errors

### FileNotFoundError: `artifacts/model.pkl` or `artifacts/preprocessor.pkl` not found

**Cause:** The training pipeline has never been run, or the artifacts were deleted.

**Fix:** Run the training pipeline as shown in Option 1 above.

### ValueError: Input has different number of features than expected

**Cause:** The form submitted different column names or values than the preprocessor expects.

**Fix:** Check that the form keys in `app.py` match the column names in the training data (`notebook/data/stud.csv`).

### Connection refused or 500 error on `/predictdata`

**Cause:** The Flask app crashed or the prediction pipeline failed.

**Fix:** Check the terminal output for the full traceback. Look for missing dependencies, missing artifacts, or schema mismatches.

## Debugging workflow

1. Check the full error traceback in the Flask terminal output.
2. Identify which component failed (ingestion, transformation, model training, or prediction).
3. Refer to the relevant doc:
   - [docs/TRAINING_PIPELINE.md](docs/TRAINING_PIPELINE.md) for training issues
   - [docs/PREDICTION_PIPELINE.md](docs/PREDICTION_PIPELINE.md) for prediction issues
   - [docs/FLASK.md](docs/FLASK.md) for web app issues
4. If retraining, ensure `notebook/data/stud.csv` exists.
5. Run the training pipeline step-by-step to isolate the failure point.

## Need more help?

- Review the full error message and traceback.
- Check `src/logger.py` for logging setup and review the logs.
- Look at the notebooks for working examples.
- Read the component docs in [docs/COMPONENTS.md](docs/COMPONENTS.md).
