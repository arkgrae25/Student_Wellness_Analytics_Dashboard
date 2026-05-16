# Student_Wellness_Analytics_Dashboard
# Data Preprocessing & Cleaning Log (C.L.E.A.N. Framework)

**Class Section:** C303  
**Data Analysts:** Geraldine C. Mendoza, Jhulia Reign Gilhang, Janier Levine U. Santos, Sittie Aisah S. Serad  
**Target Dataset:** Student Depression & Lifestyle Dataset (100,000 Records)  
**Data Domain:** Behavioral Metrics & Academic Wellness Patterns  

---

## 💻 Technical Execution Log

### 1. Conceptualize (C)
* **Grain & Scope:** Each individual record represents a single unique survey submission tracking student behavior across a 100,000-record dataset.
* **SQL Verification Script:**
```sql
SELECT 
    COUNT(*) AS total_raw_records, 
    COUNT(DISTINCT Student_ID) AS unique_student_keys 
FROM student_data_raw;
```
* **Status:** Confirmed exactly 100,000 unique records. No duplicate `Student_ID` values exist.

### 2. Locate (L)
* **Inconsistency Check:** Audited the string-based columns (`Department` and `Gender`) for extra spaces or inconsistent values.
* **SQL Standardization Scripts:**
```sql
CREATE TABLE student_data_clean AS SELECT * FROM student_data_raw;

UPDATE student_data_clean 
SET Department = TRIM(CONCAT(UPPER(SUBSTRING(Department, 1, 1)), LOWER(SUBSTRING(Department, 2)))),
    Gender = TRIM(CONCAT(UPPER(SUBSTRING(Gender, 1, 1)), LOWER(SUBSTRING(Gender, 2))));
```

### 3. Evaluate (E)
* **Outlier Strategy:** Evaluated continuous numeric attributes (`Study_Hours`, `CGPA`) to find impossible values or human entry errors.
* **SQL Range Test Script:**
```sql
SELECT 
    MIN(Age) AS youngest, MAX(Age) AS oldest,
    MIN(CGPA) AS worst_gpa, MAX(CGPA) AS best_gpa,
    MIN(Study_Hours) AS min_study_hours, MAX(Study_Hours) AS max_study_hours
FROM student_data_clean;
```
* **Status:** All fields match expected boundaries. No records were deleted.

### 4. Augment (A)
* **Feature Engineering:** Created a custom column `Work_Life_Balance` by dividing study hours by sleep duration.
* **SQL Enrichment Script:**
```sql
ALTER TABLE student_data_clean ADD COLUMN Work_Life_Balance FLOAT;

UPDATE student_data_clean 
SET Work_Life_Balance = ROUND(Study_Hours / NULLIF(Sleep_Duration, 0), 2);
```

### 5. Note (N)
* **Audit Trail Summary:** The cleaned dataset was saved and verified. The file has been exported as a `.csv` file to hand off to the team's data modeler.
