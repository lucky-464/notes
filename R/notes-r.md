# R Notes: First Contact

Notes from the Digital Adoption project. Written as I learned R for the first time.

---

## 1. Install Once, Load Every Session

### The confusion I had

I ran `install.packages(c("fixest", "dplyr", "readr"))`. Then at the top of every script, I saw `library(fixest)`, `library(dplyr)`, `library(readr)`. I thought we were re-installing.

We were not.

### What is actually happening

**Install** puts the package on your hard drive. One-time thing. The package now exists as a folder full of files, forever, until you uninstall it.

**Library** wakes the package up for the current R session. R does not auto-load every installed package at startup. It loads a small core set and leaves the rest dormant until you ask for them.

So every fresh R session, you have to re-run `library(...)` at the top of your script. That is why it looks like repeating the install. It is not.

### The analogy

Install puts books on your shelf. `library()` pulls a book off the shelf and opens it on your desk. You can only read from the desk. When you leave the room and come back, the book is back on the shelf. Pull it off again.

### Why three packages

- **fixest** gives `feols()` — the fixed-effects estimator. This is R's equivalent of Python's `PanelOLS`.
- **dplyr** gives `%>%` (the pipe) and `filter()`. Data wrangling.
- **readr** gives `read_csv()`. Faster and cleaner than base R's `read.csv`.

Base R has `read.csv` and `subset` built in. I could have used those and skipped `readr` and `dplyr`. But `fixest` is not optional — R does not ship with a fixed-effects estimator.

---

## 2. The Formula Syntax

### How R writes a model

In Python, I wrote:

```python
PanelOLS(y, X, entity_effects=True, time_effects=True)

In R, the same model is written as a formula string:

y ~ x1 + x2 | country + year
The ~ means "regress y on." The | is fixest syntax. Everything to the left of the pipe is the regression equation. Everything to the right is the fixed effects
Why this matters
Three names, same model:

Python: entity_effects=True, time_effects=True

R: | country + year

Stata: absorb(country year)

The formula syntax is R's way of writing the same thing. Once you see the pattern, it is readable.

How I built the formula
r
formula_str <- paste(
  "SL.GDP.PCAP.EM.KD ~",
  paste(regressors, collapse = " + "),
  "| country + year"
)

form <- as.formula(formula_str)
paste() concatenates strings. collapse = " + " joins the regressor names with +. Then as.formula() converts the string into something R understands as a model. This is just string-building. There is no magic.

 3. The Reconciliation: R Matches Python
What we ran
Three models in R, matching the three in Python:

Productivity FE

Female LFP FE

Female LFP FE with interaction

The results
Spec    Python IT_lag1    R IT_lag1
Productivity    55.639    55.63855
Female LFP    −0.0175    −0.017538
Interaction (IT_lag1_x_gap)    −0.0021    −0.002079
The coefficients match to at least three decimals. That is the point of Task 3. Two different languages, two different packages, same answer.

Where they differ, and why
The standard errors differ slightly. Example: productivity SE was 33.29 in Python, 31.48 in R.

Why: clustered standard errors have a finite-sample correction factor. linearmodels and fixest use slightly different divisors. When the number of clusters is finite (86 here), the adjustment matters at the margin. The difference is expected, not a bug.

The p-values shift in the same direction. Neither crosses a significance threshold it should not cross.

What to report
In the reconciliation table, report the coefficient as identical and the SE as matching to within a few percent. Note the correction difference in the README.

4. Things I Got Wrong or Almost Got Wrong
Almost saved the model output in the wrong place
RStudio's working directory is separate from where the script file is saved. I set setwd() to the folder with panel_task2.csv. That is where etable() writes the output file. If I had not checked getwd(), the file would have gone somewhere I did not expect.

Almost confused install with library
Already covered above. It is the first thing I will forget if I come back to R in three weeks.

Almost forgot the interaction term needed mutate
In R, columns are added to a data frame with mutate from dplyr:

r
df <- df %>% mutate(IT_lag1_x_gap = IT_lag1 * gender_gap)
In Python it was df["IT_lag1_x_gap"] = df["IT_lag1"] * df["gender_gap"]. Same idea, different syntax.

5. Quick Reference
Concept    R    Python
Load package    library(fixest)    import linearmodels
Read CSV    read_csv("file.csv")    pd.read_csv("file.csv")
Pipe    %>%    . method chaining
Filter rows    filter(!is.na(x))    df.dropna(subset=["x"])
Add column    mutate(y = a * b)    df["y"] = df["a"] * df["b"]
Fixed effects    y ~ x | fe1 + fe2    PanelOLS(..., entity_effects=True, time_effects=True)
Cluster SEs    cluster = ~country    cov_type="clustered", cluster_entity=True
Save table    etable(m, file = "...")    open("...", "w").write(str(res.summary))
text


