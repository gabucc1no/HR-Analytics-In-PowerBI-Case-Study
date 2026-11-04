# 🏢 HR Analytics & Workforce Intelligence | Power BI Case Study

[![Power BI](https://img.shields.io/badge/Power_BI-Executive_Dashboard-F2C811?style=for-the-badge&logo=powerbi&logoColor=black)](https://powerbi.microsoft.com/)
[![Data Model](https://img.shields.io/badge/Schema-Kimball_Snowflake-0D9488?style=for-the-badge)](#)
[![Status](https://img.shields.io/badge/Status-Completed-success?style=for-the-badge)](#)

> An end-to-end People Analytics business intelligence solution developed for **Atlas Labs**, a technology enterprise. Designed to evaluate workforce demographics, track multi-year employee performance and sentiment cycles, and isolate the core behavioral and operational drivers behind employee attrition across 1,470 personnel records.

---

## 🖼️ Executive Dashboard Showcase

| 1. High-Level Workforce Overview | 2. Diversity, Demographics & Equity |
| :---: | :---: |
| ![Workforce Overview](images/1_overview.png) | ![Demographics](images/2_demographics.png) |

| 3. Individual Performance & Review Tracker | 4. Employee Attrition & Flight Risk Diagnostics |
| :---: | :---: |
| ![Performance Tracker](images/3_performance_tracker.png) | ![Attrition Diagnostics](images/4_attrition.png) |

---

## 📌 Executive Summary & Key Workforce KPIs

Based on active and historical personnel records at Atlas Labs:

| Metric | Measured Value | Business Interpretation & Context |
| :--- | :---: | :--- |
| **Total Headcount Recorded** | **1,470** | Total cumulative employees captured in the data model |
| **Active Workforce** | **1,233** | Retained workforce contributing to ongoing operations |
| **Historical Exits (Inactive)** | **237** | Documented voluntary and involuntary departures |
| **Company Attrition Rate** | **16.1%** | Baseline organizational turnover benchmark |
| **Workforce Age Span** | **18 – 51 (Avg: 29.0)** | Strong concentration within entry- and early-career tiers |
| **Peak Overtime Attrition** | **~30.0%** | Staff subject to mandatory overtime exit at nearly 3x the baseline rate |

---

## ⚙️ Data Preparation & Transformation Pipeline (Power Query / ETL)

The raw data comprised transactional HR event logs and normalized dimension lookups. An end-to-end Power Query ETL pipeline was established to clean, structure, and shape the tables before ingestion into the data model:

```text
Raw Sources (CSV/Excel)
   │
   ├── Fact Table Ingestion ──► Cast Data Types ──► Validate Review Cycles ──► Clean Score Keys
   │
   ├── DimEmployee Profiling ─► Clean Strings ──► Parse Attrition Dates ──► Age & Tenure Binning
   │
   └── Dimension Decoupling ──► Normalize Lookup Codes (1:M) ──► Connect to Kimball Snowflake

```

### 1. Data Cleaning & Type Enforcement

* **Datatype Casting:** Explicitly converted dates (`HireDate`, `AttritionDate`, `ReviewDate`) into standardized ISO dates using Power Query locale-aware parsing to avoid runtime type coercion.
* **Null & Anomaly Handling:** Audited `AttritionDate` records—imputing active personnel via `ISBLANK` to distinguish active headcount from historical exit cohorts without record loss.
* **Deduplication & Key Verification:** Enforced entity integrity across primary keys (`EmployeeID`, `PerformanceID`) to guarantee unambiguous 1:M relationships without many-to-many cross-filtering issues.

### 2. Feature Engineering & Column Transformations

* **Dynamic Age Bins:** Segmented continuous employee age into strategic 10-year generational cohorts (`<20`, `20-29`, `30-39`, `40-49`, `50+`) to isolate early-career flight risk patterns.
* **Conditional Tenure & Role Categorization:** Implemented multi-condition classification logic via `SWITCH(TRUE(), ...)` to segment personnel by career longevity and department seniority:
```dax
Tenure_Segment = 
SWITCH(
    TRUE(),
    'DimEmployee'[YearsAtCompany] <= 1, "0-1 Years (Early Flight Risk)",
    'DimEmployee'[YearsAtCompany] <= 3, "2-3 Years (Transition)",
    'DimEmployee'[YearsAtCompany] <= 5, "4-5 Years (Mid-Tenure)",
    "6+ Years (Tenured Staff)"
)

```


* **Time-Span Derivations:** Calculated relative tenure, time since last promotion, and employee age dynamically using `DATEDIFF()` against rolling date boundaries.

---

## 🏗️ Relational Data Modeling & Architecture

The report implements a **Kimball Snowflake Schema** to maintain data integrity across one-to-many relationships, separating snapshot facts from normalized dimensional entities:

```text
                  ┌───────────────────────┐
                  │   DimEducationLevel   │
                  └───────────┬───────────┘
                              │ (1:M)
   ┌──────────────┐ (1:M) ┌───┴───────────┐ (1:M) ┌────────────────────────┐
   │   DimDate    ├───────┤  DimEmployee  ├───────┤  FactPerformanceRating  │
   └──────────────┘       └───────────────┘       └───┬────────────────┬───┘
                                                      │                │
                                                (1:M) │          (1:M) │
                                      ┌───────────────┴──┐   ┌─────────┴────────────┐
                                      │  DimRatingLevel  │   │  DimSatisfiedLevel   │
                                      └──────────────────┘   └──────────────────────┘

```

* **Fact Table (`FactPerformanceRating`):** Central fact storing recurring annual reviews, evaluation scores (Manager & Self), and granular satisfaction indices (1–5 scale).
* **Primary Dimension (`DimEmployee`):** Stores static and slowly-changing personnel data, including hire dates, termination dates, department, job role, and compensation bands.
* **Normalized Sub-Dimensions:** `DimEducationLevel`, `DimRatingLevel`, and `DimSatisfiedLevel` normalize categorical descriptions from numerical codes, optimizing VertiPaq engine memory usage.
* **Dedicated Time Dimension (`DimDate`):** Supports time-series drilldowns across years, quarters, and months for hiring trajectory modeling without relying on auto-datetime hierarchies.

---

## 📐 Selected DAX Measures

```dax
// Cumulative personnel count
TotalEmployees = DISTINCTCOUNT('DimEmployee'[EmployeeID])

// Retained personnel
ActiveEmployees = 
CALCULATE(
    [TotalEmployees],
    ISBLANK('DimEmployee'[AttritionDate])
)

// Departed personnel
InactiveEmployees = 
CALCULATE(
    [TotalEmployees],
    NOT(ISBLANK('DimEmployee'[AttritionDate]))
)

// Organizational turnover benchmark
% Attrition Rate = 
DIVIDE(
    [InactiveEmployees],
    [TotalEmployees],
    0
)

// Dynamic Date Dimension Generation
DimDate = 
ADDCOLUMNS(
    CALENDAR(DATE(2010, 1, 1), DATE(2025, 12, 31)),
    "Year", YEAR([Date]),
    "Quarter", "Q" & FORMAT([Date], "Q"),
    "Month", FORMAT([Date], "MMM"),
    "MonthNumber", MONTH([Date])
)

```

---

## 🔍 Core Business Insights & Diagnostic Findings

### 1. Overtime Demands Multiply Departure Risk

* Employees working under mandatory **Overtime requirements exhibit an attrition rate near 30.0%**, compared to approximately **10.0% for non-overtime staff**.
* Overtime fatigue serves as one of the single largest operational contributors to voluntary turnover across Atlas Labs.

### 2. High Early-Tenure Flight Risk (The "First-Year Cliff")

* Attrition peaks severely during years **0 and 1 (exceeding 30%–34%)**, before declining sharply after year 2.
* Employees who remain past year 3 stabilize below a 15% exit rate, indicating onboarding and early career alignment represent the most vulnerable operational stages.

### 3. Role-Specific Attrition Concentration

* **Sales Representatives** (~40%) and **Human Resources Recruiters** (~38%) endure the highest department attrition rates.
* In contrast, specialized technology roles (such as Engineering Managers and Senior Software Engineers) record strong retention with attrition rates below 8%.

### 4. Travel Frequency & Work-Life Equilibrium

* Staff categorized as **Frequent Travellers encounter ~25% attrition**, notably exceeding non-traveling peers (~8%), demonstrating an inverse relationship between sustained travel requirements and long-term retention.

### 5. Multi-Year Sentiment Tracking (Individual Drill-Down)

* The dedicated **Performance Tracker** reveals that employee sentiment (Job Satisfaction, Work-Life Balance, and Environment Rating) typically displays a steady downward trend across consecutive review cycles 12 to 24 months before formal exit dates occur.

---

## 💡 Strategic HR Recommendations

1. **Structured 90-Day Early Onboarding Checkpoints:** Address the 30%+ turnover among year 0–1 hires by establishing cross-departmental mentorship frameworks and structured milestone reviews within the first 6 months.
2. **Overtime Audit & Workload Rebalancing:** Because overtime employees leave at nearly 3x the baseline rate, Atlas Labs leadership should audit teams with chronic overtime and redistribute project allocations or expand headcount capacity.
3. **Automated Sentiment Early-Warning Indicators:** Leverage the Performance Tracker to set automated alert flags whenever an employee’s combined satisfaction scores drop below 3 for two consecutive review cycles.
4. **Compensation & Travel Band Adjustments:** Calibrate compensation bands and travel stipends for frequent-travel roles and entry-level sales professionals experiencing disproportionate turnover.
