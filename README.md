# AMEX-CAMPUS-CHALLENGE-2026
# BAEP — Breakage-Adjusted Economic Profit

**Round 1**

Rank 500,000 Premier cardmembers by profitability to the issuer. The submission is a per-member score; the metric is the overlap between the predicted top 20% (100,000 rows) and the hidden true top 20%, split 70/30 public/private, under an integrity review.

BAEP scores each member with a **single closed-form economic-profit equation** — accounting margin charged for the balance-sheet risk it rides on — applied identically to every member. Every coefficient traces to a public benchmark (Amex 10-K, Basel IRB, the Premier product sheet); **none is fitted to leaderboard feedback**.

Final Leaderboard score - 0.92
---

## Why economic profit, not accounting profit

Two members with equal margin are not equally valuable if one carries a large risky revolving balance and the other is a capital-light transactor: the first consumes expected loss and regulatory capital the second does not. Ranking by economic profit taxes that difference explicitly — how an issuer actually prioritises a portfolio.

## The equation

```
EconomicProfit =
    0.023*(f6+f7+f8+f9+f10)                     discount revenue (avg discount rate 2.3%)
  + 0.17*f1*(1 - f11)                           net interest on the PERFORMING balance only
  + 575*f20 + 175*f19                           fee income (card mid-band + supplementary)
  - 0.011 * rho_i * (5*(f6+f9) + (f7+f8+f10))   breakage-adjusted rewards liability
  - (35*f13 + f14 + 15*f15 + f16)               benefit / statement-credit costs
  - 0.85 * f11 * EAD                            expected loss (PD x LGD x exposure)
  - 0.0096 * EAD                                economic-capital charge (12% hurdle x 8% ratio)
  - 140*f2 - 620*f3                             retention + collections cost

  EAD   = f1 + 0.40*max(f17 - f1, 0)            drawn balance + 40% CCF on undrawn line
  rho_i = w*clip(f21/earned, 0, 1.2) + (1-w)*rho_bar,   w = earned/(earned + 50000)
```

## Two mechanisms that distinguish this from a linear P&L

**Breakage-adjusted rewards.** Points earned are not points paid for — members let a share expire. Rewards are costed at each member's own redemption propensity `rho_i` (from f21 ÷ points earned), credibility-smoothed toward the population mean (`rho_bar = 0.452`, K = 50,000 points) so thin-history members are not assigned extreme ratios. Mirrors points-liability accounting.

**Basel-style exposure.** The lend line `f17` enters as *risk, not revenue*. Exposure at default uses a 40% credit-conversion factor on the undrawn portion — a large committed line to a risky member destroys value before a dollar is drawn. Expected loss is PD (`f11`) × LGD (85%) × EAD, and an economic-capital charge taxes the capital that exposure consumes.

## Feature treatment

| Feature | Role |
|---|---|
| f6–f10 | category spend → discount revenue + points earn |
| f1 | revolve balance → net interest (performing) + drawn exposure |
| f17 | lend line → undrawn exposure (risk, not revenue) |
| f19, f20 | accounts → fee income |
| f11 | risk score → probability of default |
| f13–f16 | benefit usage → benefit cost |
| f2, f3 | cancellation / collections flags → attrition + recovery cost |
| f21 | points redeemed → redemption propensity |
| f5 | **excluded** — decoy (corr 0.10 with sum f6–f10, 13× scale mismatch) |
| f18 | **excluded** — duplicate of f17 (corr 0.90) |
| f4 | **excluded** — points stock, not a flow |
| f12 / f22 / f23 | **excluded** — engagement, no direct P&L linkage |

Missing `f11` → population median; block-missing spend/benefit/line/redemption → 0 (non-enrolment).

## Results (offline, no leaderboard input)

| Check | Value |
|---|---|
| Overlap-metric band | ~0.92 |
| Aggregate EL / aggregate EAD | 3.06% (premium charge-off benchmark ~1.5–3%) |
| Revenue mix | discount 38.1% · NII 17.4% · fees 44.5% |
| Top-20% vs rest — mean spend | $104,079 vs $20,639 |
| Top-100k stability under ±25% block perturbation | 90.6% – 98.8% |

The ~0.92 is the true accuracy of a first-principles economic model with **zero leaderboard information** baked into its coefficients — so public and private scores should agree up to sampling noise, and every coefficient survives a "why that value?" question in review.

## Usage

```bash
python3 baep_pipeline.py <data.csv> <submission_template.xlsx>
```

Deterministic. Writes `submission_BAEP.xlsx` (Predictions + Profitability Framework sheets, template-conformant) and `baep_diagnostics.txt`. Tie-break: monotone 1e-6 offset in (score, id) order — below any real gap, never reorders economically distinct members.

## Repository structure

```
baep_pipeline.py        production pipeline (load → prep → score → densify → write)
BAEP_pipeline.ipynb     annotated notebook version
BAEP_DOCUMENTATION.md    full technical write-up
baep_diagnostics.txt     offline validation output
```

## Requirements

```
numpy · pandas · scipy · openpyxl
```
