# 📊 Student Wellness Dashboard

**Course Section:** C303  
**Technical Preprocessing & Implementation Lead:** Geraldine C. Mendoza  
**Project Group Members:** Geraldine C. Mendoza, Jhulia Reign Gilhang, Janier Levine U. Santos, Sittie Aisah S. Serad  
**Dataset Scope:** 100,000 Student Records (Sourced via Kaggle)  

---

## 🏛️ System Architecture & Rationale for Tool Selection

For the data cleaning and data preparation phase, I made the conscious decision to utilize **XAMPP** and **MySQL Workbench** as our core backend database environment instead of relying on Power BI's Power Query tools. While performing raw data preparation inside Power BI might initially seem more convenient since modeling and visualizations happen within that same software, a dedicated relational database management system (RDBMS) offers superior advantages for a professional engineering pipeline:

* **Modular Architecture:** Separating our database storage layer (MySQL) from our presentation layer (Power BI) mirrors enterprise-grade standards, ensuring a decoupled, stable, and completely organized project workflow.
* **Academic Accountability:** Writing explicit SQL scripts rather than clicking visual GUI utility buttons provides a clear, reproducible code log. This serves as transparent, undeniable proof of independent work for our evaluation.
* **Performance Optimization:** Performing resource-intensive cleaning operations over 100,000 records at the server level ensures that the raw data is optimized before ingestion. This minimizes memory usage, eliminates data loading lag inside the Power BI cache, and makes it much easier to construct clean, high-performance Star Schema relationships.

---

## 🧼 1. Data Preprocessing & Cleaning (The C.L.E.A.N. Framework)

The data preprocessing phase strictly follows the structured **C.L.E.A.N. Framework** to guarantee maximum data quality and structure before any modeling begins:

### ⚙️ Environment Setup & Ingestion
1. **Infrastructure Initialization:** Launched Apache and MySQL services within the XAMPP Control Panel.
2. **Database Workspace Configuration:** Established a local connection (`127.0.0.1:3306`) in MySQL Workbench using default `root` access. 
3. **Schema Instantiation:** Executed foundational environment scripts to isolate our project scope:
   ```sql
   CREATE DATABASE IF NOT EXISTS student_wellness_db;
   USE student_wellness_db;
   ```
4. **Data Wizard Loading:** Utilized the Table Data Import Wizard to ingest the raw `student_lifestyle_100k.csv` dataset directly into our schema workspace.

### 💻 Core C.L.E.A.N. Script Execution
```sql
-- Use our active project schema
USE student_wellness_db;

-- =========================================================================
-- [C] CONCEPTUALIZE PHASE: Establish data boundaries and ingestion counts
-- =========================================================================
SELECT COUNT(*) AS total_raw_records, COUNT(DISTINCT Student_ID) AS unique_student_keys 
FROM student_lifestyle_100k;
-- Status: Verified exactly 100,000 unique records; zero primary key duplicates.

-- =========================================================================
-- [L] LOCATE PHASE: Create production table copy and standardize categories
-- =========================================================================
DROP TABLE IF EXISTS student_wellness_cleaned;
CREATE TABLE student_wellness_cleaned AS SELECT * FROM student_lifestyle_100k;

-- Temporarily disable safe updates to allow table-wide string cleaning
SET SQL_SAFE_UPDATES = 0;

-- Clean hidden trailing spaces and standardize casing to proper format
UPDATE student_wellness_cleaned 
SET Department = TRIM(CONCAT(UPPER(SUBSTRING(Department, 1, 1)), LOWER(SUBSTRING(Department, 2)))),
    Gender = TRIM(CONCAT(UPPER(SUBSTRING(Gender, 1, 1)), LOWER(SUBSTRING(Gender, 2))));

-- =========================================================================
-- [E] EVALUATE PHASE: Check numerical ranges for out-of-bounds outliers
-- =========================================================================
SELECT 
    MIN(Age) AS youngest, MAX(Age) AS oldest,
    MIN(CGPA) AS worst_gpa, MAX(CGPA) AS best_gpa,
    MIN(Study_Hours) AS min_study_hours, MAX(Study_Hours) AS max_study_hours
FROM student_wellness_cleaned;
-- Status: Outlier validation complete. Age (18-24) and CGPA (0.0-4.0) fall inside human bounds.

-- =========================================================================
-- [A] AUGMENT PHASE: Engineer the mandatory composite calculations
-- =========================================================================
-- Add a fresh float column to store the work-life balance ratio metric
ALTER TABLE student_wellness_cleaned ADD COLUMN Work_Life_Balance FLOAT;

-- Calculate: Study Hours divided by Sleep Duration (using NULLIF to protect against division by zero errors)
UPDATE student_wellness_cleaned 
SET Work_Life_Balance = ROUND(Study_Hours / NULLIF(Sleep_Duration, 0), 2);

-- Re-enable safe updates for database security
SET SQL_SAFE_UPDATES = 1;

-- =========================================================================
-- [N] NOTE PHASE: Final quality assurance check before export
-- =========================================================================
SELECT Student_ID, Gender, Department, Study_Hours, Sleep_Duration, Work_Life_Balance 
FROM student_wellness_cleaned 
LIMIT 10;
```
*Following data validation, the entire 100,000-row table was cleanly exported via the data result panel as `student_wellness_cleaned.csv` to be passed over to the modeling phase.*

---

## 🗄️ 2. Analysis & Data Modeling (The Star Schema Transformation)

To prevent visual lag, broken filters, and memory bloating, the flat, cleaned CSV file was transformed into a professional, relational **Star Schema Data Model** inside Power BI Desktop:

### 🧩 Entity-Relationship Architecture
The single table was broken down into a central central **Fact Table** surrounded by descriptive lookup **Dimension Tables**:
* `fact_student_wellness`: Holds numerical metrics and foreign keys (`Student_ID`, `dept_id`, `behavior_id`, `CGPA`, `Stress_Level`, `Work_Life_Balance`, `Depression`).
* `student_dim`: Stores unique demographic information (`Student_ID`, `Age`, `Gender`, `Physical_Activity`).
* `department_dim`: Standardized index lookup for fields (`dept_id`, `Department`).
* `behavior_dim`: Tracks distinct academic lifestyle profiles (`behavior_id`, `Sleep_Duration`, `Study_Hours`, `Social_Media_Hours`).

### 🔗 Schema Relationship Links
The tables are securely linked in Power BI's Model View using **Many-to-One ($*:1$) relationships**, filtering unidirectionally down from the dimensions to the central fact table:
* `fact_student_wellness[Student_ID]` ➡️ `student_dim[Student_ID]`
* `fact_student_wellness[dept_id]` ➡️ `department_dim[dept_id]`
* `fact_student_wellness[behavior_id]` ➡️ `behavior_dim[behavior_id]`

---

## 📐 3. The Pyramid Analytical Framework (DAX Implementation)

The dashboard logic adopts a structured **Data Analytics Pyramid Framework** to organize information flow smoothly, starting from high-level summaries down to granular user-driven exploratory insights:

### 🔼 Tier 1: The Pyramid Base (Macro Baseline Tracking)
Tracks total foundational volume counters across the entire ingested model structure.
```dax
Total Students = COUNT(fact_student_wellness[Student_ID])
```

### 🔼 Tier 2: The Pyramid Mid-Tier (Core Performance Metrics)
Calculates standard averages to monitor ongoing baseline academic and lifestyle balances.
```dax
Average CGPA = AVERAGE(fact_student_wellness[CGPA])
```
```dax
Average Stress Level = AVERAGE(fact_student_wellness[Stress_Level])
```

### 🔼 Tier 3: The Pyramid Peak (Advanced Critical Risk Indicators)
Isolates critical target mental health and student burnout distributions using conditional filters.
```dax
Depression Rate = 
DIVIDE(
    COUNTROWS(
        FILTER(
            fact_student_wellness,
            fact_student_wellness[Depression] = "Yes"
        )
    ),
    COUNTROWS(fact_student_wellness)
)
```

---

## 📊 4. Interactive Dashboard Architecture (The D.A.S.H. Layout)

The dashboard user interface features a cohesive, wellness-themed dark green and gold color scheme mapped to an intuitive **F-Pattern visual layout** that aligns seamlessly with natural human scanning behaviors:

1. **Visual Header Anchor (Top-Left):** Houses the main title text box labeled **STUDENT WELLNESS DASHBOARD** to establish immediate context.
2. **Macro Summary Strip (Top-Horizontal Row):** Hosts our 5 foundational KPI summary cards (Total Records, Average Balance, Stress Rate, Depression %, and Average CGPA) to ensure primary metrics are processed first.
3. **Core Operational Visuals (Center Canvas):** 
   * **Line and Clustered Column Chart:** Compares student sleep duration counts alongside stress rate patterns mapped across academic departments.
   * **Scatter / Bubble Chart:** Identifies trends correlating a student's average resting sleep duration against their final academic CGPA standing.
4. **Categorical Splits (Lower Footer Row):** Houses the **Gender Depression Donut Split** and the **Stacked Department Stress Distribution Chart** to cleanly divide group metrics.
5. **Interactive Controls (Left-Sidebar Margin):** Implements dynamic, multi-select slicing controls (`Gender`, `Department`, `Stress Level`, `Depression Status`) to allow stakeholders to drill down and explore custom demographic populations instantly with zero relationship errors.
