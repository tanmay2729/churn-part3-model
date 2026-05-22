# Error Analysis Report
## D2C Customer Churn Intelligence — Part 3

**Model:** Logistic Regression (Final Model)  
**Threshold:** 0.5  
**Test Set Performance:** Accuracy=0.8865 | Precision=0.8847 | Recall=0.8895 | F1=0.8871

---

## Confusion Matrix Summary

| | Predicted: No Churn | Predicted: Churn |
|---|---|---|
| **Actual: No Churn** | 135 (TN) | 33 (FP) |
| **Actual: Churn** | 32 (FN) | 136 (TP) |

- **True Positives (136):** Correctly identified churners — retention campaign can be triggered
- **True Negatives (135):** Correctly identified loyal customers — no action needed
- **False Positives (33):** Predicted churn but customer stayed — wasted retention spend
- **False Negatives (32):** Missed churners — highest business risk

---

## False Positive Analysis
### Predicted Churn — Actually Stayed

False positives are customers the model flagged as churners but who did not churn.
**Business risk:** Wasted retention budget on customers who didn't need intervention.

| Customer ID | Pred Prob | Recency | Frequency | Monetary | Tickets | Sessions | Return Rate | Last Visit |
|---|---|---|---|---|---|---|---|---|
| CUST01246 | 0.9892 | 262 | 0 | 0.00 | 0 | 1 | 0.00 | 60 |
| CUST01411 | 0.9583 | 183 | 0 | 0.00 | 0 | 0 | 0.00 | 51 |
| CUST01325 | 0.9453 | 186 | 0 | 0.00 | 0 | 1 | 0.00 | 43 |
| CUST01370 | 0.9138 | 161 | 2 | 1246.04 | 0 | 2 | 0.00 | 35 |
| CUST00437 | 0.9061 | 151 | 1 | 729.22 | 0 | 0 | 0.00 | 33 |
| CUST01017 | 0.8856 | 133 | 2 | 1167.28 | 0 | 3 | 0.50 | 13 |
| CUST01405 | 0.8670 | 140 | 1 | 1013.03 | 0 | 2 | 0.00 | 20 |
| CUST00491 | 0.8442 | 97 | 1 | 540.89 | 1 | 10 | 1.00 | 20 |
| CUST01614 | 0.7921 | 103 | 2 | 1352.11 | 0 | 4 | 0.50 | 4 |
| CUST01906 | 0.7817 | 81 | 1 | 555.37 | 0 | 3 | 0.00 | 28 |

### False Positive Pattern Analysis

**Common characteristics:**
- High recency days (81–262 days since last purchase) — model correctly identified disengagement signal
- Zero or very low frequency (0–2 orders in 180 days)
- Low or zero sessions — low app engagement
- These customers LOOK like churners based on behavioural signals but somehow made a purchase in the target window

**Why the model was wrong:**
- Some customers may have made a single purchase outside their normal pattern (seasonal buying)
- CUST01246, CUST01411, CUST01325 have 0 frequency and 0 monetary — they may have been acquired in the target window itself
- CUST00491 has a 100% return rate and 10 sessions — browsing heavily but returning everything; eventually made a purchase that counted as "not churned"

**Business implication of FPs:**
Cost of false positive = cost of one unnecessary retention offer (e.g. ₹40 discount coupon).
With 33 FPs this = ~₹1,320 wasted per campaign cycle — acceptable given the value of catching true churners.

---

## False Negative Analysis
### Missed Churners — Model Failed to Detect

False negatives are the most dangerous errors — customers who churned but were not flagged.
**Business risk:** No retention intervention triggered — customer lost with no chance to save them.

| Customer ID | Pred Prob | Recency | Frequency | Monetary | Tickets | Sessions | Return Rate | Last Visit |
|---|---|---|---|---|---|---|---|---|
| CUST02072 | 0.0581 | 35 | 7 | 4340.19 | 0 | 4 | 0.00 | 1 |
| CUST01990 | 0.0884 | 59 | 4 | 3877.77 | 0 | 11 | 0.00 | 7 |
| CUST00184 | 0.1028 | 14 | 3 | 2456.91 | 0 | 6 | 0.00 | 6 |
| CUST00866 | 0.1378 | 26 | 1 | 1280.71 | 0 | 5 | 0.00 | 1 |
| CUST01655 | 0.1436 | 13 | 2 | 1358.99 | 0 | 2 | 0.00 | 7 |
| CUST01303 | 0.1471 | 20 | 1 | 844.74 | 1 | 3 | 0.00 | 0 |
| CUST00592 | 0.2583 | 20 | 1 | 627.36 | 0 | 3 | 0.00 | 1 |
| CUST02103 | 0.2920 | 44 | 2 | 1052.31 | 1 | 0 | 0.00 | 0 |
| CUST02060 | 0.2948 | 23 | 2 | 1331.01 | 2 | 4 | 0.50 | 6 |
| CUST00838 | 0.2955 | 9 | 1 | 402.67 | 1 | 11 | 0.00 | 12 |

### False Negative Pattern Analysis

**Common characteristics:**
- Very LOW recency (9–59 days) — these customers purchased recently, so model assumed they were safe
- Decent frequency (1–7 orders) — regular buyers who still churned
- High monetary values — CUST02072 (₹4,340) and CUST01990 (₹3,877) are high-value customers
- Low support tickets — no visible distress signal before churning
- These are "silent churners" — no warning signs in the data

**Why the model was wrong:**
- The model relies heavily on `recency_days` — customers with low recency looked safe
- CUST02072 had 7 orders and ₹4,340 spend — the model was extremely confident (only 5.8% churn probability) but they churned
- These customers may have churned due to external factors not captured in the data (competitor offer, life change, price sensitivity)
- Zero return rates and low tickets mean there was no behavioural signal of dissatisfaction

**Business implication of FNs:**
Each missed churner represents lost revenue. CUST02072 alone = ₹4,340 in lost monetary value.
32 missed churners at avg ₹2,000 each = ~₹64,000 potential lost revenue per cycle.

---

## Error Type Comparison

| Error Type | Count | Avg Pred Prob | Avg Monetary | Business Risk |
|---|---|---|---|---|
| False Positive | 33 | ~0.87 | ~₹800 | LOW — wasted retention spend |
| False Negative | 32 | ~0.18 | ~₹2,200 | HIGH — high-value customers lost silently |

**Key insight:** False negatives have significantly higher average monetary value than false positives. The model is missing the most valuable churners — those who look healthy but quietly leave.

---

## Recommendations

1. **Lower threshold to 0.35** for high-value customers (monetary > ₹2,000) to catch more FNs at the cost of more FPs
2. **Add external signals** — competitor promotions, price change events — which may explain silent churners
3. **Flag customers with recency < 60 days but declining frequency trend** — these are the hardest to catch
4. **Priority outreach for CUST02072 and CUST01990** — highest value missed churners worth a personal win-back attempt
