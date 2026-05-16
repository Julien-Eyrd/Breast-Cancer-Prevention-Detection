# 🧬 Breast Cancer Survival Prediction: High-Dimensional Genomic Modeling

[![R](https://img.shields.io/badge/R-278A4E?style=flat&logo=r&logoColor=white)](https://www.r-project.org/)
[![Bioinformatics](https://img.shields.io/badge/Domain-Bioinformatics-blue)](https://en.wikipedia.org/wiki/Bioinformatics)
[![Machine Learning](https://img.shields.io/badge/Field-Machine__Learning-orange)](https://en.wikipedia.org/wiki/Machine_learning)

[cite_start]This repository contains an end-to-end data science and bioinformatics project focused on predicting **breast cancer survival outcomes**[cite: 11, 14]. [cite_start]By integrating clinical variables with high-dimensional gene expression profiles ($p \gg n$), we evaluate multiple predictive modeling and regularization strategies to identify critical prognostic patterns and optimize clinical risk stratification[cite: 63, 143, 614, 615].

---

## 📌 Table of Contents
* [Project Overview](#-project-overview)
* [Dataset & High-Dimensionality Challenge](#-dataset--high-dimensionality-challenge)
* [Methodology & Pipeline](#-methodology--pipeline)
* [Model Architecture](#-model-architecture)
* [Key Results & Performance Summary](#-key-results--performance-summary)
* [Key Insights](#-key-insights)
* [Project Structure](#-project-structure)
* [Authors & Acknowledgments](#-authors--acknowledgments)

---

## 🔍 Project Overview

[cite_start]Breast cancer is highly heterogeneous, making accurate survival prediction essential for tailoring patient prognosis[cite: 62]. 

[cite_start]**Objective:** Build optimized classification models to predict patient vital status (`alive` vs. `deceased`) by leveraging both macro-level clinical data and micro-level gene expressions[cite: 63, 64, 595].

---

## 📊 Dataset & High-Dimensionality Challenge

### Data Breakdown
* [cite_start]**Observations:** 1,022 unique patients (after cleaning) [cite: 105]
* [cite_start]**Clinical Variables:** 24 features (e.g., age at diagnosis, tumor histological subtypes, AJCC T-stage) [cite: 67, 577, 672]
* [cite_start]**Genomic Features:** 5,000 continuous gene expression profiles [cite: 68]
* [cite_start]**Target Imbalance:** **Strongly skewed** outcome — only ~12% of patients are deceased[cite: 215, 600].

### The $p \gg n$ Problem
[cite_start]With **5,011 total variables** against only **1,022 observations**, standard Generalized Linear Models (GLMs) fail to converge or severely overfit[cite: 144, 145, 146]. [cite_start]This project implements and compares advanced feature reduction, unsupervised clustering, and regularization techniques to handle this bottleneck[cite: 159, 308, 614].

---

## ⚙️ Methodology & Pipeline

### 1. Data Pre-Processing & Cleaning
* [cite_start]**Factor Conversion:** Categorical and character variables were converted into native R factors[cite: 577].
* [cite_start]**Sparsity Management:** Grouped low-frequency levels to prevent model destabilization during cross-validation[cite: 102, 581, 608].
* [cite_start]**Variance Filtering:** Eliminated uninformative variables with near-zero variance or baseline constants (e.g., gender)[cite: 103, 104, 582].

### 2. Gene Expression Clustering (Exploratory)
[cite_start]To reduce dimensionality, **K-means clustering ($k=5$ or $k=4$)** was applied to the scaled genomic features[cite: 160, 585]. 
* [cite_start]**Validation:** Verified cluster biological relevance against the established **PAM50 intrinsic molecular classification** (Basal-like, Her2-enriched, Luminal A, Luminal B, Normal-like)[cite: 161, 587].
* [cite_start]**Result:** Strong alignment on major subtypes, with expected groupings between the highly similar Luminal A and B classes[cite: 162, 588, 589].

### 3. Splitting & Imbalance Strategies
[cite_start]The data was split into a **70-80% Training / Cross-Validation** set and a **20-30% Untouched Test Set**[cite: 214, 593, 594]. [cite_start]To handle the 12% minority class survival imbalance, we tested two independent approaches[cite: 215, 600]:
1. [cite_start]**Upsampling:** Resampling the training partition to balance both classes explicitly[cite: 217, 601].
2. [cite_start]**Class Weighting:** Incorporating a cost-sensitive loss function directly into model training to heavily penalize misclassifications of deceased patients[cite: 218, 604, 605].

---

## 🤖 Model Architecture

[cite_start]We implemented and optimized a wide spectrum of regression models using `R` and the `glmnet` ecosystem[cite: 614, 617]:

### Baseline Models (Low-Dimensionality Feature Space)
* [cite_start]**Full Logistic Regression:** Baseline model using the clustered representation dataset[cite: 286, 287, 345].
* [cite_start]**Stepwise Variable Selection:** Parsimonious models optimized via **AIC** and **BIC** criteria using greedy forward/backward search[cite: 290, 291, 292, 293, 612].

### Advanced Penalized Regularization (High-Dimensionality Feature Space)
* [cite_start]**Ridge Regression ($L_2$ Penalty):** Stabilizes coefficients under multicollinearity ($\alpha=0$)[cite: 310, 617].
* **Lasso Regression ($L_1$ Penalty):** Drives sparsity, performing automatic feature selection ($\alpha=1$)[cite: 310, 615, 617].
* **Elastic Net:** Balances $L_1$ and $L_2$ penalties via hyperparameter grid search ($\alpha \in [0,1]$)[cite: 310, 619, 620].

### Specialized Sparse Variations
* [cite_start]**Accept-Reject Lasso (ARL):** Uses stability selection to place a strict mathematical upper bound on false feature discoveries (PFER = 10)[cite: 454, 455, 456, 624].
* [cite_start]**Variable Decorrelation Lasso:** Clusters genes by correlation, extracting the $1^{st}$ Principal Component of each cluster to shrink 5,000 genes down to ~50 uncorrelated components[cite: 473, 474, 625].
* **UniLasso:** Employs univariate logistic regression screening to retain only statistically significant genes ($p < 0.05$) prior to full Lasso training[cite: 489, 490, 491, 625].

---

## 📈 Key Results & Performance Summary

Performance was evaluated on the untouched test set using F1-score optimization[cite: 219, 236, 597].

| Model Category | Specific Setup | Optimal Threshold | Recall | F1-Score | Clinical Utility |
| :--- | :--- | :---: | :---: | :---: | :--- |
| **Simple GLM** | Stepwise-AIC (Clustered Data) | 0.50 | 0.667 | 0.386 | Good baseline on aggregated data[cite: 346, 348]. |
| **Advanced Regularization** | **C-EnetUp** ($\alpha=0.10$) | **0.57-0.59** | **0.458** | **0.407** | **Best Balanced Performance.** Optimizes precision-recall trade-off[cite: 418, 508, 551]. |
| **Advanced Regularization** | **C-Lasso** ($\lambda_{1SE}$) | **0.10** | **0.958** | **0.251** | **Maximum Clinical Sensitivity.** Identifies 95.8% of high-risk patients[cite: 403, 509, 552]. |
| **Specialized Sparse** | Decorrelation Lasso | 0.19 | 0.420 | 0.400 | Highly stable; successfully handles deep correlation structures[cite: 476, 511]. |
| **Specialized Sparse** | UniLasso (1,878 genes selected) | 0.13 | 0.500 | 0.300 | 60% feature reduction with solid interpretability[cite: 493, 511]. |

---

## 💡 Key Insights

1. **Raw Genomic Expression Beats Clustering:** Models drawing from the complete 5,000 raw gene expressions (`C-complete`) systematically outperformed those using condensed cluster summaries (`K-clustered`), proving that cluster aggregation drops vital prognostic indicators[cite: 434, 549].
2. **Penalty Selection Changes the Objective:** * **Lasso ($L_1$)** acts as an ideal clinical screening filter by maximizing sensitivity (Recall = 0.958), catching virtually every high-risk case[cite: 436, 509].
   * [cite_start]**Ridge ($L_2$)** offers robust, stable predictions when working with smaller, highly correlated feature clusters[cite: 437].
3. [cite_start]**Threshold Tuning is Crucial:** Because of the 12% survival class imbalance, shifting the optimal threshold to $\sim 0.15$ maximizes F1 performance, while dropping it to $\sim 0.10$ maximizes clinical screening safety (Recall)[cite: 235, 512, 513, 514].

---

## 📁 Project Structure

```text
├── data/                         # (Excl. from GitHub) Raw clinical and gene expression sets
├── src/
│   ├── pre_processing.R          # Variable filtering, factor conversion, and cleanup
│   ├── gene_clustering.R         # K-means execution and PAM50 validation matrix
│   ├── simple_models.R           # Logistic regression with forward/backward stepwise selection
│   ├── penalized_models.R        # Ridge, Lasso, and Elastic Net implementations via glmnet
│   └── advanced_custom_lasso.R   # UniLasso, ARL, and Decorrelation Lasso routines
├── docs/
│   ├── EYRAUD_GOIGNARD_Report.pdf # Detailed mathematical methodology report
│   └── Presentation_Slides.pdf    # Final project presentation slide deck
└── README.md
