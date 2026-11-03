# Causal Logic and DAGs: Diagnosing Bad Controls

**ECON 5200 · Causal Machine Learning & Applied Analytics · Lab 10**

## Objective

I diagnosed regressions that went wrong because of the controls chosen, and I used a causal graph (DAG) to decide which variables to adjust for instead of choosing them by hand.

## Methodology

- Ran a regression where the control C was caused by both the treatment and the outcome (a collider), and compared it with the regression without C. Then I held C nearly fixed to show the association it creates.
- Ran a regression with a control W measured before treatment that was a collider of two unobserved causes (M-bias), to show that "only control for pre-treatment variables" is not enough.
- Separated the total effect of a training program from its direct effect by adding and removing the mediator (skill).
- Compared the LaLonde (1986) NSW experimental benchmark with an observational estimate that uses CPS men as the comparison group, before and after adjusting for eight pre-treatment covariates.
- Used a `dag_utils.py` module that finds backdoor paths, checks whether a path is blocked, and returns the minimal valid adjustment sets for a DAG.
- Asked an AI which controls to use in a simulated job-training study, checked its answer against the study's DAG with `dag_utils`, and revised my prompt once.

## Key Findings

**Controlling for a collider made a null effect look significant.** The true effect was zero. Adding C moved the estimate from −0.012 to −0.593 (p < 0.001) and raised R-squared from 0.000 to 0.585. Holding C nearly fixed, the correlation between D and Y went from −0.012 to −0.563.

**A pre-treatment control can still add bias.** Controlling for W, which was measured before treatment, gave −0.201 against a true effect of zero.

**Total and direct effects answer different questions.** Without the mediator the estimate was 3.978 (true total effect 4.0). With it, the estimate was 0.988 (true direct effect 1.0). The total effect is the one that answers whether the program is worth funding.

**Adjustment moved the LaLonde estimate toward the benchmark but did not reach it.** The experimental benchmark was $1,794. The observational estimate was −$8,498 before adjustment and $699 after, still about $1,095 short. Without the experiment there would be no way to tell from the regression output alone.

**The AI's first adjustment set included a collider.** It proposed `prior_earnings` and `risk_score`, which left 1 of 2 backdoor paths open and gave 3.622 against a true effect of 4.0. After I added to the prompt what the risk score is computed from, it proposed only `prior_earnings`, which blocked both paths and gave 4.047.

## Repository Contents

| File | Description |
|---|---|
| `lab-ch10-diagnostic.ipynb` | Full lab notebook with rendered output |
| `src/dag_utils.py` | Backdoor paths, path blocking, adjustment-set validity and minimal adjustment sets for a DAG |

## Data

- Diagnoses 1–3 and the AI section use simulated data with a fixed seed (`RANDOM_STATE = 42`).
- The LaLonde case uses the Dehejia and Wahba (1999) files served by NBER: `nsw_dw.dta` (185 treated, 260 control) and `cps_controls.dta` (15,992 CPS men).
