# Clinical Operations & Patient Utilization Overview (Power BI)

A one-page Power BI dashboard that summarises **patient visits, diagnosis trends and patient demographics** to support clinical operations planning.

> Status: first version. Known improvements are listed in section 8.

---

## 1. Problem

Clinic managers need a quick view of **who visits, how often, and for which conditions**. This dashboard answers:

- How many patients and visits are there?
- Which diagnoses (ICD-10 codes) drive the most visits?
- What is the age mix of patients?
- Which patients are high utilizers of services?

## 2. Data

- **File:** `patient_visits.csv`
- **Size:** 200 visit records, 8 columns
- **Period:** January 2022 to December 2024
- **Source:** *(add where you got the data, for example "practice dataset from ...")*. The IDs and codes look like sample data, not real patients.

| Column | Meaning |
|---|---|
| visit_id | Visit identifier |
| patient_id | Patient identifier (42 distinct values) |
| visit_date | Date of the visit |
| date_of_birth | Patient's date of birth |
| patient_age | Patient's age |
| patient_sex | Male / Female |
| icd_code | Diagnosis code (ICD-10) |
| cpt_code | Procedure code (CPT) |

**Data checks done:** no missing values and no fully duplicated rows.

## 3. What the dashboard shows

| Visual | Purpose |
|---|---|
| Cards: Total Patients (42), Total Visits (200), Avg Visits per Patient (4.76) | Overall volume and intensity of use |
| Line chart: Visits by Year | Trend over time (2022 to 2024) |
| Bar chart: Top 5 Diagnoses (ICD-10) | Which diagnoses drive visits |
| Donut: Visits by Age Group | Visit share by age group (18-39, 40-64, 65+) |
| Table: Utilizer Status and Visits per Patient | Finds high-utilizer patients |
| Treemap: Top 10 Procedure Codes (CPT) | Most common procedures |

## 4. Key DAX measures

| Measure | Purpose |
|---|---|
| Total Patients | Distinct count of `patient_id` (42) |
| Total Diagnosis Events | Number of visit records (200) |
| Avg Visits per Patient | Total visits divided by total patients (about 4.76), using `DIVIDE` to avoid divide-by-zero |
| Visits per Patient | Total visits for each patient, using `CALCULATE` with `ALLEXCEPT` |
| Age Band | Groups patients into 0-17, 18-39, 40-64 and 65+ with `SWITCH(TRUE())` |
| Utilizer Status | Labels a patient High (5+ visits), Medium (2-4) or Low (1) |

```dax
Utilizer Status =
VAR v = [Visits per Patient]
RETURN
SWITCH(TRUE(),
    v >= 5, "High Utilizer",
    v >= 2, "Medium Utilizer",
    "Low Utilizer")
```

```dax
Age Band =
SWITCH(TRUE(),
    patient_visits[patient_age] <= 17, "0-17",
    patient_visits[patient_age] <= 39, "18-39",
    patient_visits[patient_age] <= 64, "40-64",
    "65+")
```

```dax
Visit Year = YEAR(patient_visits[visit_date])
```

```dax
Avg Visits per Patient =
DIVIDE([Total Visits], [Total Patients], 0)
```

## 5. Key insights

All numbers come from `patient_visits.csv` (200 visits).

1. **Hypertension (I10) is the top diagnosis,** with 19 of 200 visits (9.5%). Next are J45.909 and N39.0 (11 visits each), then E11.9, M54.5 and E78.5 (10 visits each).
2. **Visits peaked in 2023.** Visit counts were 71 (2022), 77 (2023) and 52 (2024). 2024 is about 32% lower than 2023.
3. **Adults aged 40-64 make up the largest share of visits (52%),** followed by ages 18-39 (33%) and 65+ (15%). *(Confirm this matches your Age Band logic.)*
4. **Women account for 117 of the 200 visits (58.5%).**
5. **Visits are concentrated in a few patients:** 20 patient IDs are High Utilizers (8 or 9 visits each) and 22 are Low Utilizers (1 visit). No patient falls in the Medium group (2 to 4 visits).

## 6. Recommendations

- Focus chronic-care programmes (blood pressure, diabetes, cholesterol) on the most frequent diagnoses.
- Review the high-utilizer patients for care-management follow-up.
- Investigate why visits fell in 2024 (data completeness or real change).

## 7. Data quality notes and how I handled them

- **Date format:** the raw file stores dates as month/day/year (for example 3/15/2022). With a day/month locale, 117 of 200 visit dates became blank and 83 were swapped. I fixed this by loading the dates with the correct (US) format, so no dates were lost.
- **Age donut:** my first version counted distinct patients and used an age calculated from today's date. I rebuilt it to count visits and use the age recorded in the data.
- **`visit_id` is not unique:** 200 rows share only 42 distinct values, so visits are counted by rows, not by distinct visit ID.
- **Patient identity:** 20 of 42 patient IDs appear with more than one sex, date of birth or age, so patient-level demographics are unreliable. I removed sex from the patient table to avoid showing the same patient twice.
- **Utilizer groups:** 20 patients are High (8 to 9 visits) and 22 are Low (1 visit). No patient falls in the Medium group.
- CPT and ICD codes are shown as codes, without descriptions.

## 8. Next steps

- Add ICD and CPT descriptions with a lookup table so charts are readable.
- Add slicers (year, sex, age band).
- Clean or re-key the patient table so each patient has one date of birth and sex.

## 9. How I used AI

I used **Claude (Anthropic)** as an assistant while learning and building this project.

| Task | AI tool | What AI did | What I did |
|---|---|---|---|
| Planning the KPIs | Claude | Suggested healthcare metrics | Chose which to include |
| DAX and age grouping | Claude | Explained and drafted formulas | Built and tested them in Power BI |
| Dashboard layout | Claude | Suggested chart types | Designed the final page |
| This README | Claude | Helped organise it | Verified the numbers |




**Prerna Keshari** - aspiring Data Analyst | Power BI, SQL, Excel, Python basics
LinkedIn: [www.linkedin.com/in/prerna-keshari-515706357](https://www.linkedin.com/in/prerna-keshari-515706357)
