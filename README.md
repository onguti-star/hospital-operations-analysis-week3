# Hospital Operations, Cost & Patient Flow Intelligence
**AnalystLab Africa Experience Lab Internship — Week 3 (Track B: Data Science)**

## 1. Project Overview & Business Context
Healthcare administrators and financial operations teams require clear visibility into patient visit volumes, encounter durations, billed claim costs, and insurance reimbursement patterns to allocate clinical resources and manage uncompensated care exposure[cite: 6]. 

This project investigates eleven years of historical hospital records from **Massachusetts General Hospital** (`2011-01-02` to `2022-02-05`), spanning **27,891 patient encounters**, **47,701 clinical procedures**, **974 unique patients**, and **10 insurance payers**. Through exploratory data analysis (EDA) and non-parametric statistical hypothesis testing in Python, this study establishes a reproducible baseline for departmental budgeting, length-of-stay benchmarking, and payer coverage evaluation.

### Core Research Questions
1. **Primary Question:** How do recorded encounter costs (`TOTAL_CLAIM_COST` and `BASE_ENCOUNTER_COST`) and insurance coverage (`PAYER_COVERAGE`) vary across hospital encounter classes (`ambulatory`, `outpatient`, `urgentcare`, `emergency`, `wellness`, and `inpatient`)?
2. **Supporting Question 1:** Are longer encounter durations (length of stay calculated from `START` and `STOP` timestamps) significantly associated with higher recorded claim costs?
3. **Supporting Question 2:** How do recorded claim costs and payer coverage ratios differ across insurance payers and frequently performed clinical procedures?

---

## 2. Dataset Architecture & Privacy Governance
The analysis integrates five relational tables from the AnalystLab Africa Healthcare Operations dataset pack:

| Table | Records | Columns | Role in Analysis |
| :--- | :--- | :--- | :--- |
| `encounters.csv` | 27,891 | 14 | Central fact table tracking visit timestamps, encounter classes, claim costs, and payer coverage (`2022-02` is a partial month ending Feb 5). |
| `procedures.csv` | 47,701 | 9 | Clinical procedures linked to encounters via `ENCOUNTER`, including SNOMED codes and `BASE_COST`. |
| `patients.csv` | 974 | 17 | Non-identifying patient demographics (`BIRTHDATE`, `GENDER`, `RACE`, `ETHNICITY`, `CITY`, `STATE`). |
| `payers.csv` | 10 | 7 | Insurance providers (`Medicare`, `Medicaid`, `Dual Eligible`, `Humana`, `Aetna`, `Blue Cross Blue Shield`, `UnitedHealthcare`, `Cigna Health`, `Anthem`, and `NO_INSURANCE`). |
| `organizations.csv` | 1 | 8 | Facility metadata identifying Massachusetts General Hospital (Boston, MA). |

### Data Hygiene & Privacy Compliance
* **Strict PII Removal:** Directly identifying patient attributes (`FIRST`, `LAST`, `ADDRESS`) were stripped immediately upon loading `patients.csv` and excluded from all outputs, charts, reports, and version control[cite: 6].
* **Domain-Aware Missing Value Handling:** Null values in `REASONCODE` / `REASONDESCRIPTION` (routine visits without a specific diagnosis), `DEATHDATE` (820 living patients), and contact fields for `NO_INSURANCE` represent valid clinical/administrative states and were preserved rather than blindly imputed.
* **Timestamp & Integrity Audit:** All five tables passed with **0 duplicate rows** and **0 negative/invalid durations** (`STOP < START`).

---

## 3. Statistical Methodology
Healthcare cost and length-of-stay distributions are heavily right-skewed—a small fraction of complex inpatient and acute care visits incur tens of thousands of dollars in claims, pulling means 3× to 44× higher than medians. Because parametric tests (such as ANOVA or Pearson correlation) assume normality, **non-parametric rank-based methods** were applied at a significance level of $\alpha = 0.05$:

* **Hypothesis 1 (Claim Costs Across Encounter Classes):** Tested via the **Kruskal-Wallis H-test** across all six encounter classes.
* **Hypothesis 2 (Encounter Duration vs. Claim Cost):** Tested via **Spearman Rank Correlation ($\rho$)** to evaluate monotonic association without outlier distortion.
* **Hypothesis 3 (Coverage Across Insurance Payers):** Tested via the **Kruskal-Wallis H-test** across all 10 payer categories.

---

## 4. Key Analytical & Statistical Findings

### 4.1 Encounter Volume & Cost Distribution by Care Setting
| Encounter Class | Visits | Share of Volume | Median Duration | Median Claim Cost | Mean Claim Cost | IQR Claim Cost |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Ambulatory** | 12,537 | 44.95% | 0.25 hrs | $573.98 | $2,894.11 | $1,124.50 |
| **Outpatient** | 6,300 | 22.59% | 0.25 hrs | $278.58 | $2,237.30 | $440.55 |
| **Urgent Care** | 3,666 | 13.14% | 0.25 hrs | $142.58 | $6,369.16 | $8,778.80 |
| **Emergency** | 2,322 | 8.33% | 1.00 hr | $146.18 | $4,629.65 | $4,142.20 |
| **Wellness** | 1,931 | 6.92% | 0.25 hrs | $1,231.44 | $4,260.71 | $790.32 |
| **Inpatient** | 1,135 | 4.07% | 24.00 hrs | $2,355.59 | $7,761.35 | $13,778.30 |

* **Care Setting Cost Divergence:** While `ambulatory` visits drive nearly half of all hospital traffic (44.95%), `inpatient` admissions generate the highest median claim cost ($2,355.59—over 8× higher than routine outpatient visits) and longest stays (median 24.00 hours; mean 36.84 hours).
* **Uninsured Care Exposure:** `Medicare` covers the largest visit volume (`11,371` encounters; mean coverage `$1,689.89`), whereas `NO_INSURANCE` is the second-largest category (`8,807` visits — **31.58%** of all encounters) with an average claim of **$5,593.20** and **$0.00** recorded coverage.
* **High-Frequency Procedure Demand:** Standardized intake and clinical assessments—including *Morse Fall Scale assessments* (`2,422` procedures; `$431.00`), *anxiety assessments* (`2,288`; `$431.00`), and *medication reconciliations* (`2,284`; `$496.00`)—drive consistent operational throughput.

### 4.2 Summary of Hypothesis Tests
| Hypothesis | Statistical Test | Test Statistic | p-value | Decision ($\alpha = 0.05$) | Practical Interpretation |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **H1:** Claim cost differs across encounter classes | Kruskal-Wallis | `H = 2025.19` | `< 0.0001` | **Reject $H_0$** | Care setting strongly impacts claim cost; inpatient stays cost >8× outpatient visits. |
| **H2:** Duration correlates with total claim cost | Spearman Rank | `rho = 0.3379` | `< 0.0001` | **Reject $H_0$** | Moderate positive link; longer stays cost more, though short high-tech procedures also drive high claims. |
| **H3:** Payer coverage differs across payers | Kruskal-Wallis | `H = 22239.03` | `< 0.0001` | **Reject $H_0$** | Sharp reimbursement gap between public plans (Medicaid/Medicare) and uninsured/commercial records. |

---

## 5. Visualisations
All charts are saved at 300 DPI inside the `Visualisations/` directory:

* **Encounter Volume by Care Setting:** `Visualisations/01_encounter_volume_by_class.png`
* **Claim Cost Distribution Across Encounter Classes (Log Scale):** `Visualisations/02_metric_by_encounter_class.png`
* **Encounter Duration vs. Total Claim Cost:** `Visualisations/03_duration_vs_metric.png`
* **Encounter Volume by Insurance Payer:** `Visualisations/04_encounters_by_payer.png`
* **Top 10 Most Frequent Clinical Procedures:** `Visualisations/05_top_procedures_frequency.png`

---

## 6. Operational Recommendations & Limitations

### Recommendations
1. **Establish Inpatient & Urgent Care Cost Benchmarks:** Set length-of-stay review triggers for inpatient admissions exceeding 36 hours and audit high-cost urgent care claims to separate routine walk-ins from complex acute interventions.
2. **Expand Front-Desk Coverage Screening:** Implement proactive insurance eligibility checks and financial counseling during ambulatory/outpatient registration to connect eligible uninsured patients (`31.58%` of visits) with Medicaid or subsidized plans.
3. **Standardize High-Volume Assessment Workflows:** Bundle recurring intake screenings (fall-risk, anxiety, medication reconciliation) into streamlined pre-visit digital workflows to reduce administrative claim fragmentation.

### Data Limitations
* **Billed Charges vs. Internal Costs:** `TOTAL_CLAIM_COST` reflects system-recorded claim figures rather than actual hospital operating expenses or realized cash collections.
* **Patient-Level Clustering:** The 27,891 encounters belong to 974 unique patients, meaning repeated visits per patient are treated as independent operational events.
* **Single-Facility Scope & Non-Causation:** Findings reflect Massachusetts General Hospital billing records and represent operational associations rather than clinical causation[cite: 6].

---

## 7. Repository Structure & How to Reproduce

```text
hospital-operations-analysis-week3/
├── Visualisations/
│   ├── 01_encounter_volume_by_class.png
│   ├── 02_metric_by_encounter_class.png
│   ├── 03_duration_vs_metric.png
│   ├── 04_encounters_by_payer.png
│   └── 05_top_procedures_frequency.png
├── hospital_operations_statistical_analysis.ipynb
├── hospital_operations_statistical_report.pdf
├── .gitignore
└── README.md