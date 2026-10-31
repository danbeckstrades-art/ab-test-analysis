# A/B Test Analysis: Moving the First Gate (Cookie Cats)

I analyzed a mobile-game A/B test with the plan fixed in advance: a power analysis, experiment validation, and a decision rule set before looking at outcomes. I also simulated how peeking and multiple metrics inflate false positives.

## Question
Cookie Cats is a mobile puzzle game. The company moved the first progress gate from level 30 (control) to level 40 (treatment). Does that change player retention?

## Data
[Mobile Games A/B Testing - Cookie Cats](https://www.kaggle.com/datasets/mursideyarkin/mobile-games-ab-testing-cookie-cats) (Kaggle, Mursideyarkin): 90,189 players with `version`, `sum_gamerounds`, `retention_1`, `retention_7`. The CSV is not included; save it as `data/cookie_cats.csv`.

## Design (I fixed this before looking at outcomes)
- Primary metric: 7-day retention. Secondary: 1-day retention. Guardrail: game rounds played.
- Two-sided test, alpha = 0.05, power = 0.80.
- Smallest effect worth acting on: 0.5 percentage points (an assumed placeholder, not a business-derived number).

## Validation and power
- No missing values or duplicate users.
- **Sample ratio mismatch:** 44,700 vs 45,489 users (p = 0.0086). It fails at 0.05 but not at the stricter 0.001 threshold often used for SRM alarms, and the cause cannot be determined from this dataset.
- With about 45,000 users per group and 18.6% baseline retention, the minimum detectable effect is 0.73 percentage points. The experiment is well powered for effects of about 1 point or more and underpowered for 0.5 points or less.

## Results

| Metric | gate_30 (control) | gate_40 (treatment) | Difference (95% CI) | p |
|---|---|---|---|---|
| 7-day retention (primary) | 19.02% | 18.20% | -0.82 pp (-1.33 to -0.31) | 0.0016 |
| 1-day retention (secondary) | 44.82% | 44.23% | -0.59 pp (-1.24 to +0.06) | 0.074 |

- Moving the gate to level 40 **reduced** 7-day retention by about 0.8 percentage points (4.3% relative). The primary result clears a Bonferroni threshold of 0.025 for the two retention metrics.
- Engagement (rounds played) shows no clear change. Medians are 17 vs 16 rounds, and three different tests give p between 0.05 and 0.07. One control-group user logged 49,854 rounds (probably a bot or logging error); I kept the user and capped values at the 99.9th percentile for mean comparisons.
- **My recommendation:** keep the gate at level 30, because the change lowered 7-day retention.

## Why the analysis was planned in advance
I simulated A/A tests with no true effect: a single planned look gives 5.0% false positives, peeking 10 times and stopping at the first p < 0.05 gives 19.2%, and testing 10 metrics gives at least one false "win" 40.3% of the time (4.9% with a Bonferroni correction).

## Limitations
- The unexplained sample ratio mismatch could bias the estimate in an unknown direction.
- Retention is not revenue. Gates also drive purchases, and the data has no purchase information.
- The 0.5 pp action threshold is an assumption, and the confidence interval straddles it, so the drop is likely at least 0.3 pp but not certainly above 0.5 pp.
- The data cannot show why the later gate hurt retention.
- The engagement test was not specified in advance, so all three tests are reported.

## Reproduce
    pip install -r requirements.txt

Place the CSV at `data/cookie_cats.csv` and run `notebooks/01_analysis.ipynb`. Random seeds are fixed at 42.

