# Credit_Portfolio_Risk_Analytics
End-to-end credit portfolio risk analysis covering portfolio exposure, credit ratings, PD, LGD, EAD, expected loss, default risk, vintage analysis, concentration risk, and macroeconomic stress testing.




# Credit Risk Analytics — Power BI Data Model

A Power BI dashboard project analyzing loan-level credit risk, portfolio performance, issuer rating migrations, macroeconomic stress scenarios, and vintage (cohort) default behavior for a simulated lending portfolio.

## Project Overview

This project builds a portfolio-level credit risk analytics model covering:

- **Loan-level risk metrics** — probability of default (PD), loss given default (LGD), expected loss (EL), risk-weighted assets (RWA)
- **Issuer credit rating migrations** — upgrades, downgrades, and default transitions over time
- **Macroeconomic stress testing** — sector-level PD/LGD sensitivity under adverse GDP, unemployment, and rate shock scenarios
- **Portfolio-level time series** — monthly EAD, expected loss, VaR/CVaR, and sector concentration (HHI)
- **Vintage (cohort) analysis** — default curves by loan origination quarter, tracked by months on books

## Data Sources

| File | Grain | Rows | Description |
|---|---|---|---|
| `loan_portfolio.csv` | 1 row per loan | 50,000 | Loan-level attributes, risk scores, and outcomes |
| `credit_ratings.csv` | 1 row per issuer per year | 17,939 | Annual rating transition history per issuer |
| `macro_stress_scenarios.csv` | 1 row per scenario per sector | 60 | Sector-level PD/LGD under stress scenarios |
| `portfolio_metrics.csv` | 1 row per month | 120 | Portfolio-wide monthly risk and performance snapshot |
| `vintage_analysis.csv` | 1 row per vintage per month-on-books | 2,160 | Cohort-level default curves by origination quarter |

## Data Model Design

The five source files do not share primary keys — they were captured at five different grains (loan, issuer, scenario, monthly snapshot, and cohort level), which is typical of real-world credit risk data marts assembled from separate systems. Rather than forcing invalid joins, this project uses a **star schema**: three shared dimension tables built in Power Query, with each fact table connected only to a dimension, never directly to another fact table.

```
                    Dim_Date
                   /    |    \
        loan_portfolio  |  portfolio_metrics
                   \     |
                    Dim_Sector
                   /    |    \
      credit_ratings  loan_portfolio  macro_stress_scenarios
                   
                    Dim_Vintage
                   /            \
        loan_portfolio      vintage_analysis
```

### Dimension tables

| Dimension | Built from | Purpose |
|---|---|---|
| `Dim_Date` | `CALENDAR()` DAX table, 2015–2024 | Connects `loan_portfolio[origination_date]` and `portfolio_metrics[date]` on a shared calendar |
| `Dim_Sector` | Deduplicated `sector` values | Connects `loan_portfolio`, `credit_ratings`, and `macro_stress_scenarios` on the 10 shared sector names |
| `Dim_Vintage` | Deduplicated `vintage` values from `vintage_analysis` | Connects `loan_portfolio` (via a derived `Vintage` column) and `vintage_analysis` |

### Derived column

`loan_portfolio` did not originally carry a vintage tag, so one was derived in Power Query to match the `YYYYQ#` format used in `vintage_analysis`:

```
Vintage = Text.From(Date.Year([origination_date])) & "Q" & Text.From(Date.QuarterOfYear([origination_date]))
```

### Relationship rules followed

- All dimension-to-fact relationships are **one-to-many, single-direction** (dimension → fact).
- No many-to-many relationships were kept in the final model. Where a direct fact-to-fact join produced a many-to-many warning (e.g. `loan_portfolio` to `vintage_analysis` on vintage, or to `credit_ratings`/`macro_stress_scenarios` on sector), a dimension table was inserted between them instead.
- `loan_portfolio[maturity_date]` and `[default_date]` are related to `Dim_Date` as **inactive** relationships, since only one active relationship can exist per table pair. These are invoked in DAX measures using `USERELATIONSHIP()` where needed (e.g. maturity-based cash flow measures).

### Known limitation

`credit_ratings` is issuer-level while `loan_portfolio` is loan-level, and the dataset provides no issuer-to-loan mapping key. The two tables are connected only through `Dim_Sector`, which supports sector-level comparisons (e.g. downgrade rate by sector) but does not allow drill-through from an individual loan to its issuer's rating history. This is called out explicitly rather than worked around, since forcing a fabricated key would misrepresent the data.

## Tools Used

- **Power BI Desktop** — data modeling, DAX measures, report visuals
- **Power Query (M)** — dimension table creation, derived columns, data cleaning

## Key Metrics Tracked

- Probability of Default (PD), Loss Given Default (LGD), Expected Loss (EL)
- Exposure at Default (EAD), Risk-Weighted Assets (RWA)
- Value at Risk (VaR 99%), Conditional VaR (CVaR 99.5%)
- Rating migration rates (upgrade/downgrade/default) by sector and year
- Stressed vs. base-case PD/LGD under macroeconomic scenarios
- Cumulative and marginal default rates by loan vintage

## Next Steps

- [ ] Build DAX measures for portfolio-level PD, LGD, and EL rollups
- [ ] Build vintage curve and rating migration visuals
- [ ] Build stress-test scenario comparison page
- [ ] Publish to Power BI Service and add screenshots to this README

