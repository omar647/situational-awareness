# Situational Awareness Portfolio

Undergraduate markets project from the
[FINM 25000 course](https://markhendricks.github.io/finm-25000/index.html) —
[project brief](https://markhendricks.github.io/finm-25000/projects/undergraduate-markets/briefs/Situational%20Awareness%20Portfolio.html),
[project guidelines](https://markhendricks.github.io/finm-25000/projects/undergraduate-markets/Project%20Guidelines.html).

The Situational Awareness portfolio is long memory, networking, and data-center
infrastructure and short application software. Equal sleeves keep the view
transparent but ignore differences in estimated risk and return. The project
preserves the mandate, builds one stabilized optimizer, and decides whether it
improves on equal sleeves after 2024.

## Setup

```bash
pip install -r requirements.txt
```

Place `project_situational_awareness_20260731.xlsx` in `data/`, then open
`situational_awareness_portfolio.ipynb` and **Run All**.

## Layout

| Path | What it holds |
|---|---|
| `situational_awareness_portfolio.ipynb` | **The submission** — all six sections, executed |
| `data/` | The course workbook |

The notebook is self-contained, as the guidelines require of the group report
("one executed `.ipynb` containing the main analysis"). It imports nothing beyond
NumPy, pandas, SciPy, and Matplotlib: the analysis engine — mandate, optimizer,
rolling backtest, attribution — is defined in its own section up front, so every
number in Sections 1–6 traces to one function in the same file.

## Data

- `data/project_situational_awareness_20260731.xlsx` — Bloomberg gross-dividend
  total-return indexes, daily, 2020-12-31 to 2026-07-31. Sheets: `descriptions`,
  `total return indexes`, `total returns`, `baseline weights`, `data source`,
  `checks`. Covers the 12 mandate names plus SPY, QQQ, and SMH. The `SNDK`
  strategy slot uses `WDC` before Sandisk begins regular-way trading and `SNDK`
  thereafter.

The file is distributed through the course, not hosted on the public site.

## The mandate

400% gross, 0% net. Long weights are nonnegative and sum to +2; short weights are
nonpositive and sum to −2.

| Sleeve | Names |
|---|---|
| Long | SNDK, MU, VRT, LITE, COHR, BE |
| Short | ADBE, CRM, ORCL, NOW, WDAY, MDB |

## The optimization rule

**Position cap.** Maximize the estimated Sharpe ratio with a zero risk-free rate,
using the sample mean and sample covariance of the trailing 756 daily returns,
subject to the two sleeve equalities, the sign constraints, and
`|w_i| <= 0.50`. Solved with SLSQP on the analytic gradient, started from the
equal-sleeve vector as the brief specifies.

## Timing

The most recent 756 daily returns at each estimate date. First estimate at the
December 31, 2024 close, re-estimated at every later month-end through June 30,
2026, each vector held over the following month's daily returns, and one final
estimate on July 31, 2026 for the allocation decision. A weight set at the close
of date *t* first earns the return on the next trading day. Equal sleeves reset on
the same dates. Financing costs, transaction costs, taxes, and market impact are
ignored.

## What is held between rebalances

The **weight vector**, not the positions — the book is marked back to its target
each day, so gross stays at 400% and net at 0% on every date, which is the
exposure the mandate specifies. Holding positions instead lets weights drift with
their own returns: Section 5.1 shows that doing so here pushes gross to 45× and
takes NAV to −$982 million through the July 2026 reversal.

## Results at a glance

- **The mandate (1.1):** equal sleeves is exactly +200% long / −200% short / 400%
  gross / 0% net — **$33,333,333 in each of the twelve names** on $100m NAV.
  Matches the workbook's `baseline weights` and `checks` sheets, and sits inside
  the 50% cap, so it is a feasible starting vector for the optimizer.
- **Why an unconstrained optimizer fails (1.2):** the sleeves are near-collinear
  (short-sleeve mean pairwise correlation **0.54**, NOW–CRM 0.71) while their
  estimated means span 30 points — about **one standard error** of a single
  estimate over 756 days. Correlation-matrix condition number **22.8**.
  Unconstrained tangency puts 70.3% of NAV in one name, runs +60.4% net, and
  places **six of twelve names on the wrong side of the mandate**.
- **The rule (2.1):** position cap — max estimated Sharpe on sample mean and
  sample covariance, every |position| ≤ 50% of NAV. SLSQP converges in 10
  iterations; the mandate holds to 12 decimals; 200 random restarts confirm the
  equal-sleeve start reaches the same optimum (gap 4×10⁻¹⁶).
- **First weights (3.1–3.2):** seven of twelve positions sit at the cap, three at
  exactly zero. Estimated Sharpe 0.7775 vs 0.3923 — all in-sample. The two largest
  changes are **NOW and ORCL, each −33.33% → 0**: ORCL on the mean (best short-sleeve
  Sharpe, 0.8822, so the most expensive to short), NOW on the covariance (average
  within-sleeve correlation **0.6206**, the highest in the portfolio).
- **Rolling weights (4.1–4.2):** all 20 vectors satisfy the mandate and the cap.
  The cap binds at **114 of 240 stock-dates**; paths are near bang-bang. Annualized
  one-way turnover **383.78% of NAV** (1.12× the same rule's 2024 cost); equal
  sleeves is exactly zero.
- **After 2024 (5.1):** **equal sleeves wins.** $100m → **$3.81bn** vs **$2.68bn**.
  Sharpe **2.3313 vs 2.1627**; the optimizer earned 13.4 points less annualized
  return on 5.4 points more volatility, and its worst day was 9.3 points worse.
- **Largest separation (5.2):** November 2025, a 26.14-point gap. The optimizer
  held **zero in LITE**, which returned **+61.32%** — $123.5m of the $200.5m
  difference. Its estimates gave no warning (LITE's trailing Sharpe was 0.886), so
  this was an outcome it could not have anticipated, not a misuse of information.
- **The reversal (6.1):** July 24–29, 2026 cost both books ~two-thirds of NAV
  (−66.91% vs −66.89%, two basis points apart). **Both sleeves lost** — longs fell
  ~25–37% while shorts rallied 20–31%. Zeroing COHR (+$657m) and the NOW short
  (+$504m) were the optimizer's best decisions; capping SNDK cost $87.7m. Its
  concentration put **23.5% of the entire loss in SNDK alone** vs 15.4% for equal
  sleeves.
- **The decision (6.2):** Criterion 1 **passes**, Criterion 2 **fails** (Sharpe
  −0.169 against a +0.25 hurdle). **The allocation stays equal sleeves.** The
  mechanism: trailing Sharpe ranked the long sleeve backwards over the test
  (Spearman **−0.657**, p = 0.16, n = 6) — VRT had the best trailing Sharpe, was
  capped at all 20 dates, and returned the sleeve's worst +113%, while LITE had the
  worst trailing Sharpe and returned +750%.

The headline levels are a property of 400% gross leverage on a fictional panel in
a violent trend — annualized volatility near 150%, so the standard error on a
nineteen-month Sharpe ratio is around 0.8, wider than the gap between the two
books. **The ranking of the two portfolios is the finding; the levels are not an
estimate of anything repeatable.**
