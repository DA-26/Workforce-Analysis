# DAX Measures

The model has 25 measures. Measures that count current employees filter on `Attrition Status = "Active"`, and those that depend on exit dates use the inactive relationship between `Date[Date]` and `Workforce Data[Exit Date]` through `USERELATIONSHIP`. Salary measures skip blank salaries, so they cover employees with a recorded salary.

## Summary

| Measure | Purpose |
|---|---|
| Total Employees | Unique employees on record, using Employee ID |
| Active Headcount | Current employees (Attrition Status = Active) |
| Headcount % of Total | Each group's share of the active headcount in the current selection |
| Active Avg Age | Average age of current employees |
| Unknown Age Headcount | Current employees whose age was missing or invalid |
| Attrition Rate | Leavers as a percentage of all employees on record |
| Exits | Leavers counted by exit date, using the inactive Date-to-Exit Date relationship |
| Avg Tenure of Leavers (Years) | Average length of service at the point of leaving |
| Early Attrition % | Share of leavers who left within their first year |
| Rated Employees | Employees with a valid performance rating |
| Unrated Employees | Employees with a missing or invalid rating |
| Avg Performance Rating | Average performance rating (1 to 5) |
| Avg Training Hours | Average training hours in the last 12 months |
| Training Participation % | Share of employees with more than 0 training hours |
| Avg Rating (Trained) | Average rating of employees with some training |
| Avg Rating (No Training) | Average rating of employees with no training hours |
| Overtime Rate | Share of employees who work overtime |
| Avg Monthly Salary | Average monthly salary of current employees with a recorded salary |
| Min Monthly Salary | Lowest monthly salary among current employees |
| Max Monthly Salary | Highest monthly salary among current employees |
| Total Monthly Payroll | Sum of monthly salaries of current employees with a recorded salary |
| Avg Salary Female | Average monthly salary of current female employees |
| Avg Salary Male | Average monthly salary of current male employees |
| Employees Without Salary | Current employees with no recorded salary |
| Gender Pay Gap % | Male average minus female average, as a percentage of the male average. A positive value means men earn more |

## Formulas

### Headcount

```
Total Employees = DISTINCTCOUNT('Workforce Data'[Employee ID])

Active Headcount = CALCULATE([Total Employees], 'Workforce Data'[Attrition Status] = "Active")

Headcount % of Total = DIVIDE([Active Headcount], CALCULATE([Active Headcount], ALLSELECTED()))

Active Avg Age = CALCULATE(AVERAGE('Workforce Data'[Age]), 'Workforce Data'[Attrition Status] = "Active")

Unknown Age Headcount = CALCULATE([Active Headcount], 'Workforce Data'[Age Band] = "Unknown")
```

### Attrition

```
Attrition Rate =
DIVIDE(
    CALCULATE([Total Employees], 'Workforce Data'[Attrition Status] = "Left"),
    [Total Employees]
)

Exits =
CALCULATE(
    [Total Employees],
    USERELATIONSHIP('Date'[Date], 'Workforce Data'[Exit Date]),
    'Workforce Data'[Attrition Status] = "Left"
)

Avg Tenure of Leavers (Years) =
CALCULATE(
    AVERAGE('Workforce Data'[Tenure at Exit (Years)]),
    USERELATIONSHIP('Date'[Date], 'Workforce Data'[Exit Date]),
    'Workforce Data'[Attrition Status] = "Left"
)

Early Attrition % =
DIVIDE(
    CALCULATE(
        [Exits],
        'Workforce Data'[Tenure at Exit (Years)] < 1
            && NOT ISBLANK('Workforce Data'[Tenure at Exit (Years)])
    ),
    [Exits]
)
```

### Performance and Training

```
Rated Employees =
CALCULATE(
    [Total Employees],
    KEEPFILTERS(NOT ISBLANK('Workforce Data'[Performance Rating]))
)

Unrated Employees = [Total Employees] - [Rated Employees]

Avg Performance Rating = AVERAGE('Workforce Data'[Performance Rating])

Avg Training Hours = AVERAGE('Workforce Data'[Training Hours (Last 12 Months)])

Training Participation % =
DIVIDE(
    CALCULATE([Total Employees], 'Workforce Data'[Training Hours (Last 12 Months)] > 0),
    CALCULATE([Total Employees], NOT ISBLANK('Workforce Data'[Training Hours (Last 12 Months)]))
)

Avg Rating (Trained) =
CALCULATE([Avg Performance Rating], 'Workforce Data'[Training Hours (Last 12 Months)] > 0)

Avg Rating (No Training) =
CALCULATE(
    [Avg Performance Rating],
    'Workforce Data'[Training Hours (Last 12 Months)] = 0
        && NOT ISBLANK('Workforce Data'[Training Hours (Last 12 Months)])
)

Overtime Rate =
DIVIDE(
    CALCULATE([Total Employees], 'Workforce Data'[Overtime] = "Yes"),
    CALCULATE([Total Employees], NOT ISBLANK('Workforce Data'[Overtime]))
)
```

`NOT ISBLANK` is used wherever a comparison could treat a blank as 0, so missing values are not counted as zero training or a low rating. `KEEPFILTERS` in `Rated Employees` lets the measure work when Performance Rating is on a chart axis.

### Compensation and Diversity

```
Avg Monthly Salary =
CALCULATE(AVERAGE('Workforce Data'[Monthly Salary (KES)]), 'Workforce Data'[Attrition Status] = "Active")

Min Monthly Salary =
CALCULATE(MIN('Workforce Data'[Monthly Salary (KES)]), 'Workforce Data'[Attrition Status] = "Active")

Max Monthly Salary =
CALCULATE(MAX('Workforce Data'[Monthly Salary (KES)]), 'Workforce Data'[Attrition Status] = "Active")

Total Monthly Payroll =
CALCULATE(SUM('Workforce Data'[Monthly Salary (KES)]), 'Workforce Data'[Attrition Status] = "Active")

Avg Salary Female = CALCULATE([Avg Monthly Salary], 'Workforce Data'[Gender] = "Female")

Avg Salary Male = CALCULATE([Avg Monthly Salary], 'Workforce Data'[Gender] = "Male")

Employees Without Salary =
CALCULATE([Active Headcount], ISBLANK('Workforce Data'[Monthly Salary (KES)]))

Gender Pay Gap % = DIVIDE([Avg Salary Male] - [Avg Salary Female], [Avg Salary Male])
```

`Avg Salary Female` and `Avg Salary Male` reuse `Avg Monthly Salary`, so they inherit its filter on current employees. A negative `Gender Pay Gap %` means women earn more than men on average. The gap is unadjusted for role, tenure or education.

## Model Dependencies

- `Exits` and `Avg Tenure of Leavers (Years)` (and so `Early Attrition %`) need the inactive relationship between `Date[Date]` and `Workforce Data[Exit Date]`.
- `Unknown Age Headcount` needs the `Age Band` column.
- `Avg Tenure of Leavers (Years)` and `Early Attrition %` need the `Tenure at Exit (Years)` column.
