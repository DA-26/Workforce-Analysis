# Power Query Cleaning Log

All cleaning was done in Power Query inside Power BI Desktop, and the source Excel file was never edited. The query takes the raw data from **1,000 rows and 16 columns** to **243 rows and 18 columns**.

The steps below are grouped by theme. The complete M script is in the `.pbix` file under **Transform data > Advanced Editor**.

## Summary

| # | Step | What was done | Result |
|---|---|---|---|
| 1 | Remove empty records | Filtered out rows with no Employee ID. 751 rows held only a hire date and 2 were otherwise empty | 247 rows |
| 2 | Standardise Employee ID | Trimmed, uppercased, added the hyphen after `EMP`, removed the `.0` suffix | Format `EMP-####` |
| 3 | Remove duplicates | 4 records had the same details but different hire dates | 243 rows |
| 4 | Standardise text | Trimmed, standardised case, merged variants and removed double spaces in Full Name | Consistent categories |
| 5 | Clean Exit Date | Parsed text dates, datetimes and Excel serial numbers into dates | One valid date type |
| 6 | Fix reversed dates | Swapped Hire Date and Exit Date in 9 records where the exit came first | No negative tenure |
| 7 | Correct Attrition Status | A blank Exit Date means an active employee | Left reduced to 35 |
| 8 | Clean numeric fields | Salary, Performance Rating, Training Hours and Age converted to numbers, with invalid values handled | Numeric columns |
| 9 | Remove Remarks | Only 16 entries and not needed | 15 columns |
| 10 | Add derived columns | Tenure at Exit (Years), Tenure Band at Exit and Age Band | 18 columns |

## Details

### Employee ID (steps 1 to 3)

The raw IDs included `emp-1164`, `EMP1759`, `EMP-1734.0` and IDs with leading spaces. IDs were trimmed and uppercased, `EMP` was replaced with `EMP-` and `EMP--` was collapsed back to `EMP-`, and `.0` was removed. Rows without an ID were then filtered out and duplicates removed. Afterwards the column showed 243 distinct and 243 unique values.

```
Table.ReplaceValue(#"Replaced Value1", "EMP--", "EMP-", Replacer.ReplaceText, {"Employee ID"})
```

### Text standardisation (step 4)

- Case and spacing were standardised in Department, Job Role, Office Location, Education Level and Gender.
- Variants were merged: `DIPLOMA` and `Diploma`, `Professional Cert` and `Professional Certification`, `M` and `Male`, `FEMALE` and `Female`, `KIGALI` and `Kigali`.
- Overtime values (`Yes`, `No`, `no`, `FALSE`, `N`, `0`) were reduced to Yes and No.
- Attrition Status values (`left`, `active`, `Currently Employed`) were reduced to Left and Active.

### Exit Date (step 5)

The raw column held real dates, datetimes (`14/02/2025 00:00:00`), text such as `05/27/2025` and `2024.11.02`, Excel serial numbers (two records), and non-dates (`Not Applicable`, `NA`, `Still Active`). A custom step converts each kind, and anything that is not a date becomes null.

```
Table.TransformColumns(#"Replaced Value2", {{"Exit Date", each
    if _ is datetime then DateTime.Date(_)
    else if _ is date then _
    else if _ is number then Date.From(_)
    else if _ is text then (
        try Date.From(Number.FromText(Text.Trim(_), "en-US"))
        otherwise try Date.FromText(Text.Replace(Text.Trim(_), ".", "-"), [Culture = "en-US"])
        otherwise try Date.FromText(Text.Replace(Text.Trim(_), ".", "-"), [Culture = "en-GB"])
        otherwise null)
    else null, type date}})
```

Text dates are read month-first, then day-first.

### Reversed dates (step 6)

9 records had an Exit Date earlier than the Hire Date and an Attrition Status of "Left". The dates were assumed to be entered the wrong way round. A temporary flag column (`Date Order Corrected`) identified them, two columns built the corrected dates, and the originals were then replaced. The flag was removed afterwards.

```
// Hire Date (corrected)
if [Hire Date] = null or [Exit Date] = null then [Hire Date] else List.Min({[Hire Date], [Exit Date]})

// Exit Date (corrected)
if [Hire Date] = null or [Exit Date] = null then [Exit Date] else List.Max({[Hire Date], [Exit Date]})
```

### Attrition Status (step 7)

The guideline for the data is that a blank Exit Date means the employee is still active. Records marked "Left" with no Exit Date were therefore set to Active. After recovering the two exit dates stored as Excel serial numbers, 37 records were marked "Left" with 2 of them having no Exit Date, which left **35 Left and 208 Active**.

```
if [Attrition Status] = "Left" and [Exit Date] = null then "Active" else [Attrition Status]
```

### Numeric fields (step 8)

| Field | Issue | Treatment |
|---|---|---|
| Monthly Salary (KES) | Text such as `KES 242,800`, one value of -105,900, 8 missing | Converted to numbers. The negative value was assumed to be a sign error and corrected to 105,900. The 8 missing salaries were left as null |
| Performance Rating | Text such as `2/5` and `1 out of 5`, 5 missing | Converted to whole numbers on the 1 to 5 scale. Missing ratings were left blank |
| Training Hours (Last 12 Months) | 15 values of `N/A`, 6 blanks and 3 negative values | `N/A` set to null, because 0 would imply no training. Negative values were corrected |
| Age | Text such as `57 yrs`, values of 200 and -5, 6 missing | Converted to numbers. Invalid values were set to null and shown as "Unknown" in the age band |

### Derived columns (step 10)

| Column | Logic |
|---|---|
| Tenure at Exit (Years) | Days between Hire Date and Exit Date, divided by 365.25. Null for employees with no Exit Date |
| Tenure Band at Exit | Tenure at exit grouped into bands, with "Active" for employees who have not left |
| Age Band | Under 25, 25-34, 35-44, 45-54, 55 and above, and Unknown for missing or invalid ages |

```
// Tenure at Exit (Years)
if [Exit Date] = null or [Hire Date] = null then null else Duration.Days([Exit Date] - [Hire Date]) / 365.25

// Age Band
if [Age] = null then "Unknown" else if [Age] < 25 then "Under 25" else if [Age] < 35 then "25-34" else if [Age] < 45 then "35-44" else if [Age] < 55 then "45-54" else "55 and above"
```

## Checks after cleaning

- Employee ID: 243 distinct values for 243 rows.
- Active (208) plus Left (35) equals 243.
- No negative values in Tenure at Exit (Years).
- Column quality showed 0% errors in every column.
- Exit Date is 86% empty, which is expected because only 35 employees have left.

## Assumptions

- A blank Exit Date means the employee is active.
- The 9 reversed date pairs were entered in the wrong order.
- Ambiguous text dates were read month-first, then day-first.
- Salaries are in KES for all offices.
- Missing ratings, salaries, training hours and ages are left blank and not estimated.
