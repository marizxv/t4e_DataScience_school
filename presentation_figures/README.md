# Presentation guideline

NOT A FINISHED PLAN, JUST A GUIDELINE. Change the order, the content and the figures however you like.

The talk is about 7-8 minutes, so around 9 slides (about 1 minute each) could work. To keep the figures readable, use at most one or two figures per slide. All numbers below are from the current notebooks (train = 30,139 people, test = 15,055 people). The figures in this folder are exported from the notebooks in high resolution.

## Possible slides

| # | Slide | Content | Figure(s) |
|---|---|---|---|
| 1 | Title | | |
| 2 | Dataset | Adult Income, 1994 US census. 48,842 people (adult.data 32,561 + adult.test 16,281), 15 variables (6 numerical, 9 categorical). Target: income >50K. We kept the original train/test split. 24.9% of people in train earn more than 50K. | |
| 3 | Preprocessing | see the list below | `03i` and/or `03k` (why the boxplots look strange) |
| 4 | What is related to income | correlations with income | one of `03a`, `03b`, `03e` |
| 5 | Hypothesis tests | table of the tests | |
| 6 | Regression | linear: equation and R²; logistic: test results | `05b` and/or `05c` |
| 7 | Decision tree | settings chosen with cross-validation, test results | `06a` and/or `06b` |
| 8 | Model comparison | table: our models and the results published with the dataset | |
| 9 | Clustering | k-means, k = 5, table of the clusters | `08a` and/or `08b` |
| 10 | Conclusion | 3-4 sentences | |

If time is short: one figure per slide, and the linear regression in one sentence.

## Numbers that could be useful

### Preprocessing
- Train and test are cleaned separately. Everything that learns from the data (encoded columns, scaling) is learned on train only and applied to test, so there is no data leak.
- Removed the dot in the test labels (`>50K.` to `>50K`).
- Deleted rows with missing values: 2,399 in train (7.37%), 1,221 in test (7.50%).
- Deleted duplicate rows: 23 in train, 5 in test.
- Outliers found with z-score and IQR, all values kept (they are real values, not errors).
- capital-gain and capital-loss: log(1 + x), skewness of capital-gain 11.9 to 3.07.
- New features: has-capital, is-married, is_us, capital-net, age-group, hours-category.
- Encoding: sex 0/1, native-country to is_us, one-hot for 5 categorical variables. Dropped education (same as education-num) and fnlwgt. 42 columns, numerical ones standardized.

### Correlations with income (train)
| Variable | r |
|---|---|
| Married-civ-spouse | 0.45 |
| education-num | 0.34 |
| has-capital | 0.31 |
| capital-gain-log | 0.29 |
| age | 0.24 |
| hours-per-week | 0.23 |
| Never-married | -0.32 |
| Own-child | -0.23 |

About `03a` (marital status): the top bar, Married-AF-spouse, is only 21 people. It may be clearer to say "45.5% of married people earn >50K, only 6.8% of the others".

### Hypothesis tests (train and test together, 45,194 people)
| Test | H0 | Result | Decision |
|---|---|---|---|
| One-sample t-test | mean working hours = 40 | 40.94 h, p < 0.001 | reject |
| One proportion z-test | share earning >50K = 25% | 24.80%, p = 0.314 | do not reject |
| Pearson correlation | age and hours not correlated | r = 0.10, p < 0.001 | reject (very weak) |
| Two-sample t-test | women and men work the same hours | 36.9 h vs 42.9 h, p < 0.001 | reject |
| Two proportions z-test | same share of women and men earn >50K | 11.4% vs 31.3%, p < 0.001 | reject |
| Chi-square goodness of fit | sex split 50/50 | 32.5% women, 67.5% men | reject |
| Chi-square independence | income does not depend on education | χ² = 5,996, p < 0.001 | reject |
| Chi-square homogeneity | income distribution is the same in all race groups | χ² = 453, p < 0.001 | reject |

### Regression
- Linear: hours-per-week = 27.33 + 0.070 × age + 0.696 × education-num + 5.73 × male. F-test significant (F = 899), but R² = 0.078 on test (MAE 7.7 h): significant, but explains very little.
- Logistic, test set: accuracy 0.849, precision 0.734, recall 0.606, F1 0.664, ROC-AUC 0.905 (always predicting <=50K would give accuracy 0.754).
- Odds ratios: Married-civ-spouse x9.1, relationship Wife x3.7, male x2.4, Exec-managerial x2.2, each step of education-num x1.32, Own-child x0.54.
- has-capital (0.00), capital-loss-log (x30) and workclass Without-pay (0.00) have extreme odds ratios. has-capital overlaps with the capital log columns and Without-pay has only 14 people, so these coefficients are not reliable and better left out.

### Decision tree
| Tree | Train accuracy | CV F1 |
|---|---|---|
| No limits | 97.7% | 0.624 |
| max_depth = 9 | 86.4% | 0.667 |
| min_samples_leaf = 20 (final) | 87.2% | 0.678 |

- The settings are chosen with 5-fold cross-validation on train. The test set is used only once, for the final tree.
- Final tree on the test set: accuracy 0.850, precision 0.742, recall 0.600, F1 0.663, ROC-AUC 0.896.
- Most important features: Married-civ-spouse (0.37), education-num (0.21), capital-gain-log (0.16), age (0.08).
- Rule-based classifier: 131 rules instead of 731 leaves, accuracy 0.839, recall 0.712, F1 0.686.

### Model comparison (test set accuracy)
| Model | Accuracy |
|---|---|
| Always predicting <=50K | 75.4% |
| Naive Bayes (published in adult.names) | 83.9% |
| C4.5 decision tree (published) | 84.5% |
| NBTree (best published) | 85.9% |
| Our logistic regression | 84.9% |
| Our decision tree | 85.0% |

### Clustering
- k-means on train, 7 standardized features: age, education-num, hours-per-week, capital-gain-log, capital-loss-log, sex, is-married. Income is not used.
- k = 2 to 10 tried. Elbow method and silhouette (0.317) both choose k = 5.

| Cluster | Who | Share | Earn >50K |
|---|---|---|---|
| 0 | young single men (~32) | 23% | 6.1% |
| 1 | capital loss (~42) | 5% | 51.6% |
| 2 | capital gain (~44, highest education) | 8% | 63.8% |
| 3 | women, mostly not married | 29% | 8.2% |
| 4 | married men (~43) | 35% | 38.6% |

## Figures in this folder
| File | Could go on |
|---|---|
| `03a_eda_income_by_marital_status.png` | correlations slide |
| `03b_eda_income_by_education.png` | correlations slide |
| `03c_eda_income_by_sex.png` | correlations slide |
| `03d_eda_income_by_age_group.png` | correlations slide |
| `03e_eda_correlation_with_income.png` | correlations slide |
| `03f_boxplot_age.png` | preprocessing slide (a normal-looking boxplot, for comparison) |
| `03g_boxplot_fnlwgt.png` | preprocessing slide (long tail to the right) |
| `03h_boxplot_education-num.png` | preprocessing slide (a few low outliers) |
| `03i_boxplot_capital-gain.png` | preprocessing slide (91.6% are 0, so there is no box, only a line at 0, and every non-zero value is an outlier dot) |
| `03j_boxplot_capital-loss.png` | preprocessing slide (the same effect, 95.3% are 0) |
| `03k_boxplot_hours-per-week.png` | preprocessing slide (47.2% work exactly 40 h, so the box is only 40-45 h and IQR marks 26% as outliers) |
| `05a_linear_actual_vs_predicted.png` | regression slide |
| `05b_logistic_confusion_matrix.png` | regression slide |
| `05c_logistic_roc_curve.png` | regression slide |
| `06a_tree_f1_vs_depth.png` | decision tree slide (the left half is enough) |
| `06b_tree_feature_importance.png` | decision tree slide |
| `06c_tree_confusion_matrix.png` | decision tree slide |
| `08a_clustering_elbow_silhouette.png` | clustering slide |
| `08b_clustering_pca_plot.png` | clustering slide |
| `08c_clustering_income_by_cluster.png` | clustering slide (or show the cluster table instead) |
