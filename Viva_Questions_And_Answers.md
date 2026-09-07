# Viva Questions & Answers — R Data Analytics Examination

Organized by topic to match Sections A, B, and C of the exam.

---

## 1. General R & Data Handling

**Q1. Why do you check `dim()`, `str()`, and `summary()` before analyzing a dataset?**
`dim()` gives the number of rows/columns so you know the dataset size. `str()` shows the data type of each variable (numeric, factor, character), which tells you if any column needs type conversion (e.g., `SeniorCitizen` stored as int instead of factor). `summary()` gives min/max/mean/quartiles for numeric columns and counts for factors, helping spot outliers, skew, or unrealistic values before doing any cleaning.

**Q2. How do you count missing values per column in R?**
`colSums(is.na(df))` — `is.na(df)` returns a logical matrix (TRUE/FALSE per cell), and `colSums()` sums the TRUEs (1s) column-wise, giving the missing count for each variable.

**Q3. What's the difference between `NA`, `NULL`, and an empty string `""` in R?**
`NA` represents a genuinely missing value and participates in type-aware logic (e.g., `NA_integer_`, `NA_character_`). `NULL` represents the absence of a value/object entirely — it has length zero and is used to represent "nothing," e.g., removing a list element. `""` is a valid, non-missing character string (just an empty one) and won't be caught by `is.na()`.

**Q4. Why do you set a seed (`set.seed()`) before splitting data or sampling?**
R's random number generator is pseudo-random — it produces the same sequence of "random" numbers if started from the same seed. Setting a seed makes the train/test split (or any random process) reproducible, so results can be verified/re-run by someone else and get identical output.

**Q5. What is the difference between a factor and a character vector in R?**
A character vector stores raw text. A factor stores categorical data as integer codes internally, mapped to a fixed set of "levels" (labels). Factors are used by modeling functions (`lm`, `rpart`, `knn`) to correctly treat a variable as categorical rather than free text, and they preserve category order/labels for plotting and levels-based logic.

---

## 2. Data Cleaning / Missing Value Imputation

**Q6. Why did you use median imputation instead of mean for `TotalCharges`/`Balance`?**
Mean is sensitive to outliers and skewed distributions — a few very large values pull the mean upward, so imputing with the mean can distort the variable's distribution. Median is the middle value and is robust to skew/outliers, so it better represents a "typical" value when the distribution isn't symmetric. (If the variable is roughly symmetric with no strong outliers, mean imputation is equally acceptable — the key is justifying the choice based on the distribution shape, which you check via a histogram or `skewness()`.)

**Q7. When would you use mode imputation instead of mean/median?**
Mode imputation is used for categorical variables, since mean/median aren't defined for categories. You replace missing values with the most frequently occurring category (e.g., filling missing `Checking account` values with the most common category, or creating an explicit "Unknown" level).

**Q8. What are the risks of imputing missing values instead of dropping rows?**
Imputation can artificially reduce variance (since you're inserting the "typical" value repeatedly) and can introduce bias if the data isn't missing completely at random. Dropping rows avoids this but loses information and can shrink your sample size, especially problematic if missingness is large or non-random (e.g., a `TotalCharges` NA might specifically occur for customers with `tenure = 0`, which is informative, not random).

**Q9. How do you detect and treat unrealistic values, like negative `tenure` or `Age` above 100?**
Detect: use `summary()`/boxplots to spot values outside a plausible range, or explicit logical checks like `sum(df$tenure < 0)`. Treat: either cap/clip them to the nearest valid boundary (Winsorizing), replace with NA and impute, or remove those rows if they're clearly data-entry errors and few in number. The choice depends on how many such records exist and whether the error pattern is understood.

**Q10. What is Winsorizing / capping, and when would you use it over removing outliers?**
Winsorizing replaces extreme values with a specified percentile boundary (e.g., anything above the 99th percentile is set to the 99th percentile value) rather than deleting the row. It's preferred when you want to retain the record's other variable values (dropping the row would lose that information) and when the outlier is a plausible-but-extreme value rather than an impossible one.

**Q11. How do you detect outliers using a boxplot?**
A boxplot shows the IQR (interquartile range = Q3 − Q1). Points beyond 1.5×IQR from Q1 or Q3 are typically flagged as outliers (shown as individual dots beyond the whiskers). In R: `boxplot.stats(x)$out` returns the actual outlier values.

---

## 3. Exploratory Data Analysis & Visualization

**Q12. Why use a boxplot to compare `MonthlyCharges` across `Churn` categories instead of a bar chart of means?**
A boxplot shows the full distribution (median, spread, skew, outliers) for each group, not just a single summary number. Two groups can have the same mean but very different spreads or shapes — a boxplot reveals that, while a bar chart of means would hide it.

**Q13. What does it mean if churned customers show a higher median `MonthlyCharges` than retained customers?**
It suggests price sensitivity may be a driver of churn — customers on higher-cost plans are more likely to leave, possibly because they feel they aren't getting proportional value, or are more open to switching to cheaper competitors.

**Q14. Why use a stacked/grouped bar chart for `Contract` type vs `Churn`, rather than a scatter plot?**
Both `Contract` and `Churn` are categorical variables. Scatter plots are for two continuous variables. A stacked or grouped bar chart (or a mosaic plot) correctly represents counts/proportions across combinations of categorical levels.

**Q15. What does the Pearson correlation coefficient between `tenure` and `MonthlyCharges` tell you, and what are its limitations?**
It quantifies the strength and direction of a *linear* relationship, ranging from −1 (perfect negative) to +1 (perfect positive); values near 0 indicate a weak linear relationship. Its main limitation is that it only captures linear association — two variables can have a strong non-linear (e.g., curved) relationship and still show a Pearson correlation near zero. It's also sensitive to outliers.

**Q16. How would you interpret a correlation heatmap in the Breast Cancer dataset?**
Cells with values close to +1 or −1 (typically shown in a strong color) indicate two predictors are highly (positively/negatively) correlated — e.g., `radius_mean` and `area_mean` are almost always strongly positively correlated since area is derived from radius. This matters for KNN because highly correlated/redundant features can dominate the distance calculation and add redundant weight to essentially the same information.

**Q17. Why is a mosaic plot sometimes preferred over a simple bar chart for two categorical variables?**
A mosaic plot encodes both the proportion within each category and the overall group size (via tile area/width), letting you see conditional relationships (e.g., proportion of "Bad" risk within each `Housing` category) more richly than a simple stacked bar, which only shows counts or proportions along one axis.

---

## 4. Decision Trees

**Q18. How does a Decision Tree decide which variable to split on at each node?**
It evaluates every candidate predictor and split point, and picks the one that produces the greatest reduction in impurity in the resulting child nodes — measured using Gini Index (`rpart`'s default for classification) or entropy/information gain. The split that most "purifies" the two child groups (makes each group as homogeneous as possible in the target class) is chosen.

**Q19. What is the Gini Index, and what does a Gini of 0 mean?**
Gini Index measures node impurity: Gini = 1 − Σ(pᵢ)², where pᵢ is the proportion of class i in the node. A Gini of 0 means the node is perfectly pure — all observations in it belong to a single class.

**Q20. What is the root node of your tree, and what does it tell you?**
The root node is the first (top) split, chosen because it's the single variable/threshold that best separates the two classes across the whole dataset. It indicates the single strongest predictor of the target — e.g., if `Checking account` is the root split in the German Credit tree, it means account status is the most informative single feature for distinguishing good vs. bad credit risk before any other variable is considered.

**Q21. What is overfitting in a Decision Tree, and how do you control it?**
Overfitting happens when the tree grows so deep it memorizes noise/idiosyncrasies of the training data (near-100% training accuracy) but generalizes poorly to new data (low test accuracy). It's controlled via pruning — setting `cp` (complexity parameter), `minsplit`, or `maxdepth` in `rpart.control()`, or post-pruning with `prune()` based on cross-validated error (`printcp()`/`plotcp()`).

**Q22. What does the `cp` (complexity parameter) control in `rpart`?**
`cp` sets the minimum improvement in fit (reduction in overall error) required for a split to be worthwhile; any split that doesn't improve the model by at least `cp` is not attempted/is pruned. Lower `cp` → deeper, more complex trees (risk of overfitting); higher `cp` → simpler trees (risk of underfitting).

**Q23. How do you read a confusion matrix, and how are accuracy, precision, and recall calculated from it?**
A confusion matrix cross-tabulates predicted vs actual classes into True Positives (TP), True Negatives (TN), False Positives (FP), and False Negatives (FN).
- Accuracy = (TP + TN) / (TP + TN + FP + FN) — overall correctness.
- Precision = TP / (TP + FP) — of predicted positives, how many were actually positive.
- Recall (Sensitivity) = TP / (TP + FN) — of actual positives, how many were correctly identified.

**Q24. In a loan-default context, would you prioritize precision or recall for the "Bad risk" class, and why?**
Generally recall, because missing an actual bad-risk borrower (a False Negative — predicting them as "Good" when they default) is more costly to a lender than incorrectly flagging a good borrower as risky (a False Positive, which just means an extra manual review). The right trade-off depends on business cost assumptions, but in credit risk, minimizing missed defaults is usually prioritized.

**Q25. Why is a Decision Tree considered a "white-box" model compared to KNN or a neural network?**
You can trace the exact sequence of if/else rules from root to leaf for any prediction, making it fully human-interpretable. KNN and neural networks make predictions based on distances or learned weights that don't translate into simple, explainable rules.

---

## 5. K-Nearest Neighbors (KNN)

**Q26. Why must you scale/normalize features before running KNN?**
KNN classifies based on distance (typically Euclidean) between points. If one feature has a much larger numeric range than others (e.g., `area_mean` in the hundreds vs. `smoothness_mean` in decimals), it will dominate the distance calculation regardless of its actual predictive importance. Scaling (min-max normalization or z-score standardization) puts all features on a comparable scale so each contributes fairly.

**Q27. What's the difference between min-max normalization and z-score standardization?**
Min-max normalization rescales values into a fixed range, usually [0,1], using (x − min)/(max − min); it's sensitive to outliers since they define the min/max. Z-score standardization transforms values to have mean 0 and standard deviation 1, using (x − mean)/sd; it handles outliers a bit better and doesn't bound values to a fixed range.

**Q28. How does the KNN algorithm actually classify a new/test observation?**
It calculates the distance from the test point to every training point, selects the *k* closest training points (neighbors), and assigns the test point the majority class among those *k* neighbors (for classification; for regression, it would average their values).

**Q29. How do you choose the value of *k*, and what happens if *k* is too small or too large?**
*k* is typically chosen via experimentation/cross-validation, trying several odd values (to avoid ties in binary classification) and picking the one with the best validation accuracy. Too small a *k* (e.g., k=1) makes the model sensitive to noise/outliers and prone to overfitting. Too large a *k* oversmooths the decision boundary, potentially ignoring local structure and underfitting, and can be biased toward the majority class in imbalanced data.

**Q30. Why did you use an odd value of k in a binary classification problem?**
To avoid ties when counting the majority class among the k nearest neighbors — with an even k, it's possible to get an equal number of neighbors from each class, requiring an arbitrary tie-breaking rule.

**Q31. What are the main drawbacks of KNN compared to a Decision Tree?**
KNN has no real "training phase" — it stores the entire training set and does the computation at prediction time, making it slow on large datasets. It's also sensitive to irrelevant/redundant features and requires careful scaling, and it offers no interpretable rule/explanation for why a particular prediction was made — unlike a Decision Tree's clear if/else path.

**Q32. Based on your EDA, which predictors seemed most influential in separating Malignant vs Benign cases, and why?**
Variables like `radius_mean`, `perimeter_mean`, `area_mean`, and `concavity_mean` typically show the clearest separation between the two diagnosis groups in boxplots (malignant cases cluster at visibly higher values) — this is confirmed by comparing group medians/IQRs across `diagnosis` and by checking which features are highly correlated with the target. (Exact features to cite should match whichever boxplots you actually generated.)

---

## 6. Cross-Cutting / Comparison Questions

**Q33. Compare Decision Tree and KNN in terms of how each handles categorical vs numeric predictors.**
Decision Trees handle both categorical and numeric predictors natively — they can split directly on category membership or on numeric thresholds. KNN requires all predictors to be numeric (or converted to numeric, e.g., via one-hot encoding for categories) since it relies on calculating a distance metric.

**Q34. Why do we split data into training and testing sets rather than evaluating the model on the full dataset?**
Evaluating on the same data used to build the model measures how well the model memorized the training data, not how well it generalizes to unseen cases. A held-out test set simulates "new" data, giving a fairer estimate of real-world predictive performance and helping detect overfitting.

**Q35. What would you do differently if your target classes were highly imbalanced (e.g., 95% "No Churn" vs 5% "Churn")?**
Accuracy becomes a misleading metric (a model predicting "No Churn" for everyone would still score 95% accuracy), so you'd rely more on precision, recall, F1-score, or the ROC-AUC. You might also apply resampling techniques (oversampling the minority class with SMOTE, or undersampling the majority class) or use class weights in the model to prevent it from ignoring the minority class.

**Q36. How would you validate that your imputation or scaling choices didn't distort the analysis?**
Compare summary statistics (mean, median, sd) and distribution shape (histogram/density plot) of each variable before and after imputation/scaling — the overall shape should be largely preserved, without introducing artificial spikes at the imputed value or drastically shifting the range beyond what scaling is expected to do.

---

*Tip: Be ready to point to the exact line of your own code/output when answering — examiners often follow up with "show me where you did that" for any answer above.*
