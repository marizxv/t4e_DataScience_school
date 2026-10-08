# Presentation plan

About 7-8 minutes, so around 9 slides (about 1 minute each). All numbers below are from the current notebooks (train = 30,139 people, test = 15,055 people). The figures in this folder are exported from the notebooks in high resolution.

## Slides

| # | Slide | Content | Figure |
|---|---|---|---|
| 1 | Title | | |
| 2 | Dataset | Adult Income, 1994 US census. 48,842 people (adult.data 32,561 + adult.test 16,281), 15 variables (6 numerical, 9 categorical). Target: income >50K. We kept the original train/test split. 24.9% of people in train earn more than 50K. | |
| 3 | Preprocessing | see the list below | |
| 4 | What is related to income | correlations with income + one chart | `03a` or `03b`, or `03e` |
| 5 | Hypothesis tests | table of the tests | |
| 6 | Regression | linear: equation + R²; logistic: test results | `05b`, `05c` (optional `05a`) |
| 7 | Decision tree | settings chosen with cross-validation, test results | `06a`, `06b` |
| 8 | Model comparison | table: our models vs. the results published with the dataset | |
| 9 | Clustering | k-means, k = 5 | `08a`, `08b`, `08c` |
| 10 | Conclusion | 3-4 sentences | |

If time is short: show only one figure on slide 4 and keep the linear regression to one sentence.

## Numbers for each slide

### Slide 3: Preprocessing
- Train and test are cleaned separately. Everything that learns from the data (encoded columns, scaling) is learned on train only and applied to test, so there is no data leak.
- Removed the dot in the test labels (`>50K.` to `>50K`).
- Deleted rows with missing values: 2,399 in train (7.37%), 1,221 in test (7.50%).
- Deleted duplicate rows: 23 in train, 5 in test.
- Outliers found with z-score and IQR, **all values kept** (they are real values, not errors).
- capital-gain and capital-loss: log(1 + x), skewness of capital-gain 11.9 to 3.07.
- New features: has-capital, is-married, is_us, capital-net, age-group, hours-category.
- Encoding: sex 0/1, native-country to is_us, one-hot for 5 categorical variables. Dropped education (same as education-num) and fnlwgt. 42 columns, numerical ones standardized.

### Slide 4: Correlations with income (train)
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

Note for `03a` (marital status): the top bar, Married-AF-spouse, is only 21 people. Better to say "45.5% of married people earn >50K, only 6.8% of the others".

### Slide 5: Hypothesis tests (train + test together, 45,194 people)
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

### Slide 6: Regression
- Linear: hours-per-week = 27.33 + 0.070 × age + 0.696 × education-num + 5.73 × male. F-test significant (F = 899), but R² = 0.078 on test (MAE 7.7 h): significant, but explains very little.
- Logistic, test set: accuracy 0.849, precision 0.734, recall 0.606, F1 0.664, ROC-AUC 0.905 (always predicting <=50K would give accuracy 0.754).
- Odds ratios: married (Married-civ-spouse) x9.1, relationship Wife x3.7, male x2.4, Exec-managerial x2.2, each step of education-num x1.32, Own-child x0.54.
- Careful: has-capital (0.00), capital-loss-log (x30) and workclass Without-pay (0.00) have extreme odds ratios. has-capital overlaps with the capital log columns and Without-pay has only 14 people, so these coefficients are not reliable. Don't put them on the slide.

### Slide 7: Decision tree
| Tree | Train accuracy | CV F1 |
|---|---|---|
| No limits | 97.7% | 0.624 |
| max_depth = 9 | 86.4% | 0.667 |
| min_samples_leaf = 20 (final) | 87.2% | 0.678 |

- The settings are chosen with 5-fold cross-validation on train. The test set is used only once, for the final tree.
- **Final tree on the test set**: accuracy 0.850, precision 0.742, recall 0.600, F1 0.663, ROC-AUC 0.896.
- Most important features: Married-civ-spouse (0.37), education-num (0.21), capital-gain-log (0.16), age (0.08).
- Rule-based classifier: 131 rules instead of 731 leaves, accuracy 0.839, recall 0.712, F1 0.686.

### Slide 8: Model comparison (test set accuracy)
| Model | Accuracy |
|---|---|
| Always predicting <=50K | 75.4% |
| Naive Bayes (published in adult.names) | 83.9% |
| C4.5 decision tree (published) | 84.5% |
| NBTree (best published) | 85.9% |
| Our logistic regression | 84.9% |
| Our decision tree | 85.0% |

### Slide 9: Clustering
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
| File | Slide |
|---|---|
| `03a_eda_income_by_marital_status.png` | 4 |
| `03b_eda_income_by_education.png` | 4 |
| `03c_eda_income_by_sex.png` | 4 (alternative) |
| `03d_eda_income_by_age_group.png` | 4 (alternative) |
| `03e_eda_correlation_with_income.png` | 4 |
| `05a_linear_actual_vs_predicted.png` | 6 (optional) |
| `05b_logistic_confusion_matrix.png` | 6 |
| `05c_logistic_roc_curve.png` | 6 |
| `06a_tree_f1_vs_depth.png` | 7 (the left half is enough) |
| `06b_tree_feature_importance.png` | 7 |
| `06c_tree_confusion_matrix.png` | 7 (alternative) |
| `08a_clustering_elbow_silhouette.png` | 9 |
| `08b_clustering_pca_plot.png` | 9 |
| `08c_clustering_income_by_cluster.png` | 9 |

## Changes needed in the current Data science.pptx
The slide numbers here are the ones in the current deck (6 = linear regression, 7 = logistic regression, 8 = decision tree).

- Slide 2: typos "eran", "tan", "rase". Use 24.9% (train).
- Slide 3: remove "Age and hours capped at the 1st/99th percentile" (we keep all values now). Duplicates: 23 in train + 5 in test (not 47). Missing rows: 2,399 + 1,221. Add one line about the train/test split.
- Slide 4: education-num 0.34, has-capital 0.31, Own-child -0.23; optionally add capital-gain-log 0.29.
- Slide 5: "Education doesn't depend on education" should be "Income doesn't depend on education". The Pearson row has "p = 0." in the H0 column.
- Slide 6: new equation: 27.33 + 0.070 × age + 0.696 × education-num + 5.73 × male. Add R² = 0.078.
- Slide 7: married x9.1 (not x8.2), Exec-managerial x2.2 (not x2.7), Own-child x0.54 (not x0.47). Add the test results.
- Slide 8: add the test results of the final tree.
- New slides: model comparison, clustering, conclusion.
