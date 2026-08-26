# Telco Customer Churn Prediction — PySpark & Databricks

> End-to-end churn analytics and machine-learning portfolio built from an **executed Databricks notebook**: business understanding → data quality → EDA → feature engineering → serverless-safe PySpark encoding → Spark ML → threshold analysis → customer scoring → retention actions.

[Executed notebook](notebooks/telco_customer_churn_databricks_with_native_plots.ipynb) · [Model card](docs/model_card.md) · [Data dictionary](docs/data_dictionary.md) · [Reproducibility](docs/reproducibility.md) · [French version](README_FR.md)

---

## Executive snapshot

| Item | Executed result |
|---|---:|
| Customers | **7,043** |
| Raw columns | **21** |
| Churners | **1,869** |
| Overall churn | **26.54%** |
| Runtime | **Databricks Free Edition / Spark 4.1.0** |
| Train / test | **5,634 / 1,409** |
| Predictive dimensions | **32** |
| Models | Logistic Regression, Random Forest |
| Champion by ROC-AUC | **Logistic Regression** |
| Logistic ROC-AUC | **0.8369** |
| Logistic PR-AUC | **0.6347** |
| Logistic churn recall @ 0.50 | **47.52%** |
| Exploratory best churn-F1 threshold | **0.30** |
| Churn recall @ 0.30 | **76.24%** |

**Execution integrity:** the supplied notebook contains **31 cells, 64 recorded outputs, 19 native PNG plots, and no error outputs**. All figures shown in this README are the original Matplotlib PNG outputs embedded in that executed notebook; they are not redrawn or restyled.

---

## 1. Business question

> **How can a telecom operator identify customers at high risk of churn early enough to prioritize retention actions?**

The project is designed as a decision-support workflow rather than a classification exercise only. It connects descriptive churn patterns to model scores, threshold choices and practical retention prioritization.

```text
Business understanding
        ↓
Data quality
        ↓
Exploratory analysis
        ↓
Cleaning + feature engineering
        ↓
Train/test split
        ↓
Serverless-safe categorical encoding
        ↓
VectorAssembler
        ↓
Logistic Regression + Random Forest
        ↓
Model evaluation
        ↓
Threshold analysis
        ↓
Customer-level scoring
        ↓
Risk segmentation
        ↓
Retention actions
```

---

## 2. Environment and data loading

The run was executed with **Spark 4.1.0** in Databricks Free Edition / serverless compute. The notebook loads the CSV through a `dataset_path` widget pointing to a Databricks Volume.

The raw dataset observed by the notebook contains **7,043 rows and 21 columns**. The raw CSV itself is not included here because it was not embedded in the notebook supplied for publication; see [`data/README.md`](data/README.md).

---

## 3. Data quality audit

The important data-quality issue is `TotalCharges`:

- **11 blank values** are found;
- all 11 rows have `tenure = 0`;
- the customers are retained;
- blank historical charges are mapped to `0.0` before casting to double.

This avoids dropping brand-new customers solely because they do not yet have accumulated historical charges.

---

# 4. Exploratory Data Analysis

The notebook explicitly treats these results as **associations, not causal effects**.

## 4.1 Target distribution

| Churn | Customers | Share |
|---|---:|---:|
| No | 5,174 | 73.46% |
| **Yes** | **1,869** | **26.54%** |

![Customer churn distribution](assets/01_target_distribution.png)

## 4.2 Contract

| Contract | Customers | Churners | Churn rate |
|---|---:|---:|---:|
| **Month-to-month** | 3,875 | 1,655 | **42.71%** |
| One year | 1,473 | 166 | 11.27% |
| Two year | 1,695 | 48 | 2.83% |

![Churn rate by contract](assets/02_contract_churn.png)

## 4.3 Internet service

| Internet service | Customers | Churners | Churn rate |
|---|---:|---:|---:|
| **Fiber optic** | 3,096 | 1,297 | **41.89%** |
| DSL | 2,421 | 459 | 18.96% |
| No | 1,526 | 113 | 7.40% |

![Churn rate by internet service](assets/03_internet_service_churn.png)

## 4.4 Payment method

| Payment method | Customers | Churners | Churn rate |
|---|---:|---:|---:|
| **Electronic check** | 2,365 | 1,071 | **45.29%** |
| Mailed check | 1,612 | 308 | 19.11% |
| Bank transfer (automatic) | 1,544 | 258 | 16.71% |
| Credit card (automatic) | 1,522 | 232 | 15.24% |

![Churn rate by payment method](assets/04_payment_method_churn.png)

## 4.5 Tech support

| TechSupport | Churn rate |
|---|---:|
| **No** | **41.64%** |
| Yes | 15.17% |
| No internet service | 7.40% |

![Churn rate by TechSupport](assets/05_tech_support_churn.png)

## 4.6 Online security

| OnlineSecurity | Churn rate |
|---|---:|
| **No** | **41.77%** |
| Yes | 14.61% |
| No internet service | 7.40% |

![Churn rate by OnlineSecurity](assets/06_online_security_churn.png)

## 4.7 Senior customers

| SeniorCitizen | Churn rate |
|---|---:|
| **1** | **41.68%** |
| 0 | 23.61% |

![Churn rate by senior-citizen status](assets/07_senior_citizen_churn.png)

This descriptive gap warrants subgroup-performance and fairness monitoring before operational use.

## 4.8 Paperless billing

| PaperlessBilling | Churn rate |
|---|---:|
| **Yes** | **33.57%** |
| No | 16.33% |

![Churn rate by paperless billing](assets/08_paperless_billing_churn.png)

## 4.9 Partner

| Partner | Churn rate |
|---|---:|
| **No** | **32.96%** |
| Yes | 19.66% |

![Churn rate by partner status](assets/09_partner_churn.png)

## 4.10 Dependents

| Dependents | Churn rate |
|---|---:|
| **No** | **31.28%** |
| Yes | 15.45% |

![Churn rate by dependents](assets/10_dependents_churn.png)

## 4.11 Gender

| Gender | Churn rate |
|---|---:|
| Female | 26.92% |
| Male | 26.16% |

![Churn rate by gender](assets/11_gender_churn.png)

The descriptive difference by gender is very small in this dataset.

## 4.12 Tenure

| Group | Average tenure | Median tenure |
|---|---:|---:|
| Non-churn | 37.57 months | 38 |
| **Churn** | **17.98 months** | **10** |

![Average tenure by churn](assets/12_avg_tenure_by_churn.png)

## 4.13 Monthly charges

| Group | Average MonthlyCharges |
|---|---:|
| Non-churn | 61.27 |
| **Churn** | **74.44** |

![Average monthly charges by churn](assets/13_avg_monthly_charges_by_churn.png)

---

# 5. Cleaning and feature engineering

`customerID` is retained for customer-level scoring but excluded from the predictive feature vector.

Two business features are engineered:

- `num_services`: counts active services among eight telecom options;
- `charges_per_month`: `TotalCharges / tenure` when tenure > 0, otherwise `MonthlyCharges`.

---

# 6. Train / test split

```python
train_df, test_df = df.randomSplit([0.8, 0.2], seed=42)
```

| Dataset | Non-churn | Churn | Total |
|---|---:|---:|---:|
| Train | 4,148 | 1,486 | **5,634** |
| Test | 1,026 | 383 | **1,409** |

The split is performed before the categorical vocabulary is learned.

---

# 7. Serverless-safe categorical encoding

Databricks Free Edition limits the total size of Spark ML model objects cached in a serverless session. Instead of fitting many `StringIndexerModel` and `OneHotEncoderModel` objects, the notebook:

1. learns category levels from the **training set only**;
2. sorts those levels deterministically;
3. omits one baseline category per feature;
4. creates dummy columns directly with PySpark expressions;
5. uses `VectorAssembler` to create the final vector.

The executed model uses **32 predictive dimensions**. This design also prevents category-vocabulary leakage from the test set.

---

# 8. Evaluation framework

The notebook evaluates ROC-AUC, PR-AUC, Accuracy, Weighted F1, churn Precision, churn Recall, churn F1 and TN / FP / FN / TP.

For retention, **false negatives are especially important** because they represent actual churners who are not flagged.

---

# 9. Logistic Regression

```python
LogisticRegression(
    featuresCol="features",
    labelCol="label",
    maxIter=30,
    regParam=0.05,
    family="binomial",
    standardization=True,
)
```

| Metric | Value |
|---|---:|
| ROC-AUC | **0.8369** |
| PR-AUC | 0.6347 |
| Accuracy | 0.7921 |
| Weighted F1 | 0.7800 |
| Churn precision | 0.6642 |
| Churn recall | 0.4752 |
| Churn F1 | 0.5540 |
| TN / FP / FN / TP | 934 / 92 / 201 / 182 |

---

# 10. Threshold analysis — Logistic Regression

![Threshold trade-off — Logistic Regression](assets/14_threshold_analysis.png)

| Threshold | Precision | Recall | Churn F1 | FP | FN |
|---:|---:|---:|---:|---:|---:|
| 0.20 | 0.4798 | 0.8381 | 0.6103 | 348 | 62 |
| 0.25 | 0.5209 | 0.8120 | 0.6347 | 286 | 72 |
| **0.30** | **0.5583** | **0.7624** | **0.6446** | 231 | 91 |
| 0.35 | 0.5904 | 0.6736 | 0.6293 | 179 | 125 |
| 0.40 | 0.6114 | 0.6162 | 0.6138 | 150 | 147 |
| 0.45 | 0.6495 | 0.5614 | 0.6022 | 116 | 168 |
| 0.50 | 0.6642 | 0.4752 | 0.5540 | 92 | 201 |
| 0.55 | 0.7023 | 0.3943 | 0.5050 | 64 | 232 |
| 0.60 | 0.7383 | 0.2872 | 0.4135 | 39 | 273 |
| 0.65 | 0.7273 | 0.1880 | 0.2988 | 27 | 311 |
| 0.70 | 0.8298 | 0.1018 | 0.1814 | 8 | 344 |

In this **exploratory test-set analysis**, threshold **0.30** maximizes churn F1 and raises recall from **47.52% to 76.24%**.

This is **not a production threshold recommendation**. The production threshold should be selected on validation data using business costs, campaign capacity and customer value.

---

# 11. Customer-level churn scoring

| Probability | Risk segment |
|---:|---|
| `< 0.30` | Low |
| `0.30–0.60` | Medium |
| `0.60–0.80` | High |
| `>= 0.80` | Critical |

| Risk segment | Customers | Average probability |
|---|---:|---:|
| Critical | **9** | 0.8117 |
| High | **750** | 0.6750 |
| Medium | **1,869** | 0.4447 |
| Low | **4,415** | 0.1151 |

![Customer risk segmentation](assets/15_risk_segmentation.png)

These bands are operational illustrations, not calibrated probability guarantees.

---

# 12. Random Forest

```python
RandomForestClassifier(
    featuresCol="features",
    labelCol="label",
    numTrees=20,
    maxDepth=4,
    maxBins=32,
    seed=42,
)
```

| Metric | Value |
|---|---:|
| ROC-AUC | 0.8279 |
| PR-AUC | **0.6367** |
| Accuracy | 0.7814 |
| Weighted F1 | 0.7546 |
| Churn precision | **0.6943** |
| Churn recall | 0.3499 |
| Churn F1 | 0.4653 |
| TN / FP / FN / TP | 967 / 59 / 249 / 134 |

## Feature importance

![Random Forest — top feature importances](assets/16_rf_feature_importance.png)

| Rank | Feature | Importance |
|---:|---|---:|
| 1 | `tenure` | **0.1987** |
| 2 | `TotalCharges` | 0.1333 |
| 3 | `InternetService__Fiber_optic` | 0.1285 |
| 4 | `Contract__Two_year` | 0.0909 |
| 5 | `PaymentMethod__Electronic_check` | 0.0809 |
| 6 | `InternetService__No` | 0.0753 |
| 7 | `MonthlyCharges` | 0.0649 |
| 8 | `TechSupport__Yes` | 0.0475 |
| 9 | `Contract__One_year` | 0.0393 |
| 10 | `charges_per_month` | 0.0286 |

Feature importance describes **model usage, not causal impact**.

---

# 13. Final model comparison

| Metric | Logistic Regression | Random Forest |
|---|---:|---:|
| **ROC-AUC** | **0.8369** | 0.8279 |
| PR-AUC | 0.6347 | **0.6367** |
| **Accuracy** | **0.7921** | 0.7814 |
| **Weighted F1** | **0.7800** | 0.7546 |
| Churn precision | 0.6642 | **0.6943** |
| **Churn recall** | **0.4752** | 0.3499 |
| **Churn F1** | **0.5540** | 0.4653 |

![Final executed model comparison](assets/17_model_comparison.png)

### Champion: Logistic Regression

The notebook explicitly prints **“Champion by ROC-AUC: Logistic Regression.”** It also provides higher churn recall and churn F1 than the Random Forest at the default threshold.

## Confusion matrices at threshold 0.50

### Logistic Regression

![Logistic Regression confusion matrix](assets/18_logistic_confusion_matrix.png)

TN=934 · FP=92 · FN=201 · TP=182

### Random Forest

![Random Forest confusion matrix](assets/19_random_forest_confusion_matrix.png)

TN=967 · FP=59 · FN=249 · TP=134

The Random Forest generates fewer false positives, but it misses substantially more churners.

---

# 14. Business recommendations

| Priority | Evidence from executed EDA | Action to test |
|---|---|---|
| 1 | Month-to-month churn **42.71%** | Longer-contract incentives among high-risk customers |
| 2 | Median churner tenure **10 months** | Stronger first-year onboarding and proactive support |
| 3 | No TechSupport **41.64%** / No OnlineSecurity **41.77%** | Targeted service bundles |
| 4 | Electronic check **45.29%** | Investigate billing friction / engagement mechanisms |
| 5 | Senior customers **41.68%** | Personalized support with an appropriate fairness review |

These are **hypotheses based on observed associations**. Business impact should be validated experimentally.

---

# 15. Production roadmap

The notebook proposes MLflow experiment tracking, Model Registry, scheduled batch scoring, Delta prediction tables, validation-based cost-aware threshold calibration, probability calibration checks, data/prediction drift monitoring, fairness monitoring, retraining strategy and CRM integration.

---

# 16. Limitations

- static dataset;
- one random train/test split;
- limited model tuning;
- threshold exploration uses the test set;
- no out-of-time validation;
- probability calibration not formally assessed;
- no causal inference;
- no campaign-cost data;
- no deployed monitoring loop.

---

# Conclusion

The **Logistic Regression** is the final champion by ROC-AUC (**0.8369**) and has materially stronger churn recall than the Random Forest at threshold 0.50. The threshold analysis shows why the operating point matters: moving from 0.50 to an exploratory 0.30 threshold increases churn recall from **47.52% to 76.24%**, at the cost of more false positives.

That trade-off—not just the model ranking—is the core business lesson of this project.
