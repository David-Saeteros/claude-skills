# Psychometrics

## When this applies
- Validating or using psychological scales
- Checking internal consistency (reliability)
- Exploring or confirming latent factor structure
- Assessing construct validity
- Scale development or adaptation
- Working with validated instruments (Big Five, NEO, HEXACO, PHQ, etc.)

---

## Recommended workflow

```
1. Reverse-score items → 2. Internal consistency → 3. EFA (if exploratory) or CFA (if confirmatory) → 4. Compute scale scores → 5. Proceed to substantive analysis
```

Never skip step 1 and 2. If the factor structure is assumed (e.g. using a published instrument), go straight to CFA.

---

## Step 1 — Reverse Scoring

Always verify which items need reversing against the instrument's manual or scoring key before aggregating.

```r
# R
max_val <- 5  # or 7, depending on scale
df <- df %>%
  mutate(
    item3_r = (max_val + 1) - item3,
    item7_r = (max_val + 1) - item7
  )
```

---

## Step 2 — Internal Consistency

### Cronbach's alpha
Acceptable: α ≥ .70; Good: α ≥ .80; Excellent: α ≥ .90
Be cautious of α inflated by large item pools.

```r
# R
library(psych)
alpha(df[, c("item1", "item2", "item3_r", "item4")])
# Check: raw_alpha, std.alpha, and item-total correlations
# Flag items with r.drop < .30 — they may not belong to the scale
```

```python
import pingouin as pg
pg.cronbach_alpha(data=df[['item1','item2','item3_r','item4']])
```

### McDonald's omega (preferred over alpha)
```r
omega(df[, item_cols])  # from psych package
# Reports omega_h (hierarchical) and omega_t (total)
```

---

## Step 3a — Exploratory Factor Analysis (EFA)

Use EFA when: the factor structure is unknown, you're developing a new scale, or you're adapting an instrument for a new population.

### Step 3a.1 — Factorability checks
```r
library(psych)
# Kaiser-Meyer-Olkin (KMO > .60 acceptable, > .80 good)
KMO(df[, item_cols])

# Bartlett's test of sphericity (want p < .05)
cortest.bartlett(df[, item_cols])
```

### Step 3a.2 — How many factors?
```r
# Parallel analysis (most recommended method)
fa.parallel(df[, item_cols], fm = "ml", fa = "fa")

# Scree plot
scree(df[, item_cols])

# Very Simple Structure
vss(df[, item_cols])
```
Parallel analysis is generally preferred. Report and triangulate across methods.

### Step 3a.3 — Run EFA
```r
# Oblique rotation (oblimin) if factors are expected to correlate (usual for personality)
# Orthogonal (varimax) if factors should be independent
efa_result <- fa(df[, item_cols], 
                 nfactors = 3,       # from parallel analysis
                 rotate = "oblimin", 
                 fm = "ml")          # maximum likelihood

print(efa_result$loadings, cutoff = .30)
```

Interpret loadings: ≥ .40 primary, < .30 cross-loading concern. Flag items with no clear primary loading or with high cross-loadings (> .30 on 2+ factors).

---

## Step 3b — Confirmatory Factor Analysis (CFA)

Use CFA when: testing a theoretically specified structure, using a published instrument, or confirming EFA results in a new sample.

```r
library(lavaan)
library(semPlot)

# Define the model
model <- '
  Factor1 =~ item1 + item2 + item3_r + item4
  Factor2 =~ item5 + item6 + item7_r + item8
'

fit <- cfa(model, data = df, estimator = "MLR")  # MLR robust to non-normality
summary(fit, fit.measures = TRUE, standardized = TRUE)

# Path diagram
semPaths(fit, "std", layout = "tree")
```

### Fit indices to report
| Index | Acceptable | Good |
|---|---|---|
| CFI | > .90 | > .95 |
| TLI | > .90 | > .95 |
| RMSEA | < .08 | < .05 |
| SRMR | < .10 | < .08 |

If fit is poor: inspect modification indices, check for correlated residuals, revisit item wording.

```r
modindices(fit, sort = TRUE, maximum.number = 10)
```

---

## Step 4 — Computing Scale Scores

Once factor structure is confirmed:
```r
df <- df %>%
  mutate(
    factor1_score = rowMeans(select(., item1, item2, item3_r, item4), na.rm = TRUE),
    factor2_score = rowMeans(select(., item5, item6, item7_r, item8), na.rm = TRUE)
  )
```

For sum scores vs. mean scores: mean scores are generally preferred as they're interpretable on the original response scale and handle missing data more gracefully.

---

## Convergent & Discriminant Validity

```r
# Correlate your scale scores with theoretically related and unrelated measures
library(psych)
corr.test(df[, c("factor1_score", "factor2_score", "related_scale", "unrelated_scale")])

# Average Variance Extracted (AVE) — from CFA
# AVE > .50 = convergent validity supported
# AVE > squared correlation between factors = discriminant validity supported
```

---

## Common Instruments (recognition only — always verify scoring with the manual)

- **Big Five / NEO-PI-R / BFI:** 5 factors (O, C, E, A, N); various item lengths
- **HEXACO:** 6 factors, adds Honesty-Humility
- **PHQ-9 / PHQ-4:** Depression/anxiety screening; sum score
- **DASS-21:** Depression, Anxiety, Stress; subscale sum scores × 2
- **Rosenberg Self-Esteem Scale:** 10 items, items 2,5,6,8,9 reverse-scored

---

## Reporting Standards

- Report KMO, Bartlett's test, parallel analysis rationale for EFA
- Report all fit indices for CFA
- Report loadings table (EFA) or standardised path coefficients (CFA)
- Report reliability (α and/or ω) per subscale
- Report mean and SD of scale scores
