# Analyst Notes — What I Learned Building This

Not a summary of findings — that's in the README. This is the honest version of how the analysis actually went.

---

## What surprised me most

I went in expecting a retention problem. When a business asks "who are our customers and are they coming back" the assumption is that some cohorts retain better than others and the job is to find which ones and why.

The data said something different. Less than 1% retention across every single cohort, every single month, with almost no variation. That's not a retention problem — that's the business model. Olist sells furniture and appliances. Nobody buys a couch twice in the same year.

That single realization changed the entire framing of the project. The interesting questions shifted from "how do we improve retention" to "who are the rare customers who DO come back, and what made them different." That's how the RFM segmentation and churn prediction became the most valuable parts of the analysis.

---

## The query that slowed me down most

The cohort retention matrix. The logic itself is straightforward — find each customer's first order month, calculate how many months later each subsequent order happened, pivot into a matrix. But the `transform('min')` step is easy to get wrong.

```python
# this is the line that trips people up
df['cohort'] = df.groupby('customer_unique_id')['order_month'] \
                  .transform('min')
```

The key is `transform` rather than `agg`. If you use `agg` you collapse the DataFrame to one row per customer and lose all the order-level detail you need for the rest of the analysis. `transform` calculates the minimum per group but writes the result back to every row in the original DataFrame — so each order row now knows what cohort it belongs to. Took a bit to get right.

---

## The device bucketing equivalent — RFM scoring

In this project the equivalent of the Yammer device bucketing problem was the RFM scoring. The naive approach is to hardcode the segment rules:

```python
if r >= 4 and f >= 4 and m >= 4: return 'Champion'
```

This works but it's brittle. The thresholds are arbitrary and the segments will shift completely if the data distribution changes. A more robust approach would be to use k-means clustering to let the data define the natural segments rather than imposing them manually. I'd do that in a follow-up — it would make the segmentation defensible rather than rule-based.

---

## The class imbalance problem

The churn model had a 97/3 class split — 97% of customers churned, 3% returned. If you train a model on that imbalance without fixing it, the model learns to just predict "churned" for everyone and gets 97% accuracy. That sounds great until you realize it's completely useless — it never identifies the customers who will actually return.

The fix was downsampling — taking a random sample of the majority class (churned) to match the size of the minority class (returned), then training on the balanced dataset. This drops overall accuracy to 71% but the model now actually identifies returners rather than ignoring them.

That tradeoff — lower headline accuracy in exchange for a model that's actually useful — is one of the most important practical lessons in machine learning for analysts.

---

## What the feature importance told me

Monetary value at 78.8% importance was the dominant predictor of whether a customer returns. That's a stronger signal than I expected. The implication is straightforward: Olist's highest-spend customers are its most loyal customers, and the business should treat them accordingly — faster delivery, proactive communication, priority support.

Delivery speed at 11.6% was the operational insight. It's the one lever the business can pull that directly affects whether a customer comes back. A customer who waited 20 days for their first delivery is measurably less likely to place a second order than one who waited 7 days.

---

## What I'd do differently

**Use k-means for RFM segmentation** instead of manual rules. Let the data define the clusters rather than imposing arbitrary thresholds.

**Add product categories as a feature** in the churn model. A customer who bought a phone case might return in a month. A customer who bought a refrigerator almost certainly won't return for years. The category of the first purchase is probably a strong predictor that I didn't include.

**Time-series validation** on the churn model. The current train/test split is random — some 2016 orders end up in the test set, some 2018 orders in the training set. A proper time-based split (train on all orders before a cutoff date, test on orders after) would give a more honest estimate of real-world performance.
