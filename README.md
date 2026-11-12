# Workforce Analytics (Power BI)

An end-to-end workforce analytics project that turns a messy HR dataset into a five-page Power BI report covering headcount, attrition, performance and training, and compensation and diversity. The work follows the CRISP-DM framework.

The company is a pan-African technology services company with offices in Nairobi, Mombasa, Kisumu, Kampala, Dar es Salaam and Kigali (four countries), six departments and 24 job roles.

---

## Business Problem

The company holds employee data from 2017 to 2025 but has no clear tracking of headcount or workforce trends, and the growing workforce makes them hard to monitor. The business wants to understand its employee dynamics, with the aim of improving retention and performance as part of workforce planning. This project is the first step and it answers five objectives:

1. **Headcount:** know current employee headcount by office location, department, job role, education level, gender and age.
2. **Performance and training:** analyse performance ratings and training participation by department, with overtime included.
3. **Attrition:** measure the average tenure of leavers and evaluate attrition patterns over time across gender, office location, department and education level.
4. **Compensation and diversity:** analyse salary distribution and diversity balance across the workforce.
5. **Summary view:** provide a detailed and summarised view of the company workforce.

## Key Findings

| Area | Finding |
|---|---|
| Workforce | 208 current employees. Kenya (three of the six offices) accounts for 50% of them, and the other three offices have 16% to 17% each. Customer Support is the largest department (44). The 25 to 34 age band is the largest (55), and the average age is about 39. Gender is balanced (49.5% female, 50.5% male), but men are the majority in Human Resources and among Diploma and Master's holders. |
| Attrition | 35 of 243 employees on record have left (14.4%). Exits peaked in 2024 (13 of the 35). Finance (21.6%) and Customer Support (18.5%) have the highest rates, and Operations (2.7%) the lowest. By office, Kisumu (22.9%) and Mombasa (17.5%) are highest. Kenya has the highest country rate (16.1%), and Uganda the lowest (10.5%). |
| Tenure | Leavers stayed 2.9 years on average, and 17% of leavers left within their first year. |
| Performance | The average rating is 2.89 out of 5. Sales (3.11) and Human Resources (3.08) are highest, and Customer Support (2.47) is lowest. |
| Training and overtime | Training participation is 97.3%, so almost everyone trains and participation cannot explain differences in rating. Average training hours are similar across ratings (36 to 43). Ratings are almost identical with and without overtime (2.89 and 2.90), although 54.7% of employees work overtime. |
| Compensation | For current employees with a recorded salary, the average monthly salary is Ksh148.6K and monthly payroll is Ksh29.7M. 8 current employees have no recorded salary. The company-wide gender pay gap is small (2.7% of the male average) but varies by group. Women earn 15.2% less than men among Diploma holders, and the gap is widest by country in Uganda (13.0%). In the other direction, women earn 14.1% more than men among Professional Certification holders and average more in Sales. In Engineering, men average Ksh239K against Ksh212K for women. |

These are descriptive findings. The data shows where patterns exist but not why they occur.

## Dashboard Pages

| Page | What it shows |
|---|---|
| Executive Summary | Headline KPIs and one chart per objective, by department |
| Workforce Overview | Active headcount by education, age band, department, location and job role, split by gender |
| Departmental Performance | Ratings, training hours and overtime by department and rating level |
| Attrition Analysis | Attrition rate, exits by country and year, tenure of leavers, and attrition by office, department and education |
| Compensation and Diversity | Average salary by gender across department, education level and country, plus headcount by gender |

## Dashboard Preview

Executive Summary

<img width="975" height="548" alt="image" src="https://github.com/user-attachments/assets/ba7f5326-1ee1-4dc9-bc14-95c40d1780c6" />


Workforce Overview

<img width="975" height="550" alt="image" src="https://github.com/user-attachments/assets/f5549079-7862-4c85-a010-478b53dba431" />



Departmental Performance

<img width="975" height="546" alt="image" src="https://github.com/user-attachments/assets/a3022717-daaa-4ec9-9594-e7908e4d34cc" />



Attrition Analysis

<img width="975" height="549" alt="image" src="https://github.com/user-attachments/assets/2cdb4fe6-e1b3-4d02-b6ee-66e4665b7bdb" />



Compensation and Diversity

<img width="975" height="549" alt="image" src="https://github.com/user-attachments/assets/96239a9c-750a-41c3-a984-a324ff8bfdad" /> 


## Approach (CRISP-DM)

1. **Business understanding:** defined the business problem and the five objectives.
2. **Data understanding:** profiled the Excel file in Excel and in Power Query to find the quality problems.
3. **Data preparation:** cleaned and standardised the data in Power Query.
4. **Modelling:** converted the flat table into a star schema and built 25 DAX measures.
5. **Evaluation:** interpreted each dashboard against its objective, and checked totals across visuals.
6. **Deployment:** five-page report with page navigation, published to Power BI Service.

## Data Understanding

The source is an Excel file with 1,000 rows and 16 fields. Only 247 rows held employee data. The other 753 rows contained only a hire date, and two of those were otherwise blank. Other observations:

- The Remarks column had only 16 entries.
- Exit Date is blank for active employees, by design.
- 8 records had no salary, and there were invalid ages (200 and -5).
- 4 records were duplicates with different hire dates.
- 2 records were marked "Left" with no Exit Date.
- 9 records had an Exit Date earlier than the Hire Date.

## Data Preparation Highlights

| Issue | Treatment |
|---|---|
| Rows with only a hire date | Filtered out using Employee ID, leaving 247 rows |
| 4 duplicate records | Removed, leaving 243 rows |
| Inconsistent Employee IDs, categories (`DIPLOMA` and `Diploma`, `M` and `Male`) and spacing | Trimmed, standardised case and merged variants |
| Exit Date held text, datetimes and Excel serial numbers | Parsed into proper dates. Two serial numbers were converted back to dates. Non-dates became null |
| 9 records with Exit Date earlier than Hire Date, all marked "Left" | Assumed the dates were entered the wrong way round and swapped them. A temporary flag column identified the records |
| Records marked "Left" with no Exit Date | Applied the guideline that a blank Exit Date means an active employee, using a custom column |
| Invalid ages (-5, 200) and 6 missing ages | Set to null and grouped as "Unknown" in the age band |
| 5 missing performance ratings | Left blank and excluded from rating measures |
| 15 training-hour values of `N/A` and 3 negative values | `N/A` set to null instead of 0, because 0 would imply no training. The 3 negative values were corrected |
| 8 missing salaries and one salary of -105,900 | Missing salaries (all 8 belong to current employees) were left as null and excluded from the salary measures, in line with the treatment of ratings, ages and training hours. The negative salary was assumed to be a sign error and corrected to 105,900 |
| Remarks column | Removed, as it was mostly empty and not needed |
| Added columns | Tenure at Exit (Years), Tenure Band at Exit and Age Band |

Final dataset: **243 employees** (208 active, 35 left). Full details are in [`Docs/power-query-steps.md`](Docs/power-query-steps.md).

## Data Model

A star schema with `Workforce Data` as the fact table and five dimension tables: Date (marked as a date table), Department, Education, Job Role and Location (with Country as an attribute).

- The Date table has an active relationship to Hire Date and an inactive relationship to Exit Date. The exit-based measures (`Exits` and `Avg Tenure of Leavers (Years)`) activate it with `USERELATIONSHIP`.
- The 25 DAX measures cover headcount, attrition, performance and training, and compensation. The full list, with purposes, is in [`Docs/dax-measures.md`](Docs/dax-measures.md).

## Assumptions and Limitations

- **Small sample.** 243 records and 35 leavers, so splits by department, office or gender can rest on a handful of people. For example, Operations' 100% early attrition is a single leaver, and Nairobi's 50% is one of two.
- **Snapshot data.** One rating and one training figure per employee, so the data cannot show cause and effect. A weak link between training and ratings does not show that training has no value.
- **Training groups.** With 97.3% participation, the group with no training is very small, so its average rating should not be used to draw conclusions.
- **Missing salaries.** 8 records (about 3%) have no salary and are excluded from the salary measures, so total payroll covers recorded salaries only.
- **Unadjusted pay gap.** It compares averages without controlling for role, department, education or tenure.
- **Currency.** Salaries are treated as KES for all offices, because the data has no currency field.
- **Date assumptions.** Nine date pairs were swapped, and a blank Exit Date was treated as an active employee.
- **Attrition rate definition.** Leavers as a share of all employees on record (cumulative), not a per-period rate.
- **No reasons for leaving** are recorded, so the drivers of attrition cannot be confirmed.

## Recommendations

1. **Review training** content and targeting by department and job role instead of adding hours, starting with Customer Support, then Operations and Finance.
2. **Monitor workload.** Operations and Finance have the highest overtime rates (64.9%), so check whether staffing levels need to change.
3. **Investigate the 2024 exit spike** and introduce exit interviews or surveys to understand the reasons for leaving.
4. **Focus retention** on Kisumu and Mombasa, and on Finance and Customer Support.
5. **Strengthen first-year onboarding,** since 17% of leavers left within a year.
6. **Review pay in the groups with larger gaps** by comparing individual job roles and tenure. Examples to start with are Customer Support roles, Sales Managers (women earn more) and HR Officers (men earn more).

## Skills Demonstrated

- **Data cleaning in Power Query:** text standardisation, date parsing (including Excel serial numbers), conditional logic and documented assumptions.
- **Data modelling:** converting a flat table into a star schema, with an active and an inactive relationship to a Date table.
- **DAX:** 25 measures, including `USERELATIONSHIP`, `KEEPFILTERS`, `ALLSELECTED` and null-safe comparisons.
- **Dashboard design:** five linked pages with a consistent layout, navigation and an executive summary.
- **Analysis and communication:** findings, limitations and recommendations written for HR and senior management, following CRISP-DM.

## Documentation

| File | Contents |
|---|---|
| [`Docs/power-query-steps.md`](Docs/power-query-steps.md) | Cleaning log with the key M formulas |
| [`Docs/dax-measures.md`](Docs/dax-measures.md) | All 25 measures with purposes and formulas |
| [`Docs/data-dictionary.md`](Docs/data-dictionary.md) | Fields, dimension tables and key definitions |

## Folder Contents

```
Workforce-analysis/
README.md
Report/          Written project report (PDF)
Dashboards/      Screenshots of each report page
Data/            Dataset (provided for a capstone project)
Powerbi/         Power BI file (.pbix)
Docs/
    power-query-steps.md
    dax-measures.md
    data-dictionary.md
```
## Tools

Power BI Desktop and Power BI Service, Power Query (M), DAX, Excel (for initial exploration).

## Author

**Doreen Awuor** 
Data Analytics
[LinkedIn](https://www.linkedin.com/in/doreenawuor/) | [GitHub](https://github.com/DA-26) 

