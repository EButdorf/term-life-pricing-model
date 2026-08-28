# Term Life Insurance Pricing Model

A from-scratch actuarial pricing model that calculates the **net level annual premium** for a level term life insurance policy. Built in Excel using a published Society of Actuaries (SOA) mortality table and the **equivalence principle**, the core pricing framework used in life insurance.

The model is fully dynamic: change the issue age, term, benefit amount, or interest rate, and the premium recalculates automatically.

---

## What it does

Given a policyholder profile and a set of economic assumptions, the model answers a fundamental insurance question: *what is the fair annual premium for this policy?*

For the base case — a **male, age 40, preferred non-smoker, $250,000 benefit, 20-year level term, 5% interest** — the model produces a net annual premium of **$333.87**.

---

## Methodology

The model is built on the **equivalence principle**: at issue, the expected present value (EPV) of the premiums the insurer collects must equal the expected present value of the death benefits it expects to pay.

Each future cash flow is weighted by two things — the **probability** it occurs and a **discount factor** that converts it to today's dollars — and then summed across every policy year.

**EPV of death benefits** (benefit paid at the end of the year of death):

```
EPV_benefits = Σ  (t-1)p_x · q_(x+t-1) · B · v^t        for t = 1 … n
```

**EPV of premiums** (level premium of 1, paid at the start of each year the policyholder is alive):

```
EPV_premium_per_1 = Σ  (t-1)p_x · v^(t-1)               for t = 1 … n
```

**Net level annual premium:**

```
Premium = EPV_benefits / EPV_premium_per_1
```

Where:
- `q_(x+t-1)` = probability of death during policy year *t* (from the mortality table)
- `(t-1)p_x` = probability the policyholder survives to the start of year *t* (a running product of yearly survival probabilities)
- `B` = death benefit
- `v = 1 / (1 + i)` = annual discount factor at interest rate *i*
- `n` = policy term

---

## Data source

Mortality rates come from the **Society of Actuaries Mortality and Other Rate Tables (MORT)** database at [mort.soa.org](https://mort.soa.org).

- **Table:** 2017 Loaded CSO Preferred Structure — Nonsmoker, Preferred, Male, Age Last Birthday (ALB)
- **Table ID:** 3310
- **Structure:** Select & Ultimate, with a 25-year select period (durations 1–25)

Because the policy is newly underwritten, the model uses the **select** mortality rates for the issue age, read across policy durations. Selection matters: a freshly underwritten preferred life has meaningfully lower mortality in the early policy years than an equivalent attained-age ultimate rate would imply.

---

## Model structure

**Inputs** (assumption cells, all downstream formulas reference these):

| Input | Base case |
|---|---|
| Issue age | 40 |
| Term (years) | 20 |
| Benefit | $250,000 |
| Interest rate | 5% |

**Calculation table** (one row per policy year):

| Column | Meaning |
|---|---|
| Year / Age | Policy year and attained age |
| q(x) | Probability of death that year (looked up from the SOA table by issue age and duration) |
| p(x) | Probability of surviving the year, `1 − q(x)` |
| Alive at start | Probability of surviving to the start of the year (running product of prior survival) |
| Prem discount | Discount factor for a start-of-year premium, `v^(t−1)` |
| Benefit discount | Discount factor for an end-of-year benefit, `v^t` |
| EPV benefit | Expected present value of the death benefit that year |
| EPV premium | Expected present value of $1 of premium that year |

The two EPV columns are summed and divided to produce the premium.

**Dynamic design:**
- Year and age columns use spill formulas (`SEQUENCE`) that resize automatically with the term.
- Calculation formulas are guarded (`IF(year is blank, "", …)`) so the table blanks out cleanly when the term shrinks, and the summary sums cover the full block so they never need adjusting.
- **Data validation** restricts inputs to the table's valid range: term 1–25 (the select period) and issue age 18–95.

---

## Selected results

Base case profile (male, preferred non-smoker, $250,000 benefit, 5% interest), varying the term:

| Term | Net annual premium |
|---|---|
| 10 years | $171.18 |
| 20 years | $333.87 |
| 25 years | $454.39 |

Premiums rise with term, as expected: a longer term covers more years and reaches into higher-mortality ages.

As a reasonableness check, these pure-mortality premiums sit just below real-world quotes for a comparable healthy 40-year-old — consistent with the fact that market premiums also load in expenses, profit, and commissions on top of the underlying mortality cost.

---

## Assumptions and limitations

This is a foundational pricing model built to demonstrate the actuarial framework end to end. It makes several simplifying assumptions, stated here for transparency:

- **Net premium only.** The model prices the pure cost of mortality. It does not load for expenses, commissions, profit margin, or contingencies, so results are lower than a market premium.
- **Loaded valuation table.** The 2017 CSO table includes conservative margins intended for reserving/valuation. A true pricing exercise would use a best-estimate (basic/experience) table; premiums here are therefore somewhat conservative.
- **No lapses.** Every policyholder is assumed to persist for the full term. Real pricing reflects lapse/surrender behavior.
- **Deterministic, level interest.** A single flat interest rate is used, with no stochastic or term-structure modeling.
- **Select period only.** The model relies on select rates, which limits the term to the 25-year select period. Longer terms would require splicing in ultimate rates.
- **Single mortality basis.** One table (preferred non-smoker, male) is used; other risk classes would require the corresponding tables.

---

## How to use

1. Open the workbook.
2. Set the four inputs (issue age, term, benefit, interest rate); inputs are validated to the supported ranges.
3. The calculation table and the annual premium update automatically.

---

## Possible extensions

- **Policy reserves** — project the reserve held over the life of the contract (the natural next step in valuation).
- **Python implementation** — reproduce the engine in Python for extensibility and reproducibility.
- **Sensitivity analysis** — a table or chart showing how the premium responds to changes in age, term, and interest rate.
- **Expense and profit loading** — extend the net premium to a gross (market-style) premium.
- **Additional risk classes** — support smoker/non-smoker and male/female tables.

---

## Author

Emanuel Butdorf — B.S. Mathematics (Actuarial Science) & B.B.A. Finance. Built as an independent project to apply actuarial pricing concepts to real published mortality data.
