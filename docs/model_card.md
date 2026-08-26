# Model card

## Intended use

Customer churn risk ranking for a telecom retention use case. The notebook asks: **How can a telecom operator identify customers at high risk of churn early enough to prioritize retention actions?**

## Training setup

- Databricks Free Edition / serverless environment
- Spark **4.1.0**
- random split: **80% / 20%**, `seed=42`
- executed split: **5,634 train / 1,409 test**
- predictive dimensions: **32**
- category vocabulary learned from the training set only
- deterministic dummy encoding implemented with Spark column expressions

## Models

### Logistic Regression
- `maxIter=30`
- `regParam=0.05`
- `family="binomial"`
- `standardization=True`

### Random Forest
- `numTrees=20`
- `maxDepth=4`
- `maxBins=32`
- `seed=42`

## Executed test metrics

| Metric | Logistic Regression | Random Forest |
|---|---:|---:|
| ROC-AUC | **0.8369** | 0.8279 |
| PR-AUC | 0.6347 | **0.6367** |
| Accuracy | **0.7921** | 0.7814 |
| Weighted F1 | **0.7800** | 0.7546 |
| Churn precision | 0.6642 | **0.6943** |
| Churn recall | **0.4752** | 0.3499 |
| Churn F1 | **0.5540** | 0.4653 |

The notebook reports **Logistic Regression** as the champion by ROC-AUC. For retention, its higher churn recall and churn F1 are also operationally relevant.

## Confusion matrices at threshold 0.50

- Logistic Regression: TN=934, FP=92, FN=201, TP=182
- Random Forest: TN=967, FP=59, FN=249, TP=134

## Threshold analysis

The notebook explores thresholds from 0.20 to 0.70 on the test probabilities. Threshold **0.30** maximizes churn F1 in this exploratory test-set analysis:

- precision: **0.5583**
- recall: **0.7624**
- churn F1: **0.6446**
- FP: **231**
- FN: **91**

This is explicitly **not a production threshold recommendation**. Production threshold selection should be performed on validation data using business costs, capacity and customer value.

## Risk segmentation

Illustrative score bands used in the notebook:

- Low: `< 0.30`
- Medium: `0.30–0.60`
- High: `0.60–0.80`
- Critical: `>= 0.80`

Executed distribution: Low 4,415; Medium 1,869; High 750; Critical 9.

## Limitations

- static dataset
- one random train/test split
- limited model tuning
- threshold exploration performed on the test set
- no out-of-time validation
- probability calibration not formally assessed
- no causal inference
- no campaign-cost data
- no production monitoring loop

Feature importance and EDA associations must not be interpreted as causal effects.
