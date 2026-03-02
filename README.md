# DEGO Course Project - Team 6

## Team & Roles

#### \- Product Lead: Ines Azeredo, 73259
Project coordination, governance framing, executive summary, Q&A lead

#### \- Data Engineer: Marnix Hazelhoff, 71906
Data loading, cleaning, transformation

#### \- Data Scientist: Carolina Diogo, 71947
Bias detection, fairness metrics, statistical analysis

#### \- Governance Officer: Nick Hammerschmid, 70159
GDPR mapping, AI Act assessment, policy recommendations

## Project Description

This project audits NovaCred’s historical credit application dataset to assess data quality, detect algorithmic bias, and propose governance controls aligned with GDPR and the EU AI Act. Our objective is to evaluate whether NovaCred’s credit decision pipeline demonstrates fairness, transparency, and regulatory compliance.

## Project Structure Initialization
The repository was organized into dedicated folders to ensure clarity, maintainability and auditability:

- `data/` – Dataset files, raw and cleaned
- `notebooks/` – Jupyter analysis notebooks 
- `reports/` – Final deliverables, exported visual evidence (PNG files)  
- `presentation/` – Slides for final delivery  
- `src/` – Python source code and utility functions

This structure ensures separation of raw data, processed outputs, and analytical artifacts.

---

## Data Engineering Pipeline

The data engineering workflow, led by **Marnix Hazelhoff (Data Engineer)**, followed a structured, reproducible pipeline:

#### 1. Data Quality Analysis Notebook
A comprehensive data quality assessment notebook was developed, mapping issues to specific quality dimensions:

- **Uniqueness:** Duplicate detection
- **Completeness:** Missing value analysis
- **Consistency:** Inconsistent categorical encoding and date formats
- **Validity:** Logical constraint checks (e.g., DTI > 1, negative credit history months)

Visual audit evidence was generated and exported to the `reports/` folder, including:

- `date_formats.png` – highlights inconsistent date formats
- `gender_raw.png` – shows inconsistent gender encoding (e.g., "Male"/"M", "Female"/"F")
- `missing_values.png` – visual overview of null distributions across features

---

### 2. Data Cleaning & Standardization

The dataset was then cleaned and standardized:

- Duplicate records removed
- Gender values harmonized (Male, Female, Unknown)
- Date formats standardized to ISO format (YYYY-MM-DD)
- Annual income imputed (flag retained for transparency)
- Extreme DTI values flagged rather than silently altered
- Logical inconsistencies identified (e.g., spending exceeding income)

The cleaned dataset was exported to `data/cleaned_credit_applications.csv` to enable downstream bias analysis.

---

### 3. Iterative Improvement

Following peer feedback, the notebook was refined to include:

- Improved methodological explanations
- An added accuracy/validation section
- Clearer documentation of assumptions and transformation logic

This ensured the engineering process remained transparent, reproducible, and audit-ready.

### 4. To Do
- Bias detection, fairness metrics, statistical analysis (in progress, by Carolina)
- GDPR mapping, AI Act assessment, policy recommendations (Nick)