# TECHNICAL REPORT: FRAUD DETECTION WITH MACHINE LEARNING

## Table of Contents

1. [Data Preprocessing](#1-data-preprocessing)
   - A. Examining the Dataset
   - B. Data Cleaning
   - C. Feature Engineering
   - D. Outlier Analysis & Scaling & Covariance
2. [Model Training and Performance Measurement](#2-model-training-and-performance-measurement)
   - E. Logistic Regression (LR)
   - F. Random Forest (RF)
   - G. XGBoost (XGB)
   - H. Balanced Random Forest (BRF)
   - I. Ensemble Learning and Voting Classifier
   - J. Feature Selection
   - K. Performance Improvement Efforts
3. [Model/Performance Evaluation](#3-modelperformance-evaluation)
4. [Model Deployment (REST API)](#4-model-deployment-rest-api)

---

## 1) Data Preprocessing

In this stage of the project, the dataset was examined and cleaned with the aim of reaching the most information with the fewest variables.

The initial features were selected from the cleaned data and model training began. If the results are not as desired, the feature selection may be changed later.

### A. Examining the Dataset

The dataset of 849,564 rows was examined in terms of columns, values, and data types. Inferences were made about the columns that may be useful in fraud detection, those that will not be used, those that will be encoded numerically, those that will be scaled, and those that contain null values. After the general dataset check, the rows where IsFraudTransaction is 1 were filtered and examined in order to understand the fraudulent transactions. Ideas were developed about columns that may be important, and the values of the columns were examined. These inferences were used when selecting features.

Some inferences were made based on the initial examination:

- There are invalid device models (KUVEYT TURK KATILIM BANKASI ÇEKİLİŞ ONAY, KUVEYT TURK KATILIM BANKASI CAGRI MERKEZI).
- Fraud is observed in transactions with the same IP_Subnet (91.151.xxx, 178.240.xxx).
- Transactions with a low unique IP count may be closer to being fraud.
- CustomerTenure may be important; if it is high, the transaction may not be fraud.

Although some inconsistencies and patterns were detected in the data, many of them appeared to stem from the synthetic nature of the data and were not taken into account:

- The title in the receiver's name does not match the receiver's occupation. The receiver's name and gender do not match. Education information and occupation do not match (e.g., a high-school graduate who is a dentist).
- Such inconsistencies affected the choices made in grouping the data. For example, while occupations can normally be grouped by income level, in this case grouping by frequency was preferred.

### B. Data Cleaning

Unique values and strings that we cannot make sense of without NLP/LLMs (BusinessKey, ReceiverName, SenderName, CustomerName) were removed. The TransactionChannel column, which contains only a single value, was also removed.

The following null-handling strategies were applied to the columns containing null values:

- **DayType:** Null handling was performed by converting public holiday information to numeric form.
- **CustomerMaritalStatus:** Since the number of null values is small enough to be ignored (188/849564), these rows were removed.
- **CustomerEducation:** After converting the column to numeric form with ordinal encoding according to education levels, the average education value was assigned to the null records.

Categorical data was converted to numeric form with encoding.

- Since the values of the CustomerSegment, TransactionType, DeviceOSName, DayType, IsFractionalAmount, and CustomerEducation columns are known and ordered, these columns were converted to numeric form with ordinal encoding.
- CustomerMaritalStatus, CustomerGender, CustomerProfession (after grouping), and CustomerAge (after binning) were converted to numeric form with one-hot encoding.
  - Since CustomerProfession has high cardinality, the use of target encoding was also considered. It was not preferred because it could cause overfitting for occupations with few transactions.
  - One-hot encoding was preferred for CustomerAge because it can capture non-linear behavior.

### C. Feature Engineering

The following new features were derived to improve model performance:

- **CustomerProfession Grouping:** The 100+ titles were simplified into the 10 (N) most frequent groups and 'Other'. The value of N may be changed based on the results. High values of N were not preferred in order to prevent overfitting.
- **IsFlaggedDeviceModel:** Invalid/suspicious strings in device names were examined. Suspicious models that contain words that make business sense, such as 'Kuveyt Turk' and 'Onay', or that are longer than a valid device model (35 characters), were flagged.
- **TransactionDate:** For the purpose of pattern detection, the date information was split into hour, day of the week, and month.
- **RecentSimilarTransactionsCount:** The velocity of transactions with similar amounts within the last t hours was calculated with optimal values. When TP/FP analysis was performed, it was seen that it did not give successful results.
  - Different time intervals (1h, 2h, 3h, 6h) were tried.
  - Groupings by AccountNumber and/or DeviceId were tried.
  - Different TransactionAmount roundings (100, 1000) were tried.
- **CustomerAge:** The CustomerAge data was grouped (binning) into bins created according to business logic (18-25, 25-40, 40-55, 55+). The optimal bins were selected based on the density of the bins tried.
- **IP and Subnet Analysis:** Transactions with the same IP_Subnet and a low UniqueIPCount were observed to be suspicious. These assumptions were turned into inferences by grouping the IP_Subnet data by frequency and by outlier analysis for the UniqueIPCount data.
  - Subnets with a fraud rate higher than 15% and that received at least 5 transactions were flagged as IsHighRiskIPSubnet.
  - A risk flag was created for UniqueIPCount based on the Q1 value obtained from the outlier analysis.
    - Because of the high number of FP values, a better value for the risk threshold was investigated. However, since the volume of flagged data decreased greatly while precision increased, it was decided to continue with Q1 unless feature optimization is required.
- **IsSharedDevice:** Cases where multiple accounts log in from the same device were detected.

### D. Outlier Analysis & Scaling & Covariance

The Interquartile Range (IQR) method was used for outlier analysis. In the TransactionAmount and CustomerTenure columns, the fraud rate among the outliers was found to be twice the global rate. This shows that high-amount transactions and long-tenured customers are more likely to indicate fraudulent transactions. These data points were not deleted; instead, they were scaled using the median-based RobustScaler, which is less affected by outliers.

The covariances of the generated features were checked. Linearly related features should be removed so that highly correlated features (r > 0.9) do not affect model performance. No column pair with a correlation above 90% was found.

---

## 2) Model Training and Performance Measurement

Since the dataset is imbalanced, a Stratified Split was used. The data was split into 80% train and 20% test. All features processed with the goal of predicting IsFraudTransaction were given to the models initially.

### E. Logistic Regression (LR)

It was configured for the imbalanced dataset (class_weight='balanced') and tried with all features.

| Class | Precision | Recall | F1-Score | ROC AUC Score |
|:-----:|:---------:|:------:|:--------:|:-------------:|
| 0 | 1.00 | 0.94 | 0.97 | 0.9592 |
| 1 | 0.08 | 0.85 | 0.15 | |

*Table 1: Logistic Regression performance measurement*

The Recall value is high, as desired, but the Precision value is very low. Since there are many FPs, it produces false alarms.

### F. Random Forest (RF)

It was tried with 100 decision trees (n_estimators=100) and balanced weighting (class_weight='balanced').

| Class | Precision | Recall | F1-Score | ROC AUC Score |
|:-----:|:---------:|:------:|:--------:|:-------------:|
| 0 | 1.00 | 1.00 | 1.00 | 0.9171 |
| 1 | 0.91 | 0.47 | 0.62 | |

*Table 2: Random Forest performance measurement*

The Precision value is high, as desired, but the Recall value is very low for fraud detection. It produces very few false alarms, but misses more than half of the fraudulent transactions.

### G. XGBoost (XGB)

It was tried with scaled weighting according to the minority fraud data.

| Class | Precision | Recall | F1-Score | ROC AUC Score |
|:-----:|:---------:|:------:|:--------:|:-------------:|
| 0 | 1.00 | 0.97 | 0.98 | 0.9529 |
| 1 | 0.13 | 0.80 | 0.23 | |

*Table 3: XGBoost performance measurement*

The Recall value is high, as desired, but the Precision value is very low. Since there are many FPs, it produces false alarms. It gives better results than LR, and since Recall is prioritized for fraud detection, it is more preferable than RF.

### H. Balanced Random Forest (BRF)

It was tried because it was developed specifically for imbalanced datasets. Although it gives more accurate results than RF for the fraud class, this method was not continued with because RF balances the Voting Classifier.

| Class | Precision | Recall | F1-Score | ROC AUC Score |
|:-----:|:---------:|:------:|:--------:|:-------------:|
| 0 | 1.00 | 0.93 | 0.96 | 0.9588 |
| 1 | 0.07 | 0.87 | 0.13 | |

*Table 4: Balanced Random Forest performance measurement*

### I. Ensemble Learning and Voting Classifier

The LR and XGB results show high recall and low precision, while the RF results show the opposite. A method that combines these results was tried. The previously trained and evaluated models (LR + RF + XGB) were combined with a Soft Voting Ensemble. It was also tried with BRF instead of RF, but since BRF gave results close to the other two models, the success decreased when the average was taken.

| Class | Precision | Recall | F1-Score | ROC AUC Score |
|:-----:|:---------:|:------:|:--------:|:-------------:|
| 0 | 1.00 | 0.99 | 0.99 | 0.9642 |
| 1 | 0.31 | 0.75 | 0.44 | |

*Table 5: Voting Classifier (LR + RF + XGB) performance measurement*

|  | Predicted 0 | Predicted 1 |
|:--|:-----------:|:-----------:|
| **Actual 0** | TN: 167060 | FP: 1748 |
| **Actual 1** | FN: 272 | TP: 796 |

*Table 6: Voting Classifier Confusion Matrix*

Precision and F1 Score improve considerably. No serious drop is seen in Recall either. It is the model with the highest F1 Score and ROC AUC Score. It was decided to use this method.

StratifiedKFold was applied to the selected model to ensure the reliability (robustness) of the results.

| Metric | Result |
|:--|:--|
| Average Recall | 0.7394 (+/- 0.0154) |
| Average Precision | 0.3030 (+/- 0.0079) |
| Average ROC AUC | 0.9566 (+/- 0.0041) |

*Table 7: Voting Classifier Cross Validation results*

### J. Feature Selection

To improve the initial performance obtained using all features, different feature selections were tried with Feature Importance, L1 Regularization (Lasso), and Recursive Feature Elimination.

In tree-based models (Random Forest, XGBoost), the features that affect the result the most/least were found by looking at Feature Importance. Performance was measured by experimentally removing the N least important features.

- Even though the features are not important on their own, since they can be used relationally, there was no major change in the results, and there were occasional decreases. This approach was abandoned.

| Class | Precision | Recall | F1-Score | ROC AUC Score |
|:-----:|:---------:|:------:|:--------:|:-------------:|
| 0 | 1.00 | 0.97 | 0.98 | 0.9480 |
| 1 | 0.13 | 0.79 | 0.23 | |

*Table 8: XGBoost performance with the 10 least important features (<0.0067) removed*

For LR, the coefficients of unimportant features were reduced by applying a penalty with L1 Regularization, and the change in performance was examined.

- Different penalty coefficients (C) were tried. For C = 0.01, 6 features were eliminated and the result improved very slightly. However, since the initial Precision result for LR was very low, this improvement was not sufficient.

| Class | Precision | Recall | F1-Score | ROC AUC Score |
|:-----:|:---------:|:------:|:--------:|:-------------:|
| 0 | 1.00 | 0.99 | 0.99 | 0.9642 |
| 1 | 0.31 | 0.75 | 0.44 | |

*Table 9: LR performance with 6 features removed for C = 0.01*

Recursive Feature Elimination (RFE) was tried for LR, and the optimal number of features was found to be 33. The Transaction_day_of_week and Transaction_month features were eliminated. No improvement was seen in the results when the suggested feature selection was applied.

| Class | Precision | Recall | F1-Score | ROC AUC Score |
|:-----:|:---------:|:------:|:--------:|:-------------:|
| 0 | 1.00 | 0.94 | 0.97 | 0.9591 |
| 1 | 0.08 | 0.85 | 0.15 | |

*Table 10: LR performance after RFE*

The feature selection methods that were tried did not provide the expected improvement. Therefore, all features were used going forward.

### K. Performance Improvement Efforts

Improvement methods suggested in the literature for imbalanced datasets were tried:

- **StratifiedKFold:** In KFold Cross-Validation, the data is split into k subsets. Variance is reduced by iterating over these subsets. With StratifiedKFold, the splits are stratified (balanced for the minority data).
  - Although StratifiedKFold did not directly improve the results, it confirms that the results obtained so far are more reliable (robust). It provides stability by reducing variance. It also makes feature selection more stable. A feature that does not work well in one subset (fold) may give good results in another.
- **Balanced Bagging Classifier:** It trains multiple classifiers on different subsets and averages the results. It balances the majority class to the minority class with under-sampling, reduces bias, and improves the performance of the minority class.
  - When tried with XGBoost, the Balanced Bagging Classifier made Recall very sensitive because it under-samples from the majority class, so it is more sensitive to fraud patterns. However, since this increased FP greatly, Precision dropped very low.

| Class | Precision | Recall | F1-Score | ROC AUC Score |
|:-----:|:---------:|:------:|:--------:|:-------------:|
| 0 | 1.00 | 0.62 | 0.77 | 0.9544 |
| 1 | 0.02 | 0.97 | 0.03 | |

*Table 11: Balanced Bagging Classifier performance with XGB*

  - Therefore, in the expectation of getting better results, the method was also tried with Random Forest, which has higher Precision and lower Recall. As expected, the RF results became more balanced. However, even with this improvement, the XGBoost results are more advantageous, so its use was not preferred.

| Class | Precision | Recall | F1-Score | ROC AUC Score |
|:-----:|:---------:|:------:|:--------:|:-------------:|
| 0 | 1.00 | 0.96 | 0.98 | 0.9615 |
| 1 | 0.11 | 0.85 | 0.19 | |

*Table 12: Balanced Bagging Classifier performance with RF*

---

## 3) Model/Performance Evaluation

Since fraud detection is also done with imbalanced datasets in real life, interpreting and prioritizing the metrics is very important.

In the model choices and improvements made, Recall (Sensitivity) was evaluated first. Catching the majority of fraudulent transactions prevents the financial losses and legal problems the bank would experience. Precision is considered next. High False Positives (low precision) create false alarms for non-fraudulent transactions. Since customers' transactions may be put under review/blocked, user experience is negatively affected. The F-1 Score is looked at for the harmonic mean of these two values. The model's ability to distinguish the classes from each other is also measured with ROC AUC.

| Model | Precision | Recall | F1-Score | ROC AUC |
|:--|:-:|:-:|:-:|:-:|
| LR | 0.08 | 0.85 | 0.15 | 0.9592 |
| RF | 0.91 | 0.47 | 0.62 | 0.9171 |
| XGB | 0.13 | 0.80 | 0.23 | 0.9529 |
| BRF | 0.07 | 0.87 | 0.13 | 0.9588 |
| Ensemble | 0.31 | 0.75 | 0.44 | 0.9642 |

*Table 13: Performance comparison of the models tried*

RF is the most successful model in terms of F1-Score for the first comparison (LR-RF-XGB). Its Precision is quite high, and its Recall is also better than the Precision values of the other models. However, since the cost of missing a transaction in fraud detection can be very high, a high Recall value is prioritized.

The Recall values for LR and XGB are high, as desired. However, their Precision values are quite low. Since this will increase FPs and false alarms, various methods were tried to improve them.

BRF, which was later developed for imbalanced datasets, was also tried, but it was not used because RF achieves higher success by balancing the Voting Classifier.

Since the 3 models gave different Precision and Recall results, a method that would combine these methods was researched, and Ensemble Learning was applied to these models.

The previously trained and evaluated models (LR + RF + XGB) were combined with a Soft Voting ensemble. As seen in Table 13, Precision and F1 Score improve considerably, and no serious drop is seen in Recall. It was decided to use this method.

Ensemble Learning was also tried with the RF improved by applying the Balanced Bagging Classifier, but the results were comparatively less successful. Therefore, it was decided to use LR + RF + XGB. It was proven with StratifiedKFold that the model works robustly.

| Base Model | Method Applied | Δ Precision | Δ Recall | Δ F1-Score | Δ ROC AUC |
|:--|:--|:-:|:-:|:-:|:-:|
| LR | Lasso (C=0.01) | 0.23 | -0.10 | 0.29 | 0.0050 |
| LR | RFE (33 Features) | 0.00 | 0.00 | 0.00 | -0.0001 |
| XGB | Feature Selection (-10) | 0.00 | -0.01 | 0.00 | -0.0049 |
| XGB | Balanced Bagging | -0.11 | 0.17 | -0.20 | 0.0015 |
| RF | Balanced Bagging | -0.80 | 0.38 | -0.43 | 0.0444 |

*Table 14: Effect of feature selection and improvement attempts on performance*

To improve the results, L1 Regularization (Lasso) and Recursive Feature Elimination were tried for LR; the Balanced Bagging Classifier for RF; and Feature Importance, StratifiedKFold, and the Balanced Bagging Classifier for XGB. The Balanced Bagging Classifier was successful for RF and its Recall level was improved, but it remained below XGB.

In the future, the importance of the metrics may change according to the bank's strategic decisions, and the choices may be updated. The results may be improved by trying different feature thresholds. Features that are thought not to affect the result when eliminated may not be generated at all during data preparation.

---

## 4) Model Deployment (REST API)

The trained Ensemble model was turned into a REST API using Flask. The API passes the incoming raw data through all of the processing steps in the notebook (preprocessing & alignment) and produces real-time predictions. A test scenario was created with the test set, and the API's operation was tested.
