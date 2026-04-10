<div align="center">

<h1>Clinical Trial Patient Selection & Admission Prediction</h1>

<p>A healthcare data mining project applying machine learning to predict hospital admission types and optimize patient recruitment for clinical trials - built using Orange Data Mining on 10,000 synthetic patient records.</p>

<br/>

```
┌─────────────────────────────────────────────────────────────────┐
│  PROJECT SPEC                                                   │
├─────────────────────────────────────────────────────────────────┤
│  Tool       │  Orange Data Mining                               │
│  Domain     │  Healthcare Analytics / Clinical Research         │
│  Models     │  Neural Network  +  Gradient Boosting             │
│  Dataset    │  10,000 synthetic patient health records          │
│  Target     │  Admission Type  →  Elective / Urgent / Emergency │
│  Language   │  Python (Orange backend)                          │
│  License    │  MIT                                              │
└─────────────────────────────────────────────────────────────────┘
```

<br/>

</div>

---

Patient recruitment is one of the most expensive and failure-prone stages in clinical trial management. Studies consistently show that over 80% of clinical trials fail to meet their recruitment timelines, and a significant portion of that failure traces back to inadequate patient screening - enrolling candidates who are too clinically unstable, who drop out early, or whose admission history signals elevated trial risk.

This project addresses that problem directly. By applying a data mining pipeline to 10,000 synthetic patient health records, it builds a predictive classification system that identifies whether a patient's admission type - Elective, Urgent, or Emergency - can be predicted from their demographic profile, medical history, and billing data. The underlying hypothesis is that a patient's admission pattern is a proxy for their clinical stability, and therefore a meaningful signal for trial eligibility screening.

The project was built entirely in Orange Data Mining, a visual, node-based analytics platform widely used in biomedical and healthcare research. It covers the full analytics pipeline: data governance and preprocessing, feature importance ranking, multi-model classification, ROC-based evaluation, and a frank analytical discussion of what the results mean for real-world deployment.

---

## Table of Contents

- [The Clinical Problem](#the-clinical-problem)
- [Dataset](#dataset)
- [Repository Structure](#repository-structure)
- [The Orange Data Mining Pipeline](#the-orange-data-mining-pipeline)
- [Feature Importance Analysis](#feature-importance-analysis)
- [Model Development and Evaluation](#model-development-and-evaluation)
- [ROC Analysis and Healthcare Threshold Tuning](#roc-analysis-and-healthcare-threshold-tuning)
- [Supplementary Excel Analysis](#supplementary-excel-analysis)
- [Interpreting the 33% Accuracy Result](#interpreting-the-33-accuracy-result)
- [Findings and Clinical Recommendations](#findings-and-clinical-recommendations)
- [Future Work](#future-work)
- [Repository Structure](#repository-structure)
- [Getting Started](#getting-started)
- [Author](#author)

---

## The Clinical Problem

Clinical trials operate under strict safety and efficiency constraints. Enrolling a patient who is clinically unstable - someone likely to experience an emergency admission during the trial period - creates safety risks, increases dropout rates, inflates per-patient costs, and can compromise the statistical integrity of the entire study.

The standard approach to patient screening relies on inclusion and exclusion criteria applied manually during recruitment. This works, but it is slow, subjective, and doesn't scale. A data-driven pre-screening layer that flags high-risk candidates before they enter the manual review process could meaningfully reduce recruitment time and improve trial completion rates.

This project frames admission type prediction as the core analytical problem. The three classes - **Elective**, **Urgent**, and **Emergency** - represent a natural spectrum of clinical stability:

| Admission Type | Clinical Profile | Trial Suitability |
|:--------------|:----------------|:-----------------|
| Elective | Planned, stable condition | Generally suitable - predictable clinical trajectory |
| Urgent | Condition requiring prompt but non-critical attention | Requires case-by-case review - moderate risk |
| Emergency | Acute, unplanned, high-severity event | Typically unsuitable - unstable, high dropout and safety risk |

Predicting which category a prospective patient falls into - from their existing health record profile - is the core predictive task.

---

## Dataset

<div align="center">
<img src="https://github.com/TejashwiniSaravanan/Clinical-Trial-Patient-Selection-Optimization/raw/main/data_info.png" width="180"/>
<br/>
<sub>Dataset composition - 10,000 patient records, feature types, and class distribution</sub>
</div>

<br/>

The dataset is a synthetic patient health records corpus containing 10,000 records across a mix of demographic, clinical, and administrative features. Synthetic data was chosen deliberately - it preserves the statistical properties and feature relationships of real healthcare data while eliminating any patient privacy concerns, making it appropriate for open research and portfolio work.

| Feature | Type | Description |
|:--------|:-----|:------------|
| Age | Numeric | Patient age at time of admission |
| Gender | Categorical | Patient gender |
| Blood Type | Categorical | ABO/Rh blood group |
| Medical Condition | Categorical | Primary diagnosed condition |
| Insurance Provider | Categorical | Patient's insurance carrier |
| Billing Amount | Numeric | Total billed amount for the admission |
| Room Number | Numeric | Assigned room - proxy for ward type |
| Medication | Categorical | Primary medication administered |
| Test Results | Categorical | Outcome of diagnostic tests |
| Admission Type | Categorical | **Target variable** - Elective, Urgent, or Emergency |

**Key dataset properties:**

| Property | Value |
|:---------|:------|
| Total records | 10,000 |
| Target classes | 3 (Elective, Urgent, Emergency) |
| Missing values | None after preprocessing |
| Feature types | Mixed - numeric and categorical |
| Synthetic source | Designed to reflect real EHR distributions |

---

## Repository Structure

```
Clinical-Trial-Patient-Selection-Optimization/
│
├── data/
│   └── Patient_Health_Records_Dataset.csv     # 10,000 synthetic patient records
│
├── workflow/
│   └── Admission_Prediction_Workflow.ows      # Orange pipeline file
│
├── docs/
│   ├── Project_Report_and_Analysis.pdf        # Full project report
│   └── Model_Performance_Analysis_Matrix.xlsx # Excel confusion matrix and metrics
│
├── workflow_preview.png                        # Orange pipeline screenshot
├── feature_rankings.png                        # Feature importance output
├── roc_curves.png                              # ROC analysis for all models
├── data_info.png                               # Dataset composition screenshot
├── requirements.txt                            # Software requirements
└── README.md
```

---

## The Orange Data Mining Pipeline

<div align="center">
<img src="https://github.com/TejashwiniSaravanan/Clinical-Trial-Patient-Selection-Optimization/raw/main/workflow_preview.png" width="520"/>
<br/>
<sub>Orange workflow - data preprocessing, feature ranking, classification, and evaluation</sub>
</div>

<br/>

The pipeline was designed as a modular, reproducible flow in Orange's visual programming environment. Each node in the workflow corresponds to a discrete analytical decision, and the structure reflects a deliberate sequence: govern the data before modeling it, understand the features before selecting models, and evaluate honestly rather than optimistically.

### Stage 1 - Data Governance and Preprocessing

Three preprocessing operators were applied in sequence before any model saw the data.

**Select Columns.** Patient identifiers - names, doctor IDs, and record numbers - were removed at this stage. These fields carry no predictive signal for admission type and would introduce noise. Retaining only clinically and demographically meaningful features ensures the model learns from relevant patterns rather than memorising identifiers.

**Discretize.** Continuous variables including Age and Billing Amount were binned into categorical ranges. This decision was made for two reasons. First, it makes the model outputs more interpretable for clinical stakeholders - a prediction tied to "Age group: 60–75" is more actionable than one tied to a specific age value. Second, binning reduces the sensitivity of tree-based and rule-based models to outliers in continuous distributions, which are common in billing data.

**Purge Domain.** After column selection and discretization, the Purge Domain operator was applied to remove empty attribute values and redundant category levels that remained as artefacts of the filtering. In Orange, skipping this step can cause models to reference categories that no longer exist in the filtered dataset - a subtle source of silent errors in classification outputs. This step ensures the domain presented to each model is clean, consistent, and free of ghost categories.

### Stage 2 - Feature Importance Ranking

Before training any classification model, the **Rank** node was used to score every input feature against the target variable using two complementary metrics: **Information Gain** and **Gini Decrease**. Running both measures simultaneously provides a more robust picture of feature relevance - a feature that scores highly on both metrics is a more reliable predictor than one that appears important only under a single measure.

This step informed which features were worth retaining and which were likely to add noise without contributing predictive value.

### Stage 3 - Classification Modelling

Two classification algorithms were trained and evaluated in parallel: **Neural Network** and **Gradient Boosting**. The rationale for this pairing is that they represent fundamentally different modelling philosophies - a neural network learns distributed representations through layered non-linear transformations, while gradient boosting builds a sequential ensemble of shallow trees, each correcting the residual errors of the previous. If both models converge on similar performance, the result is more credible than if only one algorithm had been tested.

### Stage 4 - Evaluation

Both models were evaluated using **Test and Score** with cross-validation, and the outputs were assessed using the **Confusion Matrix** and **ROC Analysis** nodes. The confusion matrix breaks down prediction errors by class - essential in healthcare, where the cost of a false negative (missing an emergency patient) is structurally different from the cost of a false positive (incorrectly flagging an elective patient as urgent).

---

## Feature Importance Analysis

<div align="center">
<img src="https://github.com/TejashwiniSaravanan/Clinical-Trial-Patient-Selection-Optimization/raw/main/feature_rankings.png" width="380"/>
<br/>
<sub>Feature rankings - Information Gain and Gini Decrease scores for all input features</sub>
</div>

<br/>

The Rank node output revealed a feature landscape that is consistent with what the 33% accuracy result later confirmed: no single feature dominates the prediction of admission type, and the most informative features are not the ones a clinical researcher would expect.

**Billing Amount** ranked as the most informative feature by both Information Gain and Gini Decrease. This is notable because billing amount is an administrative variable, not a clinical one. Its predictive relevance likely reflects the way the synthetic dataset was generated - billing amounts in synthetic records are often correlated with admission severity by construction - rather than a genuine causal relationship.

**Blood Type** ranked second. In real clinical settings, blood type has minimal bearing on whether an admission is elective, urgent, or emergency. Its appearance at the top of the ranking is a signal that the synthetic dataset does not fully replicate the feature-to-outcome relationships of real EHR data - a key finding that directly informs the interpretation of model accuracy.

**Medical Condition** appeared mid-table. In a real clinical context, this would be expected to be among the strongest predictors of admission urgency. Its moderate ranking in this dataset further supports the conclusion that the synthetic data's feature correlations diverge from real-world clinical patterns.

This feature importance analysis is not just a diagnostic output - it is the analytical foundation for the primary recommendation of this project: that demographic and administrative features alone are insufficient predictors of admission type, and that clinical integration of vital signs, lab values, and comorbidity history is required before this pipeline could support real deployment.

---

## Model Development and Evaluation

Two models were trained and evaluated: a **Neural Network** and **Gradient Boosting**.

**Neural Network configuration.** Orange's neural network implementation uses a multi-layer perceptron architecture. For this dataset, it was applied without custom architecture tuning - a deliberate choice to establish a clean baseline performance before any hyperparameter optimisation. The network learns non-linear feature interactions through backpropagation, which in theory should capture complex relationships between patient demographics and admission urgency if such relationships exist in the data.

**Gradient Boosting configuration.** Gradient Boosting builds an additive ensemble of decision trees, each trained on the residual errors of the previous iteration. It typically outperforms neural networks on structured tabular data when sample sizes are moderate and feature interactions are well-defined. Its application here tests whether the ensemble approach can extract signal that the neural network cannot.

**Cross-validation.** Both models were evaluated using stratified k-fold cross-validation within Orange's Test and Score node. Stratification ensures that each fold maintains the same class distribution as the full dataset - important when dealing with multi-class healthcare targets where class imbalance could distort evaluation metrics.

**Results summary:**

| Model | Accuracy | AUC (avg) | Interpretation |
|:------|:---------|:----------|:---------------|
| Neural Network | ~33% | ~0.50 | No better than random classification across three classes |
| Gradient Boosting | ~33% | ~0.50 | Consistent with Neural Network - confirms finding is structural |
| Random Baseline | 33.3% | 0.50 | Expected performance for a 3-class problem with no signal |

Both models converged at the random baseline. The consistency between two fundamentally different algorithms makes this result analytically meaningful - it is not a model implementation problem, it is a data signal problem.

---

## ROC Analysis and Healthcare Threshold Tuning

<div align="center">
<img src="https://github.com/TejashwiniSaravanan/Clinical-Trial-Patient-Selection-Optimization/raw/main/roc_curves.png" width="380"/>
<br/>
<sub>ROC curves - Neural Network and Gradient Boosting across all three admission classes</sub>
</div>

<br/>

ROC (Receiver Operating Characteristic) analysis plots the true positive rate against the false positive rate at every possible classification threshold. An AUC of 1.0 represents perfect discrimination; an AUC of 0.50 represents random guessing - indistinguishable from a coin flip. Both models produced AUC values near 0.50 across all three admission classes, visually confirmed by ROC curves that closely track the diagonal random baseline.

This result carries a specific and important meaning in the healthcare context. In clinical trial recruitment, the consequences of misclassification are asymmetric:

| Error Type | Clinical Consequence | Relative Cost |
|:-----------|:--------------------|:-------------|
| False Negative on Emergency | Emergency patient enrolled in trial - safety risk, early dropout | High |
| False Positive on Emergency | Stable patient excluded unnecessarily - recruitment inefficiency | Moderate |
| False Negative on Elective | Suitable patient screened out | Low |

A model with AUC near 0.50 cannot reliably support threshold tuning for any of these trade-offs. Attempting to tune the classification threshold on a model without discriminative power would produce false confidence rather than genuine risk stratification.

The ROC analysis, therefore, does not represent a failure of methodology - it represents an honest and accurate characterisation of the limits of the available data, and it directly motivates the data enrichment recommendations in the findings section below.

---

## Supplementary Excel Analysis

In addition to the Orange workflow outputs, a comprehensive **Model Performance Analysis Matrix** was developed in Excel to translate the technical evaluation metrics into a format accessible to clinical and business stakeholders.

The Excel workbook contains three analytical components:

**Confusion Matrix Breakdown.** For each of the three admission classes, a detailed confusion matrix was constructed showing True Positives, True Negatives, False Positives, and False Negatives across both models. This allows stakeholders to see not just overall accuracy but precisely where each model makes errors - which classes are confused with which, and at what rates.

**Comparative Metrics.** A side-by-side comparison of Neural Network and Gradient Boosting across Precision, Recall, F1-Score, and AUC for each class. This is the most informative view of model behaviour - a model can have acceptable overall accuracy while performing poorly on a clinically critical class like Emergency.

**Cost-Benefit Projection.** A business-layer analysis that estimates the resource allocation implications of deploying a model at this accuracy level versus a manual screening baseline. This component frames the technical findings in terms of trial budget, screening time, and expected recruitment yield - the language that trial sponsors and clinical operations teams work in.

---

## Interpreting the 33% Accuracy Result

The accuracy result of approximately 33% across both models is the most important analytical finding in this project, and it requires careful interpretation rather than dismissal.

In a three-class classification problem with balanced classes, a random classifier achieves 33.3% accuracy by definition. Both models - Neural Network and Gradient Boosting - converged at this baseline. This means neither model learned anything systematic from the available features that predicted admission type better than chance.

There are three distinct reasons why this outcome is informative rather than simply negative.

**The features available in this dataset are the wrong features for this task.** Admission type - whether a patient arrived electively, urgently, or as an emergency - is determined by acute clinical events and physiological status at the time of presentation. Demographic variables, insurance providers, blood type, and billing amounts do not capture the factors that drive admission urgency. This is not a modelling failure; it is a feature availability problem. The data does not contain the signal required to solve the stated prediction problem.

**The synthetic nature of the dataset amplifies this limitation.** Real EHR datasets encode complex, domain-specific correlations between diagnoses, vital signs, medication histories, and admission patterns. Synthetic datasets, even well-constructed ones, rarely replicate these correlations at the level of fidelity required for clinical prediction tasks. The top-ranked features - Billing Amount and Blood Type - are not clinically meaningful predictors of admission urgency in real settings, which suggests the synthetic generator did not encode realistic feature-to-outcome relationships.

**This baseline result is itself a contribution.** Establishing that demographic and administrative features alone cannot predict admission type - and documenting this finding with two independent models and ROC analysis - is a valid and rigorous research outcome. It defines the minimum data requirements for a deployable version of this system and sets a clear benchmark against which future iterations can be measured.

---

## Findings and Clinical Recommendations

The combined output of the feature importance analysis, model evaluation, and ROC assessment produces a set of findings that are directly actionable for healthcare data teams and clinical trial operations:

| Finding | Evidence | Recommendation |
|:--------|:---------|:---------------|
| Demographic and administrative features cannot predict admission type | Both models at 33% accuracy, AUC ~0.50 | Do not deploy admission prediction systems using only EHR metadata - enrich with clinical features |
| Billing Amount ranks highest but is not clinically meaningful | Top Information Gain score for an administrative variable | Audit synthetic dataset generation methodology before using for clinical model validation |
| Blood Type ranks second - no clinical basis | High Gini Decrease for a clinically irrelevant feature | Treat top-ranked features from synthetic data with caution - validate against domain knowledge |
| Medical Condition has moderate ranking despite clinical relevance | Expected top predictor ranks mid-table | In real EHR data, Medical Condition combined with ICD codes would likely dominate predictions |
| Emergency misclassification carries asymmetric cost | Clinical consequence analysis | Any deployed version must prioritise Recall for the Emergency class - use threshold tuning via ROC |
| Neural Network and Gradient Boosting perform identically | Both at random baseline | Result is structural - additional models will not improve performance without better data |

**For a production-ready clinical trial patient screening system, the following data enrichment steps are required before model training:**

- Integration of structured vital signs (heart rate, blood pressure, oxygen saturation) from EHR systems
- Inclusion of laboratory results (CBC, metabolic panels, inflammatory markers)
- Historical comorbidity burden using Charlson Comorbidity Index or similar scoring
- Prior admission frequency and emergency visit history
- Diagnosis codes (ICD-10) at the primary and secondary level
- Medication complexity score as a proxy for disease burden

With these features present, the prediction task becomes tractable and the pipeline architecture established in this project provides a ready foundation for re-deployment.

---

## Future Work

Three development priorities follow directly from the findings of this project.

**Clinical data integration.** The most impactful next step is replacing or augmenting the synthetic dataset with real de-identified EHR data containing vital signs, laboratory values, and structured diagnosis codes. Even a modest dataset of a few thousand records with genuine clinical features would likely produce substantially better discrimination than 10,000 synthetic records without meaningful feature-outcome correlations.

**Advanced modelling.** Once appropriate features are available, more powerful classifiers - Random Forest, XGBoost, or a transformer-based model fine-tuned on clinical text - could be tested within the same pipeline structure. The Orange workflow is already modular enough to support model swapping without restructuring the preprocessing or evaluation stages.

**Trial eligibility scoring layer.** The current project predicts admission type as a binary proxy for clinical stability. A more clinically useful extension would replace this three-class problem with a continuous eligibility score - a probability that a given patient profile is suitable for a specific trial's inclusion criteria. This would require collaboration with clinical researchers to define the scoring rubric, but the data pipeline and evaluation framework established here would carry forward directly.

---

## Getting Started

**Requirements:** Orange Data Mining 3.x - available free at [orangedatamining.com](https://orangedatamining.com/)

**To open the workflow:**

1. Install Orange Data Mining
2. Open Orange and select **File > Open**
3. Navigate to `workflow/Admission_Prediction_Workflow.ows`
4. The full pipeline loads with all nodes connected - click **Run** to execute

**To use a different dataset:**

1. Replace the CSV file path in the **File** node at the start of the workflow
2. Ensure the target column is named `Admission Type` or update the **Select Columns** node accordingly
3. Re-run the workflow - all downstream nodes update automatically

**Software requirements** are listed in `requirements.txt`.

---

## Documentation

The full project report - including detailed methodology, model output tables, confusion matrices, and the cost-benefit analysis - is in [`docs/Project_Report_and_Analysis.pdf`](./docs/Project_Report_and_Analysis.pdf).

The Model Performance Analysis Matrix, including the Excel-based confusion matrix breakdown and comparative metrics, is in [`docs/Model_Performance_Analysis_Matrix.xlsx`](./docs/Model_Performance_Analysis_Matrix.xlsx).

---

## Author

<div align="center">

**Tejashwini Saravanan**

Post2tejashwini@gmail.com

[LinkedIn - tejashwinisaravanan](https://www.linkedin.com/in/tejashwinisaravanan/) &nbsp;|&nbsp; [GitHub - TejashwiniSaravanan](https://github.com/TejashwiniSaravanan)

</div>

---

## License

Released under the MIT License. The dataset used in this project is fully synthetic and contains no real patient information. See [LICENSE](./LICENSE) for full terms.
