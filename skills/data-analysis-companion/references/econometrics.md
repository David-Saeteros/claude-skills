# Econometrics

## When this applies
- Causal inference from observational data
- Panel data (multiple observations per unit over time)
- Difference-in-differences (DiD) designs
- Instrumental variables (IV)
- Regression discontinuity (RD)
- Heteroskedasticity-robust inference

---

## Fundamental Principle

Always be clear about whether the goal is **prediction** or **causal inference**. Most econometric methods are designed for the latter. If the user wants causal claims, the identification strategy is the most important thing to establish before any estimation.

Ask: *"Are you trying to estimate a causal effect, or predict an outcome?"*

---

## OLS Regression (baseline)

```r
library(fixest)  # preferred for econometrics in R
library(sandwich)
library(lmtest)

# Basic OLS with robust standard errors
model <- feols(outcome ~ treatment + covariate1 + covariate2, 
               data = df, 
               vcov = "hetero")  # HC1 robust SEs
summary(model)
```

```python
import statsmodels.formula.api as smf

model = smf.ols('outcome ~ treatment + covariate1 + covariate2', data=df).fit(
    cov_type='HC1'  # heteroskedasticity-robust SEs
)
print(model.summary())
```

**Always use robust standard errors in applied econometrics unless you have a specific reason not to.**

---

## Panel Data

### Fixed Effects vs. Random Effects

| Method | When to use | Key assumption |
|---|---|---|
| Fixed Effects (FE) | Unit-specific confounders likely | Unobserved heterogeneity correlated with X |
| Random Effects (RE) | Unobserved heterogeneity uncorrelated with X | Stronger, often violated |
| Pooled OLS | Benchmark only | No unobserved heterogeneity |

Run the **Hausman test** to choose between FE and RE: if p < .05, use FE.

```r
# R with fixest — two-way FE (unit + time)
model_fe <- feols(outcome ~ treatment | unit_id + year, 
                  data = panel_df,
                  vcov = ~unit_id)  # cluster by unit

# Hausman test
library(plm)
fe_model <- plm(outcome ~ treatment + covariate, data = panel_df, 
                index = c("unit_id", "year"), model = "within")
re_model <- plm(outcome ~ treatment + covariate, data = panel_df,
                index = c("unit_id", "year"), model = "random")
phtest(fe_model, re_model)
```

```python
from linearmodels import PanelOLS, RandomEffects
import statsmodels.api as sm

panel_df = df.set_index(['unit_id', 'year'])

# Fixed effects
mod_fe = PanelOLS.from_formula('outcome ~ treatment + EntityEffects + TimeEffects', 
                                 data=panel_df)
res_fe = mod_fe.fit(cov_type='clustered', cluster_entity=True)
print(res_fe.summary)
```

---

## Difference-in-Differences (DiD)

### Classic 2x2 DiD
```r
model_did <- feols(outcome ~ treat * post | unit_id + time, 
                   data = df,
                   vcov = ~unit_id)
summary(model_did)
```
The coefficient on `treat:post` is the DiD estimate.

### Parallel trends assumption
This is the critical assumption — treated and control groups would have followed the same trend absent treatment. Test with an **event study plot**:

```r
# Event study
model_es <- feols(outcome ~ i(time, treat, ref = -1) | unit_id + time,
                  data = df, vcov = ~unit_id)
iplot(model_es, main = "Event Study")
```
Pre-treatment coefficients should be close to zero and statistically insignificant.

### Staggered DiD (multiple treatment timing)
Use Callaway & Sant'Anna or Sun & Abraham estimators — the classic TWFE estimator is biased with heterogeneous treatment effects:

```r
library(did)
out <- att_gt(yname = "outcome",
              gname = "first_treat_year",
              idname = "unit_id",
              tname = "year",
              data = df)
aggte(out, type = "dynamic")  # event study
```

---

## Instrumental Variables (IV)

Use when treatment is endogenous (correlated with the error term). IV requires:
1. **Relevance:** instrument strongly predicts treatment (F-stat > 10, ideally > 15)
2. **Exclusion restriction:** instrument affects outcome ONLY through treatment

```r
# 2SLS with fixest
model_iv <- feols(outcome ~ 1 | covariate | treatment ~ instrument,
                  data = df, vcov = ~cluster_var)

# First stage F-stat — critical for weak instruments
fitstat(model_iv, "ivf")
```

**Always report the first-stage F-statistic. Weak instruments (F < 10) produce unreliable estimates.**

---

## Regression Discontinuity (RD)

Use when treatment assignment is determined by crossing a known threshold on a running variable.

```r
library(rdrobust)

# Main RD estimate
rd_out <- rdrobust(y = df$outcome, x = df$running_var, c = threshold)
summary(rd_out)

# Plot
rdplot(y = df$outcome, x = df$running_var, c = threshold)
```

Key checks:
- **Density test** (McCrary): no manipulation of the running variable
- **Bandwidth sensitivity**: results should be robust to bandwidth choices
- **Placebo thresholds**: no discontinuity at other values

---

## Heteroskedasticity & Clustering

Standard errors choice:
- **HC1/HC3 robust SEs:** when errors have non-constant variance (almost always use these)
- **Clustered SEs:** when observations within groups are correlated (panel data, students in schools, etc.)
- **Two-way clustered:** cluster by two dimensions (e.g. firm and year)

```r
# fixest handles all of these cleanly
feols(outcome ~ x, data = df, vcov = "hetero")          # robust
feols(outcome ~ x, data = df, vcov = ~cluster_var)      # one-way cluster
feols(outcome ~ x, data = df, vcov = ~unit + time)      # two-way cluster
```

---

## Reporting Standards

- Report coefficients, robust/clustered SEs, t-statistics, and p-values
- Always specify the SE type used
- Report R², within-R² for FE models
- For IV: first-stage F-stat, Sargan/Hansen J test if overidentified
- For DiD: include event study plot
- For RD: include RD plot, report bandwidth choice and robustness checks
