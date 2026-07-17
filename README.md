# ITSM Predictive Analytics Dashboard

Predicting SLA breach risk on 24,918 ITSM incidents using a comparative machine learning
pipeline (Logistic Regression, Random Forest, Gradient Boosting), surfaced in an
interactive Power BI dashboard for incident triage.

]**[Live Dashboard](https://app.powerbi.com/view?r=eyJrIjoiMDk3OThkZjAtZDY1YS00ZmY5LWJlNzAtOTZkNzQ2NzVlYTQ2IiwidCI6ImVlMTQyMGU0LWIzYTUtNGUwYi1hNTAxLTU2MTBiNTYyNmU5MCJ9)**


## Problem

IT service desks need to know which incidents are at risk of breaching their SLA
*before* they breach it, not after — early intervention is the only way to actually
prevent the breach rather than just report on it. This project builds a predictive
model that flags high-risk incidents at the point of ticket creation, using only
information available at intake (no post-resolution data), so it reflects a real,
deployable triage scenario.

## Dataset

[UCI Incident Management Process Enriched Event Log](https://archive.ics.uci.edu/dataset/498/incident+management+process+enriched+event+log)
— 141,712 event-log rows across 24,918 unique incidents, collapsed to one row per
incident (opening-snapshot features, final-state labels) to avoid data leakage.

## Approach

- **Feature engineering**: intake-time categorical fields (priority, category,
  assignment group, contact type, etc.), frequency-encoded for high-cardinality
  columns, plus time-of-open features (hour, day of week).
- **Two prediction targets**, trained and compared independently:
  - `sla_breach` — did the incident breach its SLA?
  - `reopened` — was the incident reopened after resolution?
- **Three classifiers compared head-to-head** on identical train/test splits:
  Logistic Regression, Random Forest, Gradient Boosting.

## Results

| Target | Best Model | Accuracy | Precision | Recall | F1 | ROC-AUC |
|---|---|---|---|---|---|---|
| SLA Breach | Gradient Boosting | 75.9% | 73.4% | 53.3% | 61.8% | **0.820** |
| SLA Breach | Random Forest | 72.0% | 59.4% | 74.1% | 65.9% | 0.806 |
| Reopened | (all models) | — | — | — | — | ~0.52–0.58 |

**The `reopened` target is reported as an honest negative finding, not hidden or
dressed up.** Only 1.1% of incidents were ever reopened, and intake-time features
carry almost no signal for it — reopening is driven by resolution quality, which by
definition isn't known yet when a ticket is created. All three models perform close
to random (ROC-AUC ~0.5) on this target, and that result is presented as-is.

**Top predictive features for SLA breach**: whether a knowledge article existed at
intake, assignment group, and subcategory.

## Dashboard

Three pages, built in Power BI:

1. **Executive Overview** — headline KPIs (total incidents, breach rate, compliance
   rate), top categories by volume and by breach count.
2. **Operational Analytics** — breach patterns by assignment group, location, and
   knowledge-article usage.
3. **Machine Learning Dashboard** — live model comparison, feature importance,
   and a sortable, color-coded table of the highest-risk incidents with model
   confidence scores.

## Tech Stack

- **Python** (pandas, scikit-learn) — data preparation, feature engineering, model
  training and evaluation
- **Power BI** (DAX, Power Query) — interactive dashboard and drill-through analysis
- **Dataset**: UCI Machine Learning Repository

## Repo Structure

```
├── python/
│   ├── 01_prepare_data.py          # raw event log -> one row per incident
│   ├── 02_feature_engineering.py   # encoding, time features
│   └── 03_train_models.py          # train + compare 3 models x 2 targets
├── ITSM_Predictive_Analytics.pbix
└── screenshots/
```

## Author

Kartik Kodilkar — [kartikkodilkar.org](https://kartikkodilkar.org)
