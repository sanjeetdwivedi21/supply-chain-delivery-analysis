# Supply Chain Performance Analysis: Late Delivery, Profitability & Predictive Alerts

End-to-end analysis of delivery operations for a global e-commerce company: where deliveries fail, what it costs, and a machine learning model that flags at-risk orders before they become late.

> **Headline:** 54.71% of 172,765 orders were delivered late. Shipping mode is the dominant cause (First Class 100% late, Second Class 79.8%), and a Random Forest model already identifies late orders with 74% accuracy.

**[Read the full business report (PDF, 16 pages)](Supply_Chain_Performance_Report.pdf)**

---

## Business Problem

A global e-commerce company manages shipping and delivery end to end for products such as sporting goods. Actual shipping times often deviate from the scheduled timelines, causing late deliveries and unpredictable order profitability.

**Goal:** analyse delivery operations, identify bottlenecks, and build a predictive system to reduce delays, optimise shipping decisions and improve profitability.

## Dataset

- **Source:** DataCo Supply Chain Dataset (`DataCoSupplyChainDataset.csv`, `latin-1` encoding). Download it from Kaggle and place it in the project root. The data is not included in this repository.
- **Raw size:** 180,519 rows x 53 columns, no duplicates.
- **Period:** January 2015 to January 2018 (order dates).

### Cleaning steps

| Step | Detail |
|---|---|
| Dropped columns | 33 columns removed: personal data (email, password, names, street), IDs, empty or redundant fields (e.g. `Product Description` 100% missing, `Order Zipcode` 86% missing) |
| Dropped rows | 7,754 orders with status `Shipping canceled` |
| Final dataset | **172,765 rows x 20 columns** |

### Derived metrics

- `Order Processing Time` = shipping date - order date (days)
- `Delay` = processing time - scheduled days
- `Is_Delayed` = `Delay > 0`
- `Profitability Flag` = Profit / Loss / Break-even (from `Order Profit Per Order`)
- Time features: order month, day of week, hour

> **Note on "late":** the KPIs use the computed definition `Delay > 0` (54.71%, 94,523 orders). The ML model predicts the dataset's own flag `Late_delivery_risk` (57.29%, 98,977 orders). The two are close but not identical.

---

## Key Results

| Metric | Value |
|---|---|
| Total orders analysed | 172,765 |
| Late deliveries | 94,523 (54.71%) |
| On-time deliveries | 45.29% |
| 90th percentile delay | 3 days |
| Profit on profitable orders | $7.5M |
| Net profit, all orders | $3.81M |
| Profit at risk (delayed orders) | $2.1M |
| Loss-making orders | 18.7% (32,295 orders) |
| Mean profit per order | $22.03 |
| Model accuracy (Random Forest) | 74% |

### Findings

1. **Shipping mode is the dominant bottleneck.** Delay rate by mode: Same Day 0%, Standard 39.8%, Second Class 79.8%, First Class 100%. Other dimensions (region, segment, department, payment type, status) move the delay rate by only about 1 to 4 points.
2. **Premium modes drive most delays.** First and Second Class are about 35% of orders but about 57% of late orders.
3. **Delays are a volume problem, not a unit-economics problem.** Mean profit per order stays at about $20 to $23 at every delay level.
4. **Geography is secondary.** Central Africa is the weakest region at 58.7%. Within it, `PAYMENT_REVIEW` orders are 80.0% late.
5. **Time effects are mild.** Peaks in Aug and Sep (55.4%) and Dec (55.2%); weekday differences are about 1.5 points; hourly rates range from roughly 52% to 57%.
6. **Delays are systemic.** 31% of orders are exactly 1 day late and 90% of delays are within 3 days.

---

## Machine Learning Model

**Task:** binary classification of `Late_delivery_risk` at order level.

- **Features (9):** scheduled shipment days, order month, order hour, and frequency-encoded `Type`, `Category Name`, `Customer Segment`, `Department Name`, `Order Region`, `Shipping Mode`
- **Split:** 80/20 stratified (138,212 train / 34,553 test), `random_state=42`
- **Imbalance handling:** SMOTE on the training set only (59,030 / 79,182 to 79,182 / 79,182)
- **Model:** `RandomForestClassifier` (default parameters)

| Class | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| 0 (On-time) | 0.68 | 0.73 | 0.70 | 14,758 |
| 1 (Late) | 0.79 | 0.75 | 0.77 | 19,795 |
| **Accuracy** | | | **0.74** | 34,553 |

Always predicting "late" would be right 57.3% of the time, so the model adds about 17 points of accuracy.

### Known limitations

- Scheduled days map one-to-one to shipping mode, so a large share of the accuracy likely comes from that single signal. Feature importance should be reported.
- One default-parameter model and one split; no cross-validation or tuning yet.
- Frequency encoding was computed before the split (it does not use the target, but should be fitted on the training set in production).

---

## Strategic Recommendations

| Priority | Action |
|---|---|
| Critical | Audit First Class and Second Class shipping capacity and carrier SLAs |
| High | Deploy the predictive alert system (pilot) |
| High | Resolve payment processing bottlenecks, with automated escalation of stalled reviews |
| Medium | Build seasonal surge capacity plans (Aug, Sep, Dec) |
| Medium | Default eligible orders to Standard Class |
| Medium | Investigate high-delay departments and regions |
| Low | Review pricing and discounts on loss-making orders (18.7%) |
| Low | Retrain the model quarterly and add carrier, weather and warehouse features |

**Illustrative impact:** First Class at 80% on-time and Second Class at 60% on-time would cut the late rate from 54.7% to about 34.7%. Reducing Standard Class delays to about 32% as well brings it to about 30%.

### Targets

| Priority area | Current | Target |
|---|---|---|
| Late delivery rate | 54.71% | < 30% within 12 months |
| First Class on-time rate | 0% | > 80% |
| Second Class on-time rate | 20.2% | > 60% |
| Predictive model accuracy | 74% | > 82% |
| Loss-making orders | 18.7% | < 12% |
| Profit at risk | $2.1M | Reduce by 40% |

---

## Repository Structure

```
.
├── README.md
├── supply_chain_delivery_analysis.ipynb   # full analysis
├── Supply_Chain_Performance_Report.pdf    # 16-page business report
└── DataCoSupplyChainDataset.csv           # not included, download separately
```

## Tech Stack

Python, pandas, NumPy, Matplotlib, Seaborn, scikit-learn, imbalanced-learn (SMOTE)

## How to Run

```bash
# 1. Clone the repo and enter it
git clone https://github.com/sanjeetdwivedi21/supply-chain-delivery-analysis.git
cd supply-chain-delivery-analysis

# 2. Install dependencies
pip install pandas numpy matplotlib seaborn scikit-learn imbalanced-learn jupyter

# 3. Place DataCoSupplyChainDataset.csv in the project root, then:
jupyter notebook supply_chain_delivery_analysis.ipynb
```

