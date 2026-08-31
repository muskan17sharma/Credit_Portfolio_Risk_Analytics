# file-wise column details  


## 1. loan_portfolio
loan_id – Unique identifier for each individual loan in the portfolio.
origination_date – Date the loan was issued/disbursed to the borrower.
maturity_date – Date on which the loan is scheduled to be fully repaid.
maturity_months – Total tenure of the loan in months from origination to maturity.
sector – Industry sector of the borrower (e.g., Technology, Healthcare).
loan_type – Type/category of loan product, such as mortgage or term loan.
collateral – Whether the loan is secured (backed by collateral) or unsecured.
initial_rating – Credit rating assigned to the borrower/loan at origination.
credit_score – Numeric credit score of the borrower at the time of origination.
ead – Exposure at Default; the outstanding loan amount at risk if default occurs.
coupon_rate – Interest rate (%) charged on the loan.
leverage – Borrower's leverage ratio, indicating debt relative to assets/earnings.
interest_coverage – Ratio showing borrower's ability to pay interest from earnings (EBIT/interest expense).
debt_to_equity – Ratio of borrower's total debt to shareholder equity, a leverage/solvency measure.
pd_annual – Probability of Default; annual likelihood the borrower defaults on the loan.
lgd – Loss Given Default; the proportion of exposure expected to be lost if default happens.
el – Expected Loss; average anticipated loss (EAD × PD × LGD).
unexpected_loss – Potential loss beyond the expected loss, used for capital buffer calculations.
rwa – Risk-Weighted Assets; exposure adjusted for credit risk, used in capital adequacy calculations.
defaulted – Flag (0/1) indicating whether the loan has defaulted.
default_date – Date on which the loan defaulted (blank if not defaulted).
survival_months – Number of months the loan remained active/performing.
recovery_rate – Percentage of exposure recovered after default.
loss_given_default – Actual loss incurred as a percentage of exposure after default (realized LGD).



## 2. credit_ratings
issuer_id – Unique identifier for the borrower/issuer whose rating is tracked.
sector – Industry sector to which the issuer belongs.
year – Year in which the rating observation/transition is recorded.
from_rating – Credit rating of the issuer at the start of the period.
to_rating – Credit rating of the issuer at the end of the period.
upgraded – Flag (0/1) indicating if the issuer's rating improved during the year.
downgraded – Flag (0/1) indicating if the issuer's rating worsened during the year.
defaulted – Flag (0/1) indicating if the issuer defaulted during the year.
notches_moved – Number of rating notches the issuer moved up or down (e.g., A→BBB = -1 notch).



## 3. vintage_analysis
vintage – The origination quarter/cohort of loans being tracked (e.g., 2015Q1).
months_on_books – Number of months since the vintage cohort originated.
n_loans_originated – Total number of loans originated in that vintage cohort.
n_active – Number of loans from the cohort still active (not defaulted/closed) at this point.
n_defaulted_cumulative – Cumulative count of loans from the cohort that have defaulted so far.
cumulative_default_rate – Total percentage of the cohort that has defaulted since origination.
marginal_default_rate – Percentage of the cohort that defaulted in this specific month only.
avg_pd_at_origination – Average probability of default assigned to the cohort at loan issuance.
avg_credit_score – Average borrower credit score for the cohort.



## 4. portfolio_metrics
date – The month/period for which portfolio-level metrics are calculated.
n_active_loans – Number of loans active in the portfolio on that date.
total_ead – Total Exposure at Default across the entire portfolio.
total_el – Total Expected Loss across the entire portfolio.
total_rwa – Total Risk-Weighted Assets across the portfolio.
el_rate – Expected Loss as a percentage of total exposure (Total EL / Total EAD).
avg_pd – Average probability of default across all active loans.
avg_lgd – Average loss given default across all active loans.
var_99 – Value at Risk at 99% confidence; potential loss not expected to be exceeded 99% of the time.
cvar_995 – Conditional VaR at 99.5%; average loss in the worst 0.5% of scenarios (tail risk measure).
sector_hhi – Herfindahl-Hirschman Index measuring sector concentration risk in the portfolio.
new_defaults – Number of new loan defaults recorded in that period.
gdp_growth – Macroeconomic GDP growth rate (%) for that period.
unemployment – Macroeconomic unemployment rate (%) for that period.
policy_rate – Central bank policy interest rate for that period.
credit_spread_bps – Credit spread (in basis points) reflecting market credit risk pricing for that period.



## 5. macro_stress_scenarios
scenario – Name of the stress scenario being modeled (e.g., baseline, recession).
gdp_shock_pp – Shock applied to GDP growth, in percentage points, under this scenario.
unemp_shock_pp – Shock applied to unemployment rate, in percentage points, under this scenario.
rate_shock_pp – Shock applied to interest rates, in percentage points, under this scenario.
credit_spread_bps – Change in credit spread (basis points) assumed under this scenario.
sector – Industry sector to which the stress test is applied.
base_pd – Baseline (non-stressed) probability of default for the sector.
stressed_pd – Probability of default under the stress scenario.
pd_uplift_pp – Increase in PD (percentage points) due to the stress scenario.
pd_multiplier – Factor by which PD increases under stress (stressed PD ÷ base PD).
base_lgd – Baseline loss given default assumption for the sector.
stressed_lgd – Loss given default assumption under the stress scenario.
total_ead – Total exposure at default for the sector under this scenario.
expected_loss_base – Expected loss calculated using baseline (non-stressed) assumptions.
expected_loss_stress – Expected loss calculated using stressed assumptions.
el_increase_pct – Percentage increase in expected loss between baseline and stressed scenario.