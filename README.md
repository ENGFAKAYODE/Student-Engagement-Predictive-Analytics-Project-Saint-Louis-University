# Student-Engagement-Predictive-Analytics-Project-Saint-Louis-University
This project focuses on the intersection of EdTech and Data Science by analyzing student engagement data from the Saint Louis University (SLU) Wise Opportunity platform. The workflow involved cleaning and pre-processing complex behavioral datasets to uncover trends in user drop-offs. Using Machine Learning, I developed a predictive model to identify "at-risk" students, allowing for the creation of targeted interventions and strategic recommendations to boost successful project completions.

## Table of Contents

1. [Introduction](#1-introduction)
2. [Objective](#2-objective)
3. [Goals](#3-goals)
4. [Other Skills Demonstrated](#4-other-skills-demonstrated)
5. [Data Overview](#5-data-overview)
6. [Data Transformation](#6-data-transformation)
7. [Exploratory Data Analysis](#7-exploratory-data-analysis)
8. [Predictive Modelling](#8-predictive-modelling)
9. [Analysis](#9-analysis)
10. [Recommendations](#10-recommendations)
11. [Conclusion](#11-conclusion)

---

## 1. INTRODUCTION

This report presents a comprehensive Exploratory Data Analysis (EDA) and Predictive Modeling deliverable for the **RIT 0903 AI Data Analysis** programme at Excelerate. This enhanced analysis builds on the full raw dataset of **8,390 learner records** from the SLU Opportunity Platform, incorporating a critical new engineered feature `Has_Start_Date` which identifies learners who never progressed to the start stage of their assigned opportunity. The primary prediction task is **Learner Churn (Drop-off) Prediction**, identifying which learners are at risk of dropping out or withdrawing before completion.

> **Key Finding:** Gradient Boosting is the best model, achieving **94.22% accuracy** before resampling and **78.99% recall** after SMOTE rebalancing, catching 8 in 10 at-risk learners. Engagement Lag (`0.622`) and Time in Opportunity (`0.258`) are the two dominant churn predictors confirmed across both configurations. The introduction of `Has_Start_Date` revealed that **44% of raw records** represent learners who never started, a previously hidden and critical risk segment that sits entirely outside the model's prediction scope and requires a separate intervention pipeline.

---

## 2. OBJECTIVES

The primary objective of this project is to build a reliable and interpretable **Learner Churn Prediction System** using structured data from the SLU Opportunity Platform. Specifically, the analysis aims to: analyze and clean student engagement data to uncover trends in signups, completions, and drop-offs; build machine learning models to predict student behaviors and identify factors influencing engagement; and develop recommendations to improve engagement, focusing on increasing successful completions and reducing drop-offs.

---

## 3. GOALS

1. Perform comprehensive EDA on 8,390 learner records from the full raw dataset
2. Train 5 machine learning classification models and evaluate on a held-out test set
3. Identify the top churn predictors using feature importance from the best model
4. Deliver strategic, data-driven recommendations traceable to model outputs
5. Provide a sign-up volume forecast with confidence intervals for capacity planning

---

## 4. OTHER SKILLS DEMONSTRATED

Beyond core EDA and modelling, this project demonstrates proficiency in: **Feature Engineering** (constructing `Has_Start_Date`, Engagement Lag, and composite Engagement Score from raw timestamps and categorical weights); **Data Artifact Detection** (identifying and correcting Excel date-zero corruption that inflated `Time in Opportunity` values for never-started learners); **Class Imbalance Handling** (diagnosing zero-performance in Logistic Regression and SVM caused by an 8.24% churn class imbalance, and resolving it through SMOTE resampling, improving Gradient Boosting recall from 39.13% to 78.99%); **Time-Series Forecasting** (fitting a linear regression trend model to monthly sign-up data with a 95% confidence interval projection); **Visual Storytelling** (a portfolio of 13 visualizations across distribution, categorical, correlation, and model evaluation plots); and **Strategic Communication** (translating statistical findings into executive-ready intervention proposals grounded in model outputs).

---

## 5. DATA SUMMARY

| Metric | Value |
|---|---|
| Total Records | 8,390 |
| Total Features | 16+ |
| Total Churners | 691 |
| Global Churn Rate | 8.24% |

### 5.1 Data Dictionary

Source: SLU Opportunity-Wise Data (SLU_Opportunity_Wise_Data.xlsx). The raw dataset contains 8,558 records across 16 columns, capturing learner engagement across various opportunity types.

| Column | Description |
|---|---|
| Learner SignUp DateTime | Date and time the learner signed up |
| Opportunity ID / Name | Unique identifier and name of the opportunity |
| Opportunity Category | Type: Course, Internship, Event, Competition, Engagement |
| Opportunity Start / End Date | Duration boundaries of the opportunity |
| First Name | Learner's first name |
| Date of Birth | Used to derive the Age feature |
| Gender | Male, Female, Other, Don't want to specify |
| Country | Learner's country (top: USA, India, Nigeria, Ghana) |
| Institution Name | Learner's institution (5 missing values) |
| Current/Intended Major | Field of study (5 missing values) |
| Status Description / Code | Learner's current status in the opportunity |
| Apply Date | Date the learner applied |
| Entry Created At | System entry timestamp |

### 5.2 Data Cleaning Process, Issues Encountered and Resolutions

The raw dataset of 8,558 records underwent several cleaning steps. The cleaned dataset resulted in 4,682 usable records across 39 columns.

**1. Text Standardisation:** Applied Title Case (Text.Proper) to Institution Name, Country, and Current/Intended Major.

**2. Handling Errors and Missing Values:** Removed rows with errors in 'Learner SignUp DateTime' and Apply Date. Filtered out rows where 'Opportunity Start Date' was null, as these could not be used for duration-based calculations.

| Column | Missing Count | Resolution |
|---|---|---|
| Opportunity Start Date | 3,794 (44%) | Missing values led to engineering of `Has_Start_Date` |
| Institution Name | 5 | Retained as null, not critical for analysis |
| Current/Intended Major | 5 | Retained as null, not critical for analysis |

**3. Data Type Conversions:** Converted Learner SignUp DateTime, Opportunity End Date, Opportunity Start Date and Date of Birth to DateTime type. Converted Age to Integer type.

**4. Derived / Extracted Columns:** Age was calculated as the difference between current date and Date of Birth divided by 365. Learner SignUp Year, Month, and Day were extracted from Learner SignUp DateTime.

#### 5.2.1 Issues Encountered and Resolutions

**Issue 1:** Categorical Inconsistencies in Institution Name. Entries like 'Federal University Lokoja' and 'Federal University Lokoja, Nigeria' referred to the same institution. Standardised using Text.Proper and manual grouping.

**Issue 2:** Invalid Datetime Hour Overflow (24:xx:xx). Some datetime values contained impossible hours (e.g., 24:00:09). Resolved by replacing '24:' with '00:' and adding 1 day.

```
// Power Query M Code
fixed = Text.Replace(raw, " 24:", " 00:"),
dt = DateTime.FromText(fixed),
corrected = if hour >= 24 then
  DateTime.From(Date.AddDays(DateTime.Date(dt), 1)) + DateTime.Time(dt) else dt
```

**Issue 3:** Negative Time in Opportunity Value. Dates were flipped when opportunity end date was less than start date.

```
= IF([@[Opportunity End Date]] < [@[Opportunity Start Date]],
    [@[Opportunity Start Date]] - [@[Opportunity End Date]],
    [@[Opportunity End Date]] - [@[Opportunity Start Date]])
```

**Issue 4:** Nested Table Values in Power Query. The Learner SignUp DateTime column returned 'Table' objects. Resolved using Table.FirstValue() to extract the correct scalar value.

### 5.3 Feature Engineering

14 new features were engineered to enrich the dataset for machine learning:

| Feature Name | Derived From | Rationale |
|---|---|---|
| Age | Date of Birth + Current Date | Captures learner demographic profile |
| Time in Opportunity | Apply Date + Opportunity End Date | Measures active engagement duration |
| Engagement Lag | SignUp DateTime + Apply Date | Time between signup and application |
| Learner SignUp Year | Learner SignUp DateTime | Enables trend analysis across years |
| Learner SignUp Month | Learner SignUp DateTime | Enables seasonal pattern analysis |
| Learner SignUp Day | Learner SignUp DateTime | Identifies peak signup days |
| Normalised Age | Age | Rescales Age to 0-1 for fair model weighting |
| Normalised Time in Opportunity | Time in Opportunity | Rescales duration to 0-1 range |
| Normalised Opportunity Category | Encoded Opportunity Category | Brings category score to 0-1 scale |
| Encoded Gender | Gender | Converts Male and Female to binary values |
| Encoded Country | Country | One hot encodes top 4 countries |
| Encoded Opportunity Category | Opportunity Category | One hot encodes each opportunity type |
| Age x Time in Opportunity | Age + Time in Opportunity | Interaction feature |
| Engagement Score | Normalised Age + Time + Category | Composite engagement metric |

#### 5.3.1 Feature Engineering Calculations

**1. Age**
```
Age = (DateTime.LocalNow() - [Date of Birth]) / 365
```

**2. Min-Max Normalisation**
```
=([@Age] - MIN([Age])) / (MAX([Age]) - MIN([Age]))
An Age of 23 in a range of 16-60 normalises to approximately 0.16
```

**3. Encoded Country (Others)**
```
=IF(OR([@Country]="United States",[@Country]="India",
       [@Country]="Nigeria",[@Country]="Ghana"),0,1)
```

**4. Opportunity Category Encoding:** Encoded based on expected engagement level 1-5 (Internship=5, Course=4, Competition=3, Engagement=2, Event=1).
```
=IF([@[Opportunity Category]]="Internship",5,
  IF([@[Opportunity Category]]="Course",4,
    IF([@[Opportunity Category]]="Competition",3,
      IF([@[Opportunity Category]]="Engagement",2,
        IF([@[Opportunity Category]]="Event",1,0)))))
```

**5. Engagement Score**
```
Engagement Score = (0.5 x Time_Normalized) + 
                   (0.2 x Age_Normalized) + 
                   (0.3 x OpportunityCategory_Normalized)
```
Time in Opportunity is weighted 50% as the most direct measure of active engagement. Opportunity Category is weighted 30%, reflecting varying commitment levels. Age contributes 20% as a demographic influence factor.

### 5.4 Data Validation

| Validation Check | Method | Outcome |
|---|---|---|
| Datetime format consistency | Checked all datetime columns in Power Query | Passed |
| Null value check | Reviewed column quality indicators | SignUp DateTime 100% valid |
| Age range check | Filtered for values below 10 or above 100 | No unrealistic values |
| Normalised value range | Verified min and max of normalised columns | All within 0-1 |
| Engagement Score range | Checked min and max of Engagement Score | Within expected bounds |
| Category encoding accuracy | Spot checked encoded vs original columns | Correctly matched |
| Duplicate check | Checked for duplicate Learner + Opportunity ID | No duplicates found |
| Record count reconciliation | Compared raw vs cleaned row counts | 8,558 to 4,682 |

---

## 6. DATA TRANSFORMATION

| Transformation | Description |
|---|---|
| `Has_Start_Date` Engineering | Binary flag (0/1) to separate never-started from active learners |
| Excel Date-Zero Correction | Corrupted `Time in Opportunity` values recalculated in Python; `NaN` used instead of misleading `today()` |
| Engagement Score Recalculation | Recalculated exclusively for started learners after date-zero correction |
| Engagement Lag Correction | Negative lag values applied absolute values |
| Outlier Correction | Plausible outliers corrected after Excel Date-Zero Correction |
| Encoding | Categorical features label-encoded for model compatibility |
| Train-Test Split | 80/20 stratified split preserving 8.24% churn proportion |

### 6.1 Outlier and Anomaly Summary

| Feature | Nature of Outlier | Action Taken |
|---|---|---|
| Time in Opportunity | Max = 45,630 hrs (~5.2 yrs) | Retained for started learners; NaN for never-started |
| Engagement Lag | Negative values (min = -570 hrs) | Retained as a high-motivation pre-signup cohort |
| Age | Max = 60; Min = 15 | Retained; mean imputation for nulls |
| Engagement Score | Spike at 2.05+ for `Has_Start_Date = 0` | Recalculated after date-zero correction |
| `Has_Start_Date = 0` | 3,708 rows (44% of dataset) | Flagged rather than deleted |

---

## 7. EXPLORATORY DATA ANALYSIS

### 7.1 Key Numerical Features — Statistical Summary

| Feature | Mean | Std Dev | Min | Median | Max |
|---|---|---|---|---|---|
| Age (years) | 26.6 | 4.6 | 15 | 26 | 60 |
| Engagement Score | 1.60 | 0.48 | 0.30 | 1.57 | 2.16 |
| Sign-ups (monthly avg) | 724 | — | 352 | — | 1,167 |

### 7.2 Distribution and Outlier Analysis

**Age Distribution** is right-skewed, with the majority of learners aged 18-30 and a mean of 26.6 years. Outliers above age 45 represent non-traditional learners.

**Engagement Score** shows a bimodal distribution with a cluster at 0.30-0.40 (Event learners) and 2.05-2.10 (Internship learners). The 1.10 threshold is the primary early-intervention trigger point.

**Engagement Score Distribution by Opportunity Category:** Internships dominate platform engagement with a median score of **2.06**, which is 61% higher than Courses (1.28) and more than 5 times higher than Events (0.37). The steep drop-off across categories suggests learners treat opportunity types very differently in terms of depth of involvement.

| Category | Median Score | vs. Internship |
|---|---|---|
| Internship | 2.06 | (baseline) |
| Course | 1.28 | -38% |
| Competition | 0.97 | -53% |
| Engagement | 0.68 | -67% |
| Event | 0.37 | -82% |

### 7.3 Categorical Distribution Summary

**Opportunity Category:** Internship leads at 63.4% (5,317 records). Course is second at 23.7% (1,992). Together they represent 87.1% of all platform activity.

**Geographic and Gender Breakdown:** The **United States** (3,869) and **India** (2,811) account for approximately **79.7%** of all learners. Nigeria leads African representation at **729 learners**, with Ghana (262) and Pakistan (218) rounding out the top five. A **male majority** is consistent across all markets.

| Country | Female | Male | Total | % Male |
|---|---|---|---|---|
| United States | 1,726 | 2,143 | 3,869 | 55% |
| India | 1,015 | 1,796 | 2,811 | 64% |
| Nigeria | 331 | 398 | 729 | 55% |
| Ghana | 67 | 195 | 262 | 74% |
| Pakistan | 91 | 127 | 218 | 58% |

**Learner Status Distribution:** Rejected and Team Allocated together account for **80%** of all learner records. Only **9%** reach an active Started state, and **7.2%** drop out, meaning attrition nearly matches active participation.

| Status | Count | Share |
|---|---|---|
| Rejected | 3,490 | 41.6% |
| Team Allocated | 3,221 | 38.4% |
| Started | 752 | 9.0% |
| Dropped Out | 605 | 7.2% |
| Applied | 105 | 1.3% |
| Waitlisted | 102 | 1.2% |
| Withdraw | 86 | 1.0% |
| Rewards Award | 29 | 0.3% |

> Of learners who progress past rejection, the **dropout rate relative to starters is approximately 45%**, and the full funnel yield is under **0.5%** of total records.

**Churn by Opportunity Category:** Internships account for **92.5%** of all churned learners (660 dropouts), disproportionate even relative to their 63% activity share.

| Category | Learners Lost | Churn Share |
|---|---|---|
| Internship | 660 | 92.5% |
| Course | 24 | ~4.5% |
| Competition | 5 | ~2.5% |
| Event | 2 | ~0.5% |
| Engagement | 0 | ~0.0% |

> Reducing Internship dropout by even **10%** (~66 learners) would nearly **triple** the current Rewards Award completion count of 29.

### 7.4 Seasonal Patterns

**Sign-Ups by Month:**

| Month | Sign-Ups | Note |
|---|---|---|
| August | 1,167 | Peak month, 57% above average |
| January | 992 | New Year motivation effect |
| February | 923 | Strong winter performance |
| November | 352 | Weakest month of the year |

**Sign-Ups by Day of Week:** Thursday leads (1,385), followed by Friday (1,328) and Monday (1,316). Weekends fall approximately 30% below the 1,199 daily average.

### 7.5 Feature Correlation Heatmap

Strong multicollinearity exists across engineered time and engagement features. **Normalized Time in Opportunity**, **New Time in Opportunity**, and **Engagement Lag** form a near-perfect correlation cluster (r = 0.97-1.00).

| Feature Pair | Correlation | Strength |
|---|---|---|
| Engagement Lag to Normalized Age | 1.00 | Perfect positive |
| New Time in Opportunity to Normalized Time in Opportunity | 1.00 | Perfect positive |
| Age x Time in Opportunity to Engagement Score | 0.98 | Very strong positive |
| Engagement Score to New Time in Opportunity | 0.97 | Very strong positive |
| Has_Start_Date to Engagement Score | -0.98 | Very strong negative |
| Has_Start_Date to New Time in Opportunity | -0.88 | Strong negative |
| Normalized Opportunity Category to Engagement Score | 0.84 | Strong positive |

**`Has_Start_Date`** is the only feature with strong negative correlations (-0.53 to -1.0), making it a high-signal binary flag for predictive models. The interaction term **Age x Time in Opportunity** (r = 0.98 with Engagement Score) outperforms either variable alone as a composite feature.

### 7.6 The `Has_Start_Date` Feature

A critical binary feature engineered to distinguish learners who have an Opportunity Start Date recorded (`= 1`) from those who do not (`= 0`). Learners with `Has_Start_Date = 0` never progressed to the start stage. This group represents approximately **44% of the full dataset** and constitutes the **highest-risk segment** on the platform, entirely invisible in cleaned data and outside the churn model's prediction scope.

---

## 8. PREDICTIVE MODELLING

The primary prediction task is **Learner Churn (Drop-off) Prediction**. The target variable `Is_Dropoff` combines 'Dropped Out' (7.2%, 605 learners) and 'Withdraw' (1%, 86 learners), giving a total churn rate of **8.24% (691 learners)**.

### 8.1 Model Performance — Before SMOTE

Five classification algorithms were evaluated using an 80/20 stratified train-test split on the original imbalanced dataset:

| Model | Accuracy | Precision | Recall | F1-Score |
|---|---|---|---|---|
| Logistic Regression | 0.9178 | 0.0000 | 0.0000 | 0.0000 |
| Decision Tree | 0.9207 | 0.5180 | 0.5217 | 0.5199 |
| SVM | 0.9178 | 0.0000 | 0.0000 | 0.0000 |
| Random Forest | 0.9303 | 0.5913 | 0.4928 | 0.5375 |
| **Gradient Boosting** | **0.9422** | **0.8060** | **0.3913** | **0.5268** |

Gradient Boosting wins on Accuracy (94.22%) and Precision (80.60%). However, its Recall of 39.13% means it misses 60% of actual churners. Logistic Regression and SVM scored zero F1 entirely, predicting "no churn" for every learner due to the 11:1 class imbalance.

### 8.2 Feature Importance — Gradient Boosting

**Engagement Lag** alone accounts for **62.2%** of predictive power. Combined with **Time in Opportunity** (25.8%), the top two features explain **88%** of the model's decisions. Demographics contribute just 2.4%.

| Rank | Feature | Importance | Cumulative |
|---|---|---|---|
| 1 | Engagement Lag | 0.622 | 62.2% |
| 2 | Time in Opportunity | 0.258 | 88.0% |
| 3 | Engagement Score | 0.085 | 96.5% |
| 4 | Encoded_Category_Adv | 0.011 | 97.6% |
| 5 | Encoded_Country_Adv | 0.011 | 98.7% |
| 6 | Age | 0.008 | 99.5% |
| 7 | Encoded_Gender_Adv | 0.005 | 100.0% |

The model is identity-agnostic, predicting churn based on *when* and *how long* a learner engages, not who they are.

### 8.3 Confusion Matrix — Gradient Boosting (Before SMOTE)

|  | Predicted No Churn | Predicted Churn |
|---|---|---|
| **Actual No Churn** | 1,527 | 13 |
| **Actual Churn** | 84 | 54 |

The high False Negative count (84) relative to True Positives (54) reflects the class imbalance challenge. SMOTE is the direct path to improving recall.

### 8.4 SMOTE Resampling — Improving Churn Detection Recall

The original training set suffered from severe **class imbalance**: Non-Churners at 6,159 records, Churners at 553 records, and an imbalance ratio of approximately 11:1. This caused Gradient Boosting to achieve a misleadingly high accuracy of **94.22%** while only catching **39.13%** of actual churners. Logistic Regression and SVM scored **zero F1** entirely. A model that ignores the minority class entirely can still score 91%+ accuracy simply because 91% of learners genuinely do not churn. Accuracy is the wrong metric when classes are this imbalanced.

**SMOTE (Synthetic Minority Over-sampling Technique)** generates *synthetic* churner records by interpolating between existing minority class examples, forcing the model to learn what a churner actually looks like instead of defaulting to the majority class. SMOTE was applied **exclusively to the training set**. The test set was never resampled and retains the real-world 8.24% churn distribution.

```python
from imblearn.over_sampling import SMOTE

smote = SMOTE(random_state=42)
X_train_resampled, y_train_resampled = smote.fit_resample(X_train_scaled, y_train)
```

| Split | Before SMOTE | After SMOTE |
|---|---|---|
| Churners (training) | 553 | 6,159 |
| Non-Churners (training) | 6,159 | 6,159 |
| Ratio | 11:1 | 1:1 (balanced) |
| Test set | Unchanged | Unchanged |

**Model Performance — Before vs After SMOTE:**

| Model | Metric | Before SMOTE | After SMOTE | Change |
|---|---|---|---|---|
| Logistic Regression | Recall | 0.00% | 60.14% | Now functional |
| Logistic Regression | F1 | 0.000 | 0.274 | Fixed |
| Decision Tree | Recall | 52.17% | 58.70% | +6.5% |
| Decision Tree | F1 | 0.520 | 0.475 | Stable |
| SVM | Recall | 0.00% | 68.12% | Now functional |
| SVM | F1 | 0.000 | 0.264 | Fixed |
| Random Forest | Recall | 49.28% | 57.97% | +8.7% |
| Random Forest | F1 | 0.538 | 0.520 | Stable |
| **Gradient Boosting** | **Recall** | **39.13%** | **78.99%** | **+39.9%** |
| **Gradient Boosting** | **F1** | **0.527** | **0.498** | **Stable** |

**Gradient Boosting — Full Metric Comparison:**

| Metric | Before SMOTE | After SMOTE | Verdict |
|---|---|---|---|
| Accuracy | 94.22% | 86.89% | Expected drop |
| Precision | 80.60% | 36.33% | Acceptable trade-off |
| Recall | 39.13% | **78.99%** | Primary goal achieved |
| F1-Score | 0.527 | 0.498 | Comparable |
| Churners Missed (FN) | ~84 | ~29 | 55 fewer missed |

> **55 additional at-risk learners identified per prediction cycle** — purely from rebalancing the training data. No new features, no new data collected.

**Confusion Matrix — Before SMOTE:**

|  | Predicted No Churn | Predicted Churn |
|---|---|---|
| **Actual No Churn** | 1,527 | 13 |
| **Actual Churn** | 84 | 54 |

**Confusion Matrix — After SMOTE:**

|  | Predicted No Churn | Predicted Churn |
|---|---|---|
| **Actual No Churn** | ~1,440 | ~100 |
| **Actual Churn** | ~29 | ~109 |

The model now catches **~109 churners** vs **54 before**, at the cost of flagging ~100 non-churners unnecessarily instead of 13. On a digital platform where interventions are low-cost automated messages, this trade-off is acceptable.

**Threshold Tuning — Gradient Boosting After SMOTE:**

| Threshold | Precision | Recall | F1-Score | Recommendation |
|---|---|---|---|---|
| **0.5** | 36.33% | 78.99% | **0.498** | Recommended — best F1 |
| 0.4 | 34.47% | 80.43% | 0.483 | Marginal recall gain |
| 0.3 | 29.35% | 85.51% | 0.437 | High recall, low precision |
| 0.2 | 17.25% | 89.86% | 0.289 | Too aggressive |

**Why After SMOTE Is the Production Model:**

| Decision Factor | Before SMOTE | After SMOTE | Better For This Project |
|---|---|---|---|
| Learners correctly flagged at risk | 54 | ~109 | After |
| Learners wrongly flagged | 13 | ~100 | Before |
| Churners silently missed | 84 | ~29 | After |
| Cost of false alarm | Low (nudge email) | Low (nudge email) | Neutral |
| Cost of missed churner | High (permanent dropout) | High (permanent dropout) | Neutral |
| Serves platform retention KPI | Partially | Yes | After |

> **Design Decision:** The post-SMOTE Gradient Boosting model at threshold 0.5 is adopted as the final production model. SMOTE did not make the model more accurate. It made the model more **useful**. A 94% accurate model that ignores 84 real churners does not serve the platform. An 87% accurate model that catches 109 of them does.

---

## 9. ANALYSIS

### 9.1 Key Insights

**Insight 1 — The Model Now Catches 8 in 10 At-Risk Learners:** After SMOTE rebalancing, recall improved from **39.13% to 78.99%**, and the system now correctly identifies approximately **109 churners** per prediction cycle compared to just 54 before. The 55 additional at-risk learners identified represent real people who would previously have received no intervention whatsoever. The accuracy drop from 94.22% to 86.89% is not a failure — it is the model learning to stop ignoring the minority class and start doing its actual job.

**Insight 2 — Timing Is the Dominant Churn Signal, Confirmed by SMOTE:** Even after rebalancing with 6,159 synthetic churner records, **Engagement Lag (0.622)** and **Time in Opportunity (0.258)** remained the top two features, explaining **88% of the model's decisions**. Their survival through such a dramatic training distribution shift confirms they are genuinely predictive signals, not artefacts of class imbalance. The model is identity-agnostic — demographics contribute just 2.4% even after resampling.

**Insight 3 — Two Models Were Previously Broken, Now Fixed:** Before SMOTE, Logistic Regression and SVM scored **zero F1**, predicting "no churn" for every learner. After SMOTE all five models are genuinely operational:

| Model | Recall Before | Recall After | Status |
|---|---|---|---|
| Logistic Regression | 0.00% | 60.14% | Now functional |
| SVM | 0.00% | 68.12% | Now functional |
| Decision Tree | 52.17% | 58.70% | Improved |
| Random Forest | 49.28% | 57.97% | Improved |
| **Gradient Boosting** | **39.13%** | **78.99%** | Best recall |

**Insight 4 — Random Forest Is the Precision-Conscious Alternative:** After SMOTE, Random Forest achieves the **highest F1 (0.520)** with stronger precision (47.06%) than Gradient Boosting (36.33%). If interventions scale to high-cost actions such as paid mentors or scholarship flags, Random Forest becomes the more appropriate model without sacrificing much recall.

| Model | Precision | Recall | F1 | Best For |
|---|---|---|---|---|
| Gradient Boosting | 36.33% | 78.99% | 0.498 | Maximum recall |
| Random Forest | 47.06% | 57.97% | 0.520 | Best precision-recall balance |

**Insight 5 — 44% of Learners Never Started and the Model Cannot Reach Them:** SMOTE improved recall for learners who started and dropped out. It cannot address the **3,708 learners (44%)** with `Has_Start_Date = 0`, those allocated but never begun. These learners produce no behavioural timing signals so the model has nothing to predict from. They require a completely separate intervention system triggered by allocation status, not behaviour. The churn model protects learners in the journey; the pre-start activation problem requires a separate pipeline entirely.

**Insight 6 — Internship Churn Is the Platform's Highest-Value Problem:** Internships account for **92.5% of all churners (660 dropouts)** and the highest engagement scores (2.06 median). With 79% recall post-SMOTE, approximately **521 of 660 internship churners** would now be flagged in time for intervention, compared to roughly 258 before resampling. Retaining just 20% of flagged internship churners through successful intervention (~104 learners) would nearly **quadruple** the current completion count of 29.

**Insight 7 — August Is a Capacity Problem the Model Can Help Pre-empt:** Sign-ups spike 57% above average in August (1,167 vs 724 avg). With a functional 79% recall model, the platform can begin proactive risk scoring from June, entering the August surge with an existing at-risk registry rather than scrambling to identify churners after backlogs have already formed.

**Insight 8 — Thursday and Friday Are the Highest-Leverage Communication Days:** Thursday (1,385 signups) and Friday (1,328) significantly outperform the weekly average of 1,199. Model-triggered intervention communications sent mid-to-late week reach learners at their highest engagement window, maximising open rates and response to nudges.

**Insight 9 — Gender Gaps Signal Untapped Growth Markets:** India (64% male) and Ghana (74% male) show the sharpest gender imbalances of any top-five country. Nigeria, Ghana, and Pakistan together represent just 14.2% of learners despite being active and growing EdTech markets. These gaps are not just equity issues — they represent measurable acquisition and diversification opportunities.

### 9.2 Summary Table

| # | Insight | Source |
|---|---|---|
| 1 | Model now catches 8 in 10 churners (recall 78.99%) | Post-SMOTE modelling |
| 2 | Engagement Lag (62.2%) confirmed dominant after SMOTE | Feature importance |
| 3 | LR and SVM now functional — all 5 models operational | Post-SMOTE modelling |
| 4 | Random Forest best F1 (0.520) for precision-sensitive use | Post-SMOTE modelling |
| 5 | 44% never-started learners outside model scope entirely | EDA + Has_Start_Date |
| 6 | 92.5% of churn from internships — 521 now catchable | EDA + Post-SMOTE recall |
| 7 | August surge catchable with June pre-scoring | Seasonal EDA |
| 8 | Thursday-Friday peak days for nudge communications | Temporal EDA |
| 9 | India and Ghana gender gaps = growth opportunity | Demographic EDA |

---

## 10. RECOMMENDATIONS

**Recommendation 1 — Deploy the SMOTE-Trained Model for Weekly Live Risk Scoring:** Recall improved to 78.99%, making the model operationally viable for production. The pre-SMOTE model caught only 39% of churners, making deployment irresponsible. Deploy the SMOTE-trained Gradient Boosting model to generate **weekly risk scores** for every active learner at threshold 0.5. Route flagged learners into a tiered response: automated nudge email at flag, peer mentor if no response within 48 hours, and programme coordinator escalation at 72 hours. Run the model on a **rolling weekly basis** so newly at-risk learners are caught early rather than after sustained disengagement. Use **Random Forest as the secondary model** for high-cost interventions (paid mentors, scholarship flags) where its stronger precision (47%) matters more than maximum recall.

**Recommendation 2 — Build a 48-Hour Engagement Lag Trigger as the Primary Intervention:** Engagement Lag holds a feature importance of 0.622, making it the single strongest churn predictor, confirmed stable after SMOTE rebalancing. Its dominance survived resampling, proving it is a genuine signal not a statistical artefact. Implement automated triggers at **24 hours and 48 hours** post-signup for learners who have not yet applied. The 24-hour message should be warm and motivational. The 48-hour message should be direct, with a one-click apply link, deadline visibility, and a peer success story from the same opportunity category. Any learner who has not applied by **72 hours** is automatically scored by the model and routed to a human mentor if risk score exceeds 0.5. During August (peak surge), reduce the trigger window to **24 hours** given elevated churn risk during high-intake periods.

**Recommendation 3 — Build a Separate Pre-Start Activation Pipeline for Never-Started Learners:** 3,708 learners (44%) with `Has_Start_Date = 0` are entirely outside the churn model's prediction scope. SMOTE improved recall for learners who started but cannot score those who produce no behavioural signals because they never began. Create a dedicated **Activation Pipeline** in the platform. Upon team allocation, trigger a 48-hour countdown. If no start activity after 48 hours, send an activation prompt. If no activity after 7 days, escalate to a coordinator. Introduce a mandatory lightweight onboarding step — profile completion, availability confirmation, or a welcome check-in — as both a commitment signal and an activation gate. Track this cohort under a new **Pending Activation** status so it becomes visible in platform reporting rather than disappearing into cleaned-data exclusions.

**Recommendation 4 — Launch an Internship-Specific Milestone Retention Programme:** Internships account for 92.5% of churners. The post-SMOTE model can now flag approximately **521 of 660 internship churners**, up from ~258 before resampling. The question shifts from "can we find them" to "what do we do when we find them." Structure internship journeys into **modular milestones** at weeks 1, 3, and 6. Any learner who misses a milestone check-in and carries a model risk score above 0.5 receives peer mentor outreach within 24 hours. Create an **Internship Early Warning Dashboard** visible to programme coordinators showing risk scores, Engagement Lag values, and milestone completion status for all active internship learners. Retaining just **20% of flagged internship churners** through successful interventions (~104 learners) would nearly **quadruple** the current 29 Rewards Award completions.

**Recommendation 5 — Scale Infrastructure Ahead of August Using June Pre-Scoring:** August signups run 57% above average. With 79% recall, the model can now be used proactively during surge periods rather than reactively after backlogs form. Run the first full platform risk scoring in **June 2026**, entering August with an existing at-risk registry. Scale support staff, automated systems, and partner opportunity slots from June. Alert partner organisations in **May** to prepare additional internship and course slots. During August, increase model scoring from **weekly to twice weekly** to catch newly at-risk learners faster during the high-intake period. Schedule all model-triggered communications on **Thursday or Friday** — peak engagement days — to maximise open and response rates.

**Recommendation 6 — Close Gender and Geographic Gaps as a Growth Strategy:** India is 64% male and Ghana is 74% male, representing the sharpest gender imbalances in the top five. Nigeria, Ghana, and Pakistan together represent only 14.2% of learners despite being active and growing EdTech markets. Launch **female-targeted outreach campaigns** in India and Ghana specifically, partnering with women-in-tech organisations, university societies, and professional networks in those markets. For Nigeria, Ghana, and Pakistan, invest in localised content and regional support contacts to convert interest into activation. Monitor gender balance as a **platform health KPI** alongside retention and completion rates, not just as an equity metric.

---

## 11. CONCLUSION

This enhanced analysis successfully extended the EDA and predictive modelling pipeline by incorporating the full **8,390-record dataset**, the engineered `Has_Start_Date` feature, and SMOTE resampling to address class imbalance. Five machine learning models were trained and evaluated across both imbalanced and resampled configurations. **Gradient Boosting** emerged as the definitive production model, achieving **94.22% accuracy** before resampling and **78.99% recall** after SMOTE, catching approximately **109 at-risk learners** per prediction cycle compared to 54 without resampling.

The most significant finding is that **44% of the raw dataset — 3,708 learners — never progressed past the allocation stage**. This population sits entirely outside the churn model's scope and requires a dedicated Activation Pipeline as a separate system. The churn model and the activation pipeline together address the platform's full retention problem: one for learners in the journey, one for learners who never began it.

**Engagement Lag (0.622)** and **Time in Opportunity (0.258)** account for **88% of predictive power** across both model configurations, confirming that behavioural timing signals dominate demographic features regardless of resampling method.

> **SMOTE did not make the model more accurate. It made the model more useful. The platform now has a system capable of finding the right learners at the right time. The remaining question is purely operational: what happens after the flag fires. Data does not solve problems. People with data do.**
