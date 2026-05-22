# Model Card
## D2C Customer Churn Prediction Model

---

## Model Overview

| Field | Details |
|---|---|
| **Model Name** | D2C Churn Prediction Model v1.0 |
| **Model Type** | Logistic Regression (sklearn Pipeline) |
| **Version** | 1.0 |
| **Date** | 2025-09-30 |
| **Owner** | Data Analytics Team |

---

## Intended Use

**Primary use case:** Identify customers likely to churn in the next 60 days so that the retention team can trigger targeted interventions before the customer leaves.

**Intended users:**
- Customer success team — to prioritise outreach
- Marketing team — to design targeted retention campaigns
- CRM system — for automated risk scoring

**Out-of-scope uses:**
- This model should NOT be used to deny customers any service or benefit
- Should NOT be used for customer acquisition decisions
- Should NOT be used beyond 60-day churn prediction horizon
- Should NOT be applied to customers with less than 30 days of history

---

## Data Used

**Primary dataset:** `rfm_modeling_snapshot.csv`  
**Snapshot date:** 2025-09-30  
**Target window:** 2025-10-01 to 2025-11-29 (60 days)  
**Total customers:** 2,400  
**Train/Val/Test split:** Pre-assigned by dataset (consistent evaluation)

**Features used (25 total):**

| Category | Features |
|---|---|
| Demographics | city_tier, age_group, acquisition_channel, loyalty_tier, preferred_category, marketing_consent |
| RFM | recency_days, frequency_180d, monetary_180d |
| Order behaviour | return_rate_180d, avg_discount_pct_180d, avg_rating_180d, category_diversity_180d |
| Support | ticket_count_90d, negative_ticket_rate_90d, avg_resolution_hours_90d |
| Engagement | sessions_30d, product_views_30d, cart_adds_30d, wishlist_adds_30d, abandoned_carts_30d, email_opens_30d, campaign_clicks_30d, last_visit_days_ago |
| Tenure | days_since_signup |

**Leakage prevention:**
- Used ONLY data available on or before 2025-09-30
- Excluded: churn_next_60d (target), split (assignment), customer_id (identifier), snapshot_date (constant)
- Post-snapshot orders (order_date > 2025-09-30) excluded from all features

---

## Model Approach

**Algorithm:** Logistic Regression with StandardScaler preprocessing in a sklearn Pipeline

**Why Logistic Regression:**
Three models were trained and compared — Logistic Regression, XGBoost, and Random Forest. Logistic Regression outperformed both on all metrics (ROC-AUC: 0.8856 vs 0.8822 and 0.8793). This is consistent with the dataset characteristics — pre-engineered RFM features with near-linear relationships to churn probability, and a relatively small sample size of 2,400 customers.

**Training approach:**
- StandardScaler fitted on training data only — prevents scaling leakage
- class_weight='balanced' to handle near-equal class distribution
- Pre-assigned train/validation/test splits used for consistent evaluation

**Decision threshold:** 0.5  
**Threshold justification:** At threshold 0.5, the model achieves balanced precision (0.8847) and recall (0.8895). Since false negatives (missed churners) carry higher business cost than false positives (wasted retention offers), a threshold of 0.5 was selected as it maximises F1 while maintaining recall above 0.88. Teams targeting very high recall (>0.93) can lower threshold to 0.35 at the cost of more false positives.

---

## Performance

### Test Set Metrics (Final)

| Metric | Value |
|---|---|
| Accuracy | 0.8865 |
| Precision | 0.8847 |
| Recall | 0.8895 |
| F1 Score | 0.8871 |
| ROC-AUC | 0.8856 |
| PR-AUC | 0.8778 |
| Selected Threshold | 0.5 |

### Confusion Matrix (Test Set)

| | Predicted: No Churn | Predicted: Churn |
|---|---|---|
| **Actual: No Churn** | 135 (TN) | 33 (FP) |
| **Actual: Churn** | 32 (FN) | 136 (TP) |

### Model Comparison

| Model | Accuracy | F1 | ROC-AUC |
|---|---|---|---|
| **Logistic Regression (selected)** | **0.8865** | **0.8871** | **0.8856** |
| XGBoost | 0.7976 | 0.7703 | 0.8822 |
| Random Forest | 0.8006 | 0.7649 | 0.8793 |

### Top 5 Features (by coefficient magnitude)

| Feature | Coefficient | Direction |
|---|---|---|
| recency_days | +1.7384 | Higher recency → more likely to churn |
| monetary_180d | -0.4483 | Higher spend → less likely to churn |
| return_rate_180d | +0.3446 | Higher returns → more likely to churn |
| ticket_count_90d | -0.3108 | More tickets → less likely to churn (engaged) |
| negative_ticket_rate_90d | +0.3047 | More negative tickets → more likely to churn |

---

## Limitations

1. **Small dataset:** Trained on 2,400 customers — performance may vary on larger or different customer populations
2. **Silent churners:** Model struggles to identify high-value customers who churn without any behavioural warning signs (see error analysis — CUST02072, CUST01990)
3. **Static snapshot:** Model uses a single snapshot date (2025-09-30) — does not capture trend information (e.g. declining frequency over time)
4. **No external signals:** Does not incorporate competitor activity, price changes, or macro-economic factors that may trigger churn
5. **Category-specific behaviour:** Single model for all product categories — churn patterns may differ significantly between Skin Care, Hair Care, Makeup etc.
6. **Temporal validity:** Model should be retrained every 3-6 months as customer behaviour patterns evolve

---

## Ethical Risks

1. **Differential treatment risk:** If certain demographic groups (age, city tier) are disproportionately flagged as churners, retention resources may be inequitably distributed. Regular fairness audits by segment are recommended.

2. **Privacy:** Model uses behavioural tracking data (sessions, clicks, app activity). Customers who opted out of marketing_consent should be excluded from automated campaign triggers.

3. **Reinforcement bias:** If the model consistently misses certain customer types (e.g. silent high-value churners), those customers will never receive retention offers, potentially making the business less equitable over time.

4. **Over-intervention risk:** Frequent retention campaigns triggered by model predictions may erode customer trust if they feel over-targeted. Campaign frequency should be capped per customer.

---

## Monitoring Needs

| Check | Frequency | Action if failed |
|---|---|---|
| ROC-AUC on new data | Monthly | Retrain if drops below 0.82 |
| Churn rate drift | Monthly | Retrain if actual churn rate shifts by >5% |
| Feature distribution shift | Quarterly | Investigate and retrain |
| False negative rate on high-value customers | Monthly | Lower threshold or add segment-specific model |
| Model fairness by age group and city tier | Quarterly | Audit and adjust if bias detected |

---

## When the Model Should NOT Be Used

- When customer has fewer than 30 days of history in the system
- When making decisions that affect customer pricing or service access
- As the sole decision-maker — always combine with human judgment for high-stakes interventions
- When the snapshot date is more than 3 months old without retraining
- For customers who have explicitly opted out of all marketing communications
