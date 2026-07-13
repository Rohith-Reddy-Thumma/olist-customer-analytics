# Limitations

Every analysis has gaps. These are mine.

---

## 1. The churn definition is a proxy, not the real thing

"Churned" here means a customer never placed a second order in the dataset. But the dataset ends in late 2018. A customer who placed their first order in October 2018 and their second order in January 2019 would be labeled "churned" in this analysis - incorrectly.

The churn label is only reliable for customers whose first order was early enough in the dataset that they had a reasonable window to return. Customers from late 2018 are almost certainly mislabeled.

A proper churn definition would set a fixed observation window — for example, "churned if no second order within 180 days of first purchase" - and only include customers who have been in the dataset long enough for that window to close.

---

## 2. The RFM segments are rule-based, not data-driven

The eight customer segments (Champion, Loyal Customer, At Risk, etc.) are defined by manually set thresholds on R, F, and M scores. These thresholds are reasonable but arbitrary. A different analyst might draw the boundaries differently and get different segment compositions.

A more defensible approach would use k-means clustering to let the data define natural groupings rather than imposing them manually. The current segmentation is useful for illustration but shouldn't be treated as ground truth.

---

## 3. The churn model has only 3 features

Recency, monetary, and average delivery days explain 71% of churn correctly. That's decent for three variables but leaves significant unexplained variance.

Features that were available but not included:
- Product category (a phone case buyer may return sooner than a sofa buyer)
- Payment method (installment buyers may behave differently than single-payment)
- Customer state / region (geographic differences in behavior)
- Number of items per order
- Review score (if available - a bad review is a strong churn signal)

Adding these would likely improve model accuracy meaningfully.

---

## 4. Class imbalance handling via downsampling loses data

To handle the 97/3 class imbalance, the majority class (churned) was downsampled to match the minority class (returned). This means the model was trained on about 5,600 rows out of 93,000 available. The other 87,000+ rows were discarded.

An alternative approach — SMOTE (Synthetic Minority Oversampling Technique) - generates synthetic examples of the minority class rather than discarding majority examples. This would allow the model to train on the full dataset and might produce better results.

---

## 5. The cohort analysis only covers 2017

The retention heatmap and revenue charts are filtered to 2017 cohorts for clarity. The full dataset spans 2016-2018. Including all years would give a more complete picture but would also introduce edge effects - 2016 cohorts have more months of data than 2018 cohorts, making comparisons across cohort years misleading without careful normalization.

---

## What would strengthen this analysis

- A proper time-based train/test split for the churn model
- K-means clustering for RFM segmentation
- A fixed observation window for the churn label
- Additional features in the predictive model (product category, review score)
- SMOTE instead of downsampling for class imbalance
- Full cohort analysis across all years with normalized time windows
