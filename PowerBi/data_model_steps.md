# Power BI Data Model — Build Log

This document walks through how the data model for the Credit Risk Analytics dashboard was built, including the problems encountered and how each was resolved. It's meant to show the reasoning behind the model, not just the final result.

## 1. Starting Point

Five CSV files were loaded into Power BI:

- `loan_portfolio.csv` (50,000 rows, loan-level)
- `credit_ratings.csv` (17,939 rows, issuer-level)
- `macro_stress_scenarios.csv` (60 rows, scenario/sector-level)
- `portfolio_metrics.csv` (120 rows, monthly portfolio-level)
- `vintage_analysis.csv` (2,160 rows, cohort-level)

On load, Power BI's autodetect found **no relationships** between any of the tables.

## 2. Diagnosing Why

Checked each table's key columns directly rather than guessing:

- `loan_portfolio.loan_id` and `credit_ratings.issuer_id` have zero overlapping values — they identify different entities (loans vs. issuing companies) at different grains. No valid direct join exists between these two tables.
- `sector` is spelled identically across `loan_portfolio`, `credit_ratings`, and `macro_stress_scenarios` (10 shared sector names) — a usable shared attribute, but not a primary key.
- `loan_portfolio.origination_date` and `portfolio_metrics.date` are both month-start dates, but represent different things: `origination_date` is a per-loan event date, while `portfolio_metrics.date` is a monthly portfolio-wide snapshot covering all loans active in that month, not just loans originated then. Joining them directly would misrepresent the data.
- `vintage_analysis.vintage` uses a `YYYYQ#` format (e.g. `2019Q3`); `loan_portfolio` had no equivalent column at all.

**Conclusion:** the tables are at five different grains and need dimension tables to connect properly — a star schema, not direct fact-to-fact joins.

## 3. Building Dim_Date

Created via DAX in Model view:

```
Dim_Date = CALENDAR(DATE(2015,1,1), DATE(2024,12,31))
```

Added helper columns (each added separately — DAX calculated columns can't be defined multiple at once in a single formula):

```
Year = YEAR(Dim_Date[Date])
MonthName = FORMAT(Dim_Date[Date], "MMMM")
Quarter = "Q" & FORMAT(Dim_Date[Date], "Q")
```

Marked the table as a Date Table (Table tools → Mark as Date Table) so time-intelligence DAX functions work correctly.

**Relationships:**
- `Dim_Date[Date]` → `loan_portfolio[origination_date]` — one-to-many, active
- `Dim_Date[Date]` → `portfolio_metrics[date]` — one-to-one (both columns are unique at this grain), single-direction

**Note:** `loan_portfolio` also has `maturity_date` and `default_date`. Only one relationship per table pair can be active, so these remain inactive by default and would be invoked in a measure with `USERELATIONSHIP()` if needed later (e.g. a "loans maturing in period X" measure).

## 4. Building Dim_Vintage

`loan_portfolio` had no vintage column, so one was derived in Power Query to match `vintage_analysis`'s format:

1. Selected `loan_portfolio` in Power Query → Add Column → Custom Column
2. Name: `Vintage`
3. Formula:
   ```
   Text.From(Date.Year([origination_date])) & "Q" & Text.From(Date.QuarterOfYear([origination_date]))
   ```
   This pulls the year and calendar quarter from `origination_date` and concatenates them into the same `YYYYQ#` text format used in `vintage_analysis`.

**First attempt (incorrect):** joined `loan_portfolio[Vintage]` directly to `vintage_analysis[vintage]`. Power BI flagged this as **many-to-many**, because neither column is unique — `loan_portfolio` has thousands of loans per vintage, and `vintage_analysis` has one row per vintage *per month-on-books* (multiple rows per vintage). A many-to-many relationship here would cause ambiguous filtering and incorrect aggregated totals.

**Fix:** built a proper dimension table:

1. Referenced `vintage_analysis` in Power Query (right-click → Reference)
2. Renamed the new query `Dim_Vintage`
3. Kept only the `vintage` column (Remove Other Columns)
4. Removed duplicates, leaving one row per unique vintage
5. Closed & Applied

**Relationships:**
- `Dim_Vintage[vintage]` → `loan_portfolio[Vintage]` — one-to-many, single-direction
- `Dim_Vintage[vintage]` → `vintage_analysis[vintage]` — one-to-many, single-direction

No warnings after this change.

## 5. Building Dim_Sector

Same many-to-many issue showed up when `sector` was used to connect `loan_portfolio` directly to `credit_ratings` and `macro_stress_scenarios` — none of these columns are unique on their own.

**Fix:** same pattern as Dim_Vintage:

1. Referenced `loan_portfolio` in Power Query
2. Renamed the new query `Dim_Sector`
3. Kept only the `sector` column, removed duplicates (10 unique sectors)
4. Closed & Applied

**Relationships:**
- `Dim_Sector[sector]` → `loan_portfolio[sector]` — one-to-many, single-direction
- `Dim_Sector[sector]` → `credit_ratings[sector]` — one-to-many, single-direction
- `Dim_Sector[sector]` → `macro_stress_scenarios[sector]` — one-to-many, single-direction



## 6. Create Calculated Measures

Seven DAX measures were created to calculate the key portfolio and credit-risk metrics.


| Measure | Purpose |
|---|---|
| **Total Loans** | Calculates the total number of loans in the portfolio |
| **Total EAD** | Calculates the total Exposure at Default |
| **Total EL** | Calculates the total Expected Loss |
| **Total RWA** | Calculates the total Risk-Weighted Assets |
| **Default Rate** | Measures the percentage of loans that have defaulted |
| **Avg PD** | Calculates the average Probability of Default |
| **Avg LGD** | Calculates the average Loss Given Default |


## 7. Final Model

A clean star schema: **three dimension tables** (`Dim_Date`, `Dim_Vintage`, `Dim_Sector`),
                     **five fact tables**, all relationships one-to-many and single-direction. No many-to-many relationships, no bidirectional cross-filtering.

```
Dim_Date ──────┬── loan_portfolio (origination_date)
               └── portfolio_metrics (date)

Dim_Sector ────┬── loan_portfolio (sector)
               ├── credit_ratings (sector)
               └── macro_stress_scenarios (sector)

Dim_Vintage ───┬── loan_portfolio (Vintage, derived)
               └── vintage_analysis (vintage)
```

**Known limitation kept in the model rather than worked around:** `credit_ratings` (issuer-level) and `loan_portfolio` (loan-level) share no direct key. They connect only through `Dim_Sector`, which supports sector-level analysis (e.g. downgrade rate by sector) but not loan-to-issuer drill-through. There's no clean way to fix this without a mapping table that doesn't exist in the source data — documenting the constraint is more accurate than forcing a relationship that would misrepresent the data.

## Lessons / Decisions Worth Explaining in an Interview

- Chose to build dimension tables instead of forcing many-to-many joins, because many-to-many relationships with bidirectional filtering can silently double-count or produce ambiguous totals — the kind of error that's hard to catch after the fact.
- Kept relationships single-direction by default; bidirectional filtering was avoided everywhere except where the grain genuinely required it.
- Called out the credit_ratings-to-loan_portfolio grain mismatch explicitly instead of hiding it, since acknowledging a data limitation is more defensible than papering over it with a fabricated join.

