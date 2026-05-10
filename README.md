# 🧪 Quantium Retail A/B Testing — Trial Store Evaluation

A rigorous **A/B testing analysis** simulating a real-world data analyst task at **Quantium**, a leading data science consultancy. The objective is to evaluate whether a new store layout trialled in three pilot stores (77, 86, and 88) led to a statistically significant uplift in chip sales and customer counts — and whether the trial should be rolled out to the wider store network.

\---

## 📌 Project Overview

The trial ran from **February 2019 to April 2019** across three stores. The analysis uses pre-trial data (before February 2019) to find the best matching control store for each trial store, then applies statistical testing to measure the true effect of the new layout.

**Dataset:** `QVI\_data.csv` — a cleaned, merged dataset produced from the EDA phase, containing transaction-level records with store numbers, loyalty card IDs, product quantities, and sales amounts.

**Trial stores and their matched controls:**

|Trial Store|Control Store|
|-|-|
|77|233|
|86|155|
|88|237|

\---

## 🔄 Main Steps

### 1\. 📥 Data Loading \& Date Parsing

Load the dataset and convert the `DATE` column from string to a proper datetime format. A `YEARMONTH` integer column (e.g. `201902` for February 2019) is created to allow easy monthly grouping and period comparisons.

### 2\. 📊 Building Monthly KPIs Per Store

Aggregate transaction data at the **store × month** level to create a performance table with the following metrics:

|Metric|Description|
|-|-|
|`totSales`|Total revenue from chip sales|
|`nCustomers`|Unique loyalty card holders (customers)|
|`nTransactions`|Unique transactions|
|`totalQty`|Total units sold|
|`nTxnPerCust`|Average transactions per customer|
|`nChipsPerTxn`|Average units per transaction|
|`avgPricePerUnit`|Average price paid per chip unit|

Only stores with a **full 12 months of history** are kept to ensure fair comparisons. Stores with gaps in their record are excluded.

### 3\. 🔍 Control Store Selection

For each trial store, the best matching control store is selected using a **dual-scoring system** applied on the pre-trial window only:

#### Correlation Score

Measures whether a candidate store moves **in the same pattern** as the trial store over time using the **Pearson correlation coefficient**. A high correlation means both stores respond to the same seasonal and trend drivers — making the comparison fair when an external event (like a holiday) affects one store.

#### Magnitude Score

Measures whether a candidate store is **roughly the same size** as the trial store. Two stores can be perfectly correlated in shape but one might be twice as large. The absolute monthly difference is normalised to produce a 0–1 score where 1 means identical size.

#### Final Score

Both metrics are computed for **total sales** and **number of customers** independently, then combined into a single `finalControlScore`:

```
finalControlScore = 0.5 × scoreNSales + 0.5 × scoreNCust
```

where each score is itself:

```
score = 0.5 × correlation + 0.5 × magnitude
```

The store with the highest `finalControlScore` is selected as the control.

### 4\. ✅ Visual Validation of Control Stores

Before running any statistical test, the selected trial-control pairs are plotted side by side across the pre-trial months to visually confirm that their sales and customer trends track each other closely. This is a sanity check — if the lines diverge in the pre-trial period, the control store is not a valid comparison.

### 5\. 📐 Scaling the Control Store

Even a well-matched control store may be slightly larger or smaller than the trial store in absolute terms. To remove this size gap, the control store's values are scaled using a pre-trial scaling factor:

```
scalingFactor = sum(trial pre-trial metric) / sum(control pre-trial metric)
```

After scaling, the pre-trial totals are identical between trial and control, so any gap that opens during the trial reflects a **real treatment effect** rather than a pre-existing size difference.

### 6\. 📉 Statistical Testing — t-test with Confidence Intervals

The percentage difference between the scaled control and the trial store is computed each month. The **pre-trial monthly differences** establish a baseline mean and standard deviation, which are used to construct a **95% confidence interval**.

For each trial month (Feb, Mar, Apr 2019):

* If the trial store's value falls **outside** the 5–95% confidence band → the difference is **statistically significant**
* Significance in **at least 2 out of 3** trial months is required to declare a meaningful uplift

This approach avoids a single-month anomaly driving the conclusion.

\---

## 📋 Results \& Interpretation

### Store 77 — ✅ Significant Uplift

Both **total sales** and **number of customers** exceeded the confidence interval in at least 2 of 3 trial months. The new store layout had a clear and statistically significant positive effect.

### Store 86 — ⚠️ Partial Uplift

**Customer counts** increased significantly across all 3 trial months, but **sales did not** increase proportionally. This suggests a pricing promotion may have been run in-store during the trial period, attracting more customers at lower margins. This anomaly should be investigated with the Category Manager before including Store 86 in the rollout decision.

### Store 88 — ✅ Significant Uplift

Like Store 77, both **total sales** and **customer counts** were significantly higher in at least 2 of 3 trial months. The new layout delivered a strong positive effect.

\---

## 💡 Overall Conclusion

The trial demonstrates a **significant positive effect on chip sales and customer counts** in two out of three stores (77 and 88). Store 86 is an anomaly likely caused by a confounding in-store promotion. The recommendation is to:

* **Roll out** the new layout to the broader store network based on results from stores 77 and 88
* **Investigate** Store 86 further with the Category Manager before including it in the conclusion

\---

## 🛠️ Tech Stack

|Library|Purpose|
|-|-|
|`pandas`|Data manipulation and monthly aggregation|
|`numpy`|Numerical operations and scaling|
|`scipy.stats`|Pearson correlation and t-test|
|`matplotlib`|Trend and confidence interval visualisations|

\---

## 📚 Context

This project is **Part 2** of the **Quantium Data Analytics Virtual Experience Program** on Forage. It builds directly on the cleaned dataset produced in Part 1 (EDA) and simulates the type of controlled experiment evaluation a junior analyst would conduct to support a business recommendation.

