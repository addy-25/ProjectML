# ProjectML — Flask

This page explains the Flask app that powers the UI and prediction endpoint in `app.py`.

## What the backend does

The backend is a small Flask server with two routes:

- `/` renders the landing page.
- `/predictdata` serves the prediction form on `GET` and returns a prediction on `POST`.

Its job is not to train the model. Training happens elsewhere in the project. The Flask app only collects input, turns it into a DataFrame, sends it through the saved preprocessing and model artifacts, and shows the result back to the user.

## Main file

- [app.py](app.py) creates the Flask application, defines routes, and starts the server.

## Route breakdown

### `/`

- Method: `GET`
- Template: `templates/index.html`
- Purpose: simple landing page or entry point for the app

### `/predictdata`

This route handles both display and prediction.

- `GET` returns `templates/home.html`, which contains the prediction form.
- `POST` reads the submitted form values, wraps them in `CustomData`, converts them to a DataFrame, and passes that DataFrame to `PredictPipeline.predict()`.

## Request flow

1. The user opens the landing page.
2. The user goes to the prediction form.
3. The form submits values like gender, ethnicity, parental education, lunch type, test prep, reading score, and writing score.
4. `app.py` reads those values from `request.form`.
5. `CustomData` converts the values into a one-row pandas DataFrame.
6. `PredictPipeline` loads the saved preprocessing pipeline and model from `artifacts/`.
7. The backend transforms the input and calls the model.
8. The prediction is rendered back into `home.html`.

## Input fields the form expects

The backend expects these keys in the submitted form:

- `gender`
- `ethnicity` for race/ethnicity
- `parental_level_of_education`
- `lunch`
- `test_preparation_course`
- `reading_score`
- `writing_score`

These become the columns used by the prediction pipeline.

## Output

The route returns the same `home.html` template, but with a `prediction` value injected into the page.

If the model returns a sequence-like object, the first prediction is shown.

## Tiny request / response example

Example form values submitted to `/predictdata`:

- `gender`: female
- `ethnicity`: group B
- `parental_level_of_education`: bachelor's degree
- `lunch`: standard
- `test_preparation_course`: completed
- `reading_score`: 72
- `writing_score`: 75

What the backend does with that submission:

1. Receives the `POST` request on `/predictdata`.
2. Reads the submitted values from `request.form`.
3. Converts them into a one-row DataFrame with `CustomData`.
4. Sends the DataFrame into `PredictPipeline.predict()`.
5. Loads `artifacts/preprocessor.pkl` and `artifacts/model.pkl`.
6. Returns the prediction to `home.html`.

## How it connects to the rest of the project

- `app.py` depends on `src/pipeline/predict_pipeline.py`.
- `PredictPipeline` depends on `artifacts/model.pkl` and `artifacts/preprocessor.pkl`.
- Those artifacts are created by the training workflow, not by the Flask app itself.

That means the web app only works correctly after the model artifacts already exist.

## Running behavior

When `app.py` is executed directly, it starts a Flask development server on:

- host: `0.0.0.0`
- port: `8008`
- debug: `True`

That makes the app available on your machine and on the network interface the container or machine exposes.

## Useful notes while reading the code

- The file currently imports `StandardScaler`, `numpy`, and `pandas`, but the route logic itself mostly relies on Flask and the prediction pipeline.
- `application = Flask(__name__)` and `app = application` are both defined so the app can be referenced in different deployment styles.
- `home.html` is where the form and prediction result are likely displayed.

## What to inspect next

If you want to understand the backend further, read these in order:

1. [app.py](app.py)
2. [src/pipeline/predict_pipeline.py](src/pipeline/predict_pipeline.py)
3. [templates/home.html](templates/home.html)
