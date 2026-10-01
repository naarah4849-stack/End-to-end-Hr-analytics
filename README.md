

# End-to-End HR Analytics Dashboard & Data Pipeline

A comprehensive human resources data project featuring programmatic data cleaning via **Python (Pandas & NumPy)**, secondary ETL modeling inside **Power Query**, and advanced data analytics with **Power BI** leveraging structured time-intelligence DAX metrics.

## 🛠️ Tech Stack & Architecture

1. **Data Engineering & Wrangling (Python):** Resolved structural anomalies, handled missing/corrupted primary keys, and performed vectorized logical cleaning using `pandas` and `numpy` in a Jupyter Notebook environment.
2. **ETL & Data Modeling (Power Query & Power BI):** Engineered data type coercion, connected and blended multiple data sources, established star-schema relationships, generated a dedicated calendar dimension (`dcalendar`), and constructed an interactive visual reporting layer.


## 📊 Dashboard Preview

<img width="1473" height="831" alt="hrgif" src="https://github.com/user-attachments/assets/4f8adefc-85b5-41e0-b56a-7258631ddadd" />


## 🧲 Dual-Phase Data Cleaning

### 1. Python (Pandas & NumPy) Phase
The raw raw dataset (`hr_dirty_portfolio.csv`) was heavily corrupted with placeholders, negative financial records, and indexing errors. The data pipeline automated the following fixes:
* **Structural Row & Column Purge:** Dropped rows and columns that were completely empty across all axis dimensions (`dropna(how="all")`).
* **Deduplication:** Sanitized the dataset by identifying and removing exact duplicate records (`drop_duplicates()`).
* **Primary Key Recovery (`employee_id`):** Isolated rows containing `"MISSING_ID"`, converted them to standard null values (`pd.NA`), and programmatically reconstructed unique IDs using a text-replacement pipeline mapped directly from the `full_name` column.
* **String Standardization:** Handled text inconsistencies in categorical fields by stripping trailing whitespaces and applying uniform title-case structures (`.str.strip().str.title()`).
* **Financial Anomaly Filtering:** Created a vectorized application rule to convert values flagged as `"CONFIDENTIAL"` or containing negative salary markers (`val < 0`) into structured null configurations (`np.nan`).
* **Pipeline Export:** Persisted the newly cleaned state down to an optimized source asset named `hr_cleaned_portfolio.csv`.

### 2. Power BI & Power Query Phase
* **Data Blending & Integration:** Imported and connected a secondary contextual data asset, merging information fields across the core tables to enable cohesive holistic analysis.
* **Time-Intelligence Foundations:** Generated a contiguous calendar dimension table (`dcalendar`) to enable seamless chronological groupings.
* **Schema Optimization:** Established relationship dimensions between facts, lookups, and calendars to build a robust data model and prevent calculation ambiguity.

## 🧮 DAX Measures Engineered

A robust set of 22 analytical DAX measures was developed to support workforce demographic calculations, headcount tracking, and Year-over-Year (YOY) time-intelligence trends.

<details>
<summary>📐 Click here to expand and view all 22 DAX Formulations</summary>

### 📋 Core Metrics & Baseline Configurations

```dax
DataVigenteMedida = MAX('hr_cleaned_portfolio (1)'[hire_date])
```

```dax
activeemployees = 
CALCULATE(
    COUNTA('hr_cleaned_portfolio (1)'[employee_id]), 
    employee_cleaned_synthetic[active] = "yes"
)
```

```dax
notactive = 
CALCULATE(
    COUNTA('hr_cleaned_portfolio (1)'[employee_id]), 
    employee_cleaned_synthetic[active] = "no"
)
```

```dax
exceeds by gender = 
CALCULATE(
    COUNTA(employee_cleaned_synthetic[gender]), 
    'hr_cleaned_portfolio (1)'[performance_score] = "Exceeds"
)
```

### 👤 Headcount Demographics (Gender Split)

```dax
Max_count_of_female = 
CALCULATE(
    COUNTA(employee_cleaned_synthetic[gender]), 
    employee_cleaned_synthetic[gender] = "Female"
)
```

```dax
Max_count_of_male = 
CALCULATE(
    COUNTA(employee_cleaned_synthetic[gender]), 
    employee_cleaned_synthetic[gender] = "Male"
)
```

```dax
Max_count_of_nonbinary = 
CALCULATE(
    COUNTA(employee_cleaned_synthetic[gender]), 
    employee_cleaned_synthetic[gender] = "Non-binary"
)
```

### 📈 Time-Intelligence & YOY Analytics (Female Metrics)

```dax
female_hiring_YTD = 
TOTALYTD(
    CALCULATE(
        COUNT('hr_cleaned_portfolio (1)'[employee_id]), 
        employee_cleaned_synthetic[gender] = "Female"
    ),
    dcalendar[Datas]
)
```

```dax
Female_hirings_LY YTD = CALCULATE([female_hiring_YTD], SAMEPERIODLASTYEAR(dcalendar[Datas]))
```

```dax
% Female_hirings_YOY = DIVIDE([female_hiring_YTD] - [Female_hirings_LY YTD], [Female_hirings_LY YTD])
```

### 📈 Time-Intelligence & YOY Analytics (Male Metrics)

```dax
Male_hirings_YTD = 
TOTALYTD(
    CALCULATE(
        COUNT('hr_cleaned_portfolio (1)'[employee_id]), 
        employee_cleaned_synthetic[gender] = "Male"
    ), 
    dcalendar[Datas]
)
```

```dax
Male_hirings_LY YTD = CALCULATE([Male_hirings_YTD], SAMEPERIODLASTYEAR(dcalendar[Datas]))
```

```dax
% male_hirings_YOY = DIVIDE([Male_hirings_YTD] - [Male_hirings_LY YTD], [Male_hirings_LY YTD])
```

### 📈 Time-Intelligence & YOY Analytics (Non-Binary Metrics)

```dax
Non-binary_hirings_YTD = 
TOTALYTD(
    CALCULATE(
        COUNT('hr_cleaned_portfolio (1)'[employee_id]), 
        employee_cleaned_synthetic[gender] = "Non-binary"
    ), 
    dcalendar[Datas]
)
```

```dax
Non-binary_hirings_LY YTD = CALCULATE([Non-binary_hirings_YTD], SAMEPERIODLASTYEAR(dcalendar[Datas]))
```

```dax
% non-binary_hirings_YOY = DIVIDE([Non-binary_hirings_YTD] - [Non-binary_hirings_LY YTD], [Non-binary_hirings_LY YTD])
```

### 📊 Global Headcount Tracking

```dax
num_employees_YTD = TOTALYTD(COUNT('hr_cleaned_portfolio (1)'[employee_id]), dcalendar[Datas])
```

```dax
num_employees_ LY YTD = CALCULATE([num_employees_YTD], SAMEPERIODLASTYEAR(dcalendar[Datas]))
```

```dax
% num_employees_YOY = DIVIDE([num_employees_YTD] - [num_employees_ LY YTD], [num_employees_ LY YTD])
```

### 💰 Payroll & Financial Operations Analysis

```dax
payroll_cost_YTD = TOTALYTD(COUNT('hr_cleaned_portfolio (1)'[salary]), dcalendar[Datas])
```

```dax
payroll_cost_LY YTD = CALCULATE([payroll_cost_YTD], SAMEPERIODLASTYEAR(dcalendar[Datas]))
```

```dax
%payroll_YOY = DIVIDE([payroll_cost_YTD] - [payroll_cost_LY YTD], [payroll_cost_LY YTD])
```

</details>

## 🗺️ Power BI Insights & Report View

* **Dynamic Chronological Performance:** Tracks historical monthly metrics against equivalent baselines from previous cycles using robust `TOTALYTD` and `SAMEPERIODLASTYEAR` formulations.
* **Workforce Demographics:** Provides a clear breakdown of employee trends across multiple gender identifiers, retention states, and job performance categories.
* **Payroll Scale Insights:** Equips stakeholders with an interactive overview of cumulative operational compensation and growth tracking metrics.
