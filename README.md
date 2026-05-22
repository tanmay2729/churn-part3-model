# Part 3: Churn Prediction Model & Model Card
## D2C Customer Churn Intelligence & Retention API — Capstone

---

## Overview

This repository contains Part 3 of the D2C Customer Churn Capstone project.

The objective is to build, evaluate, interpret, and document a churn prediction model that identifies customers likely to churn in the next 60 days.

---

## Repository Structure

```
churn-part3-model/
│
├── churn_model.ipynb       ← Main modeling notebook
├── model.pkl               ← Saved final model (Logistic Regression Pipeline)
├── label_encoders.pkl      ← Saved label encoders for categorical features
├── metrics.json            ← All model metrics in JSON format
├── error_analysis.md       ← FP/FN analysis with 10 specific customer examples
├── model_card.md           ← Structured model card
├── requirements.txt        ← Python dependencies
├── charts/                 ← All saved chart outputs
│   ├── chart1_model_comparison.png
│   ├── chart2_threshold_analysis.png
│   ├── chart3_confusion_matrix.png
│   └── chart4_feature_importance.png
└── README.md
```

---

## Dataset

The dataset package is available here:  
[Google Drive Dataset Link](https://drive.google.com/drive/folders/1PmLapJI1VSDgvl_AxARNKwM1MCd3WFX0?usp=sharing)

Download and place all CSV files in a `data/` folder before running the notebook.

**Primary file used:** `rfm_modeling_snapshot.csv` — pre-built leakage-free feature table

---

## Setup Instructions

```bash
# 1. Clone the repository
git clone https://github.com/tanmay2729/churn-part3-model.git
cd churn-part3-model

# 2. Create and activate virtual environment
python -m venv venv
venv\Scripts\activate        # Windows
source venv/bin/activate     # Mac/Linux

# 3. Install dependencies
pip install -r requirements.txt

# 4. Download dataset and place CSVs in data/ folder

# 5. Launch Jupyter and run the notebook
jupyter notebook churn_model.ipynb
```

---

## Loading the Saved Model

```python
import pickle

# Load model pipeline
with open('model.pkl', 'rb') as f:
    model = pickle.load(f)

# Load label encoders
with open('label_encoders.pkl', 'rb') as f:
    label_encoders = pickle.load(f)

# Predict on new data
# new_data must have same columns as ALL_FEATURES in notebook
proba = model.predict_proba(new_data)[:, 1]
pred  = (proba >= 0.5).astype(int)
```

---

## Model Summary

| Field | Details |
|---|---|
| Final Model | Logistic Regression (sklearn Pipeline) |
| Baseline Model | Logistic Regression |
| Stronger Models Tested | XGBoost, Random Forest |
| Why LR Selected | Outperformed both on all metrics |
| Decision Threshold | 0.5 |
| Dataset | rfm_modeling_snapshot.csv |
| Split | Pre-assigned (train/validation/test) |

---

## Test Set Performance

| Metric | Value |
|---|---|
| Accuracy | 0.8865 |
| Precision | 0.8847 |
| Recall | 0.8895 |
| F1 Score | 0.8871 |
| ROC-AUC | 0.8856 |
| PR-AUC | 0.8778 |
| TP | 136 |
| FP | 33 |
| TN | 135 |
| FN | 32 |

---

## Top Features

| Feature | Direction | Interpretation |
|---|---|---|
| recency_days | + | Higher recency = more likely to churn |
| monetary_180d | - | Higher spend = less likely to churn |
| return_rate_180d | + | Higher returns = more likely to churn |
| sessions_30d | - | More sessions = less likely to churn |
| negative_ticket_rate_90d | + | More negative tickets = more likely to churn |

---

## Leakage Prevention

- Used ONLY `rfm_modeling_snapshot.csv` — all features pre-computed as of 2025-09-30
- Excluded: `churn_next_60d`, `split`, `customer_id`, `snapshot_date`
- StandardScaler fitted on training data only via Pipeline
- Pre-assigned split column used — no random re-splitting

---

## Notes

- `data/` folder excluded from version control via `.gitignore`
- `label_encoders.pkl` must be loaded alongside `model.pkl` for correct preprocessing
- Model should be retrained every 3-6 months as behaviour patterns evolve
