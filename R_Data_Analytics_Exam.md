# R Programming Examination — Data Analytics

**Time:** 3 Hours | **Max Marks:** 60 (20 per section)

**Instructions to Candidates:**
- Answer all three sections. Each section is independent and carries 20 marks.
- Write all code in R (base R and/or tidyverse, `caret`, `rpart`, `class`, `rpart.plot`, `e1071` packages permitted unless stated otherwise).
- Include comments in your code explaining each step.
- Save your R script/R Markdown as `RollNumber_Section.R` (or `.Rmd`) for each section.
- Round all numerical answers to 2 decimal places unless stated otherwise.

---

## Section A: Data Cleaning, EDA and Visualization (20 Marks)

**Topic:** Bank Customer Churn Analytics
**Dataset:** Bank Customer Churn Prediction Dataset (variables include: `CreditScore`, `Geography`, `Gender`, `Age`, `Tenure`, `Balance`, `NumOfProducts`, `HasCrCard`, `IsActiveMember`, `EstimatedSalary`, `Exited`)
**Download:** Kaggle — "Bank Customer Churn Prediction" (search: bank customer churn dataset shubhammeshram)

### Questions

1. **(4 marks)** Import the dataset into R. Display its dimensions, structure (`str()`), and summary statistics (`summary()`). Identify and report the number of missing values in each column using a single command.

2. **(4 marks)** Handle missing values in `Balance` using an appropriate numerical imputation method (mean, median, or model-based). Justify your choice of method in a comment. Identify and treat any invalid or unrealistic values in `Age` (e.g., negative ages or ages above 100) and `CreditScore` (valid range 300–850).

3. **(6 marks)** Create suitable visualizations to:
   - a) Compare `EstimatedSalary` (or `Balance`) across `Exited` categories (churned vs retained).
   - b) Compare `Geography` distribution across `Exited` categories.
   - c) Show the proportion of `Exited` customers by `NumOfProducts`.

   (Use boxplots/violin plots for (a) and stacked bar charts or grouped bar charts for (b) and (c). Label all axes and titles appropriately.)

4. **(4 marks)** Analyze the relationship between `Age` and `Balance` using a scatter plot (color-coded by `Exited`). Calculate and interpret the correlation coefficient between `Age` and `Balance`.

5. **(2 marks)** Write two data-driven observations based on your analysis in Q3 and Q4.

---

## Section B: Data Cleaning, EDA and Decision Tree Classification (20 Marks)

**Topic:** Loan Default Risk Prediction
**Dataset:** German Credit Risk Dataset (variables include: `Age`, `Sex`, `Job`, `Housing`, `Saving accounts`, `Checking account`, `Credit amount`, `Duration`, `Purpose`, `Risk` — target: Good/Bad credit risk)
**Download:** Kaggle — "German Credit Risk" (UCI Statlog German Credit Data)

### Questions

1. **(3 marks)** Load the dataset into R as a data frame. Inspect its dimensions, structure, and summary. Identify missing values in each variable and the proportion of missing data.

2. **(3 marks)** Clean the dataset: impute missing values in categorical variables (e.g., `Saving accounts`, `Checking account`) using mode imputation or by creating a separate "Unknown" category. Convert appropriate character/categorical columns into factors, and identify/treat any unrealistic values in `Age` or `Credit amount` (e.g., zero or negative amounts).

3. **(4 marks)** Explore the relationship between `Risk` (target) and at least two predictors (e.g., `Housing`, `Purpose`, or `Duration`) using contingency tables (`table()`/`prop.table()`) and suitable visualizations (mosaic plot, grouped bar chart, or boxplot).

4. **(3 marks)** Split the cleaned dataset into 70% training and 30% testing observations using a fixed random seed (`set.seed(123)`).

5. **(5 marks)** Build and plot a Decision Tree classifier (using `rpart`/`rpart.plot`) to predict `Risk` using at least three relevant predictors (e.g., `Duration`, `Credit amount`, `Checking account`, `Age`).

6. **(2 marks)** Predict the test set classes, generate a confusion matrix, calculate classification accuracy, precision, and recall. Identify the primary (root node) splitting variable in the tree and briefly interpret what it means.

---

## Section C: Data Cleaning, EDA and K-Nearest Neighbors (KNN) Classification (20 Marks)

**Topic:** Breast Cancer Diagnosis Classification
**Dataset:** Wisconsin Breast Cancer Diagnostic Dataset (variables include: `id`, `diagnosis` (M/B), and 30 numeric features such as `radius_mean`, `texture_mean`, `perimeter_mean`, `area_mean`, `smoothness_mean`, etc.)
**Download:** Kaggle — "Breast Cancer Wisconsin (Diagnostic) Data Set"

### Questions

1. **(3 marks)** Load the dataset into R. Display its dimensions, structure, and summary. Identify the number of missing values in each column and drop any entirely empty/unnamed columns (e.g., trailing `NA` columns from CSV export).

2. **(3 marks)** Handle missing values (if any) in numeric predictor columns using an appropriate imputation method (mean/median/KNN-imputation). Check for and treat outliers in at least one variable (e.g., `area_mean` or `radius_mean`) using boxplot-based detection.

3. **(4 marks)** Explore the relationship between `diagnosis` and at least two predictors (e.g., `radius_mean`, `concavity_mean`) using boxplots grouped by diagnosis, and a correlation heatmap of the numeric predictors.

4. **(3 marks)** Normalize/scale the numeric predictor variables (min-max normalization or z-score standardization) — explain why feature scaling is essential before applying KNN.

5. **(3 marks)** Split the cleaned and scaled dataset into 70% training and 30% testing observations using a fixed random seed (`set.seed(123)`).

6. **(4 marks)** Build a KNN classifier (using the `class` or `caret` package) to classify `diagnosis` as Malignant or Benign using at least four relevant predictors. Try at least two different values of *k* (e.g., k=5 and k=9) and report which performs better.

7. **(2 marks)** Zenerate a confusion matrix and classification accuracy for the best-performing model, and write one observation on which predictor(s) appear most influential in separating the two classes based on your EDA in Q3.

---

### General Notes for All Sections
- Use `ggplot2` for all visualizations unless base R plotting is specifically preferred.
- Clearly label all plots with titles, axis labels, and legends.
- All imputation and cleaning decisions must be justified with a one-line comment in code.
- Interpret every statistical output (correlation, accuracy, confusion matrix) in plain language — do not just print the numbers.
