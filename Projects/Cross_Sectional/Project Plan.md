# College Scorecard Project Plan

## Working idea

We want to study whether schools with minority-focused institutional designations or minority-serving student compositions look different on key student outcome metrics.

The central descriptive question is:

How do HBCUs, predominantly Black institutions, and schools serving less-white student bodies differ in completion, earnings, cost, and loan default outcomes?

The central caution is:

This dataset is at the institution level, not the student level. That means we can learn about schools and school environments, but we cannot make direct claims about the experiences of individual Black students or minority students.

## Main research focus

Primary outcome: `completion_rate` — clean, intuitive, matches OLS work from class.

Sub-questions we also want to explore descriptively:
- `median_earnings_10yr_after` — do minority-designated schools show different post-graduation earnings?
- `loan_default_rate_3yr` — are default patterns different, and is that explained by income composition?

These sub-questions get exploratory treatment (summary stats, boxplots, raw comparisons) but the deeper regression modeling focuses on completion rate.

## Variable renaming

The raw data has some misleading column names. We rename on load:

| Original name | New name | Reason |
|---|---|---|
| `predominetly_black_institution` | `predominantly_black_institution` | Typo |
| `load_default_rate_3yr` | `loan_default_rate_3yr` | Typo |
| `n_students_white` | `share_white` | These are proportions (0–1), not counts |
| `n_students_black` | `share_black` | Same |
| `n_students_hispanic` | `share_hispanic` | Same |
| `n_students_asian` | `share_asian` | Same |
| `n_students_native` | `share_native` | Same |

## Main variables (after renaming)

Institutional designation variables:
- `hbcu`
- `predominantly_black_institution`

Student composition variables (proportions, 0–1):
- `share_white`
- `share_black`
- `share_hispanic`
- `share_asian`
- `share_native`

Race-specific completion rates (available in data):
- `completion_rate_white`
- `completion_rate_black`
- `completion_rate_hispanic`
- `completion_rate_asian`
- `completion_rate_native`

Main outcome variables of interest:
- `completion_rate` (primary)
- `median_earnings_10yr_after` (sub-question)
- `avg_cost_attendance` (exploratory)
- `loan_default_rate_3yr` (sub-question)

Important controls we may use later:
- `share_low_income`
- `ownership`
- `admission_rate`
- `n_students`
- `state` or `region`

## Derived variables we should create

### 1. Combined designation flag

Create:

`hbcu_or_pbi = 1` if `hbcu == 1` or `predominantly_black_institution == 1`, otherwise `0`.

Why:
- This gives us one simple institutional flag for schools with explicitly Black-serving designations.
- The name is explicit about what it combines, unlike the more ambiguous label "minority-designated."
- We can still keep `hbcu` and `predominantly_black_institution` separate later if we want to compare them.

### 2. Non-white-share measure

Create:

`share_nonwhite = share_black + share_hispanic + share_asian + share_native`

Do **not** use `1 - share_white`. The five race-share columns are not exhaustive — their sum averages about 0.87 across schools, and most schools are below 0.95. The remainder includes multiracial and unclassified students. Using `1 - share_white` would fold those students into "nonwhite," making the variable hard to interpret.

The sum-of-observed-nonwhite approach is not perfect either (it excludes multiracial/unknown), but it is more defensible and interpretable. We should note this limitation when using the variable.

Why:
- This gives us a cleaner summary of student composition than tracking several race-share variables individually.
- It helps us distinguish institutional designation from actual student composition.

### 3. Majority-nonwhite institution

Possible binary version:

`majority_nonwhite = 1` if `share_nonwhite >= 0.50`, otherwise `0`.

Why:
- This gives us a school-composition indicator that is intuitive and easy to describe.
- It is better than calling it a "minority_student designation," because the dataset does not label individual students that way.

### 4. High-Black-enrollment institution

Possible exploratory variable:

`high_black_share = 1` if `share_black >= 0.50`, otherwise `0`.

Why:
- This may help separate "Black-serving by designation" from "Black-serving by actual enrollment mix."
- It could be especially useful if we want to compare HBCUs to other schools that also enroll many Black students.

## Overall project structure

We should build the project in phases and not jump into final regressions immediately.

### Phase A. Load, clean, and restrict data

Tasks:
- Load the College Scorecard CSV
- Rename misleading columns (see renaming table above)
- Convert character numeric fields to numeric where needed
- Drop online-only institutions (`online_only == 1`)
- Check and document missingness for all main outcomes — note whether HBCUs are missing data at different rates than other schools (e.g., earnings data)
- Decide and document the analytic sample strategy (choose one):
  - **Option A — Common sample**: drop rows missing any of the four main outcomes, use the same schools for all comparisons. Cleaner for cross-outcome comparisons; loses observations.
  - **Option B — Outcome-specific samples**: each outcome uses all non-missing rows for that variable. Maximizes N per analysis; sample sizes will differ across tables and figures, which must be reported clearly on each.
  - Recommendation: use Option A for the balance table and regression, use Option B for descriptive boxplots with n printed on each panel.
- Report characteristics of dropped vs. retained schools to check for systematic differences
- Create `hbcu_or_pbi`
- Create `share_nonwhite`
- Optionally create `majority_nonwhite` and `high_black_share`
- Report final sample size and how many rows were dropped at each step

Goal:
- Make sure the variables we want are real, interpretable, and usable before we start modeling
- Document sample restrictions transparently so readers know what we kept and why

Note on group sizes:
- In this dataset: 89 HBCUs, 20 PBIs, 0 schools coded as both — the two designations are disjoint. This means the `hbcu_or_pbi` combined flag has ~109 schools, and the separate-designation model (Step 2) is more defensible than initially thought (no collinearity concern). The real small-sample issue is the PBI group at n=20, which will produce wide confidence intervals. Report group sizes on every table and treat PBI estimates with appropriate caution.

### Phase B. Exploratory visuals and raw comparisons

This phase should answer:

What is the data saying before we control for anything?

#### B1. Balance table (Table 1)

Produce a grouped comparison table showing how HBCU/PBI schools differ from other schools on both outcomes and covariates.

Continuous variables panel (mean, sd, by group):
- All four outcomes: `completion_rate`, `median_earnings_10yr_after`, `avg_cost_attendance`, `loan_default_rate_3yr`
- `share_low_income`, `share_black`, `share_nonwhite`, `admission_rate`, `n_students`

Categorical variables panel (% by group, separate from the continuous panel):
- `ownership` — report sector shares (% public, % private nonprofit, % for-profit) rather than a mean
- Any other binary/categorical covariates

Columns: non-HBCU/PBI | HBCU/PBI | difference (for continuous); non-HBCU/PBI % | HBCU/PBI % (for categorical)
Report n per group in the header row.

This is "Table 1" in the paper — it shows readers whether the two groups are comparable on background characteristics before any outcomes are discussed.

#### B2. Boxplots for each outcome by `hbcu_or_pbi`

Use boxplots for all four outcomes:
- `completion_rate` by `hbcu_or_pbi`
- `log(median_earnings_10yr_after)` by `hbcu_or_pbi` — log because earnings are right-skewed
- `avg_cost_attendance` by `hbcu_or_pbi`
- `loan_default_rate_3yr` by `hbcu_or_pbi`

Boxplots are the right choice here because `hbcu_or_pbi` is binary and the sample sizes are unbalanced.

#### B3. Race-specific completion rate comparison

Plot a boxplot of `completion_rate_black` by `hbcu_or_pbi`. This is a more targeted descriptive comparison of institution-level Black completion rates — it is still an institution-level subgroup average with varying subgroup sizes and likely missingness, not a near-student-level comparison. Report the n of schools with non-missing `completion_rate_black` per group.

#### B4. Correlation check among key predictors

Produce a correlation table for: `hbcu`, `predominantly_black_institution`, `share_nonwhite`, `share_black`, `share_low_income`, `admission_rate`.

Why: `hbcu` and `predominantly_black_institution` are disjoint (no overlap in this dataset), so the concern is not collinearity between them. The real correlation to watch is between `share_black` and `hbcu_or_pbi`, and between `share_low_income` and the designation variables. Knowing the correlation structure before regression prevents over-controlling and aids interpretation.

#### B5. Additional exploratory plots

- `completion_rate` vs `share_nonwhite` (scatter, continuous x)
- `completion_rate` vs `share_black` (scatter)
- Histograms of each outcome — use log scale for earnings
- Mean comparisons by `hbcu`, `predominantly_black_institution`, and `hbcu_or_pbi` separately

Goal:
- See whether HBCU/PBI institutions appear systematically different on outcomes and covariates

### Phase C. Interpret the exploratory findings

After the exploratory work, identify which story the data is telling:

Story 1:
- Minority-designated institutions have different completion outcomes in raw data

Story 2:
- The differences mostly reflect student composition and economic background

Story 3:
- Institutional designation and actual student composition are not the same thing, and both matter

Use Phase C to write up the narrative that connects the summary stats and plots to the regression analysis that follows.

## Main research question

Are minority-designated institutions associated with different completion rates than other schools, and how much of that difference is explained by student composition and institutional characteristics?

Why this works:
- clean outcome
- easy interpretation
- matches OLS work from class very well

Sub-questions (exploratory, lighter treatment):
- Do these schools also show different median earnings 10 years after entry?
- Are 3-year loan default rates different, and is that explained by income composition?

## Modeling roadmap after exploration

We are not executing this yet, but this is the likely sequence once the exploratory phase is done.

### Step 1. Raw difference model

For outcome `Y`:

`Y ~ hbcu_or_pbi`

Purpose:
- show the unconditional average gap

### Step 2. Separate designation model

`Y ~ hbcu + predominantly_black_institution`

Purpose:
- separate the two designations rather than combining them

Note: `hbcu` and `predominantly_black_institution` are disjoint in this dataset (89 HBCUs, 20 PBIs, 0 overlap), so this model is well-identified. The main caution here is the PBI group at n=20 — report its confidence interval width and be explicit that PBI estimates are exploratory.

### Step 3. Add student-composition controls

Examples:
- `share_nonwhite`
- `share_black`
- `share_low_income`

Purpose:
- see whether institutional designation still matters after accounting for who the school serves

### Step 4. Add institutional controls

Examples:
- `ownership`
- `admission_rate`
- `log(n_students)`
- `region` (broader) or `state` fixed effects (more absorbing)

State fixed effects absorb a lot of variation (state funding, local labor markets) and provide stronger control. Run with and without state FE and report both as a sensitivity check.

Purpose:
- reduce omitted-variable bias
- see how sensitive the designation coefficient is to geographic controls

### Step 5. (Stretch) Add flexibility or interactions if needed

If time and scope allow:
- Bins or `poly(...)` for `share_nonwhite` or `share_low_income` if linear looks too restrictive
- Interactions like `hbcu_or_pbi * share_low_income` or `hbcu * ownership`

Purpose:
- stretch goal only — prioritize getting Steps 1–4 solid first

## Important interpretation rules

We should keep these front and center in the final project:

- Use robust standard errors like `vcov = "HC1"` unless we decide clustering is justified
- Describe coefficients as associations, not causal effects
- Do not overstate statistically weak estimates
- Be careful not to imply student-level conclusions from institution-level data
- Distinguish institutional designation from student racial composition
- When reporting completion rate gaps, translate coefficients to interpretable units: "a X percentage point difference" rather than just the raw number
- Consider expressing gaps relative to the standard deviation of completion rate for effect size context
- For earnings, report in log-points but also convert to approximate percentage difference for readers

## Mechanistic framing

We are not testing causality, but the interpretation section should briefly address the possible mechanisms that could explain any associations we find:
- **Resources**: HBCUs may be less funded, affecting completion
- **Mission and peer effects**: students at HBCUs may have stronger peer networks and institutional support
- **Selection**: students who choose HBCUs may differ systematically from observably similar students at other schools

These are not claims — they are the lens through which readers should understand what "association with hbcu_or_pbi" might mean in practice.

## Immediate next steps

1. Start the Quarto file with Phase A: load, rename, restrict, and report sample sizes
2. Phase B: summary statistics table, then boxplots for all four outcomes
3. Phase C: write up what the exploration shows
4. Begin regression modeling (Steps 1–4) focused on `completion_rate`
