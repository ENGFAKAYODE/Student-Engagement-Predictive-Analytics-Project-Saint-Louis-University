# Student-Engagement-Predictive-Analytics-Project-Saint-Louis-University
This project focuses on the intersection of EdTech and Data Science by analyzing student engagement data from the Saint Louis University (SLU) Wise Opportunity platform. The workflow involved cleaning and pre-processing complex behavioral datasets to uncover trends in user drop-offs. Using Machine Learning, I developed a predictive model to identify "at-risk" students, allowing for the creation of targeted interventions and strategic recommendations to boost successful project completions.


## 📑 Table of Contents

1. [Introduction](#1-introduction)
2. [Objective](#2-objective)
3. [Goals](#3-goals)
4. [Other Skills Demonstrated](#4-other-skills-demonstrated)
5. [Data Overview](#5-data-overview)
6. [Data Transformation](#6-data-transformation)
7. [Data Modelling](#7-data-modelling)
8. [Scraping Code](#8-scraping-code)
9. [Analysis](#9-analysis)
10. [Dashboard Link](#10-dashboard-link)
11. [Recommendations](#11-recommendations)
12. [Conclusion](#12-conclusion)

---

## 1. Introduction

This report presents a comprehensive Exploratory Data Analysis (EDA) and Predictive Modeling deliverable for the **RIT 0903 AI Data Analysis** programme at Excelerate. This enhanced analysis builds on the full raw dataset of **8,390 learner records** from the SLU Opportunity Platform, incorporating a critical new engineered feature — `Has_Start_Date` — which identifies learners who never progressed to the start stage of their assigned opportunity.

The primary prediction task is **Learner Churn (Drop-off) Prediction**, identifying which learners are at risk of dropping out or withdrawing before completion.

> **🔑 Key Finding:** Gradient Boosting is the best model (Accuracy **94.22%**). Engagement Lag (`0.622`) and Time in Opportunity (`0.258`) are the two dominant churn predictors. The introduction of `Has_Start_Date` as a feature revealed that **44% of raw records** represent learners who never started — a previously hidden and critical risk segment.

---

## 2. Objective

The primary objective of this project is to build a reliable and interpretable **Learner Churn Prediction System** using structured data from the SLU Opportunity Platform. Specifically, the analysis aims to:

· Analyze and clean student engagement data to
uncover trends in signups, completions, and drop-offs.

. Build machine learning models to predict student
behaviors and identify factors influencing engagement.

. Develop recommendations to
engagement, focusing on increasing successful
completions and reducing drop-offs.

---

## 3. Goals

| # | Goal |
|---|---|
| 1 | Perform comprehensive EDA on 8,390 learner records from the full raw dataset |
| 2 | Engineer and validate the `Has_Start_Date` binary feature to capture pre-start dropout |
| 3 | Train 5 machine learning classification models and evaluate on a held-out test set |
| 4 | Identify the top churn predictors using feature importance from the best model |
| 5 | Deliver three strategic, data-driven recommendations traceable to model outputs |
| 6 | Provide a sign-up volume forecast with confidence intervals for capacity planning |

---

## 4. Other Skills Demonstrated

Beyond core EDA and modelling, this project demonstrates proficiency in:

- **Feature Engineering** — Constructing `Has_Start_Date`, Engagement Lag, and composite Engagement Score from raw timestamps and categorical weights.
- **Data Artifact Detection** — Identifying and correcting Excel date-zero corruption that inflated `Time in Opportunity` values for never-started learners.
- **Class Imbalance Awareness** — Diagnosing zero-performance in Logistic Regression and SVM caused by an 8.24% churn class imbalance, and proposing SMOTE as a resolution pathway.
- **Time-Series Forecasting** — Fitting a linear regression trend model to monthly sign-up data with a 95% confidence interval projection.
- **Visual Storytelling** — Portfolio of 13 visualizations across distribution, categorical, correlation, and model evaluation plots.
- **Strategic Communication** — Translating statistical findings into three executive-ready intervention proposals.

---

## 5. Data Overview

### 5.1 Dataset Summary

| Metric | Value |
|---|---|
| Total Records | 8,390 |
| Total Features | 16+ |
| Total Churners | 691 |
| Global Churn Rate | 8.24% |


### 5.2. Data Description

Source: SLU Opportunity-Wise Data (SLU_Opportunity_Wise_Data.xlsx)
The raw dataset contains 8,558 records across 16 columns, capturing learner engagement across various opportunity types.

Column	Description
Learner SignUp DateTime	Date and time the learner signed up
Opportunity ID / Name	Unique identifier and name of the opportunity
Opportunity Category	Type: Course, Internship, Event, Competition, Engagement
Opportunity Start / End Date	Duration boundaries of the opportunity
First Name	Learner's first name
Date of Birth	Used to derive the Age feature
Gender	Male, Female, Other, Don't want to specify
Country	Learner's country (top: USA, India, Nigeria, Ghana)
Institution Name	Learner's institution (5 missing values)
Current/Intended Major	Field of study (5 missing values)
Status Description / Code	Learner's current status in the opportunity
Apply Date	Date the learner applied
Entry Created At	System entry timestamp

### 5.3 Data Cleaning Process, Issues Encountered and Resolutions

The raw dataset of 8,558 records underwent several cleaning steps. The cleaned dataset resulted in 4,682 usable records across 39 columns.

1. Text Standardisation
Applied Title Case (Text.Proper) to Institution Name, Country, and Current/Intended Major.
2. Handling Errors and Missing Values
•	Removed rows with errors in 'Learner SignUp DateTime' and Apply Date
•	Filtered out rows where 'Opportunity Start Date' was null — could not be used for duration-based calculations

Column	Missing Count	Resolution
Opportunity Start Date	3,794 (44%)	Rows excluded from time-based calculations
Institution Name	5	Retained as null — not critical for analysis
Current/Intended Major	5	Retained as null — not critical for analysis

3. Data Type Conversions
•	Converted Learner SignUp DateTime, Opportunity End Date, Opportunity Start Date and Date of Birth to DateTime type
•	Converted Age to Integer type
4. Derived / Extracted Columns
•	Age — calculated as difference between current date and Date of Birth divided by 365
•	Learner SignUp Year, Month, and Day — extracted from Learner SignUp DateTime


5. Issues Encountered and Resolutions
Issue 1 — Categorical Inconsistencies in Institution Name
Entries like 'Federal University Lokoja' and 'Federal University Lokoja, Nigeria' referred to the same institution. Standardised using Text.Proper and manual grouping.
Issue 2 — Invalid Datetime Hour Overflow (24:xx:xx)
Some datetime values contained impossible hours (e.g., 24:00:09). Resolved by replacing '24:' with '00:' and adding 1 day.
// Power Query M Code
fixed = Text.Replace(raw, " 24:", " 00:"),
dt = DateTime.FromText(fixed),
corrected = if hour >= 24 then
  DateTime.From(Date.AddDays(DateTime.Date(dt), 1)) + DateTime.Time(dt) else dt
Issue 3 — Negative Time in Opportunity Value
Dates were flipped when opportunity end date was less than start date.
= IF([@[Opportunity End Date]] < [@[Opportunity Start Date]],
    [@[Opportunity Start Date]] - [@[Opportunity End Date]],
    [@[Opportunity End Date]] - [@[Opportunity Start Date]])
Issue 4 — Nested Table Values in Power Query
The Learner SignUp DateTime column returned 'Table' objects. Resolved using Table.FirstValue() to extract the correct scalar value.

### 5.4 Feature Engineering

14 new features were engineered to enrich the dataset for machine learning:

Feature Name	Derived From	Rationale
Age	Date of Birth + Current Date	Captures learner demographic profile
Time in Opportunity	Apply Date + Opportunity End Date	Measures active engagement duration
Engagement Lag	SignUp DateTime + Apply Date	Time between signup and application — behavioural indicator
Learner SignUp Year	Learner SignUp DateTime	Enables trend analysis across years
Learner SignUp Month	Learner SignUp DateTime	Enables seasonal/monthly pattern analysis
Learner SignUp Day	Learner SignUp DateTime	Identifies peak signup days of the week
Normalised Age	Age	Rescales Age to 0–1 for fair model weighting
Normalised Time in Opportunity	Time in Opportunity	Rescales duration to 0–1 range
Normalised Opportunity Category	Encoded Opportunity Category	Brings category score to 0–1 scale
Encoded Gender	Gender	Converts Male/Female to binary (1/0)
Encoded Country (US, India, Nigeria, Ghana, Others)	Country	One-hot encodes top 4 countries; rest grouped as Others
Encoded Opportunity Category	Opportunity Category	One-hot encodes each opportunity type as binary column
Age x Time in Opportunity	Age + Time in Opportunity	Interaction feature capturing joint influence on outcomes
Engagement Score	Normalised Age + Time + Category	Composite metric summarising overall engagement level

Feature Engineering Calculations
Example 1 — Age
Age was derived by calculating the difference between the learner's Date of Birth and the current date, divided by 365.
Age = (DateTime.LocalNow() - [Date of Birth]) / 365
Example 2 — Min-Max Normalisation
=([@Age] - MIN([Age])) / (MAX([Age]) - MIN([Age]))
An Age of 23 in a range of 16–60 normalises to approximately 0.16.
Example 3 — Encoded Country (Others)
=IF(OR([@Country]="United States",[@Country]="India",[@Country]="Nigeria",[@Country]="Ghana"),0,1)
Example 4 — Opportunity Category Encoding
Encoded based on expected engagement level 1–5 (Internship=5, Course=4, Competition=3, Engagement=2, Event=1).
=IF([@[Opportunity Category]]="Internship",5, IF([@[Opportunity Category]]="Course",4,
  IF([@[Opportunity Category]]="Competition",3, IF([@[Opportunity Category]]="Engagement",2,
    IF([@[Opportunity Category]]="Event",1,0)))))
Example 5 — Engagement Score
Engagement Score = (0.5 × Time_Normalized) + (0.2 × Age_Normalized) + (0.3 × OpportunityCategory_Normalized)
•	Time in Opportunity weighted 50% — most direct measure of active engagement
•	Opportunity Category weighted 30% — reflects varying commitment levels
•	Age contributed 20% — demographic influence factor

### 5.5 Data Validation
e. Data Validation

Validation Check	Method	Outcome
Datetime format consistency	Checked all datetime columns parsed without errors in Power Query	Passed — all remaining rows have valid datetime values
Null value check	Reviewed column quality indicators in Power Query (Valid/Error/Empty %)	Learner SignUp DateTime: 100% valid; Opportunity Start Date nulls fully removed
Age range check	Filtered Age column for values below 10 or above 100	No unrealistic age values found
Normalised value range	Verified min and max of normalised columns in Excel	All normalised columns confirmed within 0–1 range
Engagement Score range	Checked min/max of Engagement Score column	Values confirmed within expected bounds
Category encoding accuracy	Spot-checked encoded columns against original Category column	One-hot encoded values correctly matched source categories
Duplicate check	Checked for duplicate Learner + Opportunity ID combinations	No duplicates found in cleaned dataset


## 6 Exploratory Data Analysis
### 6.1 Key Numerical Features — Statistical Summary

| Feature | Mean | Std Dev | Min | Median | Max |
|---|---|---|---|---|---|
| Age (years) | 26.6 | 4.6 | 15 | 26 | 60 |
| Engagement Score | 1.60 | 0.48 | 0.30 | 1.57 | 2.16 |
| Sign-ups (monthly avg) | 724 | — | 352 | — | 1,167 |

### 6.2 Distribribution & Outlier Analysis

### 6.3 Categorical Distribution Summary

The dataset spans five opportunity categories. **Internship** dominates at 63.4% (5,317 records), followed by **Course** at 23.7% (1,992 records) — together representing 87.1% of all records.

The top two source countries are the **United States** (3,869 learners, 46.1%) and **India** (2,817 learners, 33.6%), accounting for nearly 80% of the total learner population.
Engagement Score by Opportunity Category
### 6.4 Seasonal Patterns
### 6.5 Geographic and Gender Breakdown
### 6.5 Learner Status and Feature Correlations
### 6.5 The `Has_Start_Date` Feature

A critical binary feature — `Has_Start_Date` — was engineered to distinguish learners who have an Opportunity Start Date recorded (`= 1`) from those who do not (`= 0`).

> Learners with `Has_Start_Date = 0` never progressed to the start stage of their opportunity. This group represents approximately **44% of the full dataset** and constitutes the **highest-risk segment** on the platform.

---

## 6. Data Transformation

The following transformations were applied to prepare data for modelling:

| Transformation | Description |
|---|---|
| `Has_Start_Date` Engineering | Binary flag (0/1) to separate never-started learners from active learners |
| Excel Date-Zero Correction | Corrupted `Time in Opportunity` values for `Has_Start_Date = 0` rows corrected to `NaN` |
| Engagement Score Recalculation | Recalculated exclusively for started learners after date-zero correction |
| Engagement Lag Correction | Negative lag values (pre-signup applicants) retained as a high-motivation cohort signal |
| Outlier Retention | Plausible outliers (Age max = 60, Time max = 45,630 hrs) retained with documentation |
| Encoding | Categorical features (Category, Country, Gender) label-encoded for model compatibility |
| Null Imputation | Mean imputation applied to Age nulls; `NaN` assigned for corrupted Engagement Scores |
| Train-Test Split | 80/20 stratified split preserving the 8.24% churn class proportion |

**Outlier & Anomaly Summary:**

| Feature | Nature of Outlier | Action Taken |
|---|---|---|
| Time in Opportunity | Max = 45,630 hrs (~5.2 yrs) | Retained for started learners; NaN for never-started |
| Engagement Lag | Negative values (min = −570 hrs) | Retained — high-motivation pre-signup cohort |
| Age | Max = 60; Min = 15 | Retained; mean imputation for nulls |
| Engagement Score | Spike at 2.05+ for `Has_Start_Date = 0` | Recalculated after date-zero correction |
| `Has_Start_Date = 0` | 3,708 rows (44% of dataset) | Flagged rather than deleted |

---

## 7. Data Modelling

### 7.1 Model Performance Comparison

Five classification algorithms were evaluated using an 80/20 stratified train-test split:

| Model | Accuracy | Precision | Recall | F1-Score |
|---|---|---|---|---|
| Logistic Regression | 0.9178 | 0.0000 | 0.0000 | 0.0000 |
| Decision Tree | 0.9207 | 0.5180 | 0.5217 | 0.5199 |
| SVM | 0.9178 | 0.0000 | 0.0000 | 0.0000 |
| Random Forest | 0.9303 | 0.5913 | 0.4928 | 0.5375 |
| **Gradient Boosting ★** | **0.9422** | **0.8060** | **0.3913** | **0.5268** |

> **🔑 Key Finding:** Gradient Boosting wins on Accuracy (94.22%) and Precision (80.60%). However, its Recall of 39.13% means it misses 60% of actual churners. SMOTE or class-weight balancing is the critical next step.

### 7.2 Feature Importance — Gradient Boosting

| Feature | Importance Score |
|---|---|
| Engagement Lag | **0.622** |
| Time in Opportunity | **0.258** |
| Engagement Score | 0.085 |
| Encoded Category | 0.011 |
| Encoded Country | 0.011 |
| Age | 0.008 |
| Encoded Gender | 0.005 |

Engagement Lag and Time in Opportunity together account for **88% of the model's predictive power**.

### 7.3 Confusion Matrix — Gradient Boosting (Test Set)

|  | Predicted Negative | Predicted Positive |
|---|---|---|
| **Actual Negative** | 1,527 (TN) | 13 (FP) |
| **Actual Positive** | 84 (FN) | 54 (TP) |

The high False Negative count (84) relative to True Positives (54) reflects the class imbalance challenge. Implementing SMOTE is the most direct path to improving recall from 39.13% toward the 63.97% achieved in the clean-data baseline.

---

## 8. Scraping Code

The dataset was sourced from the **SLU Opportunity Platform** via structured export. Below is the core data ingestion and feature engineering pipeline used in this project:

```python
import pandas as pd
import numpy as np

# Load raw dataset
df = pd.read_csv("slu_opportunity_platform_raw.csv")

# --- Feature Engineering: Has_Start_Date ---
df["Has_Start_Date"] = df["Opportunity Start Date"].notna().astype(int)

# --- Correct Excel date-zero artifact for never-started learners ---
df.loc[df["Has_Start_Date"] == 0, "Time in Opportunity"] = np.nan

# --- Recalculate Engagement Score for started learners only ---
category_weights = {
    "Internship": 2.0, "Course": 1.2, "Competition": 0.9,
    "Engagement": 0.6, "Event": 0.3
}
df["Category_Weight"] = df["Opportunity Category"].map(category_weights)
df["Engagement Score"] = np.where(
    df["Has_Start_Date"] == 1,
    df["Normalized_Time"] * df["Category_Weight"],
    np.nan
)

# --- Engagement Lag (hours between signup and application) ---
df["Signup Date"] = pd.to_datetime(df["Signup Date"])
df["Application Date"] = pd.to_datetime(df["Application Date"])
df["Engagement_Lag"] = (
    df["Application Date"] - df["Signup Date"]
).dt.total_seconds() / 3600

# --- Label Encoding ---
from sklearn.preprocessing import LabelEncoder
for col in ["Opportunity Category", "Country", "Gender"]:
    df[f"Encoded_{col}"] = LabelEncoder().fit_transform(df[col].fillna("Unknown"))

# --- Train-Test Split (stratified) ---
from sklearn.model_selection import train_test_split
X = df[["Engagement_Lag", "Time in Opportunity", "Engagement Score",
        "Encoded_Opportunity Category", "Encoded_Country", "Age", "Encoded_Gender"]]
y = df["Churned"]
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, stratify=y, random_state=42
)
```

```python
# --- Model Training: Gradient Boosting ---
from sklearn.ensemble import GradientBoostingClassifier
from sklearn.metrics import accuracy_score, classification_report, confusion_matrix

gb_model = GradientBoostingClassifier(n_estimators=100, random_state=42)
gb_model.fit(X_train, y_train)
y_pred = gb_model.predict(X_test)

print(f"Accuracy: {accuracy_score(y_test, y_pred):.4f}")
print(classification_report(y_test, y_pred))
print(confusion_matrix(y_test, y_pred))
```

---

## 9. Analysis

### 9.1 Distribution & Outlier Analysis

- **Age Distribution** — Right-skewed; majority of learners aged 18–30, mean = 26.6 years. Outliers above age 45 represent non-traditional learners.
- **Engagement Score** — Bimodal distribution: cluster at 0.30–0.40 (Event learners) and 2.05–2.10 (Internship learners). The 1.10 threshold is the primary early-intervention trigger point.

### 9.2 Categorical Distributions

- **Opportunity Category** — Internship leads at 63.4% (5,317 records). Course is second at 23.7% (1,992). Together they represent 87.1% of all platform activity.
- **Top Countries** — USA (3,869, 46.1%) and India (2,817, 33.6%) represent 79.7% of learners. Nigeria (736, 8.8%) leads African representation.

### 9.3 Seasonal Patterns

| Month | Sign-Ups | Note |
|---|---|---|
| August | 1,167 | Peak month — 57% above average |
| January | 992 | New Year motivation effect |
| February | 923 | Strong winter performance |
| November | 352 | Weakest month of the year |

- **By Day of Week:** Thursday leads (1,385), followed by Friday (1,328) and Monday (1,316). Weekends fall ~30% below the 1,199 daily average.

### 9.4 Churn Distribution by Category

| Category | Churners | Share of Total Churn |
|---|---|---|
| Internship | 660 | **92.5%** |
| Course | 24 | 3.4% |
| Competition | 5 | 0.7% |
| Event | 2 | 0.3% |
| Engagement | 0 | 0.0% |

### 9.5 Eight Key Strategic Insights

| # | Insight |
|---|---|
| 1 | Gradient Boosting achieves **94.22% accuracy** and **80.60% precision** — SMOTE is the next step to improve recall |
| 2 | **Engagement Lag (0.622)** is the single strongest churn predictor — a 48-hour nudge is the highest-ROI intervention |
| 3 | `Has_Start_Date = 0` reveals **3,708 invisible churners** (44% of dataset) previously discarded during cleaning |
| 4 | **Internship accounts for 92.5%** of all churners — modular milestones and early monitoring are the direct fix |
| 5 | **August surges 57%** above average — capacity planning must begin in June |
| 6 | India's gender gap (64% M / 36% F) presents a clear **female outreach opportunity** |
| 7 | **Thursday–Friday** are the highest-engagement days — schedule automated comms mid-to-late week |
| 8 | Logistic Regression and SVM scored **zero F1** due to class imbalance — class weighting is mandatory before deployment |

---

## 10. Dashboard Link

> 🔗 **[View Interactive Dashboard →](#)**
>
> *(Replace `#` with your live dashboard URL — e.g., Power BI, Tableau, Looker Studio, or Streamlit)*

The dashboard includes:
- Real-time churn risk scores per learner segment
- Monthly sign-up trend with forecast band
- Engagement Score distribution by Opportunity Category
- Feature importance visualization
- Country & gender demographic breakdown

---

## 11. Recommendations

### Recommendation 1 — Automated Nudge System (Engagement Lag Threshold)

**Prediction Basis:** Engagement Lag is the dominant churn predictor (importance `0.622`). Learners who delay their application by more than 48 hours after signup are at the highest dropout risk.

- Implement automated **email and SMS triggers within 48 hours** of signup for learners who have not yet applied.
- Integrate the Gradient Boosting model into the live platform for **real-time risk scoring** of all new sign-ups.
- Assign **peer mentors** to flagged learners within 24 hours of the nudge trigger.

---

### Recommendation 2 — Pre-Start Activation Campaign (`Has_Start_Date = 0`)

**Prediction Basis:** 44% of the full dataset are learners who never started — a retention problem previously invisible in cleaned data.

- Introduce a **light activation task** ("Complete your profile within 48 hours") to filter for commitment before full resource allocation.
- Create a dedicated **"Activation" status** in the platform pipeline to track learners between allocation and first start.
- Re-run the Gradient Boosting model **weekly** to dynamically update risk scores for allocated but not-yet-started learners.

---

### Recommendation 3 — Peak Window Capacity Planning (May–August)

**Prediction Basis:** Sign-up volume surges 57% above average in August. Processing backlogs from under-scaling drive churn.

- Schedule high-intensity recruitment campaigns in **May and July** to capture peak-motivation applicants ahead of the August surge.
- Scale support staff, partner opportunity slots, and matching infrastructure **from June 2026**.
- Alert partner organisations of expected volume increases by May to prepare **additional internship and course slots**.

---

## 12. Conclusion

This enhanced analysis successfully extended the Week 2 EDA and Week 3 predictive modelling pipeline by incorporating the full **8,390-record dataset** and the new `Has_Start_Date` feature. Five machine learning models were trained and evaluated. **Gradient Boosting** emerged as the definitive best model with **94.22% accuracy** and **80.60% precision**.

The most significant finding of this analysis is that **44% of the raw dataset — 3,708 learners — never progressed past the allocation stage**. This population was previously removed during data cleaning. By retaining and flagging them with `Has_Start_Date = 0`, the analysis reveals a hidden but critical churn cohort that requires dedicated intervention strategies entirely distinct from mid-journey dropout prevention.

**Engagement Lag (0.622)** and **Time in Opportunity (0.258)** together account for **88% of the model's predictive power**, confirming that behavioural timing signals dominate demographic features.

> **🔑 The work is not finished. The model flags the at-risk learner. But someone still needs to build the nudge system, assign the mentor, and act on the score. Data does not solve problems. People with data do.**

