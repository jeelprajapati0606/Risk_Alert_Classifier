# 🚨 Risk Alert Classifier

An early-warning system for a digital banking platform that predicts **high-risk customers** (likely to default on payments or commit fraud). The project handles **highly imbalanced data**, compares several classifiers with the right metrics, and improves them with **hyperparameter tuning**.

## 📌 Problem
- **Target:** `risk_status` (0 = Low Risk, 1 = High Risk)
- **Data:** 4,600 customers, 17 raw features (demographic, transaction activity, credit behaviour)
- **Imbalance:** only about 12% of customers are high-risk
- **Business goal:** catch as many high-risk customers as possible (**minimise false negatives**)

## 📁 Repository Structure
```
├── Risk_Alert_Classifier.ipynb              # Full project (Parts A-H) with outputs
├── Risk_Alert_Classifier_Data.csv           # Dataset (put your copy here)
├── Risk_Alert_Theory_and_Final_Report.pdf   # Theory definitions + final report
├── model_comparison_results.csv             # Evaluation table of all models
├── stress_test_results.csv                  # Imbalance stress-test results
├── figures/                                 # All saved plots
└── README.md
```

## 🚀 How to Run
```bash
pip install pandas numpy matplotlib seaborn scikit-learn imbalanced-learn jupyter
jupyter notebook Risk_Alert_Classifier.ipynb
```
Then choose **Kernel → Restart & Run All**. Keep the CSV in the same folder as the notebook. Runtime is a few minutes.

## 🧭 What the Notebook Covers
| Part | Content |
|---|---|
| **A** | Theory: Logistic Regression, metrics, Type-I/II error, Precision/Recall/F1/TPR/FPR, AUC-ROC, imbalance |
| **B** | Features and target, stratified train-test split, missing values with **KNN Imputer** |
| **C** | Baseline Logistic Regression, confusion matrix, metrics, Type-I and Type-II errors |
| **D** | Impact of imbalance; Under-Sampling, Over-Sampling, **SMOTE**, **ADASYN**; before/after comparison |
| **E** | Decision Tree (overfitting check) and Random Forest |
| **F** | **Randomized Search CV**, **Grid Search CV**, tuned vs untuned |
| **G** | ROC curves, AUC-ROC comparison, best-model selection (minimise false negatives) |
| **H** | Final report, business interpretation, recommendations |

## 🔧 Method in Short
1. Drop `customer_id`; turn `last_transaction_date` into `days_since_last_txn`.
2. Stratified 80/20 split (class ratio kept in train and test).
3. Numeric: scale then **KNN Imputer (k=5)**. Categorical: most-frequent fill then one-hot. Fitted on **train only** (no data leakage).
4. Resampling is applied **only on training data**; the test set is never touched.
5. Models are compared with Recall (High Risk), F1, AUC-ROC and the confusion matrix, not accuracy alone.

## 📊 Key Findings
- A dummy model that always says "Low Risk" gets about **88% accuracy but 0% recall**, so accuracy is misleading.
- On this dataset **Logistic Regression is already near-perfect** (5-fold CV AUC about 0.9995). There is no duplicate data or target leakage; the label is almost a linear combination of several features (looks like a synthetic practice dataset).
- Because of that, balancing barely matters on the full feature set. A **stress test with 4 weaker features** shows the real imbalance effect: recall rises from about 0.76 to about 0.95 after balancing, with lower precision, and AUC unchanged.
- The unrestricted **Decision Tree overfits** (100% train vs 97% test accuracy); the **Random Forest generalises much better**.
- Tuning clearly helps the Decision Tree (recall about 0.85 to 0.94); the Random Forest was already near-perfect.
- **Selected model:** chosen by the rule *"highest recall among models within 0.03 AUC of the best, then F1, then simpler model"*. Logistic Regression and the tuned Random Forest are practically tied.

> ⚠️ The test set has only 111 high-risk customers, so a 1-2 customer difference between models is not meaningful. Validate on fresh real data before production.

## 💼 Business Meaning
- **False Positive (Type-I):** a safe customer is flagged, which means a cheap manual review.
- **False Negative (Type-II):** a risky customer is missed, which means default or fraud loss. This is much costlier.
- Flagged customers should go to a **review queue**, and the model should be monitored and retrained regularly.

## 🔮 Future Work
Probability-threshold tuning, Gradient Boosting / XGBoost, more behavioural features, model monitoring.

## 🛠 Tech Stack
Python, pandas, NumPy, scikit-learn, imbalanced-learn, matplotlib, seaborn
