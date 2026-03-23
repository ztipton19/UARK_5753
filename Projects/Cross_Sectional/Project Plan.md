# College Scorecard Project Plan

## Working idea

We want to study whether schools with minority-focused institutional designations or minority-serving student compositions look different on key student outcome metrics.

The central descriptive question is:

How do HBCUs, predominantly Black institutions, and schools serving less-white student bodies differ in completion, earnings, cost, and loan default outcomes?

The central caution is:

This dataset is at the institution level, not the student level. That means we can learn about schools and school environments, but we cannot make direct claims about the experiences of individual Black students or minority students.

## Main variables already in the data

Institutional designation variables:
- `hbcu`
- `predominetly_black_institution`

Student composition variables:
- `n_students_white`
- `n_students_black`
- `n_students_hispanic`
- `n_students_asian`
- `n_students_native`

Main outcome variables of interest:
- `completion_rate`
- `median_earnings_10yr_after`
- `avg_cost_attendance`
- `load_default_rate_3yr`

Important controls we may use later:
- `share_low_income`
- `ownership`
- `admission_rate`
- `online_only`
- `n_students`
- `state` or `region`

## Derived variables we should create

### 1. Minority-designated institution

Create:

`minority_designated = 1` if `hbcu == 1` or `predominetly_black_institution == 1`, otherwise `0`.

Why:
- This gives us one simple institutional flag for schools with explicitly Black-serving designations.
- We can still keep `hbcu` and `predominetly_black_institution` separate later if we want to compare them.

### 2. Non-white-share measure

Create:

`share_nonwhite = 1 - n_students_white`

Why:
- This gives us a cleaner summary of student composition than trying to track several race-share variables at once.
- It helps us distinguish institutional designation from actual student composition.

### 3. Majority-nonwhite institution

Possible binary version:

`majority_nonwhite = 1` if `share_nonwhite >= 0.50`, otherwise `0`.

Why:
- This gives us a school-composition indicator that is intuitive and easy to describe.
- It is better than calling it a "minority_student designation," because the dataset does not label individual students that way.

### 4. High-Black-enrollment institution

Possible exploratory variable:

`high_black_share = 1` if `n_students_black >= 0.50`, otherwise `0`.

Why:
- This may help separate "Black-serving by designation" from "Black-serving by actual enrollment mix."
- It could be especially useful if we want to compare HBCUs to other schools that also enroll many Black students.

## Overall project structure

We should build the project in phases and not jump into final regressions immediately.

### Phase A. Load and clean data

Tasks:
- Load the College Scorecard CSV
- Convert character numeric fields to numeric where needed
- Create `minority_designated`
- Create `share_nonwhite`
- Optionally create `majority_nonwhite`
- Check missingness for all main outcomes

Goal:
- Make sure the variables we want are real, interpretable, and usable before we start modeling

### Phase B. Exploratory visuals and raw comparisons

This phase should answer:

What is the data saying before we control for anything?

Plots to make:
- Scatter or jitter plot of `completion_rate` by `minority_designated`, colored by `minority_designated`
- Scatter or jitter plot of `median_earnings_10yr_after` by `minority_designated`, colored by `minority_designated`
- Scatter or jitter plot of `avg_cost_attendance` by `minority_designated`, colored by `minority_designated`
- Scatter or jitter plot of `load_default_rate_3yr` by `minority_designated`, colored by `minority_designated`

Possible better alternatives to simple scatterplots for a binary x-variable:
- jittered dot plots
- boxplots plus jitter
- violin plots plus jitter

Reason:
- Because `minority_designated` is binary, a raw scatter plot can get stacked and hard to read.

Additional exploratory plots:
- Outcome vs `share_nonwhite`
- Outcome vs `n_students_black`
- histograms of each outcome
- means of each outcome by `hbcu`, `predominetly_black_institution`, and `minority_designated`

Simple descriptive comparisons to report:
- mean `completion_rate` by group
- mean `median_earnings_10yr_after` by group
- mean `avg_cost_attendance` by group
- mean `load_default_rate_3yr` by group

Goal:
- See whether minority-designated institutions appear systematically different on these outcomes

### Phase C. Clarify the substantive story

After the exploratory work, decide which one of these stories is most interesting:

Story 1:
- Minority-designated institutions have different completion and earnings outcomes in raw data

Story 2:
- The differences mostly reflect student composition and economic background

Story 3:
- Institutional designation and actual student composition are not the same thing, and both matter

Story 4:
- Cost and debt-default patterns tell a different story than completion or earnings

Goal:
- Choose one main outcome for the deeper analysis rather than trying to do everything equally

## Candidate main research questions

### Option 1. Completion-focused

Are minority-designated institutions associated with different completion rates than other schools, and how much of that difference is explained by student composition and institutional characteristics?

Why this works:
- clean outcome
- easy interpretation
- matches OLS work from class very well

### Option 2. Earnings-focused

Are minority-designated institutions associated with different median earnings 10 years after entry, and does that relationship change after accounting for composition, cost, and selectivity?

Why this works:
- substantively interesting
- can use a log-linear model because earnings are skewed

### Option 3. Loan-default-focused

Are minority-designated institutions associated with different 3-year loan default rates, and are those differences mostly explained by low-income student share and institution type?

Why this works:
- policy-relevant
- likely to produce a nuanced story
- needs especially careful interpretation

## Modeling roadmap after exploration

We are not executing this yet, but this is the likely sequence once the exploratory phase is done.

### Step 1. Raw difference model

For outcome `Y`:

`Y ~ minority_designated`

Purpose:
- show the unconditional average gap

### Step 2. Separate designation model

`Y ~ hbcu + predominetly_black_institution`

Purpose:
- separate the two designations rather than combining them

### Step 3. Add student-composition controls

Examples:
- `share_nonwhite`
- `n_students_black`
- `share_low_income`

Purpose:
- see whether institutional designation still matters after accounting for who the school serves

### Step 4. Add institutional controls

Examples:
- `ownership`
- `admission_rate`
- `online_only`
- `log(n_students)`
- `state` or `region`

Purpose:
- reduce omitted-variable bias

### Step 5. Add flexibility if needed

If linear controls look too restrictive:
- use bins for `share_nonwhite` or `share_low_income`
- use `poly(...)`
- possibly use `binsreg` for one key continuous relationship

Purpose:
- align with class emphasis on flexibility and avoiding misleading linear assumptions

### Step 6. Interaction model

Potential interactions:
- `minority_designated * share_low_income`
- `hbcu * ownership`
- `minority_designated * share_nonwhite`

Purpose:
- test whether the relationship differs across types of institutions or student bodies

## Important interpretation rules

We should keep these front and center in the final project:

- Use robust standard errors like `vcov = "HC1"` unless we decide clustering is justified
- Describe coefficients as associations, not causal effects
- Do not overstate statistically weak estimates
- Be careful not to imply student-level conclusions from institution-level data
- Distinguish institutional designation from student racial composition

## Recommended immediate next steps

1. Confirm the derived variables we want:
   - `minority_designated`
   - `share_nonwhite`
   - maybe `majority_nonwhite`

2. Decide which exploratory plots we want first:
   - one panel for each outcome by `minority_designated`
   - or one outcome at a time with richer annotations

3. Decide our likely main outcome after the raw comparisons:
   - `completion_rate`
   - `median_earnings_10yr_after`
   - `load_default_rate_3yr`

4. Only after that, start writing code in the Quarto file

## My current recommendation

Use this workflow:

First, do exploratory visuals and mean comparisons for all four outcomes.

Then choose one main outcome for the deeper regression section.

My guess is that the most interesting final project will either be:
- `completion_rate`, because it is easy to interpret and probably central to the school story
- or `log(median_earnings_10yr_after)`, because it gives us a richer and more econometric-feeling analysis
