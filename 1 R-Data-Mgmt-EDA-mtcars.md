# Data Management and Exploratory Data Analysis in R
### A Comprehensive Guide Using the `mtcars` Dataset (R 4.3)

> **Course:** M.Sc./M.Tech Data Science
> **Topics covered:** Managing Data with R · Data Cleaning · Exploring and Understanding Data · Exploring the Structure of Data · Exploring Relationships Between Variables
> **R version:** 4.3.x
> **Dataset:** `mtcars` (built into base R)

---

## Table of Contents

1. [Introduction to the Dataset](#1-introduction-to-the-dataset)
2. [Managing Data with R](#2-managing-data-with-r)
3. [Data Cleaning](#3-data-cleaning)
4. [Exploring and Understanding Data](#4-exploring-and-understanding-data)
5. [Exploring the Structure of Data](#5-exploring-the-structure-of-data)
6. [Exploring Relationships Between Variables](#6-exploring-relationships-between-variables)
7. [Summary Cheat Sheet](#7-summary-cheat-sheet)
8. [Practice Exercises](#8-practice-exercises)

---

## 1. Introduction to the Dataset

`mtcars` ("Motor Trend Car Road Tests") is a built-in R dataset extracted from the 1974 *Motor Trend* US magazine. It comprises fuel consumption and 10 aspects of automobile design and performance for 32 automobiles (1973–74 models).

```r
# Load the dataset into your environment
data(mtcars)

# Quick peek
head(mtcars)
```

| Column | Meaning                                   |
|--------|--------------------------------------------|
| `mpg`  | Miles per US gallon                        |
| `cyl`  | Number of cylinders                        |
| `disp` | Displacement (cubic inches)                |
| `hp`   | Gross horsepower                           |
| `drat` | Rear axle ratio                            |
| `wt`   | Weight (1000 lbs)                          |
| `qsec` | 1/4 mile time (seconds)                    |
| `vs`   | Engine shape (0 = V-shaped, 1 = straight)  |
| `am`   | Transmission (0 = automatic, 1 = manual)   |
| `gear` | Number of forward gears                    |
| `carb` | Number of carburetors                      |

We will use this single dataset throughout so that students can see how every concept — from importing to visual relationships — connects into one coherent workflow.

---

## 2. Managing Data with R

Data management is the foundation of any analysis: getting data in, understanding its container (data frame), and manipulating rows/columns/values.

### 2.1 Data Frames — the Core Data Structure

`mtcars` is a **data frame**: a rectangular table where columns can be of different types (though here, all are numeric) and rows are observations.

```r
class(mtcars)      # "data.frame"
dim(mtcars)         # 32 rows, 11 columns
nrow(mtcars)         # number of observations
ncol(mtcars)         # number of variables
rownames(mtcars)    # car model names (row names, not a column!)
colnames(mtcars)    # variable names
```

> **Teaching note:** Unlike most real-world datasets, `mtcars` stores the car model as **row names**, not as a column. This is a great teaching moment — students should convert it into a proper column for tidy workflows.

```r
library(dplyr)  # part of tidyverse

mtcars_df <- mtcars %>%
  tibble::rownames_to_column(var = "model")

head(mtcars_df)
```

### 2.2 Importing and Exporting Data

Although `mtcars` ships with R, students should practice the full read/write cycle:

```r
# Export to CSV
write.csv(mtcars_df, "mtcars.csv", row.names = FALSE)

# Import back
mtcars_imported <- read.csv("mtcars.csv", stringsAsFactors = FALSE)

# Tidyverse equivalent (recommended in R 4.3 workflows)
library(readr)
mtcars_imported2 <- read_csv("mtcars.csv")
```

### 2.3 Indexing and Subsetting

```r
# Base R indexing
mtcars[1, ]              # first row
mtcars[, "mpg"]          # mpg column as a vector
mtcars[["mpg"]]          # same, list-style extraction
mtcars$mpg               # same, using $ operator

# Row and column subset together
mtcars[1:5, c("mpg", "hp", "wt")]

# Logical subsetting: cars with mpg > 25
mtcars[mtcars$mpg > 25, ]

# subset() function
subset(mtcars, mpg > 25, select = c(mpg, hp, wt))
```

### 2.4 dplyr Verbs for Data Management

`dplyr` is the modern grammar for manipulating data frames — every verb below does one clear thing.

```r
library(dplyr)

mtcars_df %>%
  filter(cyl == 6) %>%                 # keep rows
  select(model, mpg, hp, wt) %>%       # keep columns
  arrange(desc(mpg)) %>%               # sort
  mutate(kmpl = mpg * 0.425) %>%       # create new column
  head()
```

| Verb       | Purpose                          |
|------------|-----------------------------------|
| `filter()` | Subset rows by condition          |
| `select()` | Subset/reorder columns            |
| `arrange()`| Sort rows                         |
| `mutate()` | Create/transform columns          |
| `summarise()`| Collapse to summary statistics  |
| `group_by()`| Define groups for aggregation    |

### 2.5 Grouped Summaries

```r
mtcars_df %>%
  group_by(cyl) %>%
  summarise(
    avg_mpg = mean(mpg),
    avg_hp  = mean(hp),
    n_cars  = n()
  )
```

### 2.6 Recoding and Labeling Categorical-Like Variables

Several `mtcars` columns are numerically coded categories (`am`, `vs`, `cyl`, `gear`). Managing data well means converting these into meaningful factors before analysis.

```r
mtcars_df <- mtcars_df %>%
  mutate(
    am_label  = factor(am,  levels = c(0, 1), labels = c("automatic", "manual")),
    vs_label  = factor(vs,  levels = c(0, 1), labels = c("V-shaped", "straight")),
    cyl       = factor(cyl),
    gear      = factor(gear)
  )
```

---

## 3. Data Cleaning

`mtcars` is famously "clean" (no missing values, no duplicate rows), so we will **intentionally introduce common real-world problems** to teach cleaning techniques, then clean them back up. This is a standard classroom technique — simulate messiness, then fix it.

### 3.1 Checking for Missing Values

```r
# mtcars itself has no NAs — verify this first
sum(is.na(mtcars))          # 0
colSums(is.na(mtcars))      # all zeros

# Simulate missingness for teaching purposes
set.seed(123)
mtcars_dirty <- mtcars_df
mtcars_dirty$hp[sample(1:32, 3)] <- NA
mtcars_dirty$mpg[sample(1:32, 2)] <- NA

colSums(is.na(mtcars_dirty))
```

### 3.2 Handling Missing Values

```r
# Option 1: Remove rows with any NA (listwise deletion)
mtcars_complete <- na.omit(mtcars_dirty)

# Option 2: Impute with mean
mtcars_imputed <- mtcars_dirty %>%
  mutate(
    hp  = ifelse(is.na(hp),  mean(hp,  na.rm = TRUE), hp),
    mpg = ifelse(is.na(mpg), mean(mpg, na.rm = TRUE), mpg)
  )

# Option 3: Impute with median (robust to outliers)
mtcars_imputed_median <- mtcars_dirty %>%
  mutate(
    hp  = ifelse(is.na(hp),  median(hp,  na.rm = TRUE), hp),
    mpg = ifelse(is.na(mpg), median(mpg, na.rm = TRUE), mpg)
  )
```

> **Discussion point for students:** Mean imputation understates variance and can bias correlation estimates — always disclose imputation strategy in a report.

### 3.3 Detecting and Removing Duplicates

```r
# Simulate a duplicate row
mtcars_dup <- rbind(mtcars_df, mtcars_df[1, ])

duplicated(mtcars_dup)              # logical vector flagging dupes
sum(duplicated(mtcars_dup))         # count of duplicates

mtcars_clean <- mtcars_dup[!duplicated(mtcars_dup), ]
```

### 3.4 Detecting Outliers

```r
# Boxplot-based outlier detection using the IQR rule
Q1 <- quantile(mtcars$hp, 0.25)
Q3 <- quantile(mtcars$hp, 0.75)
IQR_hp <- IQR(mtcars$hp)

lower_bound <- Q1 - 1.5 * IQR_hp
upper_bound <- Q3 + 1.5 * IQR_hp

outliers <- mtcars_df %>%
  filter(hp < lower_bound | hp > upper_bound) %>%
  select(model, hp)

outliers
```

```r
boxplot(mtcars$hp, main = "Horsepower Outlier Check", ylab = "hp")
```

The Maserati Bora and similar high-horsepower models typically flag as outliers here — a good discussion point on whether outliers should be removed or are simply "true, interesting extremes."

### 3.5 Fixing Inconsistent/Invalid Values

```r
# Simulate a data entry error: negative weight
mtcars_dirty2 <- mtcars_df
mtcars_dirty2$wt[5] <- -3.44

# Detect logically impossible values
mtcars_dirty2 %>% filter(wt < 0)

# Fix (e.g., take absolute value, assuming sign error)
mtcars_dirty2 <- mtcars_dirty2 %>%
  mutate(wt = ifelse(wt < 0, abs(wt), wt))
```

### 3.6 Ensuring Correct Data Types

```r
str(mtcars_df)          # inspect types

# Force correct types after cleaning
mtcars_df <- mtcars_df %>%
  mutate(
    cyl  = as.factor(cyl),
    gear = as.factor(gear),
    carb = as.factor(carb)
  )
```

### 3.7 Data Cleaning Checklist

- [ ] Are there missing values? How should they be handled?
- [ ] Are there duplicate rows?
- [ ] Are there impossible/invalid values (negative weight, zero cylinders)?
- [ ] Are categorical variables encoded as factors, not raw numbers?
- [ ] Are there extreme outliers, and are they genuine or data errors?
- [ ] Are column names consistent and descriptive?

---

## 4. Exploring and Understanding Data

Before modeling, we build intuition: what does each variable look like, and what does the dataset "feel" like as a whole?

### 4.1 First Look

```r
head(mtcars, 10)      # first 10 rows
tail(mtcars, 5)       # last 5 rows
View(mtcars)          # opens spreadsheet-style viewer (RStudio)
```

### 4.2 Dimensions and Overview

```r
dim(mtcars)
str(mtcars)            # structure: types + first few values per column
summary(mtcars)        # five-number summary + mean per column
```

Sample output interpretation for `mpg`:

```
     mpg       
 Min.   :10.40  
 1st Qu.:15.43  
 Median :19.20  
 Mean   :20.09  
 3rd Qu.:22.80  
 Max.   :33.90  
```

**Teaching point:** Mean (20.09) > Median (19.20) suggests a slight right skew in `mpg` — some fuel-efficient cars pull the average up.

### 4.3 Univariate Descriptive Statistics

```r
mean(mtcars$mpg)
median(mtcars$mpg)
sd(mtcars$mpg)          # standard deviation
var(mtcars$mpg)         # variance
range(mtcars$mpg)       # min and max
IQR(mtcars$mpg)         # interquartile range
quantile(mtcars$mpg, probs = c(0.1, 0.25, 0.5, 0.75, 0.9))
```

### 4.4 Frequency Tables for Categorical-Like Variables

```r
table(mtcars$cyl)                     # count by cylinder count
table(mtcars$am)                      # count by transmission type
prop.table(table(mtcars$cyl))         # proportions
table(mtcars$cyl, mtcars$am)          # cross-tabulation
```

### 4.5 Visual Exploration — Univariate

```r
# Histogram
hist(mtcars$mpg,
     main = "Distribution of Miles per Gallon",
     xlab = "Miles per Gallon",
     col = "steelblue",
     breaks = 8)

# Density plot
plot(density(mtcars$mpg),
     main = "Density of MPG")

# Boxplot
boxplot(mtcars$mpg,
        main = "Boxplot of MPG",
        ylab = "mpg")

# Bar plot for categorical variable
barplot(table(mtcars$cyl),
        main = "Number of Cars by Cylinder Count",
        xlab = "Cylinders",
        ylab = "Count",
        col = "coral")
```

Using `ggplot2` (recommended for reports/publications):

```r
library(ggplot2)

ggplot(mtcars_df, aes(x = mpg)) +
  geom_histogram(binwidth = 2, fill = "steelblue", color = "white") +
  labs(title = "Distribution of MPG", x = "Miles per Gallon", y = "Count") +
  theme_minimal()
```

### 4.6 Skewness and Shape

```r
library(e1071)     # install.packages("e1071") if needed

skewness(mtcars$mpg)   # > 0 indicates right skew
kurtosis(mtcars$mpg)   # > 0 indicates heavier tails than normal
```

---

## 5. Exploring the Structure of Data

"Structure" refers to how variables are organized, typed, and interrelated at a schema level — this goes one level deeper than summary statistics.

### 5.1 `str()` in Depth

```r
str(mtcars)
```

```
'data.frame':   32 obs. of  11 variables:
 $ mpg : num  21 21 22.8 21.4 18.7 ...
 $ cyl : num  6 6 4 6 8 ...
 $ disp: num  160 160 108 258 360 ...
 ...
```

`str()` tells students, at a glance: **how many rows, how many columns, the data type of each column, and a preview of values.** This should always be the *first* command run on any unfamiliar dataset.

### 5.2 Identifying Variable Types (Measurement Scales)

| Variable | R type stored | True measurement scale       |
|----------|----------------|-------------------------------|
| `mpg`    | numeric (double)| Continuous (ratio)           |
| `disp`   | numeric         | Continuous (ratio)           |
| `hp`     | numeric         | Continuous (ratio)           |
| `drat`   | numeric         | Continuous (ratio)           |
| `wt`     | numeric         | Continuous (ratio)           |
| `qsec`   | numeric         | Continuous (ratio)           |
| `cyl`    | numeric         | Discrete/Ordinal (should be factor)|
| `vs`     | numeric (0/1)   | Nominal/binary (should be factor)|
| `am`     | numeric (0/1)   | Nominal/binary (should be factor)|
| `gear`   | numeric         | Ordinal (should be factor)   |
| `carb`   | numeric         | Discrete/nominal (should be factor)|

> **Key teaching insight:** R stores everything in `mtcars` as `numeric`, but statistically several of these are categorical. Students must learn to look past the *storage type* to the *semantic type* — this affects which statistics and plots are valid.

### 5.3 Converting Types to Reflect True Structure

```r
mtcars_structured <- mtcars_df %>%
  mutate(
    cyl  = factor(cyl, levels = c(4, 6, 8), ordered = TRUE),
    gear = factor(gear, levels = c(3, 4, 5), ordered = TRUE),
    vs   = factor(vs, labels = c("V-shaped", "straight")),
    am   = factor(am, labels = c("automatic", "manual")),
    carb = factor(carb)
  )

str(mtcars_structured)
levels(mtcars_structured$cyl)
```

### 5.4 Data Dictionary / Codebook

A well-managed dataset always ships with a **codebook**. Building one is a core "structure exploration" skill:

```r
codebook <- data.frame(
  variable = names(mtcars),
  description = c(
    "Miles per US gallon", "Number of cylinders", "Displacement (cu.in.)",
    "Gross horsepower", "Rear axle ratio", "Weight (1000 lbs)",
    "1/4 mile time", "Engine (0=V-shaped,1=straight)",
    "Transmission (0=automatic,1=manual)", "Number of forward gears",
    "Number of carburetors"
  ),
  type = sapply(mtcars, class)
)
print(codebook)
```

### 5.5 Dataset "Shape" Checks

```r
# Are variables numeric-only? Any character columns?
sapply(mtcars, class)

# Any constant (zero-variance) columns? (Would carry no information)
sapply(mtcars, function(x) length(unique(x)))

# Structural completeness — full cases
complete.cases(mtcars) %>% all()
```

### 5.6 Multi-Level / Nested Structure with `group_by`

Understanding structure also means understanding **grouping hierarchy** — e.g., cars nested within cylinder count nested within transmission type:

```r
mtcars_structured %>%
  group_by(cyl, am) %>%
  summarise(n = n(), avg_mpg = mean(mpg), .groups = "drop")
```

---

## 6. Exploring Relationships Between Variables

This is where EDA becomes genuinely analytical — moving from "what does one variable look like" to "how do variables move together."

### 6.1 Correlation Matrix (Numeric–Numeric)

```r
numeric_vars <- mtcars %>% select(mpg, disp, hp, drat, wt, qsec)

cor_matrix <- cor(numeric_vars)
round(cor_matrix, 2)
```

```r
# Visualizing the correlation matrix
library(corrplot)   # install.packages("corrplot") if needed

corrplot(cor_matrix, method = "circle", type = "upper",
         tl.col = "black", addCoef.col = "black")
```

**Key relationships to discuss with students:**
- `mpg` and `wt`: strong **negative** correlation (heavier cars are less fuel-efficient)
- `mpg` and `hp`: strong **negative** correlation
- `hp` and `disp`: strong **positive** correlation (bigger engines produce more power)
- `wt` and `disp`: strong **positive** correlation

### 6.2 Scatterplots (Continuous vs. Continuous)

```r
plot(mtcars$wt, mtcars$mpg,
     main = "MPG vs Weight",
     xlab = "Weight (1000 lbs)", ylab = "Miles per Gallon",
     pch = 19, col = "darkblue")

# Add a trend line
abline(lm(mpg ~ wt, data = mtcars), col = "red", lwd = 2)
```

```r
ggplot(mtcars_df, aes(x = wt, y = mpg)) +
  geom_point(size = 3, color = "steelblue") +
  geom_smooth(method = "lm", se = TRUE, color = "firebrick") +
  labs(title = "MPG decreases as Weight increases",
       x = "Weight (1000 lbs)", y = "Miles per Gallon") +
  theme_minimal()
```

### 6.3 Pairwise Scatterplot Matrix

```r
pairs(mtcars[, c("mpg", "hp", "wt", "disp")],
      main = "Pairwise Relationships")

# or with base pairs() enhanced panel showing correlation coefficients
panel.cor <- function(x, y, ...) {
  r <- round(cor(x, y), 2)
  text(0.5, 0.5, r, cex = 1.5)
}
pairs(mtcars[, c("mpg", "hp", "wt", "disp")],
      lower.panel = panel.smooth, upper.panel = panel.cor)
```

### 6.4 Grouped Comparisons (Categorical vs. Continuous)

```r
# Boxplot of mpg across transmission types
boxplot(mpg ~ am, data = mtcars_structured,
        main = "MPG by Transmission Type",
        xlab = "Transmission", ylab = "MPG",
        col = c("lightblue", "lightgreen"))
```

```r
ggplot(mtcars_structured, aes(x = am, y = mpg, fill = am)) +
  geom_boxplot() +
  labs(title = "Manual cars tend to have higher MPG",
       x = "Transmission", y = "Miles per Gallon") +
  theme_minimal()
```

### 6.5 Statistical Tests of Relationship

```r
# Pearson correlation test (continuous-continuous)
cor.test(mtcars$mpg, mtcars$wt, method = "pearson")

# Independent samples t-test (continuous vs binary categorical)
t.test(mpg ~ am, data = mtcars_structured)

# ANOVA (continuous vs multi-level categorical)
anova_result <- aov(mpg ~ cyl, data = mtcars_structured)
summary(anova_result)

# Chi-square test (categorical vs categorical)
chisq.test(table(mtcars_structured$cyl, mtcars_structured$am))
```

**Interpretation guide:**

| Test | Null Hypothesis | When to Use |
|------|------------------|-------------|
| `cor.test()` | True correlation = 0 | Two continuous variables |
| `t.test()`   | Group means are equal | Continuous vs. 2-level categorical |
| `aov()`      | All group means are equal | Continuous vs. 3+ level categorical |
| `chisq.test()`| Variables are independent | Two categorical variables |

### 6.6 Multivariate Visualization

```r
# Color-encode a third variable
ggplot(mtcars_structured, aes(x = wt, y = mpg, color = cyl)) +
  geom_point(size = 3) +
  labs(title = "MPG vs Weight, colored by Cylinder Count") +
  theme_minimal()

# Facet by transmission type
ggplot(mtcars_structured, aes(x = wt, y = mpg)) +
  geom_point(color = "steelblue") +
  geom_smooth(method = "lm", se = FALSE, color = "firebrick") +
  facet_wrap(~ am) +
  labs(title = "MPG vs Weight, faceted by Transmission") +
  theme_minimal()
```

### 6.7 Simple Linear Regression as a Relationship Model

```r
model <- lm(mpg ~ wt, data = mtcars)
summary(model)

# Multiple regression
model2 <- lm(mpg ~ wt + hp + cyl, data = mtcars)
summary(model2)
```

**Reading the output with students:**
- **Coefficient sign** tells direction of relationship
- **p-value** (`Pr(>|t|)`) tells statistical significance
- **R-squared** tells proportion of variance in `mpg` explained by predictors
- **Residual plots** (`plot(model)`) reveal whether linearity/homoscedasticity assumptions hold

---

## 7. Summary Cheat Sheet

| Task | Function(s) |
|------|-------------|
| Load data | `data(mtcars)` |
| Peek at data | `head()`, `tail()`, `View()` |
| Structure | `str()`, `dim()`, `class()`, `sapply(x, class)` |
| Summary stats | `summary()`, `mean()`, `median()`, `sd()`, `IQR()` |
| Subset rows/cols | `filter()`, `select()`, `[ ]`, `subset()` |
| Sort | `arrange()` |
| New columns | `mutate()` |
| Group & aggregate | `group_by()` + `summarise()` |
| Missing values | `is.na()`, `na.omit()`, `ifelse(is.na(...))` |
| Duplicates | `duplicated()` |
| Outliers | `quantile()`, `IQR()`, `boxplot()` |
| Frequency tables | `table()`, `prop.table()` |
| Univariate plots | `hist()`, `boxplot()`, `barplot()`, `geom_histogram()` |
| Correlation | `cor()`, `cor.test()`, `corrplot()` |
| Bivariate plots | `plot()`, `pairs()`, `geom_point()` |
| Group comparison | `boxplot(y ~ x)`, `t.test()`, `aov()` |
| Categorical association | `chisq.test()` |
| Modeling | `lm()`, `summary(model)` |

---

## 8. Practice Exercises

1. Convert `mtcars` row names into a proper `model` column and save it as a new CSV.
2. Introduce 5 random missing values into `qsec` and compare mean vs. median imputation — how much does each shift the overall mean of `qsec`?
3. Using the IQR rule, identify all outliers in `disp`. Are they the same cars flagged as outliers in `hp`?
4. Build a correlation matrix for all numeric variables and identify the **three strongest** positive and negative correlations (excluding self-correlation).
5. Create a boxplot comparing `qsec` across `cyl` groups. Interpret whether cylinder count affects acceleration time.
6. Fit `lm(mpg ~ wt + hp)` and interpret the coefficients and R². Then add `cyl` as a predictor and see how R² changes.
7. Using `dplyr`, find the average `mpg` for each combination of `am` and `vs`.
8. Produce a faceted `ggplot2` scatterplot of `hp` vs `qsec`, split by `am`, colored by `cyl`.

---

*Prepared for classroom use — feel free to fork, adapt, and extend for your own R (≥ 4.3) teaching materials.*
