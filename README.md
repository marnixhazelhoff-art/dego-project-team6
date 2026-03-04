# DEGO Course Project - Team 6

## Team Members

| Role | Name | Student ID |
| :--- | :--- | :--- |
| Data Engineer | Marnix Hazelhoff | 71906 |
| Data Scientist | Carolina Diogo | 71947 |
| Governance Officer | Nick Hammerschmid | 70159 |
| Product Lead | Ines Azeredo | 73259 |

## Project Description
This project audits NovaCred’s historical credit application dataset to assess data quality, detect algorithmic bias, and propose governance controls aligned with GDPR and the EU AI Act. Our objective is to evaluate whether NovaCred’s credit decision pipeline demonstrates fairness, transparency, and regulatory compliance.

## Repository Structure

```text
dego-project-team6/
├── README.md
├── data/
│   ├── raw_credit_applications.json
│   └── cleaned_credit_applications.csv
├── notebooks/
│   ├── 01-data-quality.ipynb
│   ├── 02-bias-analysis.ipynb
│   └── 03-privacy-demo.ipynb
├── src/
│   └── fairness_utils.py
└── presentation/
    └── final deliverables
```

---

## Data Engineering Pipeline
**Lead:** Marnix Hazelhoff

The data engineering workflow followed a structured, reproducible pipeline to transform raw nested JSON data into an audit-ready dataset.

### 1. Data Quality Analysis (DQA)
A comprehensive assessment was conducted in `01-data-quality.ipynb`, mapping identified issues to four standard data quality dimensions:

| Dimension | Issue Identified | Records Affected | Visual Evidence |
| :--- | :--- | :--- | :--- |
| **Uniqueness** | Duplicate `_id` and identical application records | 4 (0.8%) | `01-data-quality.ipynb` |
| **Completeness** | Missing `ssn`, `email`, and `consent_timestamps` | ~87% (timestamps) | `reports/missing_values.png` |
| **Consistency** | Fragmented gender labels (e.g., "Male"/"M") | 114 (22.7%) | `reports/gender_raw.png` |
| **Validity** | Logical errors (negative credit history months) | 2 (0.4%) | `01-data-quality.ipynb` |
| **Accuracy** | Inconsistent date formats (ISO vs. non-ISO) | 162 (32.3%) | `reports/date_formats.png` |



### 2. Data Cleaning & Standardization
To comply with **EU AI Act** requirements for high-quality training data, the following remediations were applied:

* **Deduplication:** Removed duplicate records while preserving the most complete versions.
* **Harmonization:** Standardized gender values to "Male", "Female", or "Unknown" to ensure categorical consistency.
* **Standardization:** Converted all diverse date formats to a uniform ISO 8601 (YYYY-MM-DD) format.
* **Imputation:** Missing `annual_income` values were imputed with the median, but a flag (`annual_income_imputed`) was retained to ensure transparency and auditability.
* **Constraint Enforcement:** Fixed negative values using absolute values and flagged extreme Debt-to-Income (DTI > 1) ratios to prevent silent data corruption.

The final cleaned dataset was exported to `data/cleaned_credit_applications.csv` to serve as the "Single Source of Truth" for downstream bias analysis.

### 3. Iterative Improvement
Following team feedback, the notebook was refined to include:
* **Methodological Transparency:** Improved explanations of the transformation logic.
* **Validation Section:** Added a post-cleaning check to ensure data integrity.
* **Audit Readiness:** Documented assumptions to support GDPR Accountability requirements.

---

## Bias Detection & Fairness
**Lead:** Carolina Diogo

We conducted a forensic fairness audit of NovaCred’s historical decisions to identify patterns of algorithmic discrimination. The investigation followed an iterative discovery process, moving from baseline metrics to intersectional and geographic root-cause analysis.

### 1. Gender Bias: Baseline & Proxy Analysis
Our initial audit revealed a significant disparity in how the system treats male vs. female applicants.

* **Gender Disparate Impact (DI) Ratio:** **0.767**
* **Demographic Parity Difference:** **-14.7 percentage points** (Female - Male)
* **Finding:** The DI ratio of 0.767 is below the **0.80 legal benchmark** (four-fifths rule), indicating systemic bias against female applicants.
* **Proxy Discovery:** We identified that `zip_code` serves as a high-confidence proxy for gender. In specific clusters (e.g., "The Heights"), gender density is so high that the model can "infer" gender and reproduce bias even if the explicit gender field is removed.



### 2. The "Youth Penalty": Age-Based Discrimination
A critical discovery was made regarding younger applicants. The system demonstrates a "Youth Penalty" that persists even after controlling for financial factors.

* **Unprivileged Group:** Applicants aged 18–25.
* **Age DI Ratio:** **0.684** (vs. 26–40 group).
* **Finding:** This ratio represents a severe violation of the 0.80 fairness threshold. Younger applicants are significantly less likely to be approved, even with comparable income levels.
* **Geographic Redlining:** Forensic analysis showed that the model uses geography to amplify age bias. Certain neighborhoods with high concentrations of young professionals (e.g., "Southside Tech District") are negatively weighted, effectively "redlining" youth.

### 3. Intersectional & Interaction Patterns
We explored the interaction between Age, Gender, and Geography:
* **Intersectional Gap:** The approval gap is widest for **young female applicants** in specific high-density ZIP codes.
* **Feedback Loop Risk:** Because rejected applicants generate less future data, the model's current bias creates a "Selection Bias" loop that reinforces historical under representation.

### 4. Fairness Metrics Summary
| Metric | Audit Result | Status |
| :--- | :--- | :--- |
| **Gender DI Ratio** | 0.767 | **FAIL** (< 0.80) |
| **Age DI Ratio (18-25)** | 0.684 | **FAIL** (< 0.80) |
| **Demographic Parity** | -14.7 pp | **FAIL** (Significant Gap) |

### 5. Recommended Remediation & Compliance
To bring NovaCred into alignment with the **EU AI Act** and **GDPR**, we recommend:
1.  **Eliminate Geographic Proxies:** Immediately remove `zip_code` from the model features to prevent demographic redlining.
2.  **Demographic Parity Calibration:** Apply fairness constraints to the 18–25 age bracket to meet the 0.80 benchmark.
3.  **Decision Threshold Adjustment:** Implement post-processing adjustments for female applicants to satisfy legal requirements without compromising predictive power.

---

## Data Privacy & Governance
**Lead:** Nick Hammerschmid

We evaluated NovaCred’s credit scoring system from a regulatory and ethical perspective, focusing on PII protection, GDPR compliance, and alignment with the EU AI Act.

### 1. PII Identification & Classification
We identified all Personal Identifiable Information (PII) processed by the system, categorized under GDPR Art. 4(1):

* **Direct Identifiers:** Full Name, Email, and SSN (Critical Risk).
* **Quasi-Identifiers:** ZIP code, IP address, and Date of Birth (Medium Risk).
* **Protected Attributes:** Gender and Age.
* **Behavioral Data:** Credit utilization and transaction history.

### 2. Privacy-Enhancing Techniques (PEPs)
To mitigate re-identification risks, we implemented two primary technical controls:

* **Gender Pseudonymization (Art. 4(5)):** Raw gender values were replaced with neutral tokens (G1, G2, G3). A secure mapping lookup table was created to be stored separately, ensuring attribution is impossible without authorized access.
* **Age Anonymization (Generalization):** Exact dates of birth were removed and replaced with generalized **Age Groups** (18-25, 26-40, 41-60, 60+).
* **K-Anonymity Inspired Suppression:** Applied a threshold (k=10) to suppress small groups that could lead to unique individual identification.

### 3. Regulatory Mapping: GDPR & EU AI Act
| Regulation | Requirement | NovaCred Status | Recommended Control |
| :--- | :--- | :--- | :--- |
| **GDPR Art. 5(1)(c)** | Data Minimization | High (Age generalized) | Removal of raw DOB from production. |
| **GDPR Art. 22** | Automated Decisions | Triggered (Legal effects) | Implement Human-in-the-loop oversight. |
| **GDPR Art. 32** | Security | Critical (Plaintext PII) | Encryption of SSN and email at rest. |
| **AI Act Annex III** | High-Risk AI System | Classified (Credit Scoring) | Compliance with Art. 9-15 obligations. |

### 4. Proposed Governance Controls
To transition NovaCred to a compliant state, we propose the following organizational measures:

* **Human Oversight Mechanism (Art. 14):** Mandatory manual review for borderline credit scores and cases flagged by bias monitoring.
* **Decision Logging:** A structured, immutable audit trail logging model versions, input features, and human override justifications.
* **Consent Management:** A centralized registry to track applicant consent status and timestamps for transparency.
* **Data Retention Policy:** Explicit storage limits (e.g., 24 months for rejected apps) to satisfy the Storage Limitation principle.
