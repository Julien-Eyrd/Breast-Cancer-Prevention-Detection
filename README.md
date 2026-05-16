# 🧬 Breast Cancer Survival Prediction: High-Dimensional Genomic Modeling

[![R](https://img.shields.io/badge/R-278A4E?style=flat&logo=r&logoColor=white)](https://www.r-project.org/)
[![Bioinformatics](https://img.shields.io/badge/Domain-Bioinformatics-blue)](https://en.wikipedia.org/wiki/Bioinformatics)
[![Machine Learning](https://img.shields.io/badge/Field-Machine__Learning-orange)](https://en.wikipedia.org/wiki/Machine_learning)

This repository contains an end-to-end data science and bioinformatics project focused on predicting **breast cancer survival outcomes**. By integrating clinical variables with high-dimensional gene expression profiles ($p \gg n$), we evaluate multiple predictive modeling and regularization strategies to identify critical prognostic patterns and optimize clinical risk stratification.

---

## 📌 Table of Contents
* [Project Overview](#-project-overview)
* [Dataset & High-Dimensionality Challenge](#-dataset--high-dimensionality-challenge)
* [Methodology & Pipeline](#-methodology--pipeline)
* [Model Architecture](#-model-architecture)
* [Key Results & Performance Summary](#-key-results--performance-summary)
* [Key Insights](#-key-insights)

---

## 🔍 Project Overview

Breast cancer is highly heterogeneous, making accurate survival prediction essential for tailoring patient prognosis. 

**Objective:** Build optimized classification models to predict patient vital status (`alive` vs. `deceased`) by leveraging both macro-level clinical data and micro-level gene expressions.

---

## 📊 Dataset & High-Dimensionality Challenge

### Data Breakdown
* **Observations:** 1,022 unique patients (after cleaning)
* **Clinical Variables:** 24 features (e.g., age at diagnosis, tumor histological subtypes, AJCC T-stage)
* **Genomic Features:** 5,000 continuous gene expression profiles
* **Target Imbalance:** **Strongly skewed** outcome — only ~12% of patients are deceased.

### The $p \gg n$ Problem
With **5,011 total variables** against only **1,022 observations**, standard Generalized Linear Models (GLMs) fail to converge or severely overfit. This project implements and compares advanced feature reduction, unsupervised clustering, and regularization techniques to handle this bottleneck.

---

## ⚙️ Methodology & Pipeline

### 1. Data Pre-Processing & Cleaning
* **Factor Conversion:** Categorical and character variables were converted into native R factors.
* **Sparsity Management:** Grouped low-frequency levels to prevent model destabilization during cross-validation.
* **Variance Filtering:** Eliminated uninformative variables with near-zero variance or baseline constants (e.g., gender).

### 2. Gene Expression Clustering (Exploratory)
To reduce dimensionality, **K-means clustering ($k=5$ or $k=4$)** was applied to the scaled genomic features. 
* **Validation:** Verified cluster biological relevance against the established **PAM50 intrinsic molecular classification** (Basal-like, Her2-enriched, Luminal A, Luminal B, Normal-like).
* **Result:** Strong alignment on major subtypes, with expected groupings between the highly similar Luminal A and B classes.

### 3. Splitting & Imbalance Strategies
The data was split into a **70-80% Training / Cross-Validation** set and a **20-30% Untouched Test Set**. To handle the 12% minority class survival imbalance, we tested two independent approaches:
1. **Upsampling:** Resampling the training partition to balance both classes explicitly.
2. **Class Weighting:** Incorporating a cost-sensitive loss function directly into model training to heavily penalize misclassifications of deceased patients.

---

## 🤖 Model Architecture

We implemented and optimized a wide spectrum of regression models using `R` and the `glmnet` ecosystem:

### Baseline Models (Low-Dimensionality Feature Space)
* **Full Logistic Regression:** Baseline model using the clustered representation dataset.
* **Stepwise Variable Selection:** Parsimonious models optimized via **AIC** and **BIC** criteria using greedy forward/backward search.

### Advanced Penalized Regularization (High-Dimensionality Feature Space)
* **Ridge Regression ($L_2$ Penalty):** Stabilizes coefficients under multicollinearity ($\alpha=0$).
* **Lasso Regression ($L_1$ Penalty):** Drives sparsity, performing automatic feature selection ($\alpha=1$).
* **Elastic Net:** Balances $L_1$ and $L_2$ penalties via hyperparameter grid search ($\alpha \in [0,1]$).

### Specialized Sparse Variations
* **Accept-Reject Lasso (ARL):** Uses stability selection to place a strict mathematical upper bound on false feature discoveries (PFER = 10).
* **Variable Decorrelation Lasso:** Clusters genes by correlation, extracting the $1^{st}$ Principal Component of each cluster to shrink 5,000 genes down to ~50 uncorrelated components.
* **UniLasso:** Employs univariate logistic regression screening to retain only statistically significant genes ($p < 0.05$) prior to full Lasso training.

---

## 📈 Key Results & Performance Summary

Performance was evaluated on the untouched test set using F1-score optimization.

| Model Category | Specific Setup | Optimal Threshold | Recall | F1-Score | Clinical Utility |
| :--- | :--- | :---: | :---: | :---: | :--- |
| **Simple GLM** | Stepwise-AIC (Clustered Data) | 0.50 | 0.667 | 0.386 | Good baseline on aggregated data. |
| **Advanced Regularization** | **C-EnetUp** ($\alpha=0.10$) | **0.57-0.59** | **0.458** | **0.407** | **Best Balanced Performance.** Optimizes precision-recall trade-off. |
| **Advanced Regularization** | **C-Lasso** ($\lambda_{1SE}$) | **0.10** | **0.958** | **0.251** | **Maximum Clinical Sensitivity.** Identifies 95.8% of high-risk patients. |
| **Specialized Sparse** | Decorrelation Lasso | 0.19 | 0.420 | 0.400 | Highly stable; successfully handles deep correlation structures. |
| **Specialized Sparse** | UniLasso (1,878 genes selected) | 0.13 | 0.500 | 0.300 | 60% feature reduction with solid interpretability. |

---

## 💡 Key Insights

1. **Raw Genomic Expression Beats Clustering:** Models drawing from the complete 5,000 raw gene expressions (`C-complete`) systematically outperformed those using condensed cluster summaries (`K-clustered`), proving that cluster aggregation drops vital prognostic indicators.
2. **Penalty Selection Changes the Objective:**
   * **Lasso ($L_1$)** acts as an ideal clinical screening filter by maximizing sensitivity (Recall = 0.958), catching virtually every high-risk case.
   * **Ridge ($L_2$)** offers robust, stable predictions when working with smaller, highly correlated feature clusters.
3. **Threshold Tuning is Crucial:** Because of the 12% survival class imbalance, shifting the optimal threshold to $\sim 0.15$ maximizes F1 performance, while dropping it to $\sim 0.10$ maximizes clinical screening safety (Recall).

