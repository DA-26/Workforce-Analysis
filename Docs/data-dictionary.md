# Data Dictionary

## Fact table: Workforce Data (243 rows, 18 columns)

One row per employee. The raw file had 16 fields, and the Remarks field was removed because only 16 of its 1,000 rows were filled. Three columns were added in Power Query.

| Column | Type | Description | Cleaning notes |
|---|---|---|---|
| Employee ID | Text | Unique identifier, format `EMP-####` | Standardised, blanks removed, 4 duplicates removed |
| Full Name | Text | Employee name | Trimmed, double spaces removed |
| Hire Date | Date | Date the employee joined | 9 records swapped where the exit date came first |
| Exit Date | Date | Date the employee left (blank for current employees) | Text, datetimes and Excel serial numbers parsed into dates |
| Department | Text | Business unit: Finance, Human Resources, Operations, Sales, Engineering or Customer Support | Case and variants standardised |
| Job Role | Text | Specific role within the department (24 roles) | Case and spacing standardised |
| Office Location | Text | City where the employee is based (6 offices) | Case standardised |
| Gender | Text | Female or Male | Variants such as `M` and `FEMALE` merged |
| Age | Whole number | Age of the employee | Invalid values (200, -5) set to null |
| Education Level | Text | Highest qualification: Bachelor's Degree, Master's Degree, Diploma or Professional Certification | Variants merged |
| Monthly Salary (KES) | Decimal | Gross monthly salary | Text converted to numbers, one negative value corrected, 8 missing salaries left blank |
| Performance Rating | Whole number | Most recent rating on a 1 to 5 scale | Text such as `2/5` converted, 5 missing left blank |
| Training Hours (Last 12 Months) | Whole number | Hours of training completed | `N/A` set to null, negative values corrected |
| Overtime | Text | Whether the employee regularly works overtime (Yes or No) | Variants reduced to Yes and No |
| Attrition Status | Text | Active or Left | Left with a blank Exit Date set to Active |
| Age Band | Text | Under 25, 25-34, 35-44, 45-54, 55 and above, or Unknown | Added column |
| Tenure at Exit (Years) | Decimal | Years between Hire Date and Exit Date (blank for current employees) | Added column |
| Tenure Band at Exit | Text | Tenure at exit grouped into bands, with "Active" for current employees | Added column |

## Dimension Tables

| Table | Columns | Relationship to Workforce Data |
|---|---|---|
| Date | Date, Day of Week, Fiscal Quarter Year, Month, Year, Date Hierarchy | One-to-many on Hire Date (active), and on Exit Date (inactive) |
| Department | Department, Department ID | One-to-many on Department |
| Education | Education, Education ID | One-to-many on Education Level |
| Job Role | Job Role, Job Role ID | One-to-many on Job Role |
| Location | Location, Location ID, Country | One-to-many on Office Location |

The Date table is generated in DAX with `CALENDARAUTO()` and marked as a date table. Country is an attribute of Location: Kenya (Nairobi, Mombasa, Kisumu), Uganda (Kampala), Tanzania (Dar es Salaam) and Rwanda (Kigali).

## Key Definitions

- **Current employee:** Attrition Status = Active (208 employees).
- **Leaver:** Attrition Status = Left (35 employees).
- **Attrition rate:** leavers as a share of all employees on record (35 of 243, or 14.4%).
- **Early attrition:** leavers with less than 1 year of tenure at exit.
- **Gender pay gap:** male average salary minus female average salary, as a percentage of the male average. A negative value means women earn more.
