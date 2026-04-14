# Longitudinal Data Analysis

## When this applies
- Repeated measures on the same individuals over time
- Growth curve modelling
- Mixed models / multilevel models (MLM)
- Pre-post designs with multiple time points
- Experience sampling / diary studies
- Latent growth curve models (LGM)

---

## Key Decisions at the Start

Ask the user:
1. *"How many time points?"* (2 = simple; 3+ = growth models become useful)
2. *"Are time points equally spaced?"* (unequal spacing needs to be modelled)
3. *"Is the research question about change over time, differences between groups in change, or both?"*
4. *"What's the nesting structure?"* (observations within persons; persons within sites/schools?)

---

## Data Format

Longitudinal analysis requires **long format** (one row per observation per time point).

```r
# Wide to long in R
library(tidyverse)
df_long <- df_wide %>%
  pivot_longer(cols = starts_with("score_t"),
               names_to = "time",
               names_prefix = "score_t",
               values_to = "score") %>%
  mutate(time = as.numeric(time))
```

```python
# Wide to long in Python
df_long = pd.wide_to_long(df_wide, 
                           stubnames=['score_t'], 
                           i='id', 
                           j='time').reset_index()
```

---

## Simple Pre-Post (2 time points)

For two time points with a control group, use paired t-test or repeated-measures ANOVA. For more control over covariates, use ANCOVA with baseline as covariate (preferred over change scores).

```r
# ANCOVA approach (recommended)
model <- lm(post_score ~ pre_score + group, data = df)
summary(model)
```

---

## Linear Mixed Models (LMM)

The workhorse for longitudinal data with 3+ time points. Handles:
- Missing data (under MAR assumption)
- Unequal spacing
- Individual variation in trajectories (random slopes)

```r
library(lme4)
library(lmerTest)  # adds p-values

# Random intercept model (individuals differ in baseline)
m1 <- lmer(outcome ~ time + group + (1 | id), data = df_long)

# Random slope model (individuals differ in rate of change)
m2 <- lmer(outcome ~ time + group + (1 + time | id), data = df_long)

# Group × time interaction (does trajectory differ by group?)
m3 <- lmer(outcome ~ time * group + (1 + time | id), data = df_long)

summary(m3)
anova(m1, m2, m3)  # model comparison
```

```python
import statsmodels.formula.api as smf

model = smf.mixedlm("outcome ~ time * group", 
                     data=df_long, 
                     groups=df_long["id"],
                     re_formula="~time").fit()
print(model.summary())
```

### Interpreting random effects
```r
# Variance components
VarCorr(m2)
# Random slopes plot
library(lattice)
dotplot(ranef(m2, condVar = TRUE))
```

---

## Growth Curve Models

### Polynomial growth
When trajectories are non-linear:
```r
# Quadratic growth
df_long <- df_long %>% mutate(time2 = time^2)
m_quad <- lmer(outcome ~ time + time2 + group + (1 + time | id), data = df_long)

# Compare to linear
anova(m_linear, m_quad)
```

### Centring time
Always centre time at a meaningful point (e.g., first measurement = 0, or grand mean). This affects interpretation of intercepts.

```r
df_long <- df_long %>% 
  mutate(time_c = time - min(time))  # centre at first wave
```

---

## Latent Growth Curve Models (LGM)

Use when you want to model individual differences in growth as latent variables, or when integrating into a larger SEM framework.

```r
library(lavaan)

model_lgm <- '
  # Intercept factor
  i =~ 1*t1 + 1*t2 + 1*t3 + 1*t4
  # Slope factor
  s =~ 0*t1 + 1*t2 + 2*t3 + 3*t4

  # Means (growth parameters)
  i ~ 1
  s ~ 1

  # Predict growth from covariate
  i ~ group
  s ~ group
'

fit_lgm <- growth(model_lgm, data = df_wide)
summary(fit_lgm, fit.measures = TRUE, standardized = TRUE)
```

Use the same fit indices as CFA (CFI, TLI, RMSEA, SRMR).

---

## Missing Data in Longitudinal Studies

### Types of missingness
- **MCAR** (Missing Completely At Random): safe to use listwise deletion
- **MAR** (Missing At Random): LMM with ML estimation handles this; multiple imputation is an alternative
- **MNAR** (Missing Not At Random): requires sensitivity analyses or specialised models

LMM with REML/ML handles MAR data better than listwise deletion — this is one of its key advantages.

```r
# Check missingness patterns
library(naniar)
vis_miss(df_wide)
gg_miss_upset(df_wide)  # combinations of missing variables
```

---

## Multilevel / Nested Data

When individuals are nested in groups (students in classrooms, employees in firms):

```r
# Three-level model: observations within persons within groups
m_3level <- lmer(outcome ~ time + group_level_predictor + 
                   (1 + time | person_id) + 
                   (1 | group_id), 
                 data = df_long)

# ICC — how much variance is at each level
library(performance)
icc(m_3level)
```

If ICC at the group level is > .05, multilevel modelling is warranted.

---

## Visualisation

```r
library(ggplot2)

# Spaghetti plot (individual trajectories)
ggplot(df_long, aes(x = time, y = outcome, group = id)) +
  geom_line(alpha = 0.2) +
  stat_summary(aes(group = 1), fun = mean, geom = "line", 
               color = "red", linewidth = 1.5) +
  theme_minimal()

# By group
ggplot(df_long, aes(x = time, y = outcome, group = id, color = group)) +
  geom_line(alpha = 0.2) +
  stat_smooth(aes(group = group), method = "lm", se = TRUE) +
  theme_minimal()
```

---

## Reporting Standards

- Report fixed effects (β, SE, t, p) and random effects (variance components)
- Report model comparison results (AIC, BIC, likelihood ratio test)
- Report ICC values for multilevel models
- Describe missing data handling explicitly
- Include trajectory plots
