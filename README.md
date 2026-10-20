# Healthcare Operations & Patient Risk Analytics

Data Analyst Capstone Project: Emergency Department (ED) waiting time and ICU sepsis analysis.

> **Both datasets are synthetic educational datasets.** They are not real patient data, and the findings illustrate analytical method. They are not clinical or operational evidence.

## 1. Business problem

Hospital management asked two questions:

1. **ED:** why do patients wait as long as they do for a provider, and what could management change?
2. **ICU:** which patients develop sepsis, and when does it appear?

The work turns raw data into findings and recommendations for a non-technical hospital audience. It follows seven business questions:

| # | Question |
|---|---|
| 1 | Which departments and triage groups wait longest? |
| 2 | When does waiting pressure build (arrival hour, day type, season)? |
| 3 | How are queue size and staffing related to waiting time? |
| 4 | Which ICU patient groups develop sepsis more often? |
| 5 | When does sepsis appear during an ICU stay? |
| 6 | Can ED waiting time be predicted from information known at triage? |
| 7 | Can sepsis be predicted, and what errors do the models make? |

## 2. Data

| File | Size | Description |
|---|---|---|
| `ED_Waiting_Time_Data.csv` | 6,000 x 19 | ED visits, 1 Jan to 27 Sep 2025 |
| `ICU_Sepsis_Patient_Summary.csv` | 180 x 6 | Age, gender, ICU length of stay, sepsis outcome, onset hour |
| `ICU_Sepsis_Patient_Data.csv` | 180 x 18 | Vital signs, laboratory values, treatments |
| `ICU_Sepsis_Patient_Data_Dictionary.csv` and ED data dictionary | | Column definitions |

Raw files in `data/raw/` are never edited. Cleaned versions are in `data/processed/`.

## 3. Repository structure

```
healthcare-analytics-capstone/
├── data/
│   ├── raw/          original CSV files (unchanged)
│   └── processed/    ED_Waiting_Time_Cleaned.csv, ICU_Sepsis_Merged_Cleaned.csv
├── notebooks/
│   ├── 01_ED_Analysis.ipynb
│   └── 02_ICU_Sepsis_Analysis.ipynb
├── dashboard/        Healthcare_Analytics_Dashboard.pbix (Power BI)
├── reports/          Healthcare_Analytics_Executive_Report.pdf
├── images/           dashboard screenshots and key model outputs
├── README.md
├── requirements.txt
└── .gitignore
```

## 4. Methods

**Data quality (both notebooks)**
- Audit of structure, types, missing values, duplicates, categories, plausible minimums and internal consistency.
- Every cleaning decision is recorded in a decision log inside each notebook. No rows were silently deleted, targets were never imputed, and patient IDs were never used as predictors.
- Each feature was checked for availability at prediction time to avoid leakage.
- Zeros were investigated and not treated as missing: midnight arrival hours, empty queues, infants aged 0 and zero waits were all kept and documented. A sensitivity check showed that excluding the zero waits changes model errors by under 0.2 minutes.

**ED (regression):** Linear Regression, Decision Tree and Random Forest, evaluated with MAE, RMSE and R2.

**ICU (classification):** Logistic Regression, Decision Tree and Random Forest on a stratified split, with imputation and scaling fitted on the training set only; evaluated with accuracy, precision, recall, F1, ROC-AUC and confusion matrices. Models use simple baseline settings with no parameter search.

## 5. Key findings

**ED**
- Waits rise from about 5 minutes (triage priority 1) to 67 minutes (priority 5).
- Weekend visits wait about 15 minutes longer than weekday visits (53.0 vs 37.9); the longest waits occur overnight (11 pm to 5 am).
- Average wait falls from about 64 minutes with 3 staff on duty to about 25 with 16 (association, not proven cause).
- Outpatient - General has the longest waits at every urgency level.

| ED model | MAE (min) | RMSE (min) | R2 |
|---|---|---|---|
| Linear Regression | 7.41 | 9.29 | 0.836 |
| Decision Tree | 8.99 | 11.30 | 0.758 |
| Random Forest | 7.29 | 9.16 | 0.841 |

**ICU**
- 50 of 180 patients (27.8%) developed sepsis; 68% of cases appeared within 24 hours of admission.
- Age and gender differences in sepsis rates were within chance.

| ICU model | Accuracy | Precision | Recall | F1 | ROC-AUC |
|---|---|---|---|---|---|
| Logistic Regression | 0.694 | 0.333 | 0.100 | 0.154 | 0.658 |
| Decision Tree | 0.667 | 0.250 | 0.100 | 0.143 | 0.579 |
| Random Forest | 0.750 | 0.667 | 0.200 | 0.308 | 0.713 |

A model that always predicts "no sepsis" would score 72.2% accuracy, so accuracy alone is misleading. All models caught only 1 or 2 of the 10 sepsis patients in the test set and are not suitable for clinical use.

## 6. Recommendations

1. Review weekend and overnight staffing against demand.
2. Review patient flow for urgent patients in Outpatient - General and Surgery Referral.
3. Focus sepsis monitoring on the first 24 to 48 hours of an ICU stay.
4. Do not deploy the baseline sepsis models; gather larger, time-stamped data first.

See `reports/Healthcare_Analytics_Executive_Report.pdf` for the full report.

## 7. Dashboard

The Power BI dashboard (`dashboard/Healthcare_Analytics_Dashboard.pbix`) has four pages: Executive Overview, ED Waiting Time, ICU Sepsis and Model Performance. Screenshots are in `images/`.

## 8. How to reproduce

1. Clone the repository and install the packages: `pip install -r requirements.txt`.
2. Open each notebook in Google Colab or Jupyter.
3. Upload the raw CSV files from `data/raw/` to the notebook's working folder (the notebooks read from `/content/`; change the paths if running locally).
4. Run all cells from top to bottom.

## 9. Limitations

- Synthetic data; one hospital setting; ED data covers January to September 2025 only.
- Small ICU sample (180 patients, 50 sepsis cases, only 10 in the test set), so model results are unstable.
- No timestamps on ICU measurements, so the assumption that they precede sepsis onset cannot be verified.
- 314 zero waits may be immediate care or a recording convention; the data dictionary does not settle which.
- Associations only; no causal claims.
