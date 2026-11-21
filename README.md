# GlycoQueue: Capacity-Aware Prioritisation for HbA1c Screening

**Track:** Health  
**Team:** solo

**DISCLAIMER: This project is a PROTOTYPE and is NOT medical advice or a clinical diagnosis.**

## The problem

Primary care screening programmes have finite HbA1c testing capacity. Global risk rankings can improve detection yield, but may disproportionately prioritise older adults, leaving younger adults with elevated HbA1c less likely to be screened.

GlycoQueue investigates whether alternative allocation strategies can improve age diversity while retaining a strong detection yield compared with random allocation.

## What we built

**Input:** Age + BMI  
**Output:** A screening-priority score.

A primary care coordinator could theoretically use the model to rank an eligible cohort and allocate a limited number of invitations to those most likely to have elevated HbA1c. This is a research prototype, not a clinically validated tool.

## What we added

**Our own idea:**

I trained a logistic regression model using age and BMI, then tested it across three separate NHANES survey cycles: training on 2015–2016, validating on 2017–2018, and testing on unseen 2021–2023 data.

The global ranking favoured older adults. Although it detected 199 positive cases, it caught only 6 of the 261 under-50 participants with elevated HbA1c.

To address this, I introduced alternative allocation policies, including reserving 30% of invitation capacity for the highest-scoring adults under 50. This caught 44 under-50 positive cases, with 181 total positives detected — a trade-off between overall yield and younger-adult reach.

## How it works

**Data:** NHANES 2015–2016, 2017–2018 and 2021–2023. Demographics, body measurements, diabetes questionnaire and HbA1c laboratory data were merged using `SEQN`. Eligible participants were adults aged 18+, without reported diabetes or prediabetes, with valid HbA1c measurements and no known pregnancy at examination.

Eligible sample sizes: 4,010 / 3,708 / 4,224.

Elevated HbA1c was defined as `LBXGH >= 5.7%`.

**Model:** Logistic regression using age and BMI, with median imputation and standardisation in a scikit-learn Pipeline. It was trained on 2015–2016, validated on 2017–2018 and evaluated on the separate 2021–2023 holdout.

**Policies:** At 10% screening capacity:
- **Global:** Select the highest-scoring participants overall.
- **Age-stratified:** Distribute invitations proportionally across age groups, ranking within each group.
- **Hybrid 30% / 45%:** Reserve the specified share of invitations for highest-scoring adults under 50, then fill the remainder globally.

## Results

The random-allocation baseline was approximately 98 positive cases out of 423 invitations, based on the holdout prevalence of 23.18%.

The global model ranking detected 199 positive cases, achieving approximately **2.03× lift** over random allocation.

| Policy | Precision | Recall | Lift | Under-50 recall |
|---|---:|---:|---:|---:|
| Global | 47.0% | 20.3% | 2.03× | 2.3% |
| Age-stratified | 37.1% | 16.0% | 1.60× | 21.8% |
| Hybrid 30% | 42.8% | 18.5% | 1.85× | 16.9% |
| Hybrid 45% | 38.1% | 16.4% | 1.64× | 21.5% |

**Where it fails, and why we think so:**

The hybrid 30% policy performed best on the validation cohort but did not retain that advantage on the final holdout. The model's ROC-AUC remained relatively stable (0.733 → 0.723), while the allocation policies changed in relative performance.

The later cohort had a lower elevated-HbA1c prevalence, which may partly explain the difference. We retained the original policies rather than tuning them against the holdout.

Our subgroup audit also investigates differences across age, sex and ethnicity. One limitation is that BMI does not represent metabolic risk equally across ethnic groups. Some populations, including South Asian populations, can experience increased metabolic risk at lower BMI. Our audit explores subgroup outcomes but does not establish or eliminate this potential source of bias.

## What we'd do next

- Validate the model and allocation policies on a larger, independent population.
- Investigate ethnic subgroup differences more deeply, including whether additional measures of body composition could improve fairness.

## Sources

- [NHANES – Dataset](https://wwwn.cdc.gov/nchs/nhanes/)
- [NIDDK – A1C Test](https://www.niddk.nih.gov/health-information/diagnostic-tests/a1c-test)
- [Diabetes UK – Diagnostic criteria](https://www.diabetes.org.uk/for-professionals/improving-care/clinical-recommendations-for-professionals/diagnosis-ongoing-management-monitoring/new_diagnostic_criteria_for_diabetes)
- [WHO – BMI and Asian populations](https://doi.org/10.1016/S0140-6736(03)15268-3)
- [StatPearls – Hemoglobin A1C](https://www.ncbi.nlm.nih.gov/books/NBK549816/)

**Tools:** Python, pandas, NumPy, scikit-learn, Matplotlib and Google Colab.

ChatGPT assisted with data retrieval code, debugging, technical explanations and research support, as it was my first time doing this. Gemini helped clean up and bookkeep some md cells.