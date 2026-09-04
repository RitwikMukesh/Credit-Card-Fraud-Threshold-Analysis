# Credit Card Fraud Detection — Threshold Trade-Off Analysis

An end-to-end fraud detection pipeline built on 284,807 real, anonymized credit card transactions — SQL for anomaly scoring, Python for threshold evaluation against ground truth.

## Methodology

**1. Finding the real signal.**
The dataset's `Amount` field looks like an obvious fraud indicator, but it isn't — average fraud amount ($122) is actually *higher* than average legitimate amount ($88), and the largest transactions in the dataset (up to $25,691) are never fraud. Flagging on amount deviation alone returns zero true positives.

The real signal lives in the anonymized PCA components. Correlation analysis against the `Class` label surfaced four features with the strongest relationship to fraud:

| Feature | Correlation with fraud |
|---|---|
| V17 | -0.33 |
| V14 | -0.30 |
| V12 | -0.26 |
| V10 | -0.22 |

**2. Scoring every transaction.**
An anomaly score was computed per transaction as the sum of absolute deviations of V17, V14, V12, and V10 from their dataset-wide means, using SQL window functions and CTEs. Ranking by this score alone returns a **100% fraud hit rate in the top 20 transactions** — a strong early signal that this feature set carries real predictive power.

**3. Evaluating against ground truth.**
Rather than picking one threshold and calling it done, a full sweep was run in Python across the score distribution (90th to 99.99th percentile), computing precision, recall, and false-positive rate at each point. At a Z-score threshold of ~24 (99.9th percentile), the model catches 47.6% of all fraud at 82.1% precision, with a false-positive rate of just 0.018%. At a more balanced threshold (~19), precision and recall converge around 63.6% each.

**4. A known, deliberate limitation.**
This dataset has no account or cardholder ID — only `Time`, `Amount`, and 28 anonymized PCA features. That means true per-account velocity checks (flagging rapid repeated transactions on the same card) aren't possible here, and this project doesn't fake that signal. The anomaly score is built entirely on transaction-level behavioral features instead.

## Tech stack

SQL (SQLite) · Python (Pandas, NumPy) · Tableau

## Repository structure

```
├── transactions_scored.csv     # All 284,807 transactions with computed anomaly_score
├── threshold_sweep.csv         # Precision/recall/FPR at 43 threshold points
├── credit_card_fraud.twbx      # Tableau packaged workbook
└── README.md
```
