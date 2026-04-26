<img width="710" height="340" alt="Screenshot 2026-04-22 104033" src="https://github.com/user-attachments/assets/42e8da01-e52b-4534-afa9-87ca2d09fe74" /># Student-Engagement-Predictive-Analytics-Project-Saint-Louis-University
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

## 1. INTRODUCTION

This report presents a comprehensive Exploratory Data Analysis (EDA) and Predictive Modeling deliverable for the **RIT 0903 AI Data Analysis** programme at Excelerate. This enhanced analysis builds on the full raw dataset of **8,390 learner records** from the SLU Opportunity Platform, incorporating a critical new engineered feature — `Has_Start_Date` — which identifies learners who never progressed to the start stage of their assigned opportunity.

The primary prediction task is **Learner Churn (Drop-off) Prediction**, identifying which learners are at risk of dropping out or withdrawing before completion.

> **Key Finding:** Gradient Boosting is the best model (Accuracy **94.22%**). Engagement Lag (`0.622`) and Time in Opportunity (`0.258`) are the two dominant churn predictors. The introduction of `Has_Start_Date` as a feature revealed that **44% of raw records** represent learners who never started a previously hidden and critical risk segment.

---

## 2. OBJECTIVES
The primary objective of this project is to build a reliable and interpretable **Learner Churn Prediction System** using structured data from the SLU Opportunity Platform. Specifically, the analysis aims to:

- Analyze and clean student engagement data to
uncover trends in signups, completions, and drop-offs.

- Build machine learning models to predict student
behaviors and identify factors influencing engagement.

- Develop recommendations to
engagement, focusing on increasing successful
completions and reducing drop-offs.

---

## 3. GOALS

1. Perform comprehensive EDA on 8,390 learner records from the full raw dataset 
2. Train 5 machine learning classification models and evaluate on a held-out test set 
3. Identify the top churn predictors using feature importance from the best model 
4. Deliver three strategic, data-driven recommendations traceable to model outputs 
5. Provide a sign-up volume forecast with confidence intervals for capacity planning 

---

## 4. OTHER SKILLS DEMONSTRATED

Beyond core EDA and modelling, this project demonstrates proficiency in:

- **Feature Engineering**: Constructing `Has_Start_Date`, Engagement Lag, and composite Engagement Score from raw timestamps and categorical weights.
- **Data Artifact Detection**: Identifying and correcting Excel date-zero corruption that inflated `Time in Opportunity` values for never-started learners.
- **Class Imbalance Awareness**: Diagnosing zero-performance in Logistic Regression and SVM caused by an 8.24% churn class imbalance, and proposing SMOTE as a resolution pathway.
- **Time-Series Forecasting**: Fitting a linear regression trend model to monthly sign-up data with a 95% confidence interval projection.
- **Visual Storytelling**: Portfolio of 13 visualizations across distribution, categorical, correlation, and model evaluation plots.
- **Strategic Communication**: Translating statistical findings into three executive-ready intervention proposals.

---

## 5. DATA SUMMARY

| Metric | Value |
|---|---|
| Total Records | 8,390 |
| Total Features | 16+ |
| Total Churners | 691 |
| Global Churn Rate | 8.24% |


### 5.1 Data Dictionary

Source: SLU Opportunity-Wise Data (SLU_Opportunity_Wise_Data.xlsx)
The raw dataset contains 8,558 records across 16 columns, capturing learner engagement across various opportunity types.

| Column                          | Description                                                                 |
|---------------------------------|-----------------------------------------------------------------------------|
| Learner SignUp DateTime         | Date and time the learner signed up                                        |
| Opportunity ID / Name           | Unique identifier and name of the opportunity                              |
| Opportunity Category            | Type: Course, Internship, Event, Competition, Engagement                   |
| Opportunity Start / End Date    | Duration boundaries of the opportunity                                     |
| First Name                      | Learner's first name                                                       |
| Date of Birth                   | Used to derive the Age feature                                             |
| Gender                          | Male, Female, Other, Don't want to specify                                 |
| Country                         | Learner's country (top: USA, India, Nigeria, Ghana)                        |
| Institution Name                | Learner's institution (5 missing values)                                   |
| Current/Intended Major          | Field of study (5 missing values)                                          |
| Status Description / Code       | Learner's current status in the opportunity                                |
| Apply Date                      | Date the learner applied                                                   |
| Entry Created At                | System entry timestamp                                                     |

### 5.2 Data Cleaning Process, Issues Encountered and Resolutions

The raw dataset of 8,558 records underwent several cleaning steps. The cleaned dataset resulted in 4,682 usable records across 39 columns.

**1. Text Standardisation**

Applied Title Case (Text.Proper) to Institution Name, Country, and Current/Intended Major.

**2. Handling Errors and Missing Values**
-	Removed rows with errors in 'Learner SignUp DateTime' and Apply Date
-	Filtered out rows where 'Opportunity Start Date' was null — could not be used for duration-based calculations

Column	Missing Count	Resolution
Opportunity Start Date:	3,794 (44%)	missing, led to the engineering of an additional feature Has_Start_Date
Institution Name:	5	Retained as null — not critical for analysis
Current/Intended Major:	5	Retained as null — not critical for analysis

**3. Data Type Conversions**
-	Converted Learner SignUp DateTime, Opportunity End Date, Opportunity Start Date and Date of Birth to DateTime type
-	Converted Age to Integer type

**4. Derived / Extracted Columns**
-	Age: calculated as difference between current date and Date of Birth divided by 365
-	Learner SignUp Year, Month, and Day — extracted from Learner SignUp DateTime


#### 5.3.1 Issues Encountered and Resolutions
- **Issue 1:** Categorical Inconsistencies in Institution Name
Entries like 'Federal University Lokoja' and 'Federal University Lokoja, Nigeria' referred to the same institution. Standardised using Text.Proper and manual grouping.
- **Issue 2:** Invalid Datetime Hour Overflow (24:xx:xx)
Some datetime values contained impossible hours (e.g., 24:00:09). Resolved by replacing '24:' with '00:' and adding 1 day.

// Power Query M Code

fixed = Text.Replace(raw, " 24:", " 00:"),
dt = DateTime.FromText(fixed),
corrected = if hour >= 24 then
  DateTime.From(Date.AddDays(DateTime.Date(dt), 1)) + DateTime.Time(dt) else dt
  
- **Issue 3:** Negative Time in Opportunity Value
Dates were flipped when opportunity end date was less than start date.

= IF([@[Opportunity End Date]] < [@[Opportunity Start Date]],
    [@[Opportunity Start Date]] - [@[Opportunity End Date]],
    [@[Opportunity End Date]] - [@[Opportunity Start Date]])
    
- **Issue 4:** Nested Table Values in Power Query
The Learner SignUp DateTime column returned 'Table' objects. Resolved using Table.FirstValue() to extract the correct scalar value.

### 5.4 Feature Engineering
14 new features were engineered to enrich the dataset for machine learning:

| Feature Name                          | Derived From                                 | Rationale                                                                 |
|--------------------------------------|----------------------------------------------|---------------------------------------------------------------------------|
| Age                                  | Date of Birth + Current Date                 | Captures learner demographic profile                                     |
| Time in Opportunity                  | Apply Date + Opportunity End Date            | Measures active engagement duration                                      |
| Engagement Lag                       | SignUp DateTime + Apply Date                 | Time between signup and application, behavioural indicator               |
| Learner SignUp Year                  | Learner SignUp DateTime                      | Enables trend analysis across years                                      |
| Learner SignUp Month                 | Learner SignUp DateTime                      | Enables seasonal and monthly pattern analysis                            |
| Learner SignUp Day                   | Learner SignUp DateTime                      | Identifies peak signup days of the week                                  |
| Normalised Age                       | Age                                          | Rescales Age to 0–1 for fair model weighting                             |
| Normalised Time in Opportunity       | Time in Opportunity                          | Rescales duration to 0–1 range                                           |
| Normalised Opportunity Category      | Encoded Opportunity Category                 | Brings category score to 0–1 scale                                       |
| Encoded Gender                       | Gender                                       | Converts Male and Female to binary values                                |
| Encoded Country (US, India, Nigeria, Ghana, Others) | Country                         | One hot encodes top 4 countries and groups others                        |
| Encoded Opportunity Category         | Opportunity Category                         | One hot encodes each opportunity type as a binary column                 |
| Age x Time in Opportunity            | Age + Time in Opportunity                    | Interaction feature capturing joint influence on outcomes                |
| Engagement Score                     | Normalised Age + Time + Category             | Composite metric summarising overall engagement level                    |

#### 5.4.1 Feature Engineering Calculations
- **1. Age** 
Age was derived by calculating the difference between the learner's Date of Birth and the current date, divided by 365

Age = (DateTime.LocalNow() - [Date of Birth]) / 365

- **2. Min-Max Normalisation** 

=([@Age] - MIN([Age])) / (MAX([Age]) - MIN([Age]))
An Age of 23 in a range of 16–60 normalises to approximately 0.16

- **3. Encoded Country (Others)** 

=IF(OR([@Country]="United States",[@Country]="India",[@Country]="Nigeria",[@Country]="Ghana"),0,1)

- **4. Opportunity Category Encoding**
Encoded based on expected engagement level 1–5 (Internship=5, Course=4, Competition=3, Engagement=2, Event=1).

=IF([@[Opportunity Category]]="Internship",5, IF([@[Opportunity Category]]="Course",4,
  IF([@[Opportunity Category]]="Competition",3, IF([@[Opportunity Category]]="Engagement",2,
    IF([@[Opportunity Category]]="Event",1,0)))))
    
- **5. Engagement Score** 

Engagement Score = (0.5 × Time_Normalized) + (0.2 × Age_Normalized) + (0.3 × OpportunityCategory_Normalized)

Time in Opportunity weighted 50% - most direct measure of active engagement

Opportunity Category weighted 30% - reflects varying commitment levels

Age contributed 20% - demographic influence factor

### 5.5 Data Validation

| Validation Check              | Method                                                                 | Outcome                                                                 |
|------------------------------|------------------------------------------------------------------------|-------------------------------------------------------------------------|
| Datetime format consistency  | Checked all datetime columns parsed without errors in Power Query      | Passed, all remaining rows have valid datetime values                  |
| Null value check             | Reviewed column quality indicators in Power Query (Valid, Error, Empty percent) | Learner SignUp DateTime is 100 percent valid, Opportunity Start Date nulls fully removed |
| Age range check              | Filtered Age column for values below 10 or above 100                   | No unrealistic age values found                                        |
| Normalised value range       | Verified min and max of normalised columns in Excel                    | All normalised columns confirmed within 0 to 1 range                   |
| Engagement Score range       | Checked minimum and maximum of Engagement Score column                 | Values confirmed within expected bounds                                |
| Category encoding accuracy   | Spot checked encoded columns against original Category column          | One hot encoded values correctly matched source categories             |
| Duplicate check              | Checked for duplicate Learner and Opportunity ID combinations          | No duplicates found in cleaned dataset                                 |
| Record count reconciliation  | Compared raw versus cleaned row counts                                 | Raw 8,558 reduced to Cleaned 4,682, reduction explained by null rows and errors |


---

## 6. DATA TRANSFORMATION

The following transformations were applied to prepare data for modelling:

| Transformation | Description |
|---|---|
| `Has_Start_Date` Engineering | Binary flag (0/1) to separate never-started learners from active learners |
| Excel Date-Zero Correction | Corrupted `Time in Opportunity` values for `Has_Start_Date = 0` in excel was recalculated in Python
`NaN` was used instead of the misleading today() function used in excel |
| Engagement Score Recalculation | Recalculated exclusively for started learners after date-zero correction |
| Engagement Lag Correction | Negative lag values (pre-signup applicants) applied absolute values |
| Outlier Correction | Plausible outliers (Age max = 60, Time max = 45,630 hrs) corrected after Excel Date-Zero Correction |
| Encoding | Categorical features (Category, Country, Gender) label-encoded for model compatibility |
| Train-Test Split | 80/20 stratified split preserving the 8.24% churn class proportion |

### 6.2 OUTLIER AND ANOMALY SUMMARY

| Feature | Nature of Outlier | Action Taken |
|---|---|---|
| Time in Opportunity | Max = 45,630 hrs (~5.2 yrs) | Retained for started learners; NaN for never-started |
| Engagement Lag | Negative values (min = −570 hrs) | Retained — high-motivation pre-signup cohort |
| Age | Max = 60; Min = 15 | Retained; mean imputation for nulls |
| Engagement Score | Spike at 2.05+ for `Has_Start_Date = 0` | Recalculated after date-zero correction |
| `Has_Start_Date = 0` | 3,708 rows (44% of dataset) | Flagged rather than deleted |

---
## 7 Exploratory Data Analysis
### 7.1 Key Numerical Features — Statistical Summary

| Feature | Mean | Std Dev | Min | Median | Max |
|---|---|---|---|---|---|
| Age (years) | 26.6 | 4.6 | 15 | 26 | 60 |
| Engagement Score | 1.60 | 0.48 | 0.30 | 1.57 | 2.16 |
| Sign-ups (monthly avg) | 724 | — | 352 | — | 1,167 |

### 7.2 Distribribution & Outlier Analysis
- **Age Distribution** - Right-skewed; majority of learners aged 18–30, mean = 26.6 years. Outliers above age 45 represent non-traditional learners.
<img width="871" height="359" alt="Screenshot 2026-04-22 101010" src="https://github.com/user-attachments/assets/9e85ff12-604e-4684-a202-6315099963cf" />

- **Engagement Score** - Bimodal distribution; cluster at 0.30–0.40 (Event learners) and 2.05–2.10 (Internship learners). The 1.10 threshold is the primary early-intervention trigger point.
  <img width="876" height="367" alt="Screenshot 2026-04-22 101024" src="https://github.com/user-attachments/assets/f0a8dadf-32dd-4c2b-a105-5ce9d6101da1" />

  - **Engagement Score Distribution by Opportunity Category**
Internships dominate platform engagement with a median score of **2.06** — **61% higher** than
Courses (1.28) and **more than 5× higher** than Events (0.37). The steep drop-off across
categories suggests learners treat opportunity types very differently in terms of depth of
involvement.

<img width="870" height="373" alt="Screenshot 2026-04-22 101158" src="https://github.com/user-attachments/assets/9983f90a-75ee-430b-b0ff-93718cd91676" />


### 7.3 Categorical Distribution Summary
- **Opportunity Category** — Internship leads at 63.4% (5,317 records). Course is second at 23.7% (1,992). Together they represent 87.1% of all platform activity.
 
  <img width="878" height="390" alt="Screenshot 2026-04-22 101041" src="https://github.com/user-attachments/assets/ef943a4e-d8be-4839-9c23-6bdaec53f37f" />
  
- **Geographic and Gender Breakdown** 
  
The **United States** (3,869) and **India** (2,811) account for ~**79.7%** of all learners.
Nigeria leads African representation at **729 learners**, with Ghana (262) and Pakistan (218)
rounding out the top five. A **male majority** is consistent across all markets.
<img width="871" height="375" alt="Screenshot 2026-04-22 101141" src="https://github.com/user-attachments/assets/d9dcca1d-c22b-40e2-a8f2-9850e0494f65" />


- **Learners Status Description** 
  <img width="549" height="264" alt="Screenshot 2026-04-25 164445" src="https://github.com/user-attachments/assets/088da9a1-492d-46bc-84af-00fc3d9585b4" />

**Rejected** and **Team Allocated** together account for **80%** of all learner records,
dominating the funnel. Only **9%** of learners reach an active *Started* state, and
**7.2%** drop out — meaning attrition nearly matches active participation.
> Of learners who progress past rejection, the **dropout rate relative to starters is ~45%**

- **Where are the most churners coming from**
  **Churn Analysis: Contribution & Absolute Count by Opportunity Category**

**Internships overwhelmingly drive platform churn.** They account for **92.5%** of all
churned learners and the highest absolute dropout count at **660**, dwarfing every
other category combined. This mirrors Internships' dominance in volume, but the
churn concentration is disproportionate even relative to their 63% activity share.

<img width="270" height="120" alt="Screenshot 2026-04-22 101804" src="https://github.com/user-attachments/assets/3c6a8130-e807-47ae-8011-c86825c2dcfa" />

#### Contribution to Total Churners (%)

Internship churn at **660 learners** against a started base suggests the dropout-to-start ratio is critically high in this category. 
Since Internships also carry the **highest engagement scores (2.06)**, churn here represents the greatest lost value on the platform.
Reducing Internship dropout by even **10%** (~66 learners retained) would nearly **triple** the current Rewards Award completion count of 29.

  <img width="270" height="120" alt="Screenshot 2026-04-22 101804" src="https://github.com/user-attachments/assets/6a23c582-b3c3-4213-a0bf-33585e3682b4" />


### 7.4 Seasonal Patterns
**Sign Ups By Month**
<img width="871" height="362" alt="Screenshot 2026-04-22 101113" src="https://github.com/user-attachments/assets/38a6e450-5588-41bd-a764-84e6a5ae828d" />

| Month | Sign-Ups | Note |
|---|---|---|
| August | 1,167 | Peak month — 57% above average |
| January | 992 | New Year motivation effect |
| February | 923 | Strong winter performance |
| November | 352 | Weakest month of the year |

- **By Day of Week:** Thursday leads (1,385), followed by Friday (1,328) and Monday (1,316). Weekends fall ~30% below the 1,199 daily average which is very understandable.
- <img width="870" height="364" alt="Screenshot 2026-04-22 101129" src="https://github.com/user-attachments/assets/2c92efe7-fdd6-4141-bafb-9e53d64edbfc" />

### 7.5 Learner Status and Feature Correlations
### Feature Correlation Heatmap — Key Variables

Strong multicollinearity exists across engineered time and engagement features.
**Normalized Time in Opportunity**, **New Time in Opportunity**, and **Engagement Lag**
form a near-perfect correlation cluster (r = 0.97–1.00), signaling redundancy that
warrants feature pruning before modeling.

| Feature Pair | Correlation | Strength |
|---|---|---|
| Engagement Lag ↔ Normalized Age | 1.00 | Perfect positive |
| New Time in Opportunity ↔ Normalized Time in Opportunity | 1.00 | Perfect positive |
| New Engagement Lag ↔ Engagement Lag | 1.00 | Perfect positive |
| Age × Time in Opportunity ↔ Engagement Score | 0.98 | Very strong positive |
| Age × Time in Opportunity ↔ Normalized Time in Opportunity | 0.98 | Very strong positive |
| Engagement Score ↔ New Time in Opportunity | 0.97 | Very strong positive |
| Has_Start_Date ↔ Engagement Score | −0.98 | Very strong negative |
| Has_Start_Date ↔ New Time in Opportunity | −0.88 | Strong negative |
| Normalized Opportunity Category ↔ Engagement Score | 0.84 | Strong positive |
| New Engagement Lag ↔ New Time in Opportunity | 0.80 | Strong positive |
<img width="260" height="200" alt="Screenshot 2026-04-22 101426" src="https://github.com/user-attachments/assets/ea40a5db-04ef-4d51-afd9-fff8bcb149d0" />


> **`Has_Start_Date`** is the only feature with strong *negative* correlations (−0.53 to −1.0),
> suggesting that opportunities lacking a defined start date are systematically associated
> with lower engagement — a potentially high-signal binary flag for predictive models.
> The interaction term **Age × Time in Opportunity** (r = 0.98 with Engagement Score)
> outperforms either variable alone, making it a strong candidate as a composite feature.

### 7.6 The `Has_Start_Date` Feature

A critical binary feature — `Has_Start_Date` — was engineered to distinguish learners who have an Opportunity Start Date recorded (`= 1`) from those who do not (`= 0`).

> Learners with `Has_Start_Date = 0` never progressed to the start stage of their opportunity. This group represents approximately **44% of the full dataset** and constitutes the **highest-risk segment** on the platform.


## 8. Predictive Modelling
The primary prediction task is Learner Churn (Drop-off) Prediction, identifying which learners are at risk of dropping out or withdrawing from their assigned opportunity before completion. The target variable Is_Dropoff combines the 'Dropped Out' (7.2%, 605 learners) and 'Withdraw' (1%, 86 learners) statuses identified above on Learners Status Distribution, giving a total churn rate of 9.2% (691 learners).


### 8.1 Model Performance Comparison

Five classification algorithms were evaluated using an 80/20 stratified train-test split:

| Model | Accuracy | Precision | Recall | F1-Score |
|---|---|---|---|---|
| Logistic Regression | 0.9178 | 0.0000 | 0.0000 | 0.0000 |
| Decision Tree | 0.9207 | 0.5180 | 0.5217 | 0.5199 |
| SVM | 0.9178 | 0.0000 | 0.0000 | 0.0000 |
| Random Forest | 0.9303 | 0.5913 | 0.4928 | 0.5375 |
| **Gradient Boosting ★** | **0.9422** | **0.8060** | **0.3913** | **0.5268** |

<img width="710" height="340" alt="Screenshot 2026-04-22 104033" src="https://github.com/user-attachments/assets/823e8053-f91b-48fc-baf0-15068ae8581b" />


> **Key Finding:** Gradient Boosting wins on Accuracy (94.22%) and Precision (80.60%). However, its Recall of 39.13% means it misses 60% of actual churners. SMOTE or class-weight balancing is the critical next step.

### 8.2 Feature Importance — Gradient Boosting

The model is overwhelmingly driven by **temporal features**. **Engagement Lag** alone
accounts for **62.2%** of predictive power, and combined with **Time in Opportunity**
(25.8%), the top two features explain **88%** of the model's decisions. Demographic
and categorical encodings contribute minimally.

<img width="260" height="150" alt="Screenshot 2026-04-22 101741" src="https://github.com/user-attachments/assets/2e274696-b7e2-463b-b63f-6192440712ad" />

> **Engagement Lag** and **Time in Opportunity** are pure behavioral-timing signals —
> the model is essentially predicting outcomes based on *how long* and *how late*
> a learner engages, not who they are.
> Demographic features (Age, Gender, Country) collectively contribute just **2.4%**, suggesting the classifier is
identity-agnostic and timing-dominant.

### 8.3 Confusion Matrix — Gradient Boosting (Test Set)
<img width="155" height="135" alt="Screenshot 2026-04-22 101749" src="https://github.com/user-attachments/assets/bb201ca4-0875-4233-a2a4-2380a376a542" />

|  | Predicted Negative | Predicted Positive |
|---|---|---|
| **Actual Negative** | 1,527 (True Negative) | 13 (False Positive) |
| **Actual Positive** | 84 (False Negative) | 54 (True Positive) |

The high False Negative count (84) relative to True Positives (54) reflects the class imbalance challenge. 
Implementing SMOTE is the most direct path to improving recall from 39.13% toward the 63.97% achieved in the clean-data baseline.

---

## 8 SMOTE Resampling — Improving Churn Detection Recall

### Why SMOTE Was Needed

The original training set suffered from severe **class imbalance**:

- **Non-Churners:** 6,159 records
- **Churners:** 553 records
- **Imbalance Ratio:** ~11:1

This caused the Gradient Boosting model to achieve a misleadingly high accuracy of
**94.22%** while only catching **39.13%** of actual churners - missing 84 out of 138
at-risk learners in the test set. Logistic Regression and SVM scored **zero F1**
entirely, predicting "no churn" for every record.

> **The core problem:** A model that ignores the minority class entirely can still
> score 91%+ accuracy simply because 91% of learners genuinely do not churn.
> Accuracy is the wrong metric when classes are this imbalanced.

---

### What SMOTE Does

**SMOTE (Synthetic Minority Over-sampling Technique)** generates *synthetic* churner
records by interpolating between existing minority class examples — rather than simply
duplicating them. This forces the model to learn what a churner actually looks like
instead of defaulting to the majority class.

SMOTE was applied **exclusively to the training set**. The test set was never
resampled — it retains the real-world 8.24% churn distribution to ensure evaluation
reflects genuine platform conditions.

```python
from imblearn.over_sampling import SMOTE

smote = SMOTE(random_state=42)
X_train_resampled, y_train_resampled = smote.fit_resample(X_train_scaled, y_train)
```

**Class balance after SMOTE:**

| Split | Before SMOTE | After SMOTE |
|---|---|---|
| Churners (training) | 553 | 6,159 |
| Non-Churners (training) | 6,159 | 6,159 |
| Ratio | 11:1 | 1:1 (balanced) |
| Test set | Unchanged | Unchanged |

---

### Model Performance — Before vs After SMOTE
<img width="369" height="180" alt="image" src="https://github.com/user-attachments/assets/4efcbf0f-4231-4f54-b09c-9f9e40cd3dfc" />

| Model | Metric | Before SMOTE | After SMOTE | Change |
|---|---|---|---|---|
| Logistic Regression | Recall | 0.00% | 60.14% | ⬆ +60.14% |
| Logistic Regression | F1 | 0.000 | 0.274 | ⬆ Fixed |
| Decision Tree | Recall | 52.17% | 58.70% | ⬆ +6.5% |
| Decision Tree | F1 | 0.520 | 0.475 | ≈ stable |
| SVM | Recall | 0.00% | 68.12% | ⬆ +68.12% |
| SVM | F1 | 0.000 | 0.264 | ⬆ Fixed |
| Random Forest | Recall | 49.28% | 57.97% | ⬆ +8.7% |
| Random Forest | F1 | 0.538 | 0.520 | ≈ stable |
| **Gradient Boosting** | **Recall** | **39.13%** | **78.99%** | **⬆ +39.9%** |
| **Gradient Boosting** | **F1** | **0.527** | **0.498** | **≈ stable** |

---

### Gradient Boosting — Full Metric Comparison

| Metric | Before SMOTE | After SMOTE | Verdict |
|---|---|---|---|
| Accuracy | 94.22% | 86.89% | ⬇ Expected drop |
| Precision | 80.60% | 36.33% | ⬇ Acceptable trade-off |
| Recall | 39.13% | **78.99%** | ⬆ Primary goal achieved |
| F1-Score | 0.527 | 0.498 | ≈ Comparable |
| Churners Missed (FN) | ~84 | ~29 | ⬆ 55 fewer missed |

> **55 additional at-risk learners identified per prediction cycle** — purely from
> rebalancing the training data. No new features, no new data collected.

---

### Confusion Matrix Comparison — Gradient Boosting

**Before SMOTE**

|  | Predicted No Churn | Predicted Churn |
|---|---|---|
| **Actual No Churn** | 1,527 ✅ | 13 ❌ |
| **Actual Churn** | 84 ❌ | 54 ✅ |

**After SMOTE**

|  | Predicted No Churn | Predicted Churn |
|---|---|---|
| **Actual No Churn** | ~1,440 ✅ | ~100 ❌ |
| **Actual Churn** | ~29 ❌ | ~109 ✅ |

The model now catches **~109 churners** vs **54 before** — at the cost of
flagging ~100 non-churners unnecessarily instead of 13.

---

### Threshold Tuning — Gradient Boosting After SMOTE

Beyond SMOTE, the decision threshold was tuned to further control the
precision-recall trade-off:

| Threshold | Precision | Recall | F1-Score | Use Case |
|---|---|---|---|---|
| **0.5** | 36.33% | 78.99% | **0.498** | ✅ Recommended — best F1 |
| 0.4 | 34.47% | 80.43% | 0.483 | Marginal recall gain |
| 0.3 | 29.35% | 85.51% | 0.437 | High recall, low precision |
| 0.2 | 17.25% | 89.86% | 0.289 | Too aggressive |

**Recommended threshold: 0.5** — delivers the strongest F1 score while
capturing nearly 8 in 10 at-risk learners. Only lower the threshold if
the platform can operationally handle a high volume of intervention flags.

---

### Why After SMOTE Is the Better Model for This Use Case

The accuracy drop from 94.22% to 86.89% is intentional and correct.

**Before SMOTE** — The model behaves like a cautious screener that rarely
flags anyone. When it does, it is usually right (80% precision). But it
silently misses **84 out of 138** genuine churners — learners who needed
help and received none.

**After SMOTE** — The model flags more aggressively. It misses only
**~29 churners**, recovering 55 additional at-risk learners per cycle.
The false alarm cost on a digital platform is negligible — an unnecessary
nudge email — while the cost of a missed churner is a permanent dropout.

| Decision Factor | Before SMOTE | After SMOTE | Better For This Project |
|---|---|---|---|
| Learners correctly flagged at risk | 54 | ~109 | ✅ After |
| Learners wrongly flagged | 13 | ~100 | ✅ Before |
| Churners silently missed | 84 | ~29 | ✅ After |
| Cost of false alarm | Low (nudge email) | Low (nudge email) | Neutral |
| Cost of missed churner | High (permanent dropout) | High (permanent dropout) | Neutral |
| Serves platform retention KPI | ❌ Partially | ✅ Yes | ✅ After |

> **Design Decision:** After SMOTE is adopted as the final production model.
> The precision trade-off is explicitly accepted given that interventions on
> this platform are low-cost (automated emails, peer nudges, mentor flags)
> and the consequences of missing a churner — permanent learner loss — far
> outweigh the cost of an unnecessary outreach message.

---

### Key Takeaway

SMOTE did not make the model more accurate. It made the model more **useful**.
A 94% accurate model that ignores 84 real churners does not serve the platform.
An 87% accurate model that catches 109 of them does.

> **Data does not solve problems. A model that catches the right people does.**



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

