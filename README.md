# 🏢 HR Analytics & Workforce Intelligence | Power BI Case Study

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
| **Company Attrition Rate** | **16.1%** | Baseline organizational attrition benchmark |
| **Youngest / Oldest Employee** | **18 / 51** | Workforce concentrated heavily within the 20–29 age bracket |
| **Peak Overtime Attrition** | **~30.0%** | Staff requiring overtime leave at nearly 3x the rate of non-overtime peers |

---

## 🔍 Core Business Insights & Diagnostic Findings

### 1. Overtime Demands Multiply Departure Risk
- Employees working under mandatory **Overtime requirements exhibit an attrition rate near 30.0%**, compared to approximately **10.0% for non-overtime staff**.
- Overtime fatigue serves as one of the single largest operational contributors to voluntary turnover across Atlas Labs.

### 2. High Early-Tenure Flight Risk (The "First-Year Cliff")
- Attrition peaks severely during years **0 and 1 (exceeding 30%–33%)**, before declining sharply after year 2.
- Employees who remain past year 3 stabilize below a 15% exit rate, indicating onboarding and early career alignment represent the most vulnerable operational stages.

### 3. Role-Specific Attrition Concentration
- **Sales Representatives** and **Human Resources** roles endure the highest department attrition rates (~38%–40%).
- In contrast, specialized technology roles (such as Engineering Managers and Senior Software Engineers) record strong retention with attrition rates below 8%.

### 4. Travel Frequency & Work-Life Equilibrium
- Staff categorized as **Frequent Travellers encounter ~25% attrition**, notably exceeding non-traveling peers (~8%), demonstrating an inverse relationship between sustained travel requirements and long-term retention.

### 5. Multi-Year Sentiment Tracking (Individual Drill-Down)
- The dedicated **Performance Tracker** reveals that employee sentiment (Job Satisfaction, Work-Life Balance, and Environment Rating) typically displays a steady downward trend across consecutive review cycles 12 to 24 months before formal exit dates occur.

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

- **Fact Table (`FactPerformanceRating`):** Central fact storing recurring annual reviews, evaluation scores (Manager & Self), and granular satisfaction indices (1–5 scale).
- **Primary Dimension (`DimEmployee`):** Stores static and slowly-changing personnel data, including hire dates, termination dates, department, job role, and compensation bands.
- **Normalized Sub-Dimensions:** `DimEducationLevel`, `DimRatingLevel`, and `DimSatisfiedLevel` normalize categorical descriptions from numerical codes.
- **Dedicated Time Dimension (`DimDate`):** Supports time-series drilldowns across years, quarters, and months for hiring trajectory modeling.

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

## 💡 Strategic HR Recommendations

1. **Structured 90-Day Early Onboarding Checkpoints:** Address the 30%+ turnover among year 0–1 hires by establishing cross-departmental mentorship frameworks and structured milestone reviews within the first 6 months.
2. **Overtime Audit & Workload Rebalancing:** Because overtime employees leave at nearly 3x the baseline rate, Atlas Labs leadership should audit teams with chronic overtime and redistribute project allocations or expand headcount capacity.
3. **Automated Sentiment Early-Warning Indicators:** Leverage the Performance Tracker to set automated alert flags whenever an employee’s combined satisfaction scores drop below 3 for two consecutive review cycles.
4. **Compensation & Travel Band Adjustments:** Calibrate compensation bands and travel stipends for frequent-travel roles and entry-level sales professionals experiencing disproportionate turnover.

---
