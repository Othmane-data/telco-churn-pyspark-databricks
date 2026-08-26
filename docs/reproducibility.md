# Reproducibility notes

## Environment

The supplied notebook executed successfully with **Spark 4.1.0** in Databricks Free Edition / serverless compute.

## Dataset path

The notebook defines a Databricks widget named `dataset_path`. Upload the CSV to a Volume and set this widget to the exact file path.

## Serverless-safe design

The notebook intentionally avoids `cache()` / `persist()` and avoids a fitted `StringIndexer` + `OneHotEncoder` preprocessing pipeline. Instead, it:

1. splits train/test first;
2. learns category levels from the training set only;
3. creates dummy variables with Spark column expressions;
4. assembles the 32-dimensional feature vector;
5. trains Logistic Regression;
6. materializes only compact scoring outputs to pandas;
7. deletes the LR model before fitting Random Forest.

This design reduces pressure on the Spark Connect ML model cache in Free Edition.

## Validation discipline

The threshold curve in the notebook is exploratory because it is computed on the test set. A production workflow should introduce a separate validation set or time-based validation window for model/threshold selection and reserve a final test set for unbiased evaluation.
