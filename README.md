# Mortgage applications submitted through a broker are twice as likely to finish

**Applications submitted directly to a lender (the retail channel) have 2.03x the odds of being abandoned compared with applications arriving through a broker (95% CI 1.97 - 2.10), and the gap widens, rather than narrows, after controlling for applicant and loan characteristics.**

Analysis of 255970 US mortgage applications filed in Colorado in 2025, using public HMDA data. This is an association, not a causal effect.

---

## The question

Around one in five mortgage applications ends without a decision. The applicant withdraws, or the file is closed because requested documents never arrived. Neither party chose that outcome. The lender has pulled credit, ordered work and paid staff time, and recovers nothing. The applicant spent weeks and ended up without a mortgage.

Unlike a denial, which is a decision. An abandonment is usually friction which can be fixed.

**Are applications submitted through third-party channels more likely to be abandoned than those submitted directly to the lender, and does any gap survive controlling for applicant demographics and loan characteristics?**

---

## Findings

**1. Retail applications abandon far more often than broker applications.**
21.1% against 12.9%, a gap of 8.2 percentage points across 226811 applications (unadjusted odds ratio 1.80).

**2. The gap is not a composition effect.**
The two channels handle different kinds of loan: 61% of broker applications are home purchases, against 40% of retail applications. But comparing abandonment rates within each loan purpose, retail applications were higher in all five categories, by between 6.9 and 12.8 percentage points.

**3. Controlling for covariates strengthens the association rather than explaining it.**
A logistic regression adjusting for loan purpose, loan type, occupancy, lien status, sex, race, ethnicity, age, income and loan amount returns an adjusted odds ratio (OR) of **2.03**, higher than the unadjusted 1.80. This is a suppression effect: the retail channel is weighted toward categories that abandon less (home improvement OR 0.58, subordinate liens OR 0.53), which was depressing its raw rate.

**4. Channel is the largest effect in the model.**
The strongest competing predictor is lien status (OR 0.53), a product characteristic rather than anything a lender controls. No applicant attribute comes close.

**5. Reported income is unrelated to abandonment.**
Applications with missing income abandon at 24.9% against 19.3% elsewhere. However, once other covariates are included, neither income (OR 1.00) nor its absence (OR 1.02) is associated with the outcome. The raw difference was confounded.

### The leading explanation, which this data cannot test

Brokers are compensated only when a loan closes, giving them reason to screen out applications unlikely to complete before submitting them. Retail branches record every enquiry that arrives. If that is what drives the gap, the selection happens before the data begins and is invisible in HMDA.

---

## What the data can and can't show

Nobody was randomly assigned to a broker. People sort themselves between the two routes, and there is no reason to assume they sort evenly, so the two groups may differ before anything happens to their application. Everything here is therefore an *association*. I can show the gap is real and that it isn't explained by the applicant characteristics in the data, but I can't prove the channel causes it.

HMDA also has *no credit score*, the single biggest factor in any lending decision. That's why this project says nothing about fair lending. The demographic variables are in the model as controls only: they check that the channel result isn't just reflecting a different kind of applicant using each channel. 

It's still the best data there is. HMDA covers every mortgage application in the state under legal mandate, and nothing else comes close at this scale. The missing credit score also matters less here than it would for a study of denials: abandonment is about whether a process finishes, not whether someone qualifies.

---

## Method

| Stage | Approach |
|---|---|
| Source | CFPB HMDA Data Browser — 2025 loan-level records, Colorado, all institutions (295125 rows × 99 fields) |
| Outcome | Binary: abandoned (`action_taken` 4, 5) vs completed (1, 2, 3) |
| Exposure | Channel proxy built from `submission_of_application` and `initially_payable_to_institution` |
| Tests | Chi-square test of independence; Cramér's V; Wilson confidence intervals |
| Model | Logistic regression (statsmodels), coefficients reported as odds ratios |

HMDA doesn't record the channel, so it had to be built from two fields. An applicant who went straight to the institution that ended up lending the money counts as retail. One who came in some other way, where the loan was still payable to that institution, counts as a broker. It's a proxy, not a label. Lenders don't all report these fields the same way, so it's a close approximation rather than the real thing. There's a third group, correspondent lending, but it's small and patchy and was left out of the model.

### Decisions worth defending

**Dropping non-applications.** `action_taken` codes 6, 7 and 8 are purchased loans and preapproval requests. A purchased loan is a closed loan bought from another institution — there is no applicant journey in that row. Excluded.

**Sentinel values.** HMDA encodes "not provided" as `8888` rather than null, so standard missingness checks report those columns as complete. Investigation showed 88% of affected rows were purchased loans (already excluded); the remaining 4404 are 1.7% of applications and were dropped.

**Missing income is not missing at random.** It correlates with the outcome (24.9% vs 19.3% abandonment) and is uneven by channel (7.0% of broker applications vs 4.5% of retail). Dropping those 11282 rows would have biased the sample toward lower abandonment, so they were retained with a median fill plus an explicit `income_missing` indicator.

**Leakage control.** `interest_rate`, `rate_spread`, `total_loan_costs`, `origination_charges` and `loan_term` are populated only for loans that closed. An abandoned application has no interest rate because no loan was ever made. All were excluded. Including any of them would let the model predict the outcome by detecting absence.

**Negative income.** 180 applications report business losses, the most extreme at −$77.8m. Taking logs requires positive values, so a signed log - sign(x)·log(1+|x|) was used instead. It keeps the direction of a negative income rather than discarding or zeroing those rows.

**Sparse categories and reference levels.** "Free Form Text Only" (18 rows) produced an unstable ethnicity coefficient (OR 5.0, 95% CI 1.9–13.2) and was collapsed into "not available". Statsmodels defaults to the alphabetically first level as reference, which put a 492-row category as the race baseline; references were set explicitly to the largest categories.

**Why the p-value is not the headline.** At n = 226811 the chi-square statistic is 1433 on 1 degree of freedom and the p-value is below floating-point precision. At this sample size significance is nearly guaranteed and carries no information, so effect sizes are reported instead. Cramér's V is 0.079, conventionally "weak", while the same gap represents roughly 18600 applications that went nowhere. Both are true and they measure different things.

---

## Repository

```
notebooks/01-explore.ipynb    Exploration, cleaning, analysis and modelling
data/raw/                     Source CSV (not committed — see below)
src/                          Reusable functions
```

The source file is not committed. To reproduce, download 2025 Colorado records from the [HMDA Data Browser](https://ffiec.cfpb.gov/data-browser/) and place the CSV in `data/raw/`.

Field definitions come from the FFIEC Filing Instructions Guide. HMDA fields are filed as integer codes; the mapping used is defined explicitly in the notebook so it can be checked against the guide.

## Tools

Python (pandas, numpy, scipy, statsmodels, plotly), Jupyter, Git.

---

## Status

Complete: exploration, cleaning, outcome and exposure definition, hypothesis testing, logistic regression.

In progress: SQL layer (DuckDB), Tableau dashboard, county-level choropleth, cost estimate, and two robustness checks: lender fixed effects, and an alternative outcome definition treating "approved but not accepted" as abandonment.
