# ProjectML — Prediction Pipeline

This document explains how the prediction pipeline turns one user input row into a model prediction.

## Where the pipeline lives

- [src/pipeline/predict_pipeline.py](src/pipeline/predict_pipeline.py)

That file defines two pieces:

- `PredictPipeline`, which loads the saved preprocessing pipeline and model and runs inference.
- `CustomData`, which turns raw form values into a pandas DataFrame.

## What the pipeline is for

The prediction pipeline is the part of the project that runs after training is complete.

It does not retrain the model. Instead, it:

1. takes a single row of input data,
2. applies the same preprocessing that was used during training,
3. loads the trained model from disk,
4. produces a prediction.

## Inputs it expects

The pipeline expects a DataFrame with these columns:

- `gender`
- `race_ethnicity`
- `parental_level_of_education`
- `lunch`
- `test_preparation_course`
- `reading_score`
- `writing_score`

`CustomData.get_data_as_dataframe()` creates exactly that shape from the Flask form values.

## Saved artifacts it depends on

The pipeline reads two files from `artifacts/`:

- `artifacts/preprocessor.pkl`
- `artifacts/model.pkl`

These files must already exist for inference to work.

If either file is missing, stale, or incompatible with the current input schema, prediction will fail.

## `CustomData`

`CustomData` is a small helper class used to convert raw values from the Flask form into a one-row DataFrame.

Its job is simple:

1. store the submitted values,
2. build a dictionary using the exact feature names expected by the model,
3. return `pandas.DataFrame(custom_data_input_dict)`.

This avoids manual DataFrame construction inside the Flask route.

## `PredictPipeline.predict()` flow

The `predict()` method follows this order:

1. build the file paths for the saved model and preprocessor,
2. load both objects with `load_object()`,
3. transform the incoming features with `preprocessor.transform(features)`,
4. run `model.predict(data_scaled)`,
5. return the prediction array.

So the output is whatever the trained model predicts for the transformed row.

## Why preprocessing happens before prediction

The model was trained on transformed data, not on the raw user input.

That means the inference path must use the same preprocessing steps that were fitted during training. This keeps training and prediction consistent.

In practice, that usually means:

- categorical values are encoded,
- numeric values are scaled or passed through the same pipeline used in training,
- the final transformed row has the same feature layout the model expects.

## Error handling

The pipeline wraps failures in `CustomException`.

That means problems like missing artifacts, wrong column names, bad input types, or preprocessing errors should surface as project-specific exceptions instead of raw tracebacks.

## Common failure points

These are the things most likely to break prediction:

- `artifacts/model.pkl` does not exist,
- `artifacts/preprocessor.pkl` does not exist,
- the form sends field names that do not match the expected column names,
- the input values cannot be converted into the expected type,
- the preprocessing pipeline was trained on a different schema.

## How it connects to the Flask app

The backend flow is:

1. `app.py` reads the submitted form values.
2. `CustomData` builds the DataFrame.
3. `PredictPipeline` loads the artifacts and predicts.
4. `app.py` sends the result back to `home.html`.

That means the Flask app is just the delivery layer; the actual inference logic lives here.

## Reading order if you are learning the project

1. [docs/FLASK.md](docs/FLASK.md)
2. [src/pipeline/predict_pipeline.py](src/pipeline/predict_pipeline.py)
3. [src/componenets/data_transformation.py](src/componenets/data_transformation.py)
4. [src/componenets/model_trainer.py](src/componenets/model_trainer.py)
