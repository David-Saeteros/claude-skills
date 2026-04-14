# Survey & Social Science Analysis

## When this applies
- Cross-sectional survey data
- Observational studies
- Likert scale instruments
- Demographic breakdowns
- Group comparison designs
- Experimental / quasi-experimental data

---

## Likert Data — Key Decisions

Before any analysis, establish:
1. **Are items part of a scale or standalone?** If scale → go to psychometrics.md first for reliability/factor checks
2. **Treat as ordinal or continuous?** For 5-point scales: ordinal methods are technically correct; for 7-point and aggregated scale scores, treating as continuous is widely accepted in practice. Be transparent about the choice.
3. **Any reverse-scored items?** Always check and recode before aggregating.

**Reverse scoring in R:**
```r
df <- df %>%
  mutate(item3_r = (max_score + 1) - item3,
         item7_r = (max_score + 1) - item7)
```

**Reverse scoring in Python:**
```python
max_score = 5
df['item3_r'] = (max_score + 1) - df['item3']
```

---

## Descriptive Analysis for Survey Data

```r
# R — grouped descriptives
library(tidyverse)

df %>%
  group_by(group_variable) %>%
  summarise(
    n = n(),
    mean = mean(outcome, na.rm = TRUE),
    sd = sd(outcome, na.rm = TRUE),
    median = median(outcome, na.rm = TRUE)
  )

# Frequency tables for categorical variables
df %>% count(categorical_var) %>% mutate(pct = n / sum(n) * 100)
```

```python
# Python
df.groupby('group_variable')['outcome'].agg(['count','mean','std','median'])
df['categorical_var'].value_counts(normalize=True) * 100
```

---

## Group Comparisons

### Choosing the right test

| Situation | Test |
|---|---|
| 2 groups, continuous outcome, normal | Independent t-test |
| 2 groups, continuous outcome, non-normal | Mann-Whitney U |
| 2+ groups, continuous outcome, normal | One-way ANOVA |
| 2+ groups, non-normal | Kruskal-Wallis |
| Paired / repeated measures, 2 time points | Paired t-test or Wilcoxon |
| Categorical outcome | Chi-square or Fisher's exact |

Always check normality (Shapiro-Wilk for n < 50, visual inspection + skewness for larger samples) and homogeneity of variance (Levene's test) before choosing.

**R example — full comparison pipeline:**
```r
library(tidyverse)
library(rstatix)

# Normality check
df %>% 
  group_by(group) %>%
  shapiro_test(outcome)

# Levene's test
df %>% levene_test(outcome ~ group)

# t-test (adjust var.equal based on Levene result)
t.test(outcome ~ group, data = df, var.equal = TRUE)

# Effect size
df %>% cohens_d(outcome ~ group)
```

**Python example:**
```python
from scipy import stats
import pingouin as pg

# Normality
stat, p = stats.shapiro(df[df['group']=='A']['outcome'])

# t-test
t, p = stats.ttest_ind(
    df[df['group']=='A']['outcome'],
    df[df['group']=='B']['outcome']
)

# Effect size
pg.ttest(df[df['group']=='A']['outcome'], df[df['group']=='B']['outcome'])
```

### Post-hoc tests (for ANOVA)
```r
# R
library(emmeans)
model <- aov(outcome ~ group, data = df)
emmeans(model, pairwise ~ group, adjust = "bonferroni")
```

---

## Regression for Survey Data

### Linear regression checklist
Before interpreting results, verify:
- [ ] Linearity (residual plots)
- [ ] Normality of residuals (Q-Q plot)
- [ ] Homoscedasticity (scale-location plot)
- [ ] No multicollinearity (VIF < 5, ideally < 3)
- [ ] Influential cases (Cook's D)

```r
# R — regression with diagnostics
model <- lm(outcome ~ predictor1 + predictor2 + covariate, data = df)
summary(model)
par(mfrow = c(2,2)); plot(model)  # diagnostic plots

# VIF
library(car)
vif(model)
```

```python
import statsmodels.formula.api as smf
from statsmodels.stats.outliers_influence import variance_inflation_factor

model = smf.ols('outcome ~ predictor1 + predictor2 + covariate', data=df).fit()
print(model.summary())

# VIF
vif_data = pd.DataFrame()
vif_data['feature'] = X.columns
vif_data['VIF'] = [variance_inflation_factor(X.values, i) for i in range(len(X.columns))]
```

### Logistic regression (binary outcome)
```r
model <- glm(outcome ~ predictor1 + predictor2, data = df, family = binomial)
summary(model)
exp(coef(model))  # odds ratios
exp(confint(model))  # CI for odds ratios
```

---

## Experimental & Quasi-Experimental Data

### Pre-post designs
Always check for baseline equivalence before analysing change scores. If groups differ at baseline, use ANCOVA with baseline as covariate rather than a simple change score analysis.

```r
# ANCOVA
model <- lm(post_score ~ pre_score + group, data = df)
summary(model)
```

### Interaction effects
When testing whether an effect differs by group (moderation):
```r
model <- lm(outcome ~ predictor * moderator, data = df)
summary(model)

# Visualise the interaction
library(interactions)
interact_plot(model, pred = predictor, modx = moderator)
```

---

## Visualisation Defaults for Survey Data

**R (ggplot2):**
```r
# Distribution
ggplot(df, aes(x = outcome)) + 
  geom_histogram(bins = 30) + 
  theme_minimal()

# Group comparison
ggplot(df, aes(x = group, y = outcome)) +
  geom_boxplot() +
  geom_jitter(width = 0.1, alpha = 0.3) +
  theme_minimal()
```

**Python (seaborn):**
```python
import seaborn as sns
import matplotlib.pyplot as plt

sns.histplot(df['outcome'], bins=30)
sns.boxplot(x='group', y='outcome', data=df)
```

---

## Reporting Standards

For social science papers:
- Report means, SDs, and sample sizes for all groups
- Report test statistic, degrees of freedom, p-value, and effect size
- Use APA format: *t*(58) = 2.34, *p* = .023, *d* = 0.61
- For regression: report β (unstandardised), SE, standardised β, t, p, and R²
- Flag any violations of assumptions and how they were handled
