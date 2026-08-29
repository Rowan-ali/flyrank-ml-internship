# Ranking Potential Content `went_dark` Risk Using Search and Engagement Signals

## Abstract

This study investigates whether February search and engagement signals can be used to rank eligible content observations according to their subsequent likelihood of becoming `went_dark` during the March outcome window. A Logistic Regression model was developed using five predictive features derived exclusively from February 2026 data while maintaining strict temporal separation between predictors and outcomes. To reduce client-level information leakage, model validation was performed using a client-grouped split, and model performance was evaluated primarily with Precision@50, reflecting the practical objective of prioritizing a limited review queue. On the held-out validation set, the selected model achieved a Precision@50 of **0.280**, correctly identifying **14 confirmed `went_dark` observations within the highest-ranked 50 predictions**, with a ROC-AUC of **0.775**. These findings support the use of the model as a decision-support tool for prioritizing content review while avoiding causal claims or guarantees regarding future search performance.

# 1. Introduction / Problem Statement

Search performance changes over time, making it difficult for analysts to determine which content should be reviewed first when resources are limited. Rather than treating every observation equally, organizations benefit from a ranking approach that prioritizes the observations most likely to require attention.

This study addresses the following research question:

> **Which eligible content observations can be ranked most effectively for potential `went_dark` risk using February search and engagement signals?**

The analysis supports a prioritization decision rather than a causal explanation. February 2026 search and engagement signals are used to construct predictive features, while March 2026 observations are used exclusively to define the `went_dark` outcome. This temporal separation prevents future information from influencing model training.

Because the positive outcome is relatively uncommon, the primary evaluation metric is **Precision@50**, which measures how many confirmed positive observations appear within the highest-ranked 50 predictions. Logistic Regression was selected as the final model based on its ranking performance and validation results.






# 2. Data

## 2.1 Data Source

This study was conducted using the **FlyRank ML Internship warehouse release**, accessed through the Hugging Face-hosted internship dataset and queried using DuckDB. The analysis was performed using public-safe identifiers, and no client names, domains, URLs, private search queries, credentials, or raw warehouse exports are included in the published outputs.

## 2.2 Feature and Outcome Windows

A strict temporal design was used throughout the study.

* **Feature window:** February 2026
* **Outcome window:** March 2026

All predictive variables were derived exclusively from the February feature window. March data were used only to construct the `went_dark` outcome variable and were not included as predictive features. This separation reduces the risk of temporal leakage and reflects the intended forecasting scenario.

## 2.3 Eligibility Criteria

The initial modeling population was restricted to observations that satisfied the predefined eligibility requirements during the February feature window. Eligible observations were required to have:

* Google Search Console data available.
* At least **100 impressions** during February 2026.
* At least **3 clicks** during February 2026.

These criteria exclude observations with insufficient search activity while preserving a consistent population for model development.

## 2.4 Outcome Definition

The target variable was defined as **`went_dark`**, representing observations that recorded **zero Google Search Console clicks during March 2026**.

The outcome was constructed only after the February feature window had ended, ensuring that future outcome information was not used during feature construction or model training.

## 2.5 Final Modeling Population

Following the validated data-construction pipeline, the final modeling population contained:

* **29,380 client-content observations**
* **30 unique clients**
* **1,182 positive (`went_dark`) observations**
* **28,198 negative observations**
* **Positive class rate: 4.023%**

Each observation represents a unique client-content pair.

## 2.6 Predictive Features

The final model uses five predictive variables derived from February 2026:

1. `gsc_impressions`
2. `gsc_avg_position`
3. `ga4_sessions`
4. `ga4_users`
5. `ga4_engaged_sessions`

These variables summarize search visibility and user engagement during the feature window and serve as the exclusive inputs to the predictive model.

## 2.7 Missing Data

Missing values were present in several Google Analytics 4 engagement variables. Rather than removing these observations, the complete validated modeling population was retained.

Missing predictor values were handled through **median imputation within the machine-learning pipeline**, with imputation statistics estimated using the training data only. This approach preserves the full modeling population while preventing information from the validation data from influencing preprocessing.

## 2.8 Data Summary

The final dataset provides a temporally consistent and public-safe modeling population suitable for ranking observations according to their subsequent `went_dark` risk. All reported results in the following sections are based on this validated population and the predefined feature and outcome windows.








# 3. Methodology

## 3.1 Research Design

This study was designed as a supervised learning ranking task. Rather than estimating the exact probability of future search performance, the objective was to prioritize eligible content observations according to their likelihood of becoming `went_dark` during the March 2026 outcome window.

The modeling pipeline follows a strict temporal design in which predictor variables are derived exclusively from February 2026, while the target is constructed only from March 2026 observations. This approach ensures that future information is not used during feature engineering or model training.

## 3.2 Target Variable

The prediction target is the binary variable `went_dark`.

An observation is labeled as positive (`went_dark = 1`) when it records **zero Google Search Console clicks during March 2026**. Otherwise, it is labeled as negative (`went_dark = 0`).

The target is created only after the feature window has ended, preventing temporal leakage.

## 3.3 Predictive Features

Five predictive variables were selected based on search visibility and user engagement during February 2026:

* `gsc_impressions`
* `gsc_avg_position`
* `ga4_sessions`
* `ga4_users`
* `ga4_engaged_sessions`

These variables summarize search exposure and user behavior before the prediction period and represent the complete feature set used by the final model.

## 3.4 Missing Data Strategy

Several Google Analytics 4 variables contained missing values. Instead of removing these observations, missing predictor values were handled through **median imputation** within the machine-learning pipeline.

The imputer was fitted using the training data only and then applied to the validation data. This procedure prevents information from the validation set from influencing preprocessing.

## 3.5 Model Selection

The final predictive model is **Logistic Regression**.

A Week-4 baseline served as the reference model for comparison. Model selection was based primarily on **Precision@50**, reflecting the practical objective of ranking a limited number of observations for manual review. When competing models achieved the same Precision@50, **ROC-AUC** was used as a secondary criterion to select the final model.

## 3.6 Validation Design

Model evaluation used a **client-grouped train-validation split** to prevent information leakage between related observations.

The final split consisted of:

* **Training observations:** 16,417
* **Validation observations:** 12,963
* **Training clients:** 24
* **Validation clients:** 6
* **Client overlap:** 0

This grouped validation strategy evaluates model performance on previously unseen clients and provides a more realistic estimate of generalization.

## 3.7 Evaluation Metrics

The primary evaluation metric was **Precision@50**, measuring the proportion of confirmed positive observations among the highest-ranked 50 predictions.

This metric aligns with the operational objective of prioritizing a small review queue rather than maximizing overall classification accuracy.

As a secondary evaluation metric, **ROC-AUC** was calculated on the same validation population to assess the model's ability to discriminate between positive and negative observations across all probability thresholds.

## 3.8 Leakage Prevention

Several safeguards were incorporated to reduce the risk of data leakage throughout the modeling process:

* Feature variables were derived exclusively from February 2026.
* March data were used only to construct the target variable.
* Median imputation was learned from the training data only.
* Training and validation sets contained different clients.
* Predictions were generated exclusively for the held-out validation population.

These safeguards ensure that the reported performance reflects information available at prediction time.

## 3.9 Methodological Summary

The final methodology combines a temporally consistent feature design, client-grouped validation, and probability-based ranking using Logistic Regression. This framework supports reliable prioritization of potential `went_dark` observations while maintaining reproducibility and minimizing information leakage.









# 4. Results

## 4.1 Validation Performance

The final Logistic Regression model was evaluated on a held-out validation population containing **12,963 observations**. Model performance was assessed using the predefined evaluation protocol described in the methodology.

The primary evaluation metric was **Precision@50**, reflecting the practical objective of ranking a limited number of observations for manual review. A secondary evaluation using **ROC-AUC** was also performed to assess the model's overall discrimination ability across probability thresholds.

## 4.2 Model Comparison

The selected Logistic Regression model was compared with the Week-4 baseline using the same validation population and evaluation procedure.

| Model               | Precision@50 |   ROC-AUC |
| ------------------- | -----------: | --------: |
| Week-4 Baseline     |        0.180 |         — |
| Logistic Regression |    **0.280** | **0.775** |

The Logistic Regression model improved Precision@50 from **0.180** to **0.280**, representing a substantial improvement in the concentration of positive observations within the highest-ranked recommendations.

## 4.3 Top-50 Ranking Performance

The final ranking produced a prioritized review queue consisting of the **50 highest-scoring observations**.

Within this Top-50 ranking:

* **14 observations** were confirmed as `went_dark`.
* **Precision@50 = 0.280**.
* Predictions were generated using estimated probabilities from the Logistic Regression model and ordered from highest to lowest predicted risk.

These results demonstrate that the model effectively concentrates positive observations near the top of the ranked list, making it suitable for prioritization rather than exhaustive classification.

## 4.4 Model Selection

Logistic Regression was selected as the final model because it achieved the strongest overall ranking performance on the predefined validation protocol.

The selection process followed the predefined decision rule:

1. Maximize **Precision@50**.
2. If competing models achieved identical Precision@50 values, select the model with the higher **ROC-AUC**.

Under this framework, Logistic Regression was chosen as the final production model.

## 4.5 Interpretation of Results

The evaluation indicates that the selected model provides meaningful decision-support for prioritizing content review.

Rather than attempting to identify every future `went_dark` observation, the model is designed to rank observations according to relative risk. This ranking enables analysts to focus limited review resources on the observations with the highest predicted likelihood of requiring attention.

The reported results should therefore be interpreted as evidence of ranking effectiveness within the evaluated validation population.

## 4.6 Results Summary

Across the held-out validation population, the Logistic Regression model consistently outperformed the Week-4 baseline according to the predefined evaluation protocol. The combination of improved Precision@50 and a ROC-AUC of **0.775** supports the selection of Logistic Regression as the final ranking model for this study.

## Figure 1. Precision@50 Comparison

![Precision@50](images/precision_at_50.png)

**Figure 1.** Comparison of Precision@50 between the Week-4 baseline and the final Logistic Regression model.

## Figure 2. ROC-AUC

![ROC-AUC](images/roc_auc.png)

**Figure 2.** Validation ROC-AUC achieved by the final Logistic Regression model.

## Figure 3. Top-50 Recommendation Composition

![Top50](images/top50_distribution.png)

**Figure 3.** Composition of the top-50 ranked recommendations.












# 5. Limitations & Honest Framing

## 5.1 Interpretation of Findings

The results presented in this study should be interpreted as evidence of predictive ranking performance within the evaluated validation population. The objective of the model is to prioritize observations according to their estimated likelihood of becoming `went_dark`, rather than to explain why individual observations experience changes in search performance.

Accordingly, the reported findings should be viewed as decision-support evidence rather than causal conclusions.

## 5.2 Dataset Scope

The analysis is based exclusively on the FlyRank ML Internship warehouse release and the predefined February–March 2026 study period.

Consequently, the reported performance reflects the characteristics of this evaluation dataset and should not be interpreted as evidence that identical performance will be achieved for different clients, industries, time periods, or search environments.

## 5.3 Class Imbalance

The final modeling population contains **29,380 observations**, of which **1,182** are positive (`went_dark`), corresponding to a positive class rate of approximately **4.02%**.

This level of class imbalance is expected for the studied outcome and reinforces the choice of a ranking-based evaluation metric rather than overall classification accuracy.

## 5.4 Missing Predictor Values

Several Google Analytics 4 engagement variables contain missing values.

Rather than excluding these observations, missing predictor values were handled using median imputation within the machine-learning pipeline. Although this approach preserves the validated modeling population, missing engagement data may reduce the amount of information available for certain observations.

## 5.5 Validation Design

Model evaluation was performed using a client-grouped validation strategy in which no client appears in both the training and validation populations.

This validation design provides a more realistic assessment of generalization while reducing the risk of client-level information leakage.

## 5.6 Practical Interpretation

The final Logistic Regression model achieved:

* **Precision@50 = 0.280**
* **14 confirmed `went_dark` observations within the Top-50 ranked recommendations**
* **ROC-AUC = 0.775**

These results indicate that the model successfully concentrates positive observations toward the top of the ranking, making it suitable for prioritizing manual review efforts.

However, the model should not be interpreted as guaranteeing that highly ranked observations will become `went_dark`, nor that lower-ranked observations are free from future risk.

## 5.7 Appropriate Use

The proposed model is intended to support prioritization decisions by identifying observations that may warrant earlier investigation.

It should be used alongside domain expertise and routine content review rather than as an automated replacement for analyst judgment.

## 5.8 Key Takeaway

Overall, the evidence supports the use of the selected Logistic Regression model as a **ranked prioritization tool** for identifying observations that may require earlier review. Within the evaluated validation population, the model improved the concentration of confirmed `went_dark` observations near the top of the recommendation list while maintaining a reproducible, leakage-aware evaluation framework.

These findings are intentionally presented as **observed predictive evidence** within the evaluated dataset. They should not be interpreted as causal explanations, universal search-engine behavior, or guarantees of future content performance. Instead, they provide practical decision-support for prioritizing limited review resources based on estimated relative risk.











# 6. Ranked Recommendations

## 6.1 Decision-Support Objective

The primary purpose of the proposed model is to support content prioritization rather than automated decision-making. By ranking observations according to their estimated `went_dark` risk, analysts can focus limited review resources on the observations most likely to require attention.

Recommendations are therefore organized as a prioritized review queue instead of binary predictions.

## 6.2 Priority Tiers

The ranked recommendations are divided into three operational priority levels based on the predicted probability generated by the final Logistic Regression model.

### Tier 1 — Immediate Review

The highest-ranked observations represent the strongest predicted risk within the validation population.

These observations should receive the earliest manual review because they have the greatest estimated likelihood of becoming `went_dark` relative to other eligible observations.

### Tier 2 — High-Priority Review

Observations in the second priority tier demonstrate elevated predicted risk but fall below the highest-ranked recommendation group.

These observations should be reviewed after Tier 1 and monitored for additional evidence before major content decisions are made.

### Tier 3 — Monitor

Lower-ranked observations remain eligible for periodic monitoring but do not require immediate manual investigation based on the current model output.

Routine monitoring and future model updates may change their relative priority.

## 6.3 Validation Results

The final ranked recommendation list was generated from the held-out validation population.

The highest-ranked **50 observations** produced:

* **14 confirmed `went_dark` observations**
* **Precision@50 = 0.280**

These results demonstrate that the model successfully concentrates positive observations within a relatively small review queue.

## 6.4 Recommended Workflow

The following workflow is recommended for operational use:

1. Generate prediction probabilities using the validated Logistic Regression model.
2. Rank all eligible observations from highest to lowest predicted probability.
3. Prioritize manual review beginning with Tier 1 observations.
4. Continue review with Tier 2 observations as resources permit.
5. Monitor lower-ranked observations through routine reporting and future model updates.

This workflow supports efficient allocation of analyst time while maintaining reproducible decision criteria.

## 6.5 Public-Safe Reporting

To preserve confidentiality, the published recommendation table replaces internal identifiers with public ranking identifiers.

No client names, domains, URLs, search queries, or private business information are included in the public outputs. The published recommendations communicate only ranking position, predicted priority, generalized reason codes, and recommended actions.

## 6.6 Practical Implications

The proposed ranking framework provides a practical method for prioritizing content review in environments where manual resources are limited.

Rather than attempting to review every eligible observation equally, analysts can focus first on the highest-ranked observations identified by the model, improving the efficiency of review activities while maintaining transparent and reproducible decision criteria.

## 6.7 Recommendation Summary

The final recommendation framework transforms model predictions into an operational decision-support process. By combining ranked probabilities, predefined priority tiers, and public-safe reporting, the proposed approach enables reproducible prioritization of potential `went_dark` observations while avoiding exposure of confidential client information.








# 7. Reproducibility

## 7.1 Reproducible Workflow

The complete analytical workflow was developed using Python within Google Colab and follows a fully reproducible pipeline. Each stage of the analysis—from data preparation to model evaluation and ranked recommendations—is documented in the accompanying notebooks.

The workflow was designed so that the complete analysis can be reproduced from the original FlyRank ML Internship warehouse release without requiring manual intervention or modification of intermediate datasets.

## 7.2 Data Processing

The analysis was performed directly on the FlyRank ML Internship warehouse hosted on Hugging Face using DuckDB.

The data-construction process applies the same predefined eligibility criteria, feature window, and outcome window each time the notebooks are executed. No manually edited datasets were used during model development.

## 7.3 Modeling Pipeline

The final modeling pipeline includes:

* Median imputation for missing predictor values.
* Logistic Regression as the final predictive model.
* Client-grouped train-validation splitting.
* Probability-based ranking.
* Evaluation using Precision@50 and ROC-AUC.

All preprocessing steps are incorporated within the machine-learning pipeline to ensure identical processing during both training and validation.

## 7.4 Validation Consistency

Model evaluation follows a fixed validation protocol using the held-out validation population.

The reported results correspond to:

* **Final modeling population:** 29,380 observations
* **Validation population:** 12,963 observations
* **Client overlap between training and validation:** 0
* **Primary evaluation metric:** Precision@50
* **Final Precision@50:** 0.280
* **Final ROC-AUC:** 0.775

These values represent the reproducible validation results reported throughout this study.

## 7.5 Repository Structure

The complete project repository includes:

* Weekly assignment notebooks.
* The capstone notebook.
* The deployed research paper.
* Supporting documentation.
* The required `submission/paper_url.txt` file containing the public URL of the deployed research paper.

This structure enables independent review of both the analytical workflow and the published research paper.

## 7.6 Public-Safe Reproducibility

To protect confidentiality, all published materials exclude client names, domains, URLs, private search queries, credentials, and raw warehouse exports.

Only aggregated analytical results, public-safe identifiers, and reproducible methodology are included in the public repository and deployed research paper.

## 7.7 Reproducibility Summary

The complete analytical workflow is reproducible using the published notebooks and the FlyRank ML Internship dataset. By documenting each stage of data preparation, model development, validation, and recommendation generation, the project supports transparent review and independent reproduction of the reported results while maintaining compliance with public-safe reporting requirements.










# 8. Acknowledgments & Data Credit

## 8.1 Acknowledgments

This project was completed as part of the **FlyRank Machine Learning Internship Capstone**. The capstone provided the research framework, evaluation criteria, and practical search-intelligence problem that guided the development of this study.

The project integrates concepts from data engineering, machine learning, search analytics, and reproducible research into a single end-to-end analytical workflow.

## 8.2 Data Credit

This study was **built on the FlyRank ML Internship dataset**, which was provided for educational and research purposes through the internship data warehouse.

The analysis was conducted using public-safe identifiers only. No client names, domains, URLs, private search queries, credentials, or raw warehouse exports are included in the published repository or deployed research paper.

Data source: **https://flyrank.ai**

## 8.3 Open Research and Reproducibility

The accompanying GitHub repository contains the complete analytical workflow, including the weekly assignment notebooks, the capstone notebook, and the materials required to reproduce the reported methodology and results using the official internship dataset.

The repository also includes the required `submission/paper_url.txt` file containing the public URL of the deployed research paper.

## 8.4 Final Statement

This study demonstrates how search and engagement signals can be transformed into a reproducible ranking framework for prioritizing potential `went_dark` observations. The proposed methodology combines temporal feature engineering, client-grouped validation, probability-based ranking, and transparent reporting to support practical decision-making while maintaining public-safe research standards.




