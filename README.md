# Cost-Sensitive Data Acquisition Under Budget Constraints

A budget-aware data acquisition framework that combines **Conformal Risk Control (CRC)**, **RGreedy** knapsack-style optimization, and **CMOS** coverage optimization to decide which incomplete records are worth completing under a fixed budget — instead of treating missing data purely as a statistical imputation problem.

> Course project — CP-610 Data Analysis, Master of Applied Computing, Wilfrid Laurier University (Dec 2025)

## Authors

| Faisal Ul Haque Mohammed 

## Problem Statement

Real-world datasets are frequently incomplete, and completing them — via third-party purchase or manual collection — costs money. Traditional missing-data techniques (mean imputation, regression imputation, GAIN) treat this purely as an inference problem and ignore the budget entirely. This project implements and evaluates the **Cost-Sensitive Data Acquisition (CDA)** framework, which instead treats data completion as a **budget-constrained optimization problem**: acquire the rows/fields with the highest estimated utility-to-cost ratio, under a formal statistical guarantee on selection error.

## Framework Design

The framework operates in two stages:

**Stage 1 — Risk-Aware Row Selection (Conformal Risk Control).**
A predictive model scores each incomplete row for its likelihood of being valuable. CRC then sets a selection threshold λ such that the expected rate of false selections stays below a user-defined risk level α. CRC is distribution-free and model-agnostic, so this guarantee holds without assumptions about the underlying data distribution.

**Stage 2 — Budget-Constrained Acquisition.**
- **RGreedy**: for row-wise acquisition (one option = one row), the problem reduces to a 0/1 knapsack solved via a greedy utility-to-cost ratio heuristic.
- **CMOS (Coverage Minimum Option Selection)**: for acquisition menus where one option can complete multiple rows/attributes at once, CMOS is used to minimize redundant coverage.

## Dataset

A synthetic customer dataset was generated (not a public/real dataset) specifically to stress-test the framework under controlled, reproducible conditions with known ground truth.

| Property | Value |
|---|---|
| Records | 1,000 |
| Features | 8 |
| Valuable customers (label_worth = 1) | 312 (31.2%) |
| Non-valuable customers (label_worth = 0) | 688 (68.8%) |
| Average acquisition cost | $7.51 ± $3.78 |
| Cost range | $2.00 – $20.00 |

**Feature schema:**

| Feature | Type | Description | Distribution / Range |
|---|---|---|---|
| customer_id | Integer | Unique identifier | 1–1,000 |
| age | Integer | Customer age | Uniform: 18–70 |
| income | Integer | Annual income (USD) | Normal: $30K–$130K (mean $80K) |
| past_purchases | Integer | Historical purchase count | Poisson (mean 8) |
| avg_order_value | Float | Average transaction value | Uniform: $20–$250 |
| missing_fields | Integer | Number of incomplete fields | Discrete: 1–4 |
| label_worth | Binary | Ground truth: valuable / not | 31.2% valuable |
| acquisition_cost | Float | Cost to complete the record | Linear in missing_fields |

Ground truth rule: a customer is labeled "valuable" if income > $70K, past purchases > 7, and average order value > $80.

Data generation included realistic correlations (age–income), non-random missingness (older customers more likely incomplete), linear cost-per-missing-field modeling, and ±15% noise injection to simulate measurement error — with quality controls for range validation, cost-consistency checks, and a controlled 30/70 class balance.

## Performance Analysis

**Budget utilization.** 76.8% of the total budget was consumed by the acquisition process, with the remaining 23.2% left unspent — indicating the utility threshold correctly stopped acquisition once remaining candidates fell below the cost-efficiency bar, rather than exhausting the budget indiscriminately.

**Rows acquired vs. budget spent.** The relationship between cumulative budget spent and rows acquired was linear (R² = 1.000), consistent with the low variance in per-row acquisition cost.

**Class distribution in acquired data.** The acquisition process preserved a reasonable class balance without collapsing toward only the majority class:

| Class | Samples Acquired |
|---|---|
| Class 1 (valuable) | 74 |
| Class 0 (not valuable) | 67 |

**Cost by completeness level:**

| Missing Fields | Average Acquisition Cost |
|---|---|
| 1 | $3.50 |
| 2 | $7.50 |
| 3 | $11.25 |
| 4 | $15.00 |

**RGreedy vs. CMOS — cumulative utility per dollar spent.** CMOS achieved consistently higher cumulative utility than RGreedy across the observed budget range, reflecting its advantage in menu-style acquisition where reducing coverage overlap matters more than the simple ratio-greedy heuristic RGreedy uses.

**Feature importance (Random Forest).**

| Feature | Importance |
|---|---|
| Income | 32% |
| Average Order Value | 25% |
| Past Purchases | 21% |
| Age | 12% |
| Missing Fields | 10% |

**Feature correlations.**
- Income ↔ Average Order Value: 0.62 (strong positive)
- Income ↔ Label Worth: 0.58 (moderate positive)
- Missing Fields ↔ Label Worth: −0.35 (moderate negative)

**Budget allocation zones identified:**

| Zone | Definition | Notes |
|---|---|---|
| High ROI | 1–2 missing fields, $3.50–$7.50 cost | Best cost-utility trade-off |
| Moderate ROI | Complete records at higher cost | Diminishing but positive returns |
| Low ROI | Highly incomplete records | Costs >$15 yield disproportionately low returns |

## Critical Reflections & Limitations

- Results demonstrate the framework's *basic behavior* on a controlled synthetic dataset — not a full theoretical re-implementation or validation against real acquisition data.
- CDA's effectiveness is dependent on accurate utility estimation, which is harder to guarantee on noisy or biased real-world data.
- The framework assumes acquisition sources are reliable and fairly priced, which may not hold in real data marketplaces.
- RGreedy and CMOS are heuristics — they trade solution quality for scalability, and this trade-off has not been benchmarked against an exact optimal solver.

## Tech Stack

Python, Pandas, NumPy, Scikit-learn, NetworkX, Jupyter Notebook


## Future Directions

- Extend CDA to task-aware objectives that optimize downstream model performance rather than dataset completeness alone
- Improve robustness to noisy or unreliable data sources
- Support multi-table, schema-heterogeneous acquisition
- Adaptive acquisition strategies that reallocate budget dynamically based on real-time feedback
- Validation against real-world (non-synthetic) acquisition datasets

Submitted for CP-610 Data Analysis, Master of Applied Computing program, Wilfrid Laurier University, Dec 20, 2025. Intended for academic and educational purposes.
