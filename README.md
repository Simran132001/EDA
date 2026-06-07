# Financial Loan Credit Risk Analytics (2M Records)

An end-to-end data science case study executing an analytical pipeline on **2,000,000 loan records**. This project covers multi-file relational data ingestion, structural quality remediation, advanced feature engineering, and statistical modeling to analyze credit defaults and Loss Given Default (LGD).

Detailed findings can be referenced in the [EDA_Executive_Summary.pdf](EDA_Executive_Summary.pdf) file.

---

## 🛠️ Tech Stack & Relational Data
* **Tools:** `Python`, `Pandas`, `NumPy`, `SciPy`, `Statsmodels`, `Scikit-learn`, `Seaborn`, `FastParquet`
* **Data Scale:** Unified Master DataFrame (2,000,000 x 60+ columns) sequentially merged from **9 separate child CSV files** via `loan_id`
* **Core Domains:** Loan mechanics, bureau performance (CIBIL), regional macroeconomics (RBI Repo, inflation), collateral assets, and repayment tracking

---

## 🧹 Key Pipeline Stages

### 1. Advanced Data Quality Remediation
* **Remediated 8 Defect Categories:** Fixed sign errors for negative loan amounts and negative incomes, treated decimal overflows where Rate > 100%, capped debt-to-income (DTI > 100%), clipped out-of-bound bureau and utilization ranges, and dropped duplicate primary keys.
* **Missing Value Classification:** Characterized missingness into **MNAR** (imputed via sentinel value `999` for clean profiles), **MAR** (grouped median imputation by employment title or home ownership), and **MCAR** (global median imputation).
* **Outlier Strategy:** Applied **Winsorization** at the 1st and 99th percentiles to highly skewed columns and Log(1+x) transformations to satisfy OLS normality assumptions.

### 2. Feature Engineering (10 New Metrics)
Engineered strategic banking features to better isolate credit danger, including:
* `emi_to_income_ratio`: Repayment burden relative to monthly income, serving as the top LGD predictor.
* `delinq_severity_score`: Recency-weighted delinquency index.
* `enq_velocity_score`: Short-term credit hunger and financial distress proxy.
* `collateral_coverage_ratio`: Recovery buffer strength based on asset value.

### 3. Statistical Modeling & Diagnostics
* **Multicollinearity:** Iteratively calculated **Variance Inflation Factors (VIF)**, dropping highly collinear features to maintain a stable threshold of VIF <= 10.
* **Linear Failure Mode:** Baseline OLS and regularized linear models (`Ridge`, `Lasso`, `ElasticNet` via `GridSearchCV`) resulted in a near-zero Test R². 
* **Conclusion:** Because LGD exhibits a **bimodal distribution** (the data peaks near 0% and ~80%), loss magnitude is highly non-linear and requires production-grade **tree-based ensemble models** (like LightGBM or Gradient Boosting) rather than linear options.

---

## 📈 Top Business Insights
* **Severe Class Imbalance:** **25:1 ratio** consisting of ~96.12% performing loans vs. ~3.88% defaults. Standard accuracy metrics are highly misleading; SMOTE or class-weight balancing is required.
* **The Grade C-D Boundary:** Risk increases monotonically by grade, but the single sharpest jump occurs between Grade C and D (**+9.4 percentage points**), defining the bank's critical risk-pricing boundary.
* **The CIBIL Limitation:** Performing borrowers average only 21 points higher than defaulters. Due to heavy distribution overlap and a small Cohen's d effect size, CIBIL cannot be used as a standalone filter.

---

## 🏦 Strategic Corporate Recommendations
1. **Hard Credit Floors:** Establish a hard floor of **680 CIBIL** to reduce prospective NPA formation by an estimated **18-22%**.
2. **Debt Service Caps:** Implement a strict **50% EMI-to-Income cap**, as exceeding this threshold results in a **~31% higher loss severity**.
3. **Risk-Based Pricing:** Levy a **150 bps risk premium** on Grade D–G products to account for the major default rate spike without hurting origination volume.
4. **Sectoral Origination Caps:** Cap lending volumes quarterly for high-risk loan purposes (small businesses, car loans) which default at **2.8x the portfolio average**.
5. **Collateral Mandates:** Enforce a collateral mandate for loans > 5 Lakh in high-risk states, as a collateral coverage ratio > 1.2 cuts realized loss severity (LGD) by **40%**.
