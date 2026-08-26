# Data dictionary

The executed notebook reports 21 raw columns. The types below follow the Spark schema printed by the notebook. Descriptions are concise interpretations of the field names used by the dataset.

| Column | Spark type | Role / interpretation |
|---|---|---|
| `customerID` | string | Customer identifier; retained for scoring, excluded from predictive features |
| `gender` | string | Customer gender category |
| `SeniorCitizen` | integer | Senior-citizen indicator |
| `Partner` | string | Whether the customer has a partner |
| `Dependents` | string | Whether the customer has dependents |
| `tenure` | integer | Months with the company |
| `PhoneService` | string | Phone service status |
| `MultipleLines` | string | Multiple-lines status |
| `InternetService` | string | Internet service category |
| `OnlineSecurity` | string | Online security service status |
| `OnlineBackup` | string | Online backup service status |
| `DeviceProtection` | string | Device protection service status |
| `TechSupport` | string | Tech support service status |
| `StreamingTV` | string | Streaming TV service status |
| `StreamingMovies` | string | Streaming movies service status |
| `Contract` | string | Contract type |
| `PaperlessBilling` | string | Paperless billing status |
| `PaymentMethod` | string | Payment method |
| `MonthlyCharges` | double | Monthly charge amount |
| `TotalCharges` | string (raw) | Historical total charges; converted to double after cleaning |
| `Churn` | string | Target label: `Yes` / `No` |

## Engineered variables

| Variable | Definition |
|---|---|
| `label` | `1.0` when `Churn == "Yes"`, otherwise `0.0` |
| `num_services` | Number of active `Yes` values across eight telecom service columns |
| `charges_per_month` | `TotalCharges / tenure` when tenure > 0; otherwise `MonthlyCharges` |

## Data-quality rule

The notebook identifies **11 blank `TotalCharges` rows**. All have `tenure = 0`; they are retained and mapped to `0.0`.
