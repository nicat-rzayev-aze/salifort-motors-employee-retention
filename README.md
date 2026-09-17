# Salifort Motors: Employee Retention Predictive Modeling & Strategic Insights
**As Google Advanced Data Analytics Professional Certificate Capstone Project**


## Executive Summary & Business Problem
Salifort Motors, a fictional automotive and engineering firm, has been grappling with high employee turnover. Unplanned attrition incurs high recruitment, onboarding, and training costs, while disrupting team continuity and draining critical institutional knowledge.

The HR department leadership commissioned this study to:
1. Identify the fundamental drivers pushing valuable employees out of the company.
2. Build an end-to-end classification model to identify retention risks before an employee departs.
3. Formulate concrete, data-backed operational interventions to improve retention and maintain balanced workplace demands.


## PACE Methodology Framework
The project strictly follows Google's **PACE** (Plan, Analyze, Construct, Execute) project lifecycle:
* **Plan:** Formulate the research inquiry, identify stakeholders (HR leadership, Executive board), address ethical data usage concerns, and establish business-centric evaluation metrics (prioritizing Recall/F1-score over raw Accuracy).
* **Analyze:** Systematically clean the dataset, resolve high deduplication volumes, check outliers, and execute extensive multivariate exploratory data analysis (EDA).
* **Construct:** Execute feature engineering (ordinal & one-hot encoding), partition stratified training/testing subsets, develop baseline parametric (Logistic Regression) and non-parametric ensemble (Random Forest) models.
* **Execute:** Quantify performance gaps, compute model-agnostic and tree-based feature importances, and translate algorithmic findings into executive-level recommendations.


## Data Cleaning & Preprocessing
The primary dataset (`HR_capstone_dataset.csv`) contains **14,999 observations** across **10 organizational variables**.

* **Null Values:** Zero missing entries detected across all features.
* **Duplicate Slices:** Identified and eliminated **3,008 duplicate records** (~20% of raw inputs), bringing the active dataset to **11,991 unique employee profiles**. Retaining these records would have caused severe data leakage and artificial over-optimism in evaluation.
* **Outlier Strategy:** Anomalies in `time_spend_company` (long-tenure employees > 5 years) were mathematically verified using the Interquartile Range ($IQR = Q3 - Q1$, $Upper = Q3 + 1.5 \times IQR$). These records were kept intact as they reflect real, seasoned organizational talent rather than measurement errors.
* **Feature Encoding:**
  * `salary`: Ordinal transformation (`low: 0`, `medium: 1`, `high: 2`).
  * `department`: One-Hot Encoded via `pd.get_dummies(drop_first=True)` to avoid the dummy variable trap.
  * Target split: Stratified 75/25 train-test split (`stratify=y`) preserving the 83.4% / 16.6% class distribution.


## Exploratory Data Analysis & Discovery of 3 Churn Archetypes
Multi-dimensional cross-examination of workload metrics against satisfaction uncovered three distinct clusters among exiting personnel:

1. **The Overworked & Burned-Out Group (Extreme Overload):**
   * Metrics: Assigned to 6 or 7 projects; working 240–310 monthly hours; satisfaction plummeted below 0.15.
   * Finding: **100% of employees assigned to 7 projects left the company.** Over-allocation is a deterministic cause of departure.
2. **The Underutilized Group (Lack of Engagement):**
   * Metrics: Assigned to only 2 projects; logging ~130–160 hours/month; satisfaction clustered around 0.40; low performance evaluations (~0.50).
   * Finding: Sub-optimal task delegation leads to disengagement and voluntary turnover.
3. **The Poached High-Performers (Unrewarded Stars):**
   * Metrics: Assigned to 4–5 projects; high monthly hours (~220–260); top evaluation scores (median >0.90); high satisfaction (0.70–0.90).
   * Finding: Top-tier contributors leaving despite being satisfied, strongly tied to prolonged tenure (3–5 years) without promotional recognition.

---

## Machine Learning Architecture & Benchmark

Given the asymmetric costs of workforce attrition, **False Negatives** (failing to identify an employee who will leave) are substantially costlier than **False Positives** (proactively checking in on an employee who intended to stay). Therefore, **Recall** and **F1-Score** serve as the governing optimization metrics.

### Model Performance Matrix

| Metric | Logistic Regression (Baseline) | Random Forest (Ensemble) | Performance Delta |
| :--- | :---: | :---: | :---: |
| **Accuracy** | 83% | **99%** | +16% |
| **Precision (Class 1 - Left)** | 0.50 | **0.99** | +49% |
| **Recall (Class 1 - Left)** | **0.18** | **0.92** | **+74%** |
| **F1-Score (Class 1 - Left)** | 0.27 | **0.95** | +68% |

### Why Logistic Regression Failed:
Logistic Regression's linear decision boundary fundamentally assumes monotonic relationships (e.g., more hours = proportionally higher or lower exit rate). Because attrition spikes at **both extremes** (very low hours AND very high hours), the linear hyperplane fails, resulting in a disastrous 18% Recall rate.

### Why Random Forest Succeeded:
Random Forest operates via non-linear conditional thresholding (`if hours > 240 AND projects >= 6 then Churn`). Its bagging mechanism and multi-tree consensus isolated the precise churn clusters identified in EDA, capturing 92% of all actual exits with a 99% precision rate.

---

## Feature Importance Ranking
Tree-based Gini-impurity evaluation established the hierarchy of retention drivers:

1. **`satisfaction_level`** (~32% impact): Dominant predictor of turnover trajectory.
2. **`number_project`** (~24% impact): Key operational trigger reflecting workload distribution.
3. **`time_spend_company`** (~18% impact): Critical career inflection mark (highest departure risk at 3–5 years).
4. **`average_monthly_hours`** (~16% impact): Chronic overwork indicator.
5. **`last_evaluation`** (~7% impact): Differentiates between underperformers and poached talent.
6. **`salary` & `department`** (<3% combined): Minor statistical influence on churn propensity, proving compensation alone does not rectify poor operational pacing.


## Strategic Recommendations for HR Leadership

1. **Enforce Project Assignment Thresholds:**
   * Institute a strict hard cap of **maximum 4 to 5 concurrent projects** per employee.
   * Completely eliminate assigning 6 or 7 projects; reallocate tasks across larger cross-functional pools.
2. **Operational Overtime Controls:**
   * Configure internal payroll/HR alerts when an employee logs in excess of **200 hours per calendar month**.
   * Normalize hours through mandatory compensatory time off or supplementary staffing.
3. **Targeted 3–5 Year Progression Reviews:**
   * Build targeted compensation and role-growth pathways for high performers (`last_evaluation` > 0.80) at the 3-year tenure mark to mitigate external market poaching.
4. **Re-engagement Framework for Underutilized Staff:**
   * Implement skill audits and mentorship structures for individuals staffed on only 2 projects to lift engagement and evaluation scores before alienation leads to resignation
