# Data

The executed notebook loads the Telco Customer Churn CSV from a Databricks Volume path configured through the `dataset_path` widget.

Observed in the executed run:

- **7,043 rows**
- **21 raw columns**
- **1,869 churners**
- overall churn rate **26.54%**
- source filename/path used in Databricks: `WA_Fn-UseC_Telco-Customer-Churn.csv` inside a Volume

The raw CSV is **not included in this repository reconstruction because it was not part of the notebook file supplied for publication**. This keeps the repository faithful to the provided source material.

To reproduce the notebook, upload the same CSV to a Databricks Volume and set the `dataset_path` widget to its exact path.
