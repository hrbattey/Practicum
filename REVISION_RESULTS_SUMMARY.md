# Revision results summary — XGBoost vs CatBoost for TB disease (BMC Public Health revision)

Generated automatically from the outputs of `Code/Practicum_REVISION.ipynb` (Restart & Run All, 0 errors). Every number below is copied programmatically from the `rev_*.csv` files in `Output/csv`; file names are given under each table so any value can be traced. Values from the submitted manuscript are in Appendix A.

## 0. How to read the numbers

- **Data / split (unchanged):** n = 795 participants (Pakistan, numeric qXR score, valid outcome); stratified 80/20 split, seed 42 → 636 train (141 TB, 22.2 %) / 159 test (35 TB, 22.0 %).
- **Outcome:** TB disease (bacteriologically confirmed or clinician-diagnosed) vs no TB disease.
- **Models:** XGBoost and CatBoost. Staged feature sets M1–M5 chosen by carry-forward selection on training-set out-of-fold mean |SHAP| (K = 10/15/15/15/15): M1 patient characteristics, + M2 symptoms, + M3 provider CXR findings, + M4 blood counts (CBC), + M5 qXR score. Hyperparameters: Optuna (30 trials, 3-fold inner CV, AUROC), search spaces unchanged; final models refit on all 636 training rows.
- **Threshold:** for each model, the threshold giving specificity ≥ 0.80 on pooled 5-fold out-of-fold predictions within the training set; applied unchanged to the test set (so test specificity can fall below 0.80).
- **95% CIs (test set, threshold fixed):** AUROC, AP (PR-AUC) and Brier — percentile bootstrap, 2,000 resamples of test rows; sensitivity, specificity, PPV, NPV — exact Clopper–Pearson.
- **Null Brier:** Brier score of predicting the training TB prevalence for everyone (≈ 0.172). A Brier at or above it means the probabilities are no better than no-skill.
- **Brier (recal.):** Brier after a logistic (Platt) recalibration fitted on that model's training out-of-fold predictions and applied unchanged to test. Discrimination and threshold metrics are always reported on the raw scale.
- **Seed:** 42 everywhere (tuning, folds, selection, final fits, bootstrap, noise control).
- Display names: 'Prior TB disease (self-report)' = `tb_history_prior`; 'History of contact with TB disease case' = household contact; 'Provider CXR interpret' = provider CXR read as TB; qXR Score = AI chest X-ray score from qXR (Qure.ai).

## 1. Changes to the analysis since submission (Methods text)

1. **Outcome leakage removed (largest change).** The original TB-history predictor (`tb_history_composite`) combined the interview question 'Have you ever had TB in the past?' with the baseline chart-review item 'Has the patient recently been diagnosed with TB?'. The chart-review item was Yes for 130/176 TB cases and 0/619 non-cases, i.e. it recorded the current diagnosis. All models now use the interview item only ('Don't know' → No; Yes 30/176 TB vs 139/619 non-TB). The original variable is kept only in a supplementary comparison (Section 6).
2. **Class-weighting bug fixed.** XGBoost's tuned class weight (`scale_pos_weight`) was dropped when the best hyperparameters were finalised, so XGBoost models trained unweighted even when tuning selected weighting. It is now applied. Consequently the statement 'neither model used class weighting' is **false** (Section 13).
3. **One set of M1–M5 models.** Figures (SHAP, PR/ROC, AP/AUC bars) and all stage tables now come from the same fitted models; previously the stage metrics table used a separately re-tuned set. Thresholds come from training out-of-fold predictions (see Section 0).
4. **Single seed (42)** for all random steps (previously 42/123/165/789 in different places).
5. **CIs:** test-set CIs as in Section 0; recalibrated Brier and no-skill Brier reported alongside the raw Brier.
6. **Figure 5 (5 × 3 SHAP grid):** panel labels A–C, 'ATB' text → 'TB disease', 300 dpi.

## 2. Study population — Table 1 (n = 795)

Chi-square, or Fisher's exact when any expected count < 5 (Freeman–Halton for > 2 levels); Student t-test for continuous variables. File: `rev_table1_analytic_sample.csv`.

| Variable | Level | Overall (N=795) | TB disease (N=176) | No TB disease (N=619) | p-value | Test |
|---|---|---|---|---|---|---|
| Patient characteristics |  |  |  |  |  |  |
| Age, mean (SD) |  | 37.0 (17.3) | 36.3 (17.4) | 37.3 (17.3) | 0.498 | t-test |
| Sex, n (%) | Female | 416 (52.3) | 77 (43.8) | 339 (54.8) | 0.010 | Chi-square |
|  | Male | 379 (47.7) | 99 (56.2) | 280 (45.2) |  |  |
| Race, n (%) | Asian | 785 (98.7) | 172 (97.7) | 613 (99.0) | 0.241 | Fisher's exact |
|  | Other | 10 (1.3) | 4 (2.3) | 6 (1.0) |  |  |
| BMI, mean (SD) |  | 20.9 (4.8) | 19.2 (3.8) | 21.4 (4.9) | <0.001 | t-test |
| Diabetes, n (%) | Diabetes | 50 (6.3) | 19 (10.8) | 31 (5.0) | 0.005 | Chi-square |
|  | No diabetes | 745 (93.7) | 157 (89.2) | 588 (95.0) |  |  |
| Asthma, n (%) | Asthma | 39 (4.9) | 6 (3.4) | 33 (5.3) | 0.298 | Chi-square |
|  | No asthma | 756 (95.1) | 170 (96.6) | 586 (94.7) |  |  |
| BCG vax, n (%) | Unknown | 486 (61.1) | 111 (63.1) | 375 (60.6) | 0.771 | Fisher's exact (Freeman–Halton) |
|  | No | 8 (1.0) | 1 (0.6) | 7 (1.1) |  |  |
|  | Yes | 301 (37.9) | 64 (36.4) | 237 (38.3) |  |  |
| History of contact with TB disease case, n (%) | No | 531 (66.8) | 109 (61.9) | 422 (68.2) | 0.121 | Chi-square |
|  | Yes | 264 (33.2) | 67 (38.1) | 197 (31.8) |  |  |
| Prior TB disease (self-report), n (%) | No | 626 (78.7) | 146 (83.0) | 480 (77.5) | 0.122 | Chi-square |
|  | Yes | 169 (21.3) | 30 (17.0) | 139 (22.5) |  |  |
| QFT result, n (%) | Indeterminate | 37 (4.7) | 8 (4.5) | 29 (4.7) | <0.001 | Chi-square |
|  | Negative | 300 (37.7) | 33 (18.8) | 267 (43.1) |  |  |
|  | Positive | 458 (57.6) | 135 (76.7) | 323 (52.2) |  |  |
| Symptoms |  |  |  |  |  |  |
| Persistent cough, n (%) | No | 151 (19.0) | 20 (11.4) | 131 (21.2) | 0.003 | Chi-square |
|  | Yes | 644 (81.0) | 156 (88.6) | 488 (78.8) |  |  |
| Shortness breath, n (%) | No | 402 (50.6) | 95 (54.0) | 307 (49.6) | 0.305 | Chi-square |
|  | Yes | 393 (49.4) | 81 (46.0) | 312 (50.4) |  |  |
| Fever, n (%) | No | 43 (5.4) | 5 (2.8) | 38 (6.1) | 0.088 | Chi-square |
|  | Yes | 752 (94.6) | 171 (97.2) | 581 (93.9) |  |  |
| Fatigue, n (%) | No | 90 (11.3) | 10 (5.7) | 80 (12.9) | 0.007 | Chi-square |
|  | Yes | 705 (88.7) | 166 (94.3) | 539 (87.1) |  |  |
| Hemoptysis, n (%) | No | 707 (88.9) | 153 (86.9) | 554 (89.5) | 0.338 | Chi-square |
|  | Yes | 88 (11.1) | 23 (13.1) | 65 (10.5) |  |  |
| Weight loss, n (%) | No | 272 (34.2) | 31 (17.6) | 241 (38.9) | <0.001 | Chi-square |
|  | Yes | 523 (65.8) | 145 (82.4) | 378 (61.1) |  |  |
| Night sweats, n (%) | No | 541 (68.1) | 96 (54.5) | 445 (71.9) | <0.001 | Chi-square |
|  | Yes | 254 (31.9) | 80 (45.5) | 174 (28.1) |  |  |
| CXR findings (provider-interpreted) |  |  |  |  |  |  |
| Infiltrates, n (%) | No | 415 (52.2) | 28 (15.9) | 387 (62.5) | <0.001 | Chi-square |
|  | Yes | 380 (47.8) | 148 (84.1) | 232 (37.5) |  |  |
| Nodules, n (%) | No | 700 (88.1) | 115 (65.3) | 585 (94.5) | <0.001 | Chi-square |
|  | Yes | 95 (11.9) | 61 (34.7) | 34 (5.5) |  |  |
| Cavities, n (%) | No | 745 (93.7) | 148 (84.1) | 597 (96.4) | <0.001 | Chi-square |
|  | Yes | 50 (6.3) | 28 (15.9) | 22 (3.6) |  |  |
| Pleural effusion, n (%) | No | 750 (94.3) | 164 (93.2) | 586 (94.7) | 0.451 | Chi-square |
|  | Yes | 45 (5.7) | 12 (6.8) | 33 (5.3) |  |  |
| Lymphadenopathy, n (%) | No | 683 (85.9) | 153 (86.9) | 530 (85.6) | 0.659 | Chi-square |
|  | Yes | 112 (14.1) | 23 (13.1) | 89 (14.4) |  |  |
| Provider CXR interpret, n (%) | No | 344 (43.3) | 18 (10.2) | 326 (52.7) | <0.001 | Chi-square |
|  | Yes | 451 (56.7) | 158 (89.8) | 293 (47.3) |  |  |
| CXR (qXR) |  |  |  |  |  |  |
| qXR Score, mean (SD) |  | 0.42 (0.41) | 0.84 (0.27) | 0.30 (0.35) | <0.001 | t-test |
| Blood values (CBC) |  |  |  |  |  |  |
| WBC, mean (SD) |  | 9.34 (5.26) | 10.73 (4.47) | 8.95 (5.40) | <0.001 | t-test |
| NEU, mean (SD) |  | 63.3 (12.4) | 72.0 (10.4) | 60.9 (11.8) | <0.001 | t-test |
| LYM, mean (SD) |  | 27.1 (11.1) | 19.1 (8.8) | 29.4 (10.7) | <0.001 | t-test |
| P_LCR, mean (SD) |  | 27.5 (10.8) | 24.0 (9.3) | 28.5 (11.0) | <0.001 | t-test |
| RBC, mean (SD) |  | 4.79 (0.66) | 4.64 (0.70) | 4.83 (0.65) | <0.001 | t-test |
| HGB, mean (SD) |  | 12.8 (2.1) | 11.8 (2.1) | 13.1 (2.0) | <0.001 | t-test |
| HCT, mean (SD) |  | 39.1 (5.5) | 36.8 (5.5) | 39.7 (5.3) | <0.001 | t-test |
| MCHC, mean (SD) |  | 32.6 (2.0) | 32.1 (1.9) | 32.8 (2.0) | <0.001 | t-test |
| MCH, mean (SD) |  | 26.9 (3.6) | 25.6 (3.2) | 27.2 (3.6) | <0.001 | t-test |
| MCV, mean (SD) |  | 81.8 (8.4) | 79.5 (7.4) | 82.5 (8.5) | <0.001 | t-test |
| RDW_CV, mean (SD) |  | 14.8 (2.4) | 15.5 (2.8) | 14.6 (2.2) | <0.001 | t-test |
| RDW_SD, mean (SD) |  | 43.9 (5.7) | 44.5 (5.8) | 43.8 (5.7) | 0.126 | t-test |
| PLT, mean (SD) |  | 307.3 (118.6) | 370.6 (142.8) | 289.4 (104.1) | <0.001 | t-test |
| MPV, mean (SD) |  | 10.1 (1.5) | 9.6 (1.3) | 10.3 (1.5) | <0.001 | t-test |
| PCT, mean (SD) |  | 0.30 (0.10) | 0.35 (0.12) | 0.29 (0.09) | <0.001 | t-test |
| PDW, mean (SD) |  | 14.1 (3.1) | 12.9 (2.6) | 14.4 (3.1) | <0.001 | t-test |

**Outcome composition** (`rev_outcome_detail_counts.csv`):

| TB_status_detailed | No TB disease | TB disease | Total |
|---|---|---|---|
| ATB-bacteriologically confirmed | 0 | 162 | 162 |
| ATB-clinician diagnosed at baseline | 0 | 14 | 14 |
| LTBI | 323 | 0 | 323 |
| NTBI | 267 | 0 | 267 |
| Not Active TB-QFT Indeterminate | 29 | 0 | 29 |
| Total | 619 | 176 | 795 |

TB disease = 162 bacteriologically confirmed (92.0 %) + 14 clinician-diagnosed only (8.0 %).

**HIV** (`rev_hiv_counts.csv`): all 795 tested; 7 reactive → prevalence 7/795 = 0.88 % (4 with TB disease, 3 without).

| hiv_result | No TB disease | TB disease | Total |
|---|---|---|---|
| Non-Reactive | 616 | 172 | 788 |
| Reactive | 3 | 4 | 7 |
| Total | 619 | 176 | 795 |

**Age groups** (`rev_age_group_counts.csv`):

| Set | Age 5–14: n | Age 5–14: TB cases | Age ≥15: n | Age ≥15: TB cases | All ages: n | All ages: TB cases |
|---|---|---|---|---|---|---|
| Train | 43 | 6 | 593 | 135 | 636 | 141 |
| Test | 8 | 1 | 151 | 34 | 159 | 35 |
| Total | 51 | 7 | 744 | 169 | 795 | 176 |

## 3. Full-feature XGBoost vs CatBoost — nested cross-validation (Table 2 / Figure 2)

Nested 5 × 3 CV within the 636 training rows; inner tuning maximises sensitivity at specificity ≥ 0.80; per-fold thresholds; pooled out-of-fold predictions with 2,000-bootstrap CIs. File: `rev_fig2_summary_metrics_XGB_vs_CAT_bootstrap_pooled.csv`.

| Metric | CatBoost | XGBoost |
|---|---|---|
| Sens | 0.816 (0.752, 0.875) | 0.780 (0.711, 0.844) |
| Spec | 0.816 (0.781, 0.848) | 0.848 (0.815, 0.878) |
| PPV | 0.558 (0.488, 0.624) | 0.595 (0.523, 0.667) |
| NPV | 0.940 (0.917, 0.960) | 0.931 (0.908, 0.953) |
| AUC | 0.853 (0.817, 0.887) | 0.892 (0.863, 0.919) |
| PRAUC | 0.698 (0.624, 0.766) | 0.725 (0.649, 0.796) |
| Brier | 0.127 (0.115, 0.139) | 0.103 (0.089, 0.118) |

**Supplementary Figure 3 — sensitivity across specificity targets** (`rev_sensitivity_vs_spec_targets_cat_xgb.csv`):

| Specificity_Target | Model | Sensitivity | Sensitivity_CI | Achieved_Specificity | PPV | NPV | AUROC | PRAUC | Brier |
|---|---|---|---|---|---|---|---|---|---|
| 0.75 | CatBoost | 0.837 | (0.776, 0.893) | 0.79 | 0.532 | 0.944 | 0.872 | 0.71 | 0.128 |
| 0.75 | XGBoost | 0.816 | (0.750, 0.874) | 0.804 | 0.542 | 0.939 | 0.9 | 0.755 | 0.104 |
| 0.8 | CatBoost | 0.816 | (0.752, 0.875) | 0.816 | 0.558 | 0.94 | 0.853 | 0.698 | 0.127 |
| 0.8 | XGBoost | 0.78 | (0.711, 0.844) | 0.848 | 0.595 | 0.931 | 0.892 | 0.725 | 0.103 |
| 0.85 | CatBoost | 0.745 | (0.674, 0.812) | 0.869 | 0.618 | 0.923 | 0.865 | 0.721 | 0.113 |
| 0.85 | XGBoost | 0.759 | (0.688, 0.823) | 0.867 | 0.618 | 0.927 | 0.886 | 0.717 | 0.102 |
| 0.9 | CatBoost | 0.66 | (0.583, 0.734) | 0.901 | 0.655 | 0.903 | 0.885 | 0.735 | 0.108 |
| 0.9 | XGBoost | 0.716 | (0.643, 0.790) | 0.885 | 0.639 | 0.916 | 0.892 | 0.739 | 0.1 |

## 4. Staged models M1–M5 — held-out test set (n = 159, 35 TB)

**XGBoost** (`rev_metrics_by_stage_XGB_test_bootCI.csv`):

| Model | n_feats | Sens | Spec | PPV | NPV | AUROC | AP | Brier | Null Brier | Brier (recal.) | Thr |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Model 1 | 10 | 0.429 (0.263, 0.606) | 0.790 (0.708, 0.858) | 0.366 (0.221, 0.531) | 0.831 (0.750, 0.893) | 0.629 (0.519, 0.740) | 0.329 (0.224, 0.494) | 0.239 (0.215, 0.264) | 0.172 | 0.169 (0.134, 0.205) | 0.6183 |
| Model 2 | 15 | 0.514 (0.340, 0.686) | 0.774 (0.690, 0.844) | 0.391 (0.251, 0.546) | 0.850 (0.770, 0.910) | 0.663 (0.550, 0.773) | 0.391 (0.258, 0.562) | 0.250 (0.217, 0.286) | 0.172 | 0.163 (0.127, 0.200) | 0.6506 |
| Model 3 | 15 | 0.714 (0.537, 0.854) | 0.766 (0.682, 0.837) | 0.463 (0.326, 0.604) | 0.905 (0.832, 0.953) | 0.844 (0.772, 0.910) | 0.587 (0.426, 0.759) | 0.124 (0.094, 0.156) | 0.172 | 0.124 (0.091, 0.159) | 0.2239 |
| Model 4 | 15 | 0.714 (0.537, 0.854) | 0.766 (0.682, 0.837) | 0.463 (0.326, 0.604) | 0.905 (0.832, 0.953) | 0.844 (0.768, 0.913) | 0.639 (0.478, 0.810) | 0.127 (0.093, 0.163) | 0.172 | 0.118 (0.083, 0.155) | 0.3091 |
| Model 5 | 15 | 0.714 (0.537, 0.854) | 0.823 (0.744, 0.885) | 0.532 (0.381, 0.679) | 0.911 (0.842, 0.956) | 0.885 (0.819, 0.940) | 0.755 (0.605, 0.869) | 0.112 (0.077, 0.152) | 0.172 | 0.101 (0.068, 0.141) | 0.2816 |

**CatBoost** (`rev_metrics_by_stage_CAT_test_bootCI.csv`):

| Model | n_feats | Sens | Spec | PPV | NPV | AUROC | AP | Brier | Null Brier | Brier (recal.) | Thr |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Model 1 | 10 | 0.229 (0.104, 0.401) | 0.790 (0.708, 0.858) | 0.235 (0.107, 0.412) | 0.784 (0.702, 0.853) | 0.579 (0.466, 0.686) | 0.285 (0.193, 0.438) | 0.198 (0.183, 0.212) | 0.172 | 0.174 (0.136, 0.212) | 0.4181 |
| Model 2 | 15 | 0.543 (0.366, 0.712) | 0.734 (0.647, 0.809) | 0.365 (0.236, 0.510) | 0.850 (0.769, 0.912) | 0.649 (0.540, 0.752) | 0.391 (0.262, 0.552) | 0.231 (0.224, 0.237) | 0.172 | 0.165 (0.130, 0.200) | 0.5033 |
| Model 3 | 15 | 0.714 (0.537, 0.854) | 0.790 (0.708, 0.858) | 0.490 (0.348, 0.634) | 0.907 (0.836, 0.955) | 0.824 (0.743, 0.893) | 0.577 (0.424, 0.721) | 0.200 (0.191, 0.209) | 0.172 | 0.133 (0.100, 0.169) | 0.4466 |
| Model 4 | 15 | 0.743 (0.567, 0.875) | 0.831 (0.753, 0.892) | 0.553 (0.401, 0.698) | 0.920 (0.853, 0.963) | 0.837 (0.753, 0.910) | 0.625 (0.468, 0.795) | 0.125 (0.100, 0.153) | 0.172 | 0.117 (0.085, 0.152) | 0.3222 |
| Model 5 | 15 | 0.857 (0.697, 0.952) | 0.855 (0.780, 0.912) | 0.625 (0.474, 0.760) | 0.955 (0.898, 0.985) | 0.926 (0.864, 0.974) | 0.849 (0.735, 0.930) | 0.084 (0.060, 0.113) | 0.172 | 0.080 (0.052, 0.112) | 0.2493 |

**M1–M5 feature sets** (selected on training data; display names):

| Algorithm | Model | n_feats | Features |
|---|---|---|---|
| XGBoost | Model 1 | 10 | QFT result; BMI; Age; Prior TB disease (self-report); History of contact with TB disease case; Sex; Diabetes; BCG vax; Race; Asthma |
| XGBoost | Model 2 | 15 | QFT result; BMI; Weight loss; Age; Night sweats; Persistent cough; Sex; History of contact with TB disease case; Prior TB disease (self-report); Diabetes; Hemoptysis; Fatigue; Shortness breath; BCG vax; Fever |
| XGBoost | Model 3 | 15 | QFT result; Infiltrates; Provider CXR interpret; Nodules; BMI; Weight loss; Sex; Age; History of contact with TB disease case; Prior TB disease (self-report); Night sweats; BCG vax; Persistent cough; Cavities; Shortness breath |
| XGBoost | Model 4 | 15 | LYM; QFT result; Provider CXR interpret; NEU; Infiltrates; Nodules; Sex; Age; MCH; MCV; RBC; History of contact with TB disease case; Weight loss; PCT; Prior TB disease (self-report) |
| XGBoost | Model 5 | 15 | qXR Score; QFT result; Age; LYM; MCH; NEU; Prior TB disease (self-report); RBC; Sex; History of contact with TB disease case; MCV; PCT; Nodules; Weight loss; Provider CXR interpret |
| CatBoost | Model 1 | 10 | QFT result; BMI; History of contact with TB disease case; Age; Sex; Diabetes; Prior TB disease (self-report); BCG vax; Race; Asthma |
| CatBoost | Model 2 | 15 | QFT result; BMI; Weight loss; Night sweats; Persistent cough; History of contact with TB disease case; Age; Sex; Prior TB disease (self-report); Fatigue; Diabetes; Hemoptysis; Shortness breath; BCG vax; Asthma |
| CatBoost | Model 3 | 15 | Provider CXR interpret; QFT result; Nodules; BMI; Infiltrates; Weight loss; Sex; Age; History of contact with TB disease case; Prior TB disease (self-report); Cavities; Night sweats; Persistent cough; Shortness breath; Hemoptysis |
| CatBoost | Model 4 | 15 | QFT result; LYM; Provider CXR interpret; NEU; Infiltrates; MCV; Sex; Nodules; BMI; MCH; Age; PCT; PLT; PDW; WBC |
| CatBoost | Model 5 | 15 | qXR Score; NEU; LYM; Nodules; Provider CXR interpret; MCH; PLT; QFT result; PCT; MCV; BMI; Infiltrates; Age; PDW; WBC |

**Probability diagnostics** (`rev_probability_diagnostics.csv`). Brier skill = 1 − Brier / Null Brier (≤ 0 = no better than no-skill). CatBoost M1–M3 stopped after 6–13 trees (predictions bunched around 0.4–0.5); XGBoost M1–M2 probabilities are shifted upward by class weights (median prediction ≈ 0.5 vs prevalence 0.22). After training-OOF recalibration only CatBoost M1 (test AUROC 0.58) remains at no-skill.

| Algorithm | Model | Trees | Learning rate | Positive-class weight | Pred min | Pred median | Pred max | Pred SD | Brier | Null Brier | Brier skill | Calib intercept | Calib slope | Recal. slope (train OOF) | Brier (recal.) | Brier skill (recal.) | Calib slope (recal.) |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| XGBoost | Full-feature | 62 | 0.093 | 1.0 | 0.017 | 0.07 | 0.881 | 0.269 | 0.086 | 0.172 | 0.498 | 0.102 | 1.155 | 1.13 | 0.086 | 0.498 | 1.022 |
| XGBoost | Model 1 | 32 | 0.053 | 4.012 | 0.176 | 0.502 | 0.759 | 0.17 | 0.239 | 0.172 | -0.394 | -1.272 | 0.581 | 1.213 | 0.169 | 0.013 | 0.479 |
| XGBoost | Model 2 | 138 | 0.063 | 5.053 | 0.086 | 0.473 | 0.921 | 0.241 | 0.25 | 0.172 | -0.458 | -1.502 | 0.527 | 0.941 | 0.163 | 0.051 | 0.56 |
| XGBoost | Model 3 | 96 | 0.082 | 1.0 | 0.036 | 0.132 | 0.822 | 0.191 | 0.124 | 0.172 | 0.278 | -0.006 | 1.26 | 1.306 | 0.124 | 0.279 | 0.965 |
| XGBoost | Model 4 | 153 | 0.093 | 2.322 | 0.004 | 0.151 | 0.982 | 0.294 | 0.127 | 0.172 | 0.261 | -0.558 | 0.764 | 0.907 | 0.118 | 0.313 | 0.842 |
| XGBoost | Model 5 | 427 | 0.086 | 2.385 | 0.001 | 0.072 | 0.993 | 0.335 | 0.112 | 0.172 | 0.346 | -0.49 | 0.676 | 0.84 | 0.101 | 0.409 | 0.804 |
| CatBoost | Full-feature | 34 | 0.082 | 1.0 | 0.018 | 0.088 | 0.877 | 0.254 | 0.085 | 0.172 | 0.507 | 0.025 | 1.326 | 1.279 | 0.083 | 0.519 | 1.037 |
| CatBoost | Model 1 | 13 | 0.051 | 1.0 | 0.327 | 0.382 | 0.485 | 0.037 | 0.198 | 0.172 | -0.151 | -0.811 | 1.696 | 3.893 | 0.174 | -0.012 | 0.436 |
| CatBoost | Model 2 | 6 | 0.073 | 2.478 | 0.395 | 0.476 | 0.544 | 0.043 | 0.231 | 0.172 | -0.343 | -1.17 | 3.144 | 5.167 | 0.165 | 0.04 | 0.609 |
| CatBoost | Model 3 | 7 | 0.061 | 1.0 | 0.373 | 0.422 | 0.551 | 0.042 | 0.2 | 0.172 | -0.164 | -0.981 | 7.609 | 6.75 | 0.133 | 0.226 | 1.127 |
| CatBoost | Model 4 | 45 | 0.078 | 1.0 | 0.079 | 0.185 | 0.729 | 0.168 | 0.125 | 0.172 | 0.269 | -0.278 | 1.645 | 1.727 | 0.117 | 0.316 | 0.952 |
| CatBoost | Model 5 | 89 | 0.076 | 1.0 | 0.027 | 0.134 | 0.814 | 0.232 | 0.084 | 0.172 | 0.509 | -0.11 | 1.703 | 1.328 | 0.08 | 0.532 | 1.282 |

## 5. Apparent vs cross-validated vs held-out test performance (R1.4)

Apparent = refit on all 636 training rows, scored on the same rows. CV (5-fold OOF, fixed params) = same tuned params refit within 5 training folds (features/hyperparameters were chosen on all 636 rows, so this is optimistic). Nested-CV = Section 3. Test = 159 held-out rows with 95% CIs. One threshold per model. File: `rev_train_cv_test_metrics.csv` (numeric version: `rev_train_cv_test_metrics_numeric.csv`).

| Algorithm | Model | n_feats | Setting | Sens | Spec | PPV | NPV | AUROC | AP | Brier | Null Brier | Brier (recal.) | Thr |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| XGBoost | Full-feature | 40 | Apparent (train, n=636) | 0.929 | 0.824 | 0.601 | 0.976 | 0.967 | 0.912 | 0.061 |  |  | 0.1710 |
| XGBoost | Full-feature | 40 | CV: 5-fold OOF, fixed params (train) | 0.844 | 0.800 | 0.546 | 0.947 | 0.905 | 0.783 | 0.092 |  |  | 0.1710 |
| XGBoost | Full-feature | 40 | CV: nested-CV OOF (train; per-fold thr) | 0.780 | 0.848 | 0.595 | 0.931 | 0.892 | 0.725 | 0.103 |  |  | per fold |
| XGBoost | Full-feature | 40 | Held-out test (n=159) | 0.829 (0.664, 0.934) | 0.847 (0.771, 0.905) | 0.604 (0.453, 0.742) | 0.946 (0.886, 0.980) | 0.917 (0.856, 0.964) | 0.817 (0.681, 0.911) | 0.086 (0.057, 0.120) | 0.172 | 0.086 (0.056, 0.122) | 0.1710 |
| XGBoost | Model 1 | 10 | Apparent (train, n=636) | 0.674 | 0.832 | 0.534 | 0.900 | 0.857 | 0.609 | 0.190 |  |  | 0.6183 |
| XGBoost | Model 1 | 10 | CV: 5-fold OOF, fixed params (train) | 0.482 | 0.800 | 0.407 | 0.844 | 0.733 | 0.398 | 0.212 |  |  | 0.6183 |
| XGBoost | Model 1 | 10 | Held-out test (n=159) | 0.429 (0.263, 0.606) | 0.790 (0.708, 0.858) | 0.366 (0.221, 0.531) | 0.831 (0.750, 0.893) | 0.629 (0.519, 0.740) | 0.329 (0.224, 0.494) | 0.239 (0.215, 0.264) | 0.172 | 0.169 (0.134, 0.205) | 0.6183 |
| XGBoost | Model 2 | 15 | Apparent (train, n=636) | 0.709 | 0.842 | 0.562 | 0.910 | 0.882 | 0.660 | 0.181 |  |  | 0.6506 |
| XGBoost | Model 2 | 15 | CV: 5-fold OOF, fixed params (train) | 0.574 | 0.802 | 0.453 | 0.869 | 0.757 | 0.455 | 0.218 |  |  | 0.6506 |
| XGBoost | Model 2 | 15 | Held-out test (n=159) | 0.514 (0.340, 0.686) | 0.774 (0.690, 0.844) | 0.391 (0.251, 0.546) | 0.850 (0.770, 0.910) | 0.663 (0.550, 0.773) | 0.391 (0.258, 0.562) | 0.250 (0.217, 0.286) | 0.172 | 0.163 (0.127, 0.200) | 0.6506 |
| XGBoost | Model 3 | 15 | Apparent (train, n=636) | 0.809 | 0.816 | 0.556 | 0.937 | 0.894 | 0.756 | 0.103 |  |  | 0.2239 |
| XGBoost | Model 3 | 15 | CV: 5-fold OOF, fixed params (train) | 0.738 | 0.800 | 0.512 | 0.915 | 0.830 | 0.639 | 0.121 |  |  | 0.2239 |
| XGBoost | Model 3 | 15 | Held-out test (n=159) | 0.714 (0.537, 0.854) | 0.766 (0.682, 0.837) | 0.463 (0.326, 0.604) | 0.905 (0.832, 0.953) | 0.844 (0.772, 0.910) | 0.587 (0.426, 0.759) | 0.124 (0.094, 0.156) | 0.172 | 0.124 (0.091, 0.159) | 0.2239 |
| XGBoost | Model 4 | 15 | Apparent (train, n=636) | 0.979 | 0.857 | 0.660 | 0.993 | 0.980 | 0.924 | 0.057 |  |  | 0.3091 |
| XGBoost | Model 4 | 15 | CV: 5-fold OOF, fixed params (train) | 0.801 | 0.804 | 0.538 | 0.934 | 0.876 | 0.672 | 0.115 |  |  | 0.3091 |
| XGBoost | Model 4 | 15 | Held-out test (n=159) | 0.714 (0.537, 0.854) | 0.766 (0.682, 0.837) | 0.463 (0.326, 0.604) | 0.905 (0.832, 0.953) | 0.844 (0.768, 0.913) | 0.639 (0.478, 0.810) | 0.127 (0.093, 0.163) | 0.172 | 0.118 (0.083, 0.155) | 0.3091 |
| XGBoost | Model 5 | 15 | Apparent (train, n=636) | 1.000 | 0.883 | 0.709 | 1.000 | 0.991 | 0.954 | 0.038 |  |  | 0.2816 |
| XGBoost | Model 5 | 15 | CV: 5-fold OOF, fixed params (train) | 0.837 | 0.828 | 0.581 | 0.947 | 0.914 | 0.770 | 0.097 |  |  | 0.2816 |
| XGBoost | Model 5 | 15 | Held-out test (n=159) | 0.714 (0.537, 0.854) | 0.823 (0.744, 0.885) | 0.532 (0.381, 0.679) | 0.911 (0.842, 0.956) | 0.885 (0.819, 0.940) | 0.755 (0.605, 0.869) | 0.112 (0.077, 0.152) | 0.172 | 0.101 (0.068, 0.141) | 0.2816 |
| CatBoost | Full-feature | 40 | Apparent (train, n=636) | 0.950 | 0.842 | 0.632 | 0.983 | 0.980 | 0.951 | 0.053 |  |  | 0.2031 |
| CatBoost | Full-feature | 40 | CV: 5-fold OOF, fixed params (train) | 0.851 | 0.804 | 0.553 | 0.950 | 0.909 | 0.779 | 0.094 |  |  | 0.2031 |
| CatBoost | Full-feature | 40 | CV: nested-CV OOF (train; per-fold thr) | 0.816 | 0.816 | 0.558 | 0.940 | 0.853 | 0.698 | 0.127 |  |  | per fold |
| CatBoost | Full-feature | 40 | Held-out test (n=159) | 0.857 (0.697, 0.952) | 0.823 (0.744, 0.885) | 0.577 (0.432, 0.713) | 0.953 (0.894, 0.985) | 0.908 (0.836, 0.964) | 0.814 (0.681, 0.911) | 0.085 (0.057, 0.117) | 0.172 | 0.083 (0.053, 0.117) | 0.2031 |
| CatBoost | Model 1 | 10 | Apparent (train, n=636) | 0.411 | 0.832 | 0.411 | 0.832 | 0.710 | 0.469 | 0.191 |  |  | 0.4181 |
| CatBoost | Model 1 | 10 | CV: 5-fold OOF, fixed params (train) | 0.383 | 0.818 | 0.375 | 0.823 | 0.657 | 0.336 | 0.197 |  |  | 0.4181 |
| CatBoost | Model 1 | 10 | Held-out test (n=159) | 0.229 (0.104, 0.401) | 0.790 (0.708, 0.858) | 0.235 (0.107, 0.412) | 0.784 (0.702, 0.853) | 0.579 (0.466, 0.686) | 0.285 (0.193, 0.438) | 0.198 (0.183, 0.212) | 0.172 | 0.174 (0.136, 0.212) | 0.4181 |
| CatBoost | Model 2 | 15 | Apparent (train, n=636) | 0.660 | 0.766 | 0.445 | 0.888 | 0.790 | 0.479 | 0.223 |  |  | 0.5033 |
| CatBoost | Model 2 | 15 | CV: 5-fold OOF, fixed params (train) | 0.454 | 0.800 | 0.393 | 0.837 | 0.695 | 0.352 | 0.231 |  |  | 0.5033 |
| CatBoost | Model 2 | 15 | Held-out test (n=159) | 0.543 (0.366, 0.712) | 0.734 (0.647, 0.809) | 0.365 (0.236, 0.510) | 0.850 (0.769, 0.912) | 0.649 (0.540, 0.752) | 0.391 (0.262, 0.552) | 0.231 (0.224, 0.237) | 0.172 | 0.165 (0.130, 0.200) | 0.5033 |
| CatBoost | Model 3 | 15 | Apparent (train, n=636) | 0.773 | 0.786 | 0.507 | 0.924 | 0.828 | 0.659 | 0.199 |  |  | 0.4466 |
| CatBoost | Model 3 | 15 | CV: 5-fold OOF, fixed params (train) | 0.610 | 0.806 | 0.473 | 0.879 | 0.781 | 0.552 | 0.203 |  |  | 0.4466 |
| CatBoost | Model 3 | 15 | Held-out test (n=159) | 0.714 (0.537, 0.854) | 0.790 (0.708, 0.858) | 0.490 (0.348, 0.634) | 0.907 (0.836, 0.955) | 0.824 (0.743, 0.893) | 0.577 (0.424, 0.721) | 0.200 (0.191, 0.209) | 0.172 | 0.133 (0.100, 0.169) | 0.4466 |
| CatBoost | Model 4 | 15 | Apparent (train, n=636) | 0.801 | 0.851 | 0.604 | 0.938 | 0.912 | 0.781 | 0.107 |  |  | 0.3222 |
| CatBoost | Model 4 | 15 | CV: 5-fold OOF, fixed params (train) | 0.759 | 0.810 | 0.532 | 0.922 | 0.849 | 0.600 | 0.127 |  |  | 0.3222 |
| CatBoost | Model 4 | 15 | Held-out test (n=159) | 0.743 (0.567, 0.875) | 0.831 (0.753, 0.892) | 0.553 (0.401, 0.698) | 0.920 (0.853, 0.963) | 0.837 (0.753, 0.910) | 0.625 (0.468, 0.795) | 0.125 (0.100, 0.153) | 0.172 | 0.117 (0.085, 0.152) | 0.3222 |
| CatBoost | Model 5 | 15 | Apparent (train, n=636) | 0.894 | 0.859 | 0.643 | 0.966 | 0.957 | 0.887 | 0.075 |  |  | 0.2493 |
| CatBoost | Model 5 | 15 | CV: 5-fold OOF, fixed params (train) | 0.865 | 0.802 | 0.555 | 0.954 | 0.898 | 0.689 | 0.104 |  |  | 0.2493 |
| CatBoost | Model 5 | 15 | Held-out test (n=159) | 0.857 (0.697, 0.952) | 0.855 (0.780, 0.912) | 0.625 (0.474, 0.760) | 0.955 (0.898, 0.985) | 0.926 (0.864, 0.974) | 0.849 (0.735, 0.930) | 0.084 (0.060, 0.113) | 0.172 | 0.080 (0.052, 0.112) | 0.2493 |

**Paired bootstrap tests of test-set ROC-AUC** (same resampled rows for both models, B = 2,000; `rev_paired_bootstrap_auc_tests.csv`):

| Comparison (A vs B) | AUC A | AUC B | ΔAUC (A−B) | p (paired bootstrap) | B used |
|---|---|---|---|---|---|
| XGBoost: Full-feature vs Model 5 | 0.917 | 0.885 | +0.031 (+0.003, +0.062) | 0.027 | 2000 |
| CatBoost: Full-feature vs Model 5 | 0.908 | 0.926 | -0.018 (-0.039, -0.002) | 0.028 | 2000 |
| Model 5: XGBoost vs CatBoost | 0.885 | 0.926 | -0.040 (-0.080, -0.004) | 0.025 | 2000 |

## 6. TB-history leakage check and original-variable comparison (supplementary)

Components of the original composite by outcome (`rev_tbhistory_leakage_check.csv`):

| Item | Response | No TB disease | TB disease |
|---|---|---|---|
| Original composite (model input before revision) | No | 480 | 40 |
| Original composite (model input before revision) | Yes | 139 | 136 |
| Interview: ever had TB in the past? (model input now; Don't know -> No) | Don't know | 1 | 0 |
| Interview: ever had TB in the past? (model input now; Don't know -> No) | No | 479 | 146 |
| Interview: ever had TB in the past? (model input now; Don't know -> No) | Yes | 139 | 30 |
| Chart review: recently diagnosed with TB? | Don't know | 19 | 4 |
| Chart review: recently diagnosed with TB? | Missing | 4 | 0 |
| Chart review: recently diagnosed with TB? | No | 596 | 42 |
| Chart review: recently diagnosed with TB? | Yes | 0 | 130 |

M1–M5 re-run with the original composite in place of interview-only prior TB (same pipeline, test n = 159; `rev_tbhistory_original_composite_sensitivity.csv`):

| Algorithm | Model | Analysis | Sens | Spec | PPV | NPV | AUROC | AP | Brier | Null Brier | Brier (recal.) | Thr |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| XGBoost | Model 1 | Main: interview-only prior TB | 0.429 (0.263, 0.606) | 0.790 (0.708, 0.858) | 0.366 (0.221, 0.531) | 0.831 (0.750, 0.893) | 0.629 (0.519, 0.740) | 0.329 (0.224, 0.494) | 0.239 (0.215, 0.264) | 0.172 | 0.169 (0.134, 0.205) | 0.6183 |
| XGBoost | Model 1 | Supplementary: original composite | 0.686 (0.507, 0.831) | 0.871 (0.799, 0.924) | 0.600 (0.433, 0.751) | 0.908 (0.841, 0.953) | 0.806 (0.720, 0.882) | 0.530 (0.370, 0.695) | 0.142 (0.111, 0.173) | 0.172 | 0.135 (0.100, 0.173) | 0.2674 |
| XGBoost | Model 2 | Main: interview-only prior TB | 0.514 (0.340, 0.686) | 0.774 (0.690, 0.844) | 0.391 (0.251, 0.546) | 0.850 (0.770, 0.910) | 0.663 (0.550, 0.773) | 0.391 (0.258, 0.562) | 0.250 (0.217, 0.286) | 0.172 | 0.163 (0.127, 0.200) | 0.6506 |
| XGBoost | Model 2 | Supplementary: original composite | 0.657 (0.478, 0.809) | 0.879 (0.808, 0.931) | 0.605 (0.434, 0.760) | 0.901 (0.833, 0.948) | 0.817 (0.730, 0.893) | 0.589 (0.435, 0.748) | 0.133 (0.102, 0.166) | 0.172 | 0.132 (0.097, 0.170) | 0.2633 |
| XGBoost | Model 3 | Main: interview-only prior TB | 0.714 (0.537, 0.854) | 0.766 (0.682, 0.837) | 0.463 (0.326, 0.604) | 0.905 (0.832, 0.953) | 0.844 (0.772, 0.910) | 0.587 (0.426, 0.759) | 0.124 (0.094, 0.156) | 0.172 | 0.124 (0.091, 0.159) | 0.2239 |
| XGBoost | Model 3 | Supplementary: original composite | 0.771 (0.599, 0.896) | 0.847 (0.771, 0.905) | 0.587 (0.432, 0.730) | 0.929 (0.865, 0.969) | 0.891 (0.830, 0.943) | 0.683 (0.530, 0.840) | 0.114 (0.088, 0.142) | 0.172 | 0.105 (0.075, 0.140) | 0.416 |
| XGBoost | Model 4 | Main: interview-only prior TB | 0.714 (0.537, 0.854) | 0.766 (0.682, 0.837) | 0.463 (0.326, 0.604) | 0.905 (0.832, 0.953) | 0.844 (0.768, 0.913) | 0.639 (0.478, 0.810) | 0.127 (0.093, 0.163) | 0.172 | 0.118 (0.083, 0.155) | 0.3091 |
| XGBoost | Model 4 | Supplementary: original composite | 0.743 (0.567, 0.875) | 0.839 (0.762, 0.899) | 0.565 (0.411, 0.711) | 0.920 (0.854, 0.963) | 0.896 (0.835, 0.945) | 0.763 (0.618, 0.875) | 0.104 (0.073, 0.140) | 0.172 | 0.098 (0.066, 0.134) | 0.3302 |
| XGBoost | Model 5 | Main: interview-only prior TB | 0.714 (0.537, 0.854) | 0.823 (0.744, 0.885) | 0.532 (0.381, 0.679) | 0.911 (0.842, 0.956) | 0.885 (0.819, 0.940) | 0.755 (0.605, 0.869) | 0.112 (0.077, 0.152) | 0.172 | 0.101 (0.068, 0.141) | 0.2816 |
| XGBoost | Model 5 | Supplementary: original composite | 0.829 (0.664, 0.934) | 0.871 (0.799, 0.924) | 0.644 (0.488, 0.781) | 0.947 (0.889, 0.980) | 0.919 (0.857, 0.965) | 0.821 (0.691, 0.915) | 0.087 (0.058, 0.120) | 0.172 | 0.084 (0.054, 0.119) | 0.2898 |
| CatBoost | Model 1 | Main: interview-only prior TB | 0.229 (0.104, 0.401) | 0.790 (0.708, 0.858) | 0.235 (0.107, 0.412) | 0.784 (0.702, 0.853) | 0.579 (0.466, 0.686) | 0.285 (0.193, 0.438) | 0.198 (0.183, 0.212) | 0.172 | 0.174 (0.136, 0.212) | 0.4181 |
| CatBoost | Model 1 | Supplementary: original composite | 0.600 (0.421, 0.761) | 0.855 (0.780, 0.912) | 0.538 (0.372, 0.699) | 0.883 (0.812, 0.935) | 0.821 (0.724, 0.900) | 0.653 (0.495, 0.797) | 0.132 (0.107, 0.158) | 0.172 | 0.123 (0.090, 0.159) | 0.4089 |
| CatBoost | Model 2 | Main: interview-only prior TB | 0.543 (0.366, 0.712) | 0.734 (0.647, 0.809) | 0.365 (0.236, 0.510) | 0.850 (0.769, 0.912) | 0.649 (0.540, 0.752) | 0.391 (0.262, 0.552) | 0.231 (0.224, 0.237) | 0.172 | 0.165 (0.130, 0.200) | 0.5033 |
| CatBoost | Model 2 | Supplementary: original composite | 0.629 (0.449, 0.785) | 0.871 (0.799, 0.924) | 0.579 (0.408, 0.737) | 0.893 (0.823, 0.942) | 0.793 (0.693, 0.878) | 0.641 (0.485, 0.782) | 0.171 (0.157, 0.185) | 0.172 | 0.124 (0.091, 0.162) | 0.4554 |
| CatBoost | Model 3 | Main: interview-only prior TB | 0.714 (0.537, 0.854) | 0.790 (0.708, 0.858) | 0.490 (0.348, 0.634) | 0.907 (0.836, 0.955) | 0.824 (0.743, 0.893) | 0.577 (0.424, 0.721) | 0.200 (0.191, 0.209) | 0.172 | 0.133 (0.100, 0.169) | 0.4466 |
| CatBoost | Model 3 | Supplementary: original composite | 0.743 (0.567, 0.875) | 0.863 (0.790, 0.918) | 0.605 (0.444, 0.750) | 0.922 (0.858, 0.964) | 0.887 (0.821, 0.941) | 0.666 (0.510, 0.833) | 0.130 (0.109, 0.154) | 0.172 | 0.104 (0.075, 0.137) | 0.5099 |
| CatBoost | Model 4 | Main: interview-only prior TB | 0.743 (0.567, 0.875) | 0.831 (0.753, 0.892) | 0.553 (0.401, 0.698) | 0.920 (0.853, 0.963) | 0.837 (0.753, 0.910) | 0.625 (0.468, 0.795) | 0.125 (0.100, 0.153) | 0.172 | 0.117 (0.085, 0.152) | 0.3222 |
| CatBoost | Model 4 | Supplementary: original composite | 0.686 (0.507, 0.831) | 0.855 (0.780, 0.912) | 0.571 (0.410, 0.723) | 0.906 (0.838, 0.952) | 0.895 (0.835, 0.944) | 0.728 (0.574, 0.863) | 0.103 (0.071, 0.138) | 0.172 | 0.104 (0.070, 0.140) | 0.1886 |
| CatBoost | Model 5 | Main: interview-only prior TB | 0.857 (0.697, 0.952) | 0.855 (0.780, 0.912) | 0.625 (0.474, 0.760) | 0.955 (0.898, 0.985) | 0.926 (0.864, 0.974) | 0.849 (0.735, 0.930) | 0.084 (0.060, 0.113) | 0.172 | 0.080 (0.052, 0.112) | 0.2493 |
| CatBoost | Model 5 | Supplementary: original composite | 0.857 (0.697, 0.952) | 0.863 (0.790, 0.918) | 0.638 (0.485, 0.773) | 0.955 (0.899, 0.985) | 0.943 (0.895, 0.977) | 0.838 (0.702, 0.934) | 0.096 (0.075, 0.121) | 0.172 | 0.080 (0.052, 0.112) | 0.4548 |

## 7. Top-10 / top-15 one-step models (R1.2)

All 40 candidate features ranked once by training-set OOF mean |SHAP| (not the test-set permutation importance); top-k refit with the same recipe. File: `rev_topk_models.csv`; full rankings `rev_topk_shap_ranking_XGB.csv`, `rev_topk_shap_ranking_CAT.csv`.

| Algorithm | Model | n_feats | Sens | Spec | PPV | NPV | AUROC | AP | Brier | Null Brier | Brier (recal.) | Thr | Selected features |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| XGBoost | Full-feature | 40 | 0.829 (0.664, 0.934) | 0.847 (0.771, 0.905) | 0.604 (0.453, 0.742) | 0.946 (0.886, 0.980) | 0.917 (0.856, 0.964) | 0.817 (0.681, 0.911) | 0.086 (0.057, 0.120) | 0.172 | 0.086 (0.056, 0.122) | 0.171 | All 40 candidate features |
| XGBoost | Top-15 | 15 | 0.800 (0.631, 0.916) | 0.815 (0.735, 0.879) | 0.549 (0.403, 0.689) | 0.935 (0.871, 0.974) | 0.901 (0.839, 0.951) | 0.776 (0.626, 0.889) | 0.113 (0.081, 0.149) | 0.172 | 0.095 (0.065, 0.132) | 0.3565 | qXR Score; QFT result; LYM; NEU; Age; Prior TB disease (self-report); MCH; Sex; WBC; History of contact with TB disease case; RBC; RDW_SD; BMI; MCV; PCT |
| XGBoost | Top-10 | 10 | 0.771 (0.599, 0.896) | 0.806 (0.726, 0.872) | 0.529 (0.385, 0.671) | 0.926 (0.859, 0.967) | 0.903 (0.839, 0.954) | 0.798 (0.665, 0.897) | 0.102 (0.071, 0.137) | 0.172 | 0.093 (0.061, 0.129) | 0.2832 | qXR Score; QFT result; LYM; NEU; Age; Prior TB disease (self-report); MCH; Sex; WBC; History of contact with TB disease case |
| XGBoost | Model 5 | 15 | 0.714 (0.537, 0.854) | 0.823 (0.744, 0.885) | 0.532 (0.381, 0.679) | 0.911 (0.842, 0.956) | 0.885 (0.819, 0.940) | 0.755 (0.605, 0.869) | 0.112 (0.077, 0.152) | 0.172 | 0.101 (0.068, 0.141) | 0.2816 | qXR Score; QFT result; Age; LYM; MCH; NEU; Prior TB disease (self-report); RBC; Sex; History of contact with TB disease case; MCV; PCT; Nodules; Weight loss; Provider CXR interpret |
| CatBoost | Full-feature | 40 | 0.857 (0.697, 0.952) | 0.823 (0.744, 0.885) | 0.577 (0.432, 0.713) | 0.953 (0.894, 0.985) | 0.908 (0.836, 0.964) | 0.814 (0.681, 0.911) | 0.085 (0.057, 0.117) | 0.172 | 0.083 (0.053, 0.117) | 0.2031 | All 40 candidate features |
| CatBoost | Top-15 | 15 | 0.857 (0.697, 0.952) | 0.831 (0.753, 0.892) | 0.588 (0.442, 0.724) | 0.954 (0.895, 0.985) | 0.912 (0.840, 0.966) | 0.822 (0.705, 0.915) | 0.100 (0.079, 0.124) | 0.172 | 0.085 (0.055, 0.120) | 0.3033 | qXR Score; QFT result; NEU; LYM; Prior TB disease (self-report); Age; Provider CXR interpret; Nodules; MCV; Sex; MCH; BMI; Weight loss; PLT; WBC |
| CatBoost | Top-10 | 10 | 0.914 (0.769, 0.982) | 0.806 (0.726, 0.872) | 0.571 (0.432, 0.703) | 0.971 (0.917, 0.994) | 0.920 (0.860, 0.967) | 0.818 (0.686, 0.917) | 0.085 (0.058, 0.115) | 0.172 | 0.082 (0.052, 0.117) | 0.2013 | qXR Score; QFT result; NEU; LYM; Prior TB disease (self-report); Age; Provider CXR interpret; Nodules; MCV; Sex |
| CatBoost | Model 5 | 15 | 0.857 (0.697, 0.952) | 0.855 (0.780, 0.912) | 0.625 (0.474, 0.760) | 0.955 (0.898, 0.985) | 0.926 (0.864, 0.974) | 0.849 (0.735, 0.930) | 0.084 (0.060, 0.113) | 0.172 | 0.080 (0.052, 0.112) | 0.2493 | qXR Score; NEU; LYM; Nodules; Provider CXR interpret; MCH; PLT; QFT result; PCT; MCV; BMI; Infiltrates; Age; PDW; WBC |

Overlap: top-15 ∩ M5 = 12/15 features for both algorithms.

## 8. QFT and TB history (R1.2)

In the training set (n=636), a positive QFT result and a history of TB were not significantly correlated (Spearman ρ = 0.04, p = 0.3); 87 participants (13.7%) had both, 284 were QFT-positive without TB history and 53 had TB history without a positive QFT. Refitting the Model 5 XGBoost model without TB history (same hyperparameters) changed the mean |SHAP| of QFT on the test set from 0.610 to 0.623 (+2.2%), with test AUROC 0.885 vs 0.879. Household contact with a TB case (lived_with_another_tb) was retained in the XGBoost Model 5 feature set and was not retained in the CatBoost Model 5 feature set.

Training-set cross-tab (`rev_qft_tbhistory_crosstab_train.csv`):

| QFT_result_composite | TB history No | TB history Yes | Total |
|---|---|---|---|
| QFT Positive | 284 | 87 | 371 |
| QFT not positive | 212 | 53 | 265 |
| Total | 496 | 140 | 636 |

M5 XGBoost QFT mean |SHAP| with vs without TB history (`rev_qft_shap_drop_tbhistory_xgb_m5.csv`):

| Set | QFT mean|SHAP| (M5) | QFT mean|SHAP| (M5 − TB history) | Change | Change (%) |
|---|---|---|---|---|
| Test (n=159) | 0.6101 | 0.6233 | 0.0133 | 2.2 |
| Train (n=636) | 0.5991 | 0.6216 | 0.0225 | 3.7 |

## 9. Resampling sensitivity (R1.3)

Random oversampling of TB cases or undersampling of non-cases to 1:1, applied only to training data inside every fold (inner-tuning folds, threshold folds, final refit); validation and test data never resampled; class-weight ratio set to the resampled ratio (1.0). Calibration intercept = calibration-in-the-large; slope from logistic recalibration of logit(p). File: `rev_resampling_sensitivity.csv`; figure `rev_calibration_resampling_sensitivity.png`.

| Algorithm | Model | Resampling | AUROC | AP | Brier | Null Brier | Brier (recal.) | Trees | Calib intercept | Calib slope | Sens | Spec | Thr |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| XGBoost | Full-feature | Unresampled | 0.917 (0.856, 0.964) | 0.817 (0.681, 0.911) | 0.086 (0.057, 0.120) | 0.172 | 0.086 (0.056, 0.122) | 62 | 0.102 (-0.409, 0.583) | 1.155 (0.885, 1.571) | 0.829 (0.664, 0.934) | 0.847 (0.771, 0.905) | 0.171 |
| XGBoost | Full-feature | Oversampled 1:1 | 0.914 (0.849, 0.964) | 0.809 (0.664, 0.908) | 0.112 (0.083, 0.146) | 0.172 | 0.086 (0.057, 0.121) | 55 | -1.156 (-1.657, -0.688) | 1.148 (0.858, 1.648) | 0.829 (0.664, 0.934) | 0.855 (0.780, 0.912) | 0.4783 |
| XGBoost | Full-feature | Undersampled 1:1 | 0.919 (0.854, 0.970) | 0.800 (0.654, 0.914) | 0.122 (0.089, 0.158) | 0.172 | 0.085 (0.056, 0.118) | 105 | -1.516 (-2.075, -1.019) | 1.025 (0.743, 1.548) | 0.943 (0.808, 0.993) | 0.766 (0.682, 0.837) | 0.4673 |
| XGBoost | Model 5 | Unresampled | 0.885 (0.819, 0.940) | 0.755 (0.605, 0.869) | 0.112 (0.077, 0.152) | 0.172 | 0.101 (0.068, 0.141) | 427 | -0.490 (-1.242, 0.276) | 0.676 (0.504, 0.934) | 0.714 (0.537, 0.854) | 0.823 (0.744, 0.885) | 0.2816 |
| XGBoost | Model 5 | Oversampled 1:1 | 0.889 (0.821, 0.944) | 0.763 (0.608, 0.883) | 0.123 (0.089, 0.163) | 0.172 | 0.096 (0.063, 0.134) | 229 | -1.109 (-1.780, -0.435) | 0.744 (0.549, 1.057) | 0.714 (0.537, 0.854) | 0.806 (0.726, 0.872) | 0.4139 |
| XGBoost | Model 5 | Undersampled 1:1 | 0.909 (0.844, 0.960) | 0.796 (0.653, 0.899) | 0.124 (0.090, 0.163) | 0.172 | 0.091 (0.062, 0.125) | 184 | -1.488 (-2.141, -0.879) | 0.847 (0.630, 1.205) | 0.886 (0.733, 0.968) | 0.790 (0.708, 0.858) | 0.4364 |
| CatBoost | Full-feature | Unresampled | 0.908 (0.836, 0.964) | 0.814 (0.681, 0.911) | 0.085 (0.057, 0.117) | 0.172 | 0.083 (0.053, 0.117) | 34 | 0.025 (-0.443, 0.445) | 1.326 (0.976, 1.895) | 0.857 (0.697, 0.952) | 0.823 (0.744, 0.885) | 0.2031 |
| CatBoost | Full-feature | Oversampled 1:1 | 0.916 (0.861, 0.960) | 0.788 (0.640, 0.900) | 0.093 (0.061, 0.130) | 0.172 | 0.090 (0.060, 0.124) | 106 | -0.181 (-0.854, 0.448) | 0.833 (0.630, 1.166) | 0.743 (0.567, 0.875) | 0.871 (0.799, 0.924) | 0.2579 |
| CatBoost | Full-feature | Undersampled 1:1 | 0.905 (0.827, 0.964) | 0.791 (0.638, 0.906) | 0.116 (0.084, 0.154) | 0.172 | 0.086 (0.058, 0.119) | 70 | -1.442 (-2.061, -0.874) | 0.881 (0.631, 1.315) | 0.886 (0.733, 0.968) | 0.815 (0.735, 0.879) | 0.4514 |
| CatBoost | Model 5 | Unresampled | 0.926 (0.864, 0.974) | 0.849 (0.735, 0.930) | 0.084 (0.060, 0.113) | 0.172 | 0.080 (0.052, 0.112) | 89 | -0.110 (-0.529, 0.261) | 1.703 (1.291, 2.427) | 0.857 (0.697, 0.952) | 0.855 (0.780, 0.912) | 0.2493 |
| CatBoost | Model 5 | Oversampled 1:1 | 0.926 (0.865, 0.972) | 0.844 (0.726, 0.929) | 0.107 (0.083, 0.133) | 0.172 | 0.083 (0.055, 0.113) | 60 | -0.976 (-1.397, -0.601) | 1.689 (1.244, 2.528) | 0.914 (0.769, 0.982) | 0.823 (0.744, 0.885) | 0.4368 |
| CatBoost | Model 5 | Undersampled 1:1 | 0.921 (0.851, 0.976) | 0.820 (0.688, 0.925) | 0.118 (0.093, 0.146) | 0.172 | 0.082 (0.054, 0.114) | 84 | -1.160 (-1.587, -0.790) | 1.641 (1.175, 2.507) | 0.914 (0.769, 0.982) | 0.839 (0.762, 0.899) | 0.4982 |

## 10. Age, QFT and subgroups (R2.5–R2.7)

**Age ≥ 15 sensitivity analysis** — train and test restricted to age ≥ 15 within the same split (train 593 / 135 TB; test 151 / 34 TB); RFE and M1–M5 re-run. 'Main models, age≥15 test rows' = main models and thresholds scored on the same 151 patients. File: `rev_age15plus_sensitivity.csv`.

| Algorithm | Model | Analysis | n_test | TB_test | Sens | Spec | PPV | NPV | AUROC | AP | Brier | Null Brier | Brier (recal.) | Thr |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| XGBoost | Model 1 | Main analysis (all ages) | 159 | 35 | 0.429 (0.263, 0.606) | 0.790 (0.708, 0.858) | 0.366 (0.221, 0.531) | 0.831 (0.750, 0.893) | 0.629 (0.519, 0.740) | 0.329 (0.224, 0.494) | 0.239 (0.215, 0.264) | 0.172 | 0.169 (0.134, 0.205) | 0.6183 |
| XGBoost | Model 1 | Main models, age>=15 test rows | 151 | 34 | 0.441 (0.272, 0.621) | 0.778 (0.692, 0.849) | 0.366 (0.221, 0.531) | 0.827 (0.743, 0.893) | 0.624 (0.514, 0.727) | 0.330 (0.219, 0.495) | 0.242 (0.216, 0.268) | 0.174 | 0.174 (0.138, 0.212) | 0.6183 |
| XGBoost | Model 1 | Re-run on age>=15 | 151 | 34 | 0.294 (0.151, 0.475) | 0.786 (0.701, 0.857) | 0.286 (0.146, 0.463) | 0.793 (0.708, 0.863) | 0.603 (0.489, 0.712) | 0.312 (0.199, 0.461) | 0.172 (0.136, 0.210) | 0.174 | 0.177 (0.139, 0.218) | 0.2483 |
| XGBoost | Model 2 | Main analysis (all ages) | 159 | 35 | 0.514 (0.340, 0.686) | 0.774 (0.690, 0.844) | 0.391 (0.251, 0.546) | 0.850 (0.770, 0.910) | 0.663 (0.550, 0.773) | 0.391 (0.258, 0.562) | 0.250 (0.217, 0.286) | 0.172 | 0.163 (0.127, 0.200) | 0.6506 |
| XGBoost | Model 2 | Main models, age>=15 test rows | 151 | 34 | 0.500 (0.324, 0.676) | 0.778 (0.692, 0.849) | 0.395 (0.250, 0.556) | 0.843 (0.760, 0.906) | 0.660 (0.543, 0.767) | 0.391 (0.251, 0.561) | 0.249 (0.213, 0.286) | 0.174 | 0.166 (0.130, 0.207) | 0.6506 |
| XGBoost | Model 2 | Re-run on age>=15 | 151 | 34 | 0.471 (0.298, 0.649) | 0.812 (0.729, 0.878) | 0.421 (0.263, 0.592) | 0.841 (0.760, 0.903) | 0.648 (0.529, 0.759) | 0.396 (0.250, 0.571) | 0.223 (0.194, 0.252) | 0.174 | 0.169 (0.133, 0.211) | 0.593 |
| XGBoost | Model 3 | Main analysis (all ages) | 159 | 35 | 0.714 (0.537, 0.854) | 0.766 (0.682, 0.837) | 0.463 (0.326, 0.604) | 0.905 (0.832, 0.953) | 0.844 (0.772, 0.910) | 0.587 (0.426, 0.759) | 0.124 (0.094, 0.156) | 0.172 | 0.124 (0.091, 0.159) | 0.2239 |
| XGBoost | Model 3 | Main models, age>=15 test rows | 151 | 34 | 0.706 (0.525, 0.849) | 0.761 (0.673, 0.835) | 0.462 (0.322, 0.605) | 0.899 (0.822, 0.950) | 0.845 (0.769, 0.910) | 0.600 (0.434, 0.769) | 0.126 (0.094, 0.159) | 0.174 | 0.126 (0.092, 0.162) | 0.2239 |
| XGBoost | Model 3 | Re-run on age>=15 | 151 | 34 | 0.735 (0.556, 0.871) | 0.778 (0.692, 0.849) | 0.490 (0.348, 0.634) | 0.910 (0.836, 0.958) | 0.843 (0.765, 0.910) | 0.621 (0.454, 0.786) | 0.141 (0.111, 0.175) | 0.174 | 0.126 (0.093, 0.163) | 0.3958 |
| XGBoost | Model 4 | Main analysis (all ages) | 159 | 35 | 0.714 (0.537, 0.854) | 0.766 (0.682, 0.837) | 0.463 (0.326, 0.604) | 0.905 (0.832, 0.953) | 0.844 (0.768, 0.913) | 0.639 (0.478, 0.810) | 0.127 (0.093, 0.163) | 0.172 | 0.118 (0.083, 0.155) | 0.3091 |
| XGBoost | Model 4 | Main models, age>=15 test rows | 151 | 34 | 0.706 (0.525, 0.849) | 0.769 (0.682, 0.842) | 0.471 (0.329, 0.615) | 0.900 (0.824, 0.951) | 0.840 (0.750, 0.908) | 0.633 (0.456, 0.801) | 0.131 (0.096, 0.171) | 0.174 | 0.122 (0.087, 0.162) | 0.3091 |
| XGBoost | Model 4 | Re-run on age>=15 | 151 | 34 | 0.706 (0.525, 0.849) | 0.838 (0.758, 0.899) | 0.558 (0.399, 0.709) | 0.907 (0.836, 0.955) | 0.860 (0.783, 0.923) | 0.693 (0.522, 0.832) | 0.135 (0.101, 0.172) | 0.174 | 0.114 (0.081, 0.150) | 0.4739 |
| XGBoost | Model 5 | Main analysis (all ages) | 159 | 35 | 0.714 (0.537, 0.854) | 0.823 (0.744, 0.885) | 0.532 (0.381, 0.679) | 0.911 (0.842, 0.956) | 0.885 (0.819, 0.940) | 0.755 (0.605, 0.869) | 0.112 (0.077, 0.152) | 0.172 | 0.101 (0.068, 0.141) | 0.2816 |
| XGBoost | Model 5 | Main models, age>=15 test rows | 151 | 34 | 0.706 (0.525, 0.849) | 0.812 (0.729, 0.878) | 0.522 (0.369, 0.671) | 0.905 (0.832, 0.953) | 0.882 (0.815, 0.936) | 0.746 (0.592, 0.863) | 0.118 (0.082, 0.158) | 0.174 | 0.106 (0.071, 0.146) | 0.2816 |
| XGBoost | Model 5 | Re-run on age>=15 | 151 | 34 | 0.765 (0.588, 0.893) | 0.846 (0.768, 0.906) | 0.591 (0.432, 0.737) | 0.925 (0.858, 0.967) | 0.903 (0.835, 0.954) | 0.789 (0.650, 0.901) | 0.103 (0.073, 0.138) | 0.174 | 0.091 (0.061, 0.127) | 0.417 |
| CatBoost | Model 1 | Main analysis (all ages) | 159 | 35 | 0.229 (0.104, 0.401) | 0.790 (0.708, 0.858) | 0.235 (0.107, 0.412) | 0.784 (0.702, 0.853) | 0.579 (0.466, 0.686) | 0.285 (0.193, 0.438) | 0.198 (0.183, 0.212) | 0.172 | 0.174 (0.136, 0.212) | 0.4181 |
| CatBoost | Model 1 | Main models, age>=15 test rows | 151 | 34 | 0.206 (0.087, 0.379) | 0.829 (0.748, 0.892) | 0.259 (0.111, 0.463) | 0.782 (0.699, 0.851) | 0.592 (0.488, 0.702) | 0.293 (0.202, 0.444) | 0.197 (0.181, 0.214) | 0.174 | 0.175 (0.134, 0.218) | 0.4181 |
| CatBoost | Model 1 | Re-run on age>=15 | 151 | 34 | 0.382 (0.222, 0.564) | 0.795 (0.710, 0.864) | 0.351 (0.202, 0.525) | 0.816 (0.732, 0.882) | 0.650 (0.539, 0.755) | 0.385 (0.251, 0.553) | 0.167 (0.136, 0.199) | 0.174 | 0.167 (0.130, 0.207) | 0.3337 |
| CatBoost | Model 2 | Main analysis (all ages) | 159 | 35 | 0.543 (0.366, 0.712) | 0.734 (0.647, 0.809) | 0.365 (0.236, 0.510) | 0.850 (0.769, 0.912) | 0.649 (0.540, 0.752) | 0.391 (0.262, 0.552) | 0.231 (0.224, 0.237) | 0.172 | 0.165 (0.130, 0.200) | 0.5033 |
| CatBoost | Model 2 | Main models, age>=15 test rows | 151 | 34 | 0.529 (0.351, 0.702) | 0.726 (0.636, 0.805) | 0.360 (0.229, 0.508) | 0.842 (0.756, 0.907) | 0.640 (0.526, 0.743) | 0.363 (0.233, 0.526) | 0.231 (0.224, 0.238) | 0.174 | 0.169 (0.134, 0.208) | 0.5033 |
| CatBoost | Model 2 | Re-run on age>=15 | 151 | 34 | 0.382 (0.222, 0.564) | 0.752 (0.664, 0.827) | 0.310 (0.176, 0.471) | 0.807 (0.721, 0.877) | 0.658 (0.543, 0.760) | 0.383 (0.247, 0.548) | 0.169 (0.143, 0.198) | 0.174 | 0.168 (0.133, 0.209) | 0.3351 |
| CatBoost | Model 3 | Main analysis (all ages) | 159 | 35 | 0.714 (0.537, 0.854) | 0.790 (0.708, 0.858) | 0.490 (0.348, 0.634) | 0.907 (0.836, 0.955) | 0.824 (0.743, 0.893) | 0.577 (0.424, 0.721) | 0.200 (0.191, 0.209) | 0.172 | 0.133 (0.100, 0.169) | 0.4466 |
| CatBoost | Model 3 | Main models, age>=15 test rows | 151 | 34 | 0.706 (0.525, 0.849) | 0.786 (0.701, 0.857) | 0.490 (0.344, 0.637) | 0.902 (0.827, 0.952) | 0.821 (0.741, 0.891) | 0.586 (0.425, 0.739) | 0.200 (0.190, 0.210) | 0.174 | 0.133 (0.098, 0.170) | 0.4466 |
| CatBoost | Model 3 | Re-run on age>=15 | 151 | 34 | 0.735 (0.556, 0.871) | 0.761 (0.673, 0.835) | 0.472 (0.333, 0.614) | 0.908 (0.833, 0.957) | 0.833 (0.756, 0.900) | 0.597 (0.437, 0.773) | 0.159 (0.126, 0.195) | 0.174 | 0.129 (0.097, 0.165) | 0.4718 |
| CatBoost | Model 4 | Main analysis (all ages) | 159 | 35 | 0.743 (0.567, 0.875) | 0.831 (0.753, 0.892) | 0.553 (0.401, 0.698) | 0.920 (0.853, 0.963) | 0.837 (0.753, 0.910) | 0.625 (0.468, 0.795) | 0.125 (0.100, 0.153) | 0.172 | 0.117 (0.085, 0.152) | 0.3222 |
| CatBoost | Model 4 | Main models, age>=15 test rows | 151 | 34 | 0.735 (0.556, 0.871) | 0.838 (0.758, 0.899) | 0.568 (0.410, 0.717) | 0.916 (0.846, 0.961) | 0.832 (0.740, 0.908) | 0.618 (0.436, 0.798) | 0.128 (0.101, 0.156) | 0.174 | 0.121 (0.088, 0.157) | 0.3222 |
| CatBoost | Model 4 | Re-run on age>=15 | 151 | 34 | 0.735 (0.556, 0.871) | 0.829 (0.748, 0.892) | 0.556 (0.400, 0.704) | 0.915 (0.845, 0.960) | 0.862 (0.784, 0.925) | 0.659 (0.484, 0.824) | 0.116 (0.088, 0.145) | 0.174 | 0.113 (0.081, 0.147) | 0.285 |
| CatBoost | Model 5 | Main analysis (all ages) | 159 | 35 | 0.857 (0.697, 0.952) | 0.855 (0.780, 0.912) | 0.625 (0.474, 0.760) | 0.955 (0.898, 0.985) | 0.926 (0.864, 0.974) | 0.849 (0.735, 0.930) | 0.084 (0.060, 0.113) | 0.172 | 0.080 (0.052, 0.112) | 0.2493 |
| CatBoost | Model 5 | Main models, age>=15 test rows | 151 | 34 | 0.853 (0.689, 0.950) | 0.846 (0.768, 0.906) | 0.617 (0.464, 0.755) | 0.952 (0.891, 0.984) | 0.922 (0.858, 0.970) | 0.843 (0.729, 0.925) | 0.087 (0.063, 0.115) | 0.174 | 0.084 (0.056, 0.116) | 0.2493 |
| CatBoost | Model 5 | Re-run on age>=15 | 151 | 34 | 0.882 (0.725, 0.967) | 0.829 (0.748, 0.892) | 0.600 (0.452, 0.736) | 0.960 (0.902, 0.989) | 0.917 (0.846, 0.971) | 0.842 (0.729, 0.927) | 0.105 (0.085, 0.127) | 0.174 | 0.084 (0.056, 0.116) | 0.309 |

**QFT-excluded sensitivity analysis** — QFT removed from every stage (M1 starts from 9 variables); RFE and M1–M5 re-run. File: `rev_qft_excluded_sensitivity.csv`.

| Algorithm | Model | Analysis | Sens | Spec | PPV | NPV | AUROC | AP | Brier | Null Brier | Brier (recal.) | Thr |
|---|---|---|---|---|---|---|---|---|---|---|---|---|
| XGBoost | Model 1 | Main analysis | 0.429 (0.263, 0.606) | 0.790 (0.708, 0.858) | 0.366 (0.221, 0.531) | 0.831 (0.750, 0.893) | 0.629 (0.519, 0.740) | 0.329 (0.224, 0.494) | 0.239 (0.215, 0.264) | 0.172 | 0.169 (0.134, 0.205) | 0.6183 |
| XGBoost | Model 1 | QFT excluded | 0.400 (0.239, 0.579) | 0.823 (0.744, 0.885) | 0.389 (0.231, 0.565) | 0.829 (0.751, 0.891) | 0.642 (0.532, 0.748) | 0.319 (0.218, 0.483) | 0.231 (0.209, 0.254) | 0.172 | 0.166 (0.133, 0.200) | 0.5792 |
| XGBoost | Model 2 | Main analysis | 0.514 (0.340, 0.686) | 0.774 (0.690, 0.844) | 0.391 (0.251, 0.546) | 0.850 (0.770, 0.910) | 0.663 (0.550, 0.773) | 0.391 (0.258, 0.562) | 0.250 (0.217, 0.286) | 0.172 | 0.163 (0.127, 0.200) | 0.6506 |
| XGBoost | Model 2 | QFT excluded | 0.457 (0.288, 0.634) | 0.823 (0.744, 0.885) | 0.421 (0.263, 0.592) | 0.843 (0.766, 0.903) | 0.660 (0.546, 0.768) | 0.353 (0.242, 0.526) | 0.164 (0.130, 0.199) | 0.172 | 0.165 (0.131, 0.201) | 0.2742 |
| XGBoost | Model 3 | Main analysis | 0.714 (0.537, 0.854) | 0.766 (0.682, 0.837) | 0.463 (0.326, 0.604) | 0.905 (0.832, 0.953) | 0.844 (0.772, 0.910) | 0.587 (0.426, 0.759) | 0.124 (0.094, 0.156) | 0.172 | 0.124 (0.091, 0.159) | 0.2239 |
| XGBoost | Model 3 | QFT excluded | 0.714 (0.537, 0.854) | 0.815 (0.735, 0.879) | 0.521 (0.372, 0.667) | 0.910 (0.841, 0.956) | 0.845 (0.767, 0.911) | 0.610 (0.452, 0.789) | 0.124 (0.095, 0.157) | 0.172 | 0.123 (0.092, 0.157) | 0.2591 |
| XGBoost | Model 4 | Main analysis | 0.714 (0.537, 0.854) | 0.766 (0.682, 0.837) | 0.463 (0.326, 0.604) | 0.905 (0.832, 0.953) | 0.844 (0.768, 0.913) | 0.639 (0.478, 0.810) | 0.127 (0.093, 0.163) | 0.172 | 0.118 (0.083, 0.155) | 0.3091 |
| XGBoost | Model 4 | QFT excluded | 0.771 (0.599, 0.896) | 0.831 (0.753, 0.892) | 0.562 (0.412, 0.705) | 0.928 (0.863, 0.968) | 0.850 (0.767, 0.922) | 0.692 (0.528, 0.836) | 0.138 (0.108, 0.172) | 0.172 | 0.109 (0.078, 0.142) | 0.5093 |
| XGBoost | Model 5 | Main analysis | 0.714 (0.537, 0.854) | 0.823 (0.744, 0.885) | 0.532 (0.381, 0.679) | 0.911 (0.842, 0.956) | 0.885 (0.819, 0.940) | 0.755 (0.605, 0.869) | 0.112 (0.077, 0.152) | 0.172 | 0.101 (0.068, 0.141) | 0.2816 |
| XGBoost | Model 5 | QFT excluded | 0.857 (0.697, 0.952) | 0.847 (0.771, 0.905) | 0.612 (0.462, 0.748) | 0.955 (0.897, 0.985) | 0.913 (0.846, 0.964) | 0.814 (0.691, 0.909) | 0.097 (0.068, 0.130) | 0.172 | 0.088 (0.059, 0.122) | 0.3405 |
| CatBoost | Model 1 | Main analysis | 0.229 (0.104, 0.401) | 0.790 (0.708, 0.858) | 0.235 (0.107, 0.412) | 0.784 (0.702, 0.853) | 0.579 (0.466, 0.686) | 0.285 (0.193, 0.438) | 0.198 (0.183, 0.212) | 0.172 | 0.174 (0.136, 0.212) | 0.4181 |
| CatBoost | Model 1 | QFT excluded | 0.514 (0.340, 0.686) | 0.710 (0.621, 0.788) | 0.333 (0.211, 0.475) | 0.838 (0.753, 0.903) | 0.635 (0.532, 0.728) | 0.295 (0.206, 0.417) | 0.238 (0.235, 0.241) | 0.172 | 0.167 (0.131, 0.205) | 0.4851 |
| CatBoost | Model 2 | Main analysis | 0.543 (0.366, 0.712) | 0.734 (0.647, 0.809) | 0.365 (0.236, 0.510) | 0.850 (0.769, 0.912) | 0.649 (0.540, 0.752) | 0.391 (0.262, 0.552) | 0.231 (0.224, 0.237) | 0.172 | 0.165 (0.130, 0.200) | 0.5033 |
| CatBoost | Model 2 | QFT excluded | 0.486 (0.314, 0.660) | 0.758 (0.673, 0.830) | 0.362 (0.227, 0.515) | 0.839 (0.758, 0.902) | 0.656 (0.541, 0.759) | 0.355 (0.235, 0.507) | 0.237 (0.234, 0.240) | 0.172 | 0.164 (0.131, 0.197) | 0.4908 |
| CatBoost | Model 3 | Main analysis | 0.714 (0.537, 0.854) | 0.790 (0.708, 0.858) | 0.490 (0.348, 0.634) | 0.907 (0.836, 0.955) | 0.824 (0.743, 0.893) | 0.577 (0.424, 0.721) | 0.200 (0.191, 0.209) | 0.172 | 0.133 (0.100, 0.169) | 0.4466 |
| CatBoost | Model 3 | QFT excluded | 0.800 (0.631, 0.916) | 0.823 (0.744, 0.885) | 0.560 (0.413, 0.700) | 0.936 (0.872, 0.974) | 0.865 (0.793, 0.926) | 0.665 (0.515, 0.805) | 0.201 (0.189, 0.213) | 0.172 | 0.117 (0.091, 0.145) | 0.5459 |
| CatBoost | Model 4 | Main analysis | 0.743 (0.567, 0.875) | 0.831 (0.753, 0.892) | 0.553 (0.401, 0.698) | 0.920 (0.853, 0.963) | 0.837 (0.753, 0.910) | 0.625 (0.468, 0.795) | 0.125 (0.100, 0.153) | 0.172 | 0.117 (0.085, 0.152) | 0.3222 |
| CatBoost | Model 4 | QFT excluded | 0.743 (0.567, 0.875) | 0.815 (0.735, 0.879) | 0.531 (0.383, 0.675) | 0.918 (0.850, 0.962) | 0.841 (0.758, 0.911) | 0.650 (0.486, 0.801) | 0.139 (0.118, 0.161) | 0.172 | 0.118 (0.086, 0.152) | 0.3613 |
| CatBoost | Model 5 | Main analysis | 0.857 (0.697, 0.952) | 0.855 (0.780, 0.912) | 0.625 (0.474, 0.760) | 0.955 (0.898, 0.985) | 0.926 (0.864, 0.974) | 0.849 (0.735, 0.930) | 0.084 (0.060, 0.113) | 0.172 | 0.080 (0.052, 0.112) | 0.2493 |
| CatBoost | Model 5 | QFT excluded | 0.829 (0.664, 0.934) | 0.879 (0.808, 0.931) | 0.659 (0.501, 0.795) | 0.948 (0.890, 0.981) | 0.909 (0.835, 0.968) | 0.825 (0.698, 0.919) | 0.160 (0.149, 0.172) | 0.172 | 0.082 (0.054, 0.116) | 0.4307 |

Model 5 feature lists:

| Algorithm | Analysis | n_feats | Features |
|---|---|---|---|
| XGBoost | Main analysis | 15 | qXR Score; QFT result; Age; LYM; MCH; NEU; Prior TB disease (self-report); RBC; Sex; History of contact with TB disease case; MCV; PCT; Nodules; Weight loss; Provider CXR interpret |
| XGBoost | QFT excluded | 15 | qXR Score; NEU; LYM; Age; Prior TB disease (self-report); MCH; RBC; Sex; PCT; MCV; Provider CXR interpret; Weight loss; Infiltrates; BMI; Nodules |
| CatBoost | Main analysis | 15 | qXR Score; NEU; LYM; Nodules; Provider CXR interpret; MCH; PLT; QFT result; PCT; MCV; BMI; Infiltrates; Age; PDW; WBC |
| CatBoost | QFT excluded | 15 | qXR Score; NEU; LYM; HCT; Nodules; Infiltrates; MCV; RBC; WBC; PCT; BMI; Provider CXR interpret; Weight loss; MCH; PLT |

**Subgroup performance (exploratory)** — 5-fold out-of-fold predictions over all 795 participants using each algorithm's fixed M5 features and hyperparameters (tuned on the training set); sens/spec at the fixed M5 threshold. Optimistic because the training rows also informed selection/tuning. Subgroups with < 10 TB cases flagged. File: `rev_subgroup_performance_oof795.csv`.

| Algorithm | Subgroup | n | TB cases | AUROC | AP | Sens | Spec | Brier | Null Brier | Thr (fixed M5) | Flag |
|---|---|---|---|---|---|---|---|---|---|---|---|
| XGBoost | All participants | 795 | 176 | 0.905 (0.879, 0.930) | 0.765 (0.702, 0.832) | 0.812 (0.747, 0.867) | 0.840 (0.809, 0.868) | 0.096 (0.082, 0.111) | 0.172 | 0.2816 |  |
| XGBoost | Age 5–14 | 51 | 7 | 0.886 (0.612, 1.000) | 0.838 (0.501, 1.000) | 0.857 (0.421, 0.996) | 0.955 (0.845, 0.994) | 0.053 (0.012, 0.112) | 0.126 | 0.2816 | <10 TB cases — interpret with caution |
| XGBoost | Age 15–34 | 339 | 85 | 0.904 (0.862, 0.940) | 0.776 (0.676, 0.875) | 0.824 (0.726, 0.898) | 0.862 (0.814, 0.902) | 0.097 (0.073, 0.122) | 0.189 | 0.2816 |  |
| XGBoost | Age 35–54 | 246 | 47 | 0.915 (0.872, 0.952) | 0.786 (0.681, 0.877) | 0.787 (0.643, 0.893) | 0.824 (0.764, 0.874) | 0.094 (0.069, 0.119) | 0.155 | 0.2816 |  |
| XGBoost | Age ≥55 | 159 | 37 | 0.887 (0.823, 0.940) | 0.728 (0.579, 0.862) | 0.811 (0.648, 0.920) | 0.779 (0.695, 0.849) | 0.112 (0.081, 0.147) | 0.179 | 0.2816 |  |
| XGBoost | Diabetes: yes | 50 | 19 | 0.973 (0.928, 0.998) | 0.961 (0.889, 0.998) | 0.947 (0.740, 0.999) | 0.871 (0.702, 0.964) | 0.074 (0.035, 0.119) | 0.261 | 0.2816 |  |
| XGBoost | Diabetes: no | 745 | 157 | 0.898 (0.868, 0.924) | 0.741 (0.665, 0.811) | 0.796 (0.725, 0.856) | 0.838 (0.806, 0.867) | 0.098 (0.083, 0.114) | 0.166 | 0.2816 |  |
| CatBoost | All participants | 795 | 176 | 0.901 (0.875, 0.925) | 0.761 (0.700, 0.818) | 0.818 (0.753, 0.872) | 0.827 (0.795, 0.856) | 0.098 (0.086, 0.110) | 0.172 | 0.2493 |  |
| CatBoost | Age 5–14 | 51 | 7 | 0.870 (0.574, 1.000) | 0.792 (0.414, 1.000) | 0.857 (0.421, 0.996) | 0.955 (0.845, 0.994) | 0.053 (0.018, 0.102) | 0.126 | 0.2493 | <10 TB cases — interpret with caution |
| CatBoost | Age 15–34 | 339 | 85 | 0.901 (0.861, 0.938) | 0.820 (0.746, 0.882) | 0.800 (0.699, 0.879) | 0.854 (0.805, 0.895) | 0.100 (0.079, 0.120) | 0.189 | 0.2493 |  |
| CatBoost | Age 35–54 | 246 | 47 | 0.918 (0.877, 0.955) | 0.723 (0.598, 0.839) | 0.851 (0.717, 0.938) | 0.839 (0.781, 0.887) | 0.090 (0.070, 0.111) | 0.155 | 0.2493 |  |
| CatBoost | Age ≥55 | 159 | 37 | 0.868 (0.806, 0.921) | 0.660 (0.510, 0.813) | 0.811 (0.648, 0.920) | 0.705 (0.616, 0.784) | 0.122 (0.096, 0.149) | 0.179 | 0.2493 |  |
| CatBoost | Diabetes: yes | 50 | 19 | 0.959 (0.896, 1.000) | 0.915 (0.772, 1.000) | 1.000 (0.824, 1.000) | 0.871 (0.702, 0.964) | 0.104 (0.067, 0.146) | 0.261 | 0.2493 |  |
| CatBoost | Diabetes: no | 745 | 157 | 0.895 (0.865, 0.921) | 0.745 (0.675, 0.807) | 0.796 (0.725, 0.856) | 0.825 (0.792, 0.855) | 0.098 (0.085, 0.111) | 0.166 | 0.2493 |  |

## 11. Bacteriologically confirmed TB only (R3.1)

Outcome = bacteriologically confirmed TB vs no TB disease; clinician-diagnosed-only cases excluded (not recoded); original train/test assignment kept. Exclusions (`rev_bac_confirmed_exclusions.csv`):

| Set | Rows (main analysis) | Excluded: clinician-diagnosed only | Rows analysed | Bac-confirmed TB | No TB disease |
|---|---|---|---|---|---|
| Train | 636 | 11 | 625 | 130 | 495 |
| Test | 159 | 3 | 156 | 32 | 124 |
| Total | 795 | 14 | 781 | 162 | 619 |

Results (`rev_bac_confirmed_sensitivity.csv`). 'Main models, bac-confirmed test rows' = main models and thresholds scored on the same 156 test patients:

| Algorithm | Model | Analysis | n_test | TB_test | Sens | Spec | PPV | NPV | AUROC | AP | Brier | Null Brier | Brier (recal.) | Thr |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| XGBoost | Model 1 | Main analysis (all TB vs no TB) | 159 | 35 | 0.429 (0.263, 0.606) | 0.790 (0.708, 0.858) | 0.366 (0.221, 0.531) | 0.831 (0.750, 0.893) | 0.629 (0.519, 0.740) | 0.329 (0.224, 0.494) | 0.239 (0.215, 0.264) | 0.172 | 0.169 (0.134, 0.205) | 0.6183 |
| XGBoost | Model 1 | Main models, bac-confirmed test rows | 156 | 32 | 0.469 (0.291, 0.653) | 0.790 (0.708, 0.858) | 0.366 (0.221, 0.531) | 0.852 (0.774, 0.911) | 0.661 (0.547, 0.772) | 0.334 (0.218, 0.506) | 0.235 (0.211, 0.260) | 0.163 | 0.157 (0.123, 0.191) | 0.6183 |
| XGBoost | Model 1 | Re-run: bac-confirmed vs no TB | 156 | 32 | 0.531 (0.347, 0.709) | 0.815 (0.735, 0.879) | 0.425 (0.270, 0.591) | 0.871 (0.796, 0.926) | 0.695 (0.582, 0.803) | 0.380 (0.251, 0.562) | 0.213 (0.200, 0.226) | 0.163 | 0.151 (0.116, 0.185) | 0.5255 |
| XGBoost | Model 2 | Main analysis (all TB vs no TB) | 159 | 35 | 0.514 (0.340, 0.686) | 0.774 (0.690, 0.844) | 0.391 (0.251, 0.546) | 0.850 (0.770, 0.910) | 0.663 (0.550, 0.773) | 0.391 (0.258, 0.562) | 0.250 (0.217, 0.286) | 0.172 | 0.163 (0.127, 0.200) | 0.6506 |
| XGBoost | Model 2 | Main models, bac-confirmed test rows | 156 | 32 | 0.562 (0.377, 0.736) | 0.774 (0.690, 0.844) | 0.391 (0.251, 0.546) | 0.873 (0.796, 0.929) | 0.700 (0.584, 0.806) | 0.402 (0.260, 0.574) | 0.245 (0.211, 0.280) | 0.163 | 0.150 (0.116, 0.185) | 0.6506 |
| XGBoost | Model 2 | Re-run: bac-confirmed vs no TB | 156 | 32 | 0.562 (0.377, 0.736) | 0.766 (0.682, 0.837) | 0.383 (0.245, 0.536) | 0.872 (0.794, 0.928) | 0.709 (0.602, 0.811) | 0.405 (0.264, 0.579) | 0.231 (0.196, 0.267) | 0.163 | 0.150 (0.114, 0.187) | 0.634 |
| XGBoost | Model 3 | Main analysis (all TB vs no TB) | 159 | 35 | 0.714 (0.537, 0.854) | 0.766 (0.682, 0.837) | 0.463 (0.326, 0.604) | 0.905 (0.832, 0.953) | 0.844 (0.772, 0.910) | 0.587 (0.426, 0.759) | 0.124 (0.094, 0.156) | 0.172 | 0.124 (0.091, 0.159) | 0.2239 |
| XGBoost | Model 3 | Main models, bac-confirmed test rows | 156 | 32 | 0.750 (0.566, 0.885) | 0.766 (0.682, 0.837) | 0.453 (0.316, 0.596) | 0.922 (0.853, 0.966) | 0.859 (0.789, 0.918) | 0.592 (0.423, 0.765) | 0.115 (0.087, 0.144) | 0.163 | 0.115 (0.085, 0.146) | 0.2239 |
| XGBoost | Model 3 | Re-run: bac-confirmed vs no TB | 156 | 32 | 0.750 (0.566, 0.885) | 0.774 (0.690, 0.844) | 0.462 (0.322, 0.605) | 0.923 (0.854, 0.966) | 0.862 (0.788, 0.923) | 0.641 (0.470, 0.810) | 0.135 (0.106, 0.167) | 0.163 | 0.112 (0.081, 0.143) | 0.4144 |
| XGBoost | Model 4 | Main analysis (all TB vs no TB) | 159 | 35 | 0.714 (0.537, 0.854) | 0.766 (0.682, 0.837) | 0.463 (0.326, 0.604) | 0.905 (0.832, 0.953) | 0.844 (0.768, 0.913) | 0.639 (0.478, 0.810) | 0.127 (0.093, 0.163) | 0.172 | 0.118 (0.083, 0.155) | 0.3091 |
| XGBoost | Model 4 | Main models, bac-confirmed test rows | 156 | 32 | 0.750 (0.566, 0.885) | 0.766 (0.682, 0.837) | 0.453 (0.316, 0.596) | 0.922 (0.853, 0.966) | 0.868 (0.793, 0.933) | 0.659 (0.487, 0.831) | 0.116 (0.085, 0.150) | 0.163 | 0.106 (0.075, 0.141) | 0.3091 |
| XGBoost | Model 4 | Re-run: bac-confirmed vs no TB | 156 | 32 | 0.750 (0.566, 0.885) | 0.782 (0.699, 0.851) | 0.471 (0.329, 0.615) | 0.924 (0.855, 0.967) | 0.868 (0.792, 0.932) | 0.707 (0.556, 0.844) | 0.129 (0.094, 0.165) | 0.163 | 0.104 (0.073, 0.139) | 0.3927 |
| XGBoost | Model 5 | Main analysis (all TB vs no TB) | 159 | 35 | 0.714 (0.537, 0.854) | 0.823 (0.744, 0.885) | 0.532 (0.381, 0.679) | 0.911 (0.842, 0.956) | 0.885 (0.819, 0.940) | 0.755 (0.605, 0.869) | 0.112 (0.077, 0.152) | 0.172 | 0.101 (0.068, 0.141) | 0.2816 |
| XGBoost | Model 5 | Main models, bac-confirmed test rows | 156 | 32 | 0.781 (0.600, 0.907) | 0.823 (0.744, 0.885) | 0.532 (0.381, 0.679) | 0.936 (0.872, 0.974) | 0.908 (0.845, 0.957) | 0.784 (0.641, 0.899) | 0.099 (0.067, 0.134) | 0.163 | 0.087 (0.057, 0.121) | 0.2816 |
| XGBoost | Model 5 | Re-run: bac-confirmed vs no TB | 156 | 32 | 0.812 (0.636, 0.928) | 0.815 (0.735, 0.879) | 0.531 (0.383, 0.675) | 0.944 (0.882, 0.979) | 0.907 (0.832, 0.963) | 0.799 (0.655, 0.913) | 0.078 (0.046, 0.114) | 0.163 | 0.078 (0.047, 0.113) | 0.1153 |
| CatBoost | Model 1 | Main analysis (all TB vs no TB) | 159 | 35 | 0.229 (0.104, 0.401) | 0.790 (0.708, 0.858) | 0.235 (0.107, 0.412) | 0.784 (0.702, 0.853) | 0.579 (0.466, 0.686) | 0.285 (0.193, 0.438) | 0.198 (0.183, 0.212) | 0.172 | 0.174 (0.136, 0.212) | 0.4181 |
| CatBoost | Model 1 | Main models, bac-confirmed test rows | 156 | 32 | 0.250 (0.115, 0.434) | 0.790 (0.708, 0.858) | 0.235 (0.107, 0.412) | 0.803 (0.722, 0.870) | 0.592 (0.476, 0.698) | 0.279 (0.183, 0.440) | 0.194 (0.179, 0.209) | 0.163 | 0.164 (0.127, 0.201) | 0.4181 |
| CatBoost | Model 1 | Re-run: bac-confirmed vs no TB | 156 | 32 | 0.375 (0.211, 0.563) | 0.790 (0.708, 0.858) | 0.316 (0.175, 0.487) | 0.831 (0.750, 0.893) | 0.683 (0.575, 0.787) | 0.383 (0.239, 0.550) | 0.265 (0.237, 0.295) | 0.163 | 0.153 (0.118, 0.188) | 0.7077 |
| CatBoost | Model 2 | Main analysis (all TB vs no TB) | 159 | 35 | 0.543 (0.366, 0.712) | 0.734 (0.647, 0.809) | 0.365 (0.236, 0.510) | 0.850 (0.769, 0.912) | 0.649 (0.540, 0.752) | 0.391 (0.262, 0.552) | 0.231 (0.224, 0.237) | 0.172 | 0.165 (0.130, 0.200) | 0.5033 |
| CatBoost | Model 2 | Main models, bac-confirmed test rows | 156 | 32 | 0.594 (0.406, 0.763) | 0.734 (0.647, 0.809) | 0.365 (0.236, 0.510) | 0.875 (0.796, 0.932) | 0.690 (0.578, 0.796) | 0.404 (0.256, 0.572) | 0.229 (0.222, 0.235) | 0.163 | 0.152 (0.118, 0.184) | 0.5033 |
| CatBoost | Model 2 | Re-run: bac-confirmed vs no TB | 156 | 32 | 0.406 (0.237, 0.594) | 0.879 (0.808, 0.931) | 0.464 (0.275, 0.661) | 0.852 (0.778, 0.908) | 0.729 (0.623, 0.828) | 0.439 (0.287, 0.626) | 0.158 (0.130, 0.185) | 0.163 | 0.146 (0.108, 0.183) | 0.487 |
| CatBoost | Model 3 | Main analysis (all TB vs no TB) | 159 | 35 | 0.714 (0.537, 0.854) | 0.790 (0.708, 0.858) | 0.490 (0.348, 0.634) | 0.907 (0.836, 0.955) | 0.824 (0.743, 0.893) | 0.577 (0.424, 0.721) | 0.200 (0.191, 0.209) | 0.172 | 0.133 (0.100, 0.169) | 0.4466 |
| CatBoost | Model 3 | Main models, bac-confirmed test rows | 156 | 32 | 0.688 (0.500, 0.839) | 0.790 (0.708, 0.858) | 0.458 (0.314, 0.608) | 0.907 (0.836, 0.955) | 0.821 (0.739, 0.894) | 0.564 (0.397, 0.720) | 0.198 (0.189, 0.207) | 0.163 | 0.127 (0.095, 0.162) | 0.4466 |
| CatBoost | Model 3 | Re-run: bac-confirmed vs no TB | 156 | 32 | 0.750 (0.566, 0.885) | 0.790 (0.708, 0.858) | 0.480 (0.337, 0.626) | 0.925 (0.857, 0.967) | 0.861 (0.793, 0.920) | 0.600 (0.434, 0.767) | 0.119 (0.091, 0.149) | 0.163 | 0.116 (0.084, 0.150) | 0.2489 |
| CatBoost | Model 4 | Main analysis (all TB vs no TB) | 159 | 35 | 0.743 (0.567, 0.875) | 0.831 (0.753, 0.892) | 0.553 (0.401, 0.698) | 0.920 (0.853, 0.963) | 0.837 (0.753, 0.910) | 0.625 (0.468, 0.795) | 0.125 (0.100, 0.153) | 0.172 | 0.117 (0.085, 0.152) | 0.3222 |
| CatBoost | Model 4 | Main models, bac-confirmed test rows | 156 | 32 | 0.781 (0.600, 0.907) | 0.831 (0.753, 0.892) | 0.543 (0.390, 0.691) | 0.936 (0.873, 0.974) | 0.852 (0.767, 0.927) | 0.634 (0.473, 0.824) | 0.118 (0.093, 0.144) | 0.163 | 0.107 (0.077, 0.139) | 0.3222 |
| CatBoost | Model 4 | Re-run: bac-confirmed vs no TB | 156 | 32 | 0.812 (0.636, 0.928) | 0.806 (0.726, 0.872) | 0.520 (0.374, 0.663) | 0.943 (0.881, 0.979) | 0.901 (0.835, 0.957) | 0.797 (0.665, 0.904) | 0.157 (0.130, 0.187) | 0.163 | 0.092 (0.065, 0.122) | 0.5593 |
| CatBoost | Model 5 | Main analysis (all TB vs no TB) | 159 | 35 | 0.857 (0.697, 0.952) | 0.855 (0.780, 0.912) | 0.625 (0.474, 0.760) | 0.955 (0.898, 0.985) | 0.926 (0.864, 0.974) | 0.849 (0.735, 0.930) | 0.084 (0.060, 0.113) | 0.172 | 0.080 (0.052, 0.112) | 0.2493 |
| CatBoost | Model 5 | Main models, bac-confirmed test rows | 156 | 32 | 0.906 (0.750, 0.980) | 0.855 (0.780, 0.912) | 0.617 (0.464, 0.755) | 0.972 (0.922, 0.994) | 0.934 (0.873, 0.981) | 0.870 (0.771, 0.949) | 0.075 (0.054, 0.099) | 0.163 | 0.069 (0.045, 0.097) | 0.2493 |
| CatBoost | Model 5 | Re-run: bac-confirmed vs no TB | 156 | 32 | 0.875 (0.710, 0.965) | 0.855 (0.780, 0.912) | 0.609 (0.454, 0.749) | 0.964 (0.910, 0.990) | 0.931 (0.874, 0.977) | 0.858 (0.747, 0.943) | 0.075 (0.052, 0.103) | 0.163 | 0.070 (0.044, 0.101) | 0.1782 |

Model 5 feature lists:

| Algorithm | Analysis | Features |
|---|---|---|
| XGBoost | Main analysis (all TB vs no TB) | qXR Score; QFT result; Age; LYM; MCH; NEU; Prior TB disease (self-report); RBC; Sex; History of contact with TB disease case; MCV; PCT; Nodules; Weight loss; Provider CXR interpret |
| XGBoost | Re-run: bac-confirmed vs no TB | qXR Score; QFT result; RBC; MCH; Age; LYM; Prior TB disease (self-report); NEU; Sex; History of contact with TB disease case; WBC; MCV; Nodules; BMI; Infiltrates |
| CatBoost | Main analysis (all TB vs no TB) | qXR Score; NEU; LYM; Nodules; Provider CXR interpret; MCH; PLT; QFT result; PCT; MCV; BMI; Infiltrates; Age; PDW; WBC |
| CatBoost | Re-run: bac-confirmed vs no TB | qXR Score; LYM; NEU; QFT result; Nodules; HCT; Prior TB disease (self-report); BMI; Infiltrates; PLT; MCH; Age; Provider CXR interpret; WBC; MCV |

## 12. Participants without a chest X-ray (R3.2)

Enrolled Pakistan participants n = 1,129; 18 with contaminated culture excluded → 1,111 eligible: 795 with a qXR-scored CXR (modeling cohort) and 316 without (289 'No scan', 27 'Not in Qure dataset'). Predictor recoding replicates the R preprocessing and reproduces the modeling file exactly for the 795.

**Included vs excluded** (`rev_cxr_included_vs_excluded.csv`):

| Variable | Level | Included: CXR available (N=795) | Excluded: no CXR (N=316) | of which No scan (N=289) | of which Not in Qure (N=27) | p-value (included vs excluded) | Test |
|---|---|---|---|---|---|---|---|
| Age, mean (SD) |  | 37.0 (17.3) | 39.0 (16.9) | 39.1 (17.2) | 37.7 (13.6) | 0.091 | t-test |
| Sex, n (%) | Female | 416 (52.3) | 150 (47.5) | 136 (47.1) | 14 (51.9) | 0.144 | Chi-square |
|  | Male | 379 (47.7) | 166 (52.5) | 153 (52.9) | 13 (48.1) |  |  |
| BMI, mean (SD) |  | 20.9 (4.8) | 21.8 (5.1) | 21.8 (5.1) | 22.2 (5.6) | 0.006 | t-test |
| Persistent cough, n (%) | No | 151 (19.0) | 48 (15.2) | 40 (13.8) | 8 (29.6) | 0.136 | Chi-square |
|  | Yes | 644 (81.0) | 268 (84.8) | 249 (86.2) | 19 (70.4) |  |  |
| Shortness breath, n (%) | No | 402 (50.6) | 133 (42.1) | 123 (42.6) | 10 (37.0) | 0.011 | Chi-square |
|  | Yes | 393 (49.4) | 183 (57.9) | 166 (57.4) | 17 (63.0) |  |  |
| Fever, n (%) | No | 43 (5.4) | 44 (13.9) | 39 (13.5) | 5 (18.5) | <0.001 | Chi-square |
|  | Yes | 752 (94.6) | 272 (86.1) | 250 (86.5) | 22 (81.5) |  |  |
| Fatigue, n (%) | No | 90 (11.3) | 57 (18.0) | 50 (17.3) | 7 (25.9) | 0.003 | Chi-square |
|  | Yes | 705 (88.7) | 259 (82.0) | 239 (82.7) | 20 (74.1) |  |  |
| Hemoptysis, n (%) | No | 707 (88.9) | 274 (86.7) | 248 (85.8) | 26 (96.3) | 0.299 | Chi-square |
|  | Yes | 88 (11.1) | 42 (13.3) | 41 (14.2) | 1 (3.7) |  |  |
| Weight loss, n (%) | No | 272 (34.2) | 118 (37.3) | 110 (38.1) | 8 (29.6) | 0.324 | Chi-square |
|  | Yes | 523 (65.8) | 198 (62.7) | 179 (61.9) | 19 (70.4) |  |  |
| Night sweats, n (%) | No | 541 (68.1) | 223 (70.6) | 212 (73.4) | 11 (40.7) | 0.414 | Chi-square |
|  | Yes | 254 (31.9) | 93 (29.4) | 77 (26.6) | 16 (59.3) |  |  |
| Prior TB disease (self-report), n (%) | No | 626 (78.7) | 254 (80.4) | 231 (79.9) | 23 (85.2) | 0.544 | Chi-square |
|  | Yes | 169 (21.3) | 62 (19.6) | 58 (20.1) | 4 (14.8) |  |  |
| QFT result, n (%) | Indeterminate | 37 (4.7) | 20 (6.3) | 20 (6.9) | 0 (0.0) | 0.149 | Chi-square |
|  | Negative | 300 (37.7) | 133 (42.1) | 124 (42.9) | 9 (33.3) |  |  |
|  | Positive | 458 (57.6) | 163 (51.6) | 145 (50.2) | 18 (66.7) |  |  |
| TB disease, n (%) | No | 619 (77.9) | 228 (72.2) | 205 (70.9) | 23 (85.2) | 0.044 | Chi-square |
|  | Yes | 176 (22.1) | 88 (27.8) | 84 (29.1) | 4 (14.8) |  |  |

**M1/M2 on all eligible participants** — new stratified 80/20 split of 1,111 (seed 42; train 888 / 211 TB, test 223 / 53 TB; 164 test rows had a CXR); RFE stages 1–2 re-run (`rev_all_eligible_M1M2_vs_cxr_restricted.csv`). The two test sets differ, so this compares modelling samples, not one set of patients:

| Algorithm | Model | Analysis | n_test | TB_test | Sens | Spec | PPV | NPV | AUROC | AP | Brier | Null Brier | Brier (recal.) | Thr |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| XGBoost | Model 1 | CXR-restricted (main, n=795) | 159 | 35 | 0.429 (0.263, 0.606) | 0.790 (0.708, 0.858) | 0.366 (0.221, 0.531) | 0.831 (0.750, 0.893) | 0.629 (0.519, 0.740) | 0.329 (0.224, 0.494) | 0.239 (0.215, 0.264) | 0.172 | 0.169 (0.134, 0.205) | 0.6183 |
| XGBoost | Model 1 | All eligible (n=1111) | 223 | 53 | 0.491 (0.351, 0.632) | 0.776 (0.706, 0.837) | 0.406 (0.285, 0.536) | 0.830 (0.763, 0.885) | 0.700 (0.626, 0.775) | 0.369 (0.281, 0.503) | 0.206 (0.184, 0.229) | 0.181 | 0.168 (0.144, 0.194) | 0.568 |
| XGBoost | Model 2 | CXR-restricted (main, n=795) | 159 | 35 | 0.514 (0.340, 0.686) | 0.774 (0.690, 0.844) | 0.391 (0.251, 0.546) | 0.850 (0.770, 0.910) | 0.663 (0.550, 0.773) | 0.391 (0.258, 0.562) | 0.250 (0.217, 0.286) | 0.172 | 0.163 (0.127, 0.200) | 0.6506 |
| XGBoost | Model 2 | All eligible (n=1111) | 223 | 53 | 0.604 (0.460, 0.735) | 0.776 (0.706, 0.837) | 0.457 (0.337, 0.581) | 0.863 (0.798, 0.913) | 0.707 (0.627, 0.788) | 0.392 (0.297, 0.533) | 0.243 (0.212, 0.275) | 0.181 | 0.168 (0.142, 0.197) | 0.6701 |
| CatBoost | Model 1 | CXR-restricted (main, n=795) | 159 | 35 | 0.229 (0.104, 0.401) | 0.790 (0.708, 0.858) | 0.235 (0.107, 0.412) | 0.784 (0.702, 0.853) | 0.579 (0.466, 0.686) | 0.285 (0.193, 0.438) | 0.198 (0.183, 0.212) | 0.172 | 0.174 (0.136, 0.212) | 0.4181 |
| CatBoost | Model 1 | All eligible (n=1111) | 223 | 53 | 0.509 (0.368, 0.649) | 0.771 (0.700, 0.831) | 0.409 (0.290, 0.537) | 0.834 (0.767, 0.889) | 0.724 (0.655, 0.793) | 0.400 (0.304, 0.535) | 0.163 (0.139, 0.189) | 0.181 | 0.163 (0.138, 0.189) | 0.327 |
| CatBoost | Model 2 | CXR-restricted (main, n=795) | 159 | 35 | 0.543 (0.366, 0.712) | 0.734 (0.647, 0.809) | 0.365 (0.236, 0.510) | 0.850 (0.769, 0.912) | 0.649 (0.540, 0.752) | 0.391 (0.262, 0.552) | 0.231 (0.224, 0.237) | 0.172 | 0.165 (0.130, 0.200) | 0.5033 |
| CatBoost | Model 2 | All eligible (n=1111) | 223 | 53 | 0.528 (0.386, 0.667) | 0.765 (0.694, 0.826) | 0.412 (0.294, 0.538) | 0.839 (0.771, 0.893) | 0.678 (0.587, 0.764) | 0.437 (0.314, 0.571) | 0.188 (0.173, 0.203) | 0.181 | 0.166 (0.140, 0.194) | 0.374 |

**Main (CXR-trained) M1/M2 models scored on the 316 participants without a CXR** (fixed thresholds and recalibration; none of these participants were used for training; `rev_noCXR_holdout_M1M2.csv`):

| Algorithm | Model | Analysis | n_test | TB_test | Sens | Spec | PPV | NPV | AUROC | AP | Brier | Null Brier | Brier (recal.) | Thr |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| XGBoost | Model 1 | Main model, CXR test set (n=159) | 159 | 35 | 0.429 (0.263, 0.606) | 0.790 (0.708, 0.858) | 0.366 (0.221, 0.531) | 0.831 (0.750, 0.893) | 0.629 (0.519, 0.740) | 0.329 (0.224, 0.494) | 0.239 (0.215, 0.264) | 0.172 | 0.169 (0.134, 0.205) | 0.6183 |
| XGBoost | Model 1 | Main model, no-CXR participants (n=316) | 316 | 88 | 0.386 (0.284, 0.496) | 0.829 (0.774, 0.875) | 0.466 (0.348, 0.586) | 0.778 (0.720, 0.828) | 0.706 (0.637, 0.769) | 0.482 (0.387, 0.590) | 0.206 (0.188, 0.224) | 0.204 | 0.185 (0.157, 0.212) | 0.6183 |
| XGBoost | Model 2 | Main model, CXR test set (n=159) | 159 | 35 | 0.514 (0.340, 0.686) | 0.774 (0.690, 0.844) | 0.391 (0.251, 0.546) | 0.850 (0.770, 0.910) | 0.663 (0.550, 0.773) | 0.391 (0.258, 0.562) | 0.250 (0.217, 0.286) | 0.172 | 0.163 (0.127, 0.200) | 0.6506 |
| XGBoost | Model 2 | Main model, no-CXR participants (n=316) | 316 | 88 | 0.455 (0.348, 0.564) | 0.851 (0.798, 0.894) | 0.541 (0.421, 0.657) | 0.802 (0.746, 0.850) | 0.749 (0.686, 0.806) | 0.512 (0.415, 0.628) | 0.198 (0.177, 0.221) | 0.204 | 0.179 (0.152, 0.207) | 0.6506 |
| CatBoost | Model 1 | Main model, CXR test set (n=159) | 159 | 35 | 0.229 (0.104, 0.401) | 0.790 (0.708, 0.858) | 0.235 (0.107, 0.412) | 0.784 (0.702, 0.853) | 0.579 (0.466, 0.686) | 0.285 (0.193, 0.438) | 0.198 (0.183, 0.212) | 0.172 | 0.174 (0.136, 0.212) | 0.4181 |
| CatBoost | Model 1 | Main model, no-CXR participants (n=316) | 316 | 88 | 0.239 (0.154, 0.341) | 0.825 (0.769, 0.872) | 0.344 (0.227, 0.477) | 0.737 (0.679, 0.790) | 0.598 (0.527, 0.670) | 0.349 (0.279, 0.454) | 0.208 (0.197, 0.220) | 0.204 | 0.206 (0.176, 0.236) | 0.4181 |
| CatBoost | Model 2 | Main model, CXR test set (n=159) | 159 | 35 | 0.543 (0.366, 0.712) | 0.734 (0.647, 0.809) | 0.365 (0.236, 0.510) | 0.850 (0.769, 0.912) | 0.649 (0.540, 0.752) | 0.391 (0.262, 0.552) | 0.231 (0.224, 0.237) | 0.172 | 0.165 (0.130, 0.200) | 0.5033 |
| CatBoost | Model 2 | Main model, no-CXR participants (n=316) | 316 | 88 | 0.443 (0.337, 0.553) | 0.833 (0.778, 0.879) | 0.506 (0.390, 0.622) | 0.795 (0.738, 0.844) | 0.714 (0.648, 0.776) | 0.500 (0.394, 0.614) | 0.226 (0.221, 0.231) | 0.204 | 0.184 (0.157, 0.212) | 0.5033 |

## 13. Class weighting actually used

Main-analysis final M1–M5 models (`rev_class_weighting_audit.csv`):

| Algo | Source | Weighted | Detail |
|---|---|---|---|
| XGB | Final M1-M5 (Model 1) | True | scale_pos_weight=4.012 |
| XGB | Final M1-M5 (Model 2) | True | scale_pos_weight=5.053 |
| XGB | Final M1-M5 (Model 3) | False | scale_pos_weight=1.000 |
| XGB | Final M1-M5 (Model 4) | True | scale_pos_weight=2.322 |
| XGB | Final M1-M5 (Model 5) | True | scale_pos_weight=2.385 |
| CAT | Final M1-M5 (Model 1) | False | class_weights=None |
| CAT | Final M1-M5 (Model 2) | True | class_weights=[1.000, 2.478] |
| CAT | Final M1-M5 (Model 3) | False | class_weights=None |
| CAT | Final M1-M5 (Model 4) | False | class_weights=None |
| CAT | Final M1-M5 (Model 5) | False | class_weights=None |

Full-feature models:

| Algo | Source | Weighted | Detail |
|---|---|---|---|
| XGB | PI refit (full train) | False | scale_pos_weight=1.000 |
| CAT | PI refit (full train) | False | class_weights=None |
| XGB | Full-feature refit (no noise) | False | scale_pos_weight=1.000 |
| CAT | Full-feature refit (no noise) | False | class_weights=None |

→ XGBoost M1, M2, M4, M5 and CatBoost M2 used class weighting; the full-feature models did not. Per-fold nested-CV weighting and every other model are in the audit files (`rev_class_weighting_audit_summary.csv`).

## 14. Hyperparameter search spaces and selected values (full-feature models)

File: `rev_tuning_fairness_table.csv`. 'Best' values are for the full-feature (cell 6) tuning, not the M1–M5 models.

| Concept / Parameter | Purpose / Notes | XGBoost — Search space / Fixed | XGBoost — Best | CatBoost — Search space / Fixed | CatBoost — Best |
|---|---|---|---|---|---|
| Learning rate | Step size shrinkage. | [0.05, 0.1] | 0.0927503 | [0.05, 0.1] | 0.0821788 |
| Tree depth (capacity) | Model capacity. | [3, 7] (max_depth) | 3 | [3, 7] (depth) | 6 |
| L2 regularization | Overfit control. | [1.0, 10.0] (lambda) | 6.77254 | [1.0, 10.0] (l2_leaf_reg) | 6.77634 |
| L1 regularization | Sparsity (XGB only). | [0.0, 10.0] (alpha) | 2.12757 | — | — |
| Column sampling | Per-tree feature sampling. | [0.7, 0.95] (colsample_bytree) | 0.791793 | [0.7, 0.95] (rsm) | 0.732045 |
| Row sampling (Bernoulli) | Stochastic per-row. | [0.7, 0.95] (subsample) | 0.894817 | If Bernoulli → [0.7, 0.95] | 0.863162 |
| Bagging temperature (Bayesian) | Bayesian bootstrap randomness. | — | — | If Bayesian → [0.0, 1.0] | — |
| Bootstrap family | CatBoost bagging scheme. | — | — | {Bayesian, Bernoulli} | Bernoulli |
| Min child / leaf weight | Leaf stability. | [1, 10] (min_child_weight) | 2 | [1, 32] (min_data_in_leaf) | 19 |
| Min split loss (γ) | Penalty to create a new split. | [0.0, 5.0] (gamma) | 0.951369 | — | — |
| Class imbalance weighting | Optional weighting around class ratio. | use_weights∈{T,F}; w_mult∈[0.5,2.0]; scale_pos_weight = 1 if off else w_mult×SCALE_POS(3.51) | use_weights=False; scale_pos_weight=1 | use_weights∈{T,F}; w_mult∈[0.5,2.0] | use_weights=False (not used) |
| Objective / Loss | Binary loss. | binary:logistic | binary:logistic | Logloss | Logloss |
| Eval metric | Guides early stopping. | auc | auc | AUC | AUC |
| Training length control | Upper cap & early stopping patience. | ≤2000; patience=80 | best_rounds=62 | ≤2000; patience=80 | best_rounds=34 |
| Categorical handling | Native categorical support. | enable_categorical=True (fixed) | Enabled | Native CatBoost (fixed) | Enabled |
| Tree method / Seed | Determinism / backend. | tree_method=hist; seed=42 | hist; seed=42 | random_seed=42 | 42 |

## 15. Permutation importance (full-feature models, test set; Figure 3)

ΔPR-AUC (primary) and ΔAUROC (sensitivity), 200 permutations, BH-FDR q-values; a random-noise control feature is included. Files: `rev_perm_importance_{XGB,CAT}_PR-AUC_primary.csv`, `..._AUROC_sensitivity.csv`.

**XGBoost** — random-noise control ranked 10/41 by ΔPR-AUC and 20/41 by ΔAUROC. Top 12 by ΔPR-AUC:

| Display | ΔPR_AUC_mean | ΔPR_AUC_lo | ΔPR_AUC_hi | qval_PR_AUC | ΔAUROC_mean | qval_AUROC |
|---|---|---|---|---|---|---|
| qXR Score | 0.4186 | 0.4108 | 0.4264 | <0.001 | 0.2305 | <0.001 |
| Prior TB disease (self-report) | 0.0198 | 0.0176 | 0.022 | <0.001 | 0.0005 | 0.049 |
| QFT result | 0.0145 | 0.0115 | 0.0174 | <0.001 | 0.0073 | <0.001 |
| WBC | 0.011 | 0.0092 | 0.0128 | <0.001 | 0.0057 | <0.001 |
| MCV | 0.009 | 0.0083 | 0.0097 | <0.001 | 0.0019 | <0.001 |
| MCH | 0.0054 | 0.0048 | 0.0061 | <0.001 | 0.0015 | <0.001 |
| HGB | 0.005 | 0.0043 | 0.0058 | <0.001 | 0.0048 | <0.001 |
| Nodules | 0.0043 | 0.0039 | 0.0047 | <0.001 | 0.0009 | <0.001 |
| PDW | 0.0041 | 0.0036 | 0.0046 | <0.001 | 0.0017 | <0.001 |
| Random noise (control) | 0.0041 | 0.0028 | 0.0053 | <0.001 | 0.0001 | 0.641 |
| P_LCR | 0.0028 | 0.0025 | 0.0031 | <0.001 | 0.0005 | <0.001 |
| BMI | 0.002 | 0.0014 | 0.0026 | <0.001 | -0.0011 | 1.000 |

**CatBoost** — random-noise control ranked 30/41 by ΔPR-AUC and 38/41 by ΔAUROC. Top 12 by ΔPR-AUC:

| Display | ΔPR_AUC_mean | ΔPR_AUC_lo | ΔPR_AUC_hi | qval_PR_AUC | ΔAUROC_mean | qval_AUROC |
|---|---|---|---|---|---|---|
| qXR Score | 0.3983 | 0.3906 | 0.4061 | <0.001 | 0.2168 | <0.001 |
| Prior TB disease (self-report) | 0.0175 | 0.0151 | 0.0198 | <0.001 | -0.0026 | 1.000 |
| LYM | 0.0153 | 0.013 | 0.0177 | <0.001 | -0.0029 | 1.000 |
| MCV | 0.0113 | 0.0104 | 0.0122 | <0.001 | 0.0009 | <0.001 |
| BMI | 0.0091 | 0.008 | 0.0101 | <0.001 | -0.0019 | 1.000 |
| Age | 0.0071 | 0.006 | 0.0083 | <0.001 | -0.0015 | 1.000 |
| MCH | 0.0067 | 0.0054 | 0.0079 | <0.001 | 0.0005 | <0.001 |
| HGB | 0.0066 | 0.0057 | 0.0074 | <0.001 | 0.0027 | <0.001 |
| MCHC | 0.0062 | 0.0055 | 0.007 | <0.001 | -0.0003 | 1.000 |
| PCT | 0.006 | 0.0048 | 0.0072 | <0.001 | -0.0009 | 1.000 |
| QFT result | 0.0049 | 0.0027 | 0.0071 | <0.001 | 0.003 | <0.001 |
| RDW_SD | 0.0048 | 0.004 | 0.0055 | <0.001 | -0.0022 | 1.000 |

**Caution — do not use these q-values as evidence of importance.** The p/q-values come from a t-test across the 200 permutation repeats, so they reflect only permutation-to-permutation variability, not sampling uncertainty; the random-noise control itself has q < 0.001 for XGBoost ΔPR-AUC. Report importance relative to the noise control instead (features ranked below it cannot be distinguished from noise on 159 test rows).

## 16. Figures (all in `Output/Images`, 300 dpi unless noted)

| File | Content |
|---|---|
| rev_fig_ROC_PR_Calibration_Hists_XGB_left_CAT_right.png | Figure 2: nested-CV ROC, PR, calibration, probability histograms (full-feature) |
| rev_perm_importance_PR-AUCprimary_with_corr_XGB_left_CAT_right.png | Figure 3: permutation importance (ΔPR-AUC) + correlations (260 dpi) |
| rev_perm_importance_AUROC_sensitivity_with_corr_XGB_left_CAT_right.png | Supplementary: permutation importance (ΔAUROC) (260 dpi) |
| rev_fig_rfe_carryforward_xgb_cat_plus_grouped.png | Figure 5 (manuscript): 5 stages × 3 columns SHAP grid — panels A (XGBoost), B (CatBoost), C (grouped SHAP) |
| rev_fig_staged_PR_left_ROC_right_xgb_cat.png | Staged PR and ROC curves, test set (200 dpi) |
| rev_fig_bar_AP_left_AUC_right_stages_xgb_cat.png | AP and AUROC by stage, test set (200 dpi) |
| rev_fig5_shap_beeswarm_model5_xgb_vs_cat_wide_annotated.png | M5 SHAP beeswarms with mean \|SHAP\| annotations (220 dpi) |
| rev_fig5_shap_beeswarm_model5_xgb_vs_cat_wide.png | M5 SHAP beeswarms without annotations (220 dpi) |
| rev_model5_score_split_dists_xgb_cat.png | M5 split values on qXR Score (200 dpi) |
| rev_sensitivity_vs_spec_targets_cat_xgb.png | Supplementary Figure 3: sensitivity vs specificity target (220 dpi) |
| rev_train_test_distribution_horizontal.png | Supplementary Figure 1: outcome distribution in train/test |
| rev_fig_staged_PR_left_ROC_right_xgb_cat_bacteriologically_confirmed.png | Supplementary: PR/ROC, bacteriologically confirmed outcome (200 dpi) |
| rev_calibration_resampling_sensitivity.png | Supplementary: calibration, unresampled vs over/undersampled |

## 17. Points for Methods / Limitations

- TB-history leakage (Section 1, item 1) and its effect: with the original variable, test AUROC for M1 was 0.806 (XGBoost) / 0.821 (CatBoost) vs 0.629 / 0.579 now; M5 0.919 / 0.943 vs 0.885 / 0.926 now (Section 6).
- Without the leak, M1–M2 (patient characteristics ± symptoms) discriminate poorly (test AUROC 0.58–0.66); most of the gain comes from provider CXR findings (M3) and the qXR score (M5).
- Class weighting was used (Section 13); the earlier statement that neither model used it must be removed.
- Two tuning recipes: the nested-CV full-feature comparison (Table 2/Fig 2) tunes for sensitivity at specificity ≥ 0.80 with per-fold thresholds; the M1–M5 models are tuned for AUROC with one threshold from training OOF predictions.
- Thresholds target specificity ≥ 0.80 on training OOF predictions; on the test set specificity was 0.73–0.86.
- Probabilities of the weakest models are poorly calibrated (Section 4 diagnostics); raw Brier for XGBoost M1–M2 and CatBoost M1–M3 is no better than the no-skill reference; after training-OOF recalibration only CatBoost M1 remains so.
- The 'CV' column in Section 5 is not fully nested with respect to feature selection and tuning and is optimistic; apparent performance (e.g. XGBoost M5 AUROC 0.991) greatly exceeds test performance (0.885).
- Single held-out test set of 159 (35 TB): CIs are wide; model differences within overlapping CIs (and the opposite direction of full-feature vs M5 for the two algorithms) should not be over-interpreted; no multiplicity correction.
- Permutation importance on 159 test rows is noisy: the noise control ranked 10/41 (XGBoost, ΔPR-AUC), and its q-values (t-test across permutation repeats) are not valid significance tests — the noise control itself is 'significant'. Any manuscript statement that features were significantly important by permutation q-values should be replaced by comparison with the noise control.
- CBC values were mean-imputed for 4 participants using all 795 before the split (minor leakage, M4–M5 only).
- Subgroup (Section 10) and all-eligible (Section 12) analyses reuse training data and are exploratory.
- Participants without CXR differed (more TB disease 27.8 % vs 22.1 %, p = 0.044; higher BMI; less fever/fatigue).
- The fairness table reports only the full-feature models' selected hyperparameters.
- Table 1 uses Student t-tests for continuous variables, several of which (CBC) are skewed.

## Appendix A. Values in the submitted manuscript (original notebook output, verbatim)

From `Code/Practicum_FINAL_CLEANED.ipynb` (before the revision; includes the leaked TB-history variable, unweighted XGBoost and a separately re-tuned stage-metrics model set).

**Table 2 — nested-CV OOF summary:**

```
=== OOF summary with 95% CIs — pooled, fold-wise thresholds ===
                   CatBoost               XGBoost
Sens   0.837 (0.771, 0.895)  0.823 (0.755, 0.885)
Spec   0.816 (0.783, 0.847)  0.851 (0.820, 0.880)
PPV    0.565 (0.497, 0.630)  0.611 (0.542, 0.677)
NPV    0.946 (0.924, 0.966)  0.944 (0.921, 0.964)
AUC    0.880 (0.841, 0.914)  0.903 (0.873, 0.930)
PRAUC  0.737 (0.658, 0.810)  0.729 (0.646, 0.804)
Brier  0.108 (0.096, 0.122)  0.097 (0.084, 0.112)
Saved: /Users/hbattey/Desktop/Rodwell/Output/csv/fig2_summary_metrics_XGB_vs_CAT_bootstrap_pooled.csv
```

**Stage metrics (test):**

```
=== Metrics — CatBoost (Test, thr at Spec≥0.80) ===
          Sens   Spec    PPV    NPV  AUROC     AP  Brier    Thr
Model                                                          
Model 1  0.686  0.839  0.545  0.904  0.810  0.532  0.171  0.471
Model 2  0.743  0.806  0.520  0.917  0.837  0.624  0.123  0.198
Model 3  0.829  0.839  0.592  0.945  0.883  0.655  0.110  0.182
Model 4  0.714  0.831  0.543  0.912  0.889  0.742  0.109  0.268
Model 5  0.914  0.823  0.593  0.971  0.937  0.840  0.085  0.217

=== Metrics — XGBoost  (Test, thr at Spec≥0.80) ===
          Sens   Spec    PPV    NPV  AUROC     AP  Brier    Thr
Model                                                          
Model 1  0.457  0.895  0.552  0.854  0.795  0.484  0.136  0.427
Model 2  0.629  0.847  0.537  0.890  0.844  0.655  0.121  0.233
Model 3  0.743  0.879  0.634  0.924  0.888  0.640  0.120  0.280
Model 4  0.743  0.847  0.578  0.921  0.900  0.780  0.106  0.217
Model 5  0.829  0.847  0.604  0.946  0.925  0.803  0.088  0.158
Saved metrics CSVs.
```

**AP/AUC by stage (figure models):**

```
Saved: /Users/hbattey/Desktop/Rodwell/Output/Images/fig_bar_AP_left_AUC_right_stages_xgb_cat.png
              AP              AUC        
Algo    CatBoost XGBoost CatBoost XGBoost
Model                                    
Model 1    0.626   0.616    0.819   0.812
Model 2    0.661   0.579    0.806   0.821
Model 3    0.729   0.710    0.884   0.891
Model 4    0.754   0.761    0.891   0.903
Model 5    0.863   0.810    0.934   0.929
```

**Supplementary Figure 3:**

```
=== Nested CV at target specificity 0.75 ===

=== Nested CV at target specificity 0.80 ===

=== Nested CV at target specificity 0.85 ===

=== Nested CV at target specificity 0.90 ===
   Specificity_Target     Model  Sensitivity  Sensitivity_CI  \
0                0.75  CatBoost     0.836879  (0.777, 0.892)   
1                0.75   XGBoost     0.836879  (0.771, 0.894)   
2                0.80  CatBoost     0.836879  (0.773, 0.894)   
3                0.80   XGBoost     0.822695  (0.757, 0.882)   
4                0.85  CatBoost     0.815603  (0.748, 0.876)   
5                0.85   XGBoost     0.787234  (0.719, 0.851)   
6                0.90  CatBoost     0.751773  (0.680, 0.817)   
7                0.90   XGBoost     0.687943  (0.611, 0.762)   

   Achieved_Specificity       PPV       NPV     AUROC     PRAUC     Brier  
0              0.826263  0.578431  0.946759  0.903747  0.737298  0.094301  
1              0.822222  0.572816  0.946512  0.898768  0.752714  0.096168  
2              0.816162  0.564593  0.946136  0.880249  0.736970  0.108407  
3              0.850505  0.610526  0.943946  0.903346  0.728739  0.097410  
4              0.860606  0.625000  0.942478  0.857518  0.674291  0.126430  
5              0.882828  0.656805  0.935760  0.895573  0.732562  0.099781  
6              0.905051  0.692810  0.927536  0.886682  0.710214  0.105938  
7              0.915152  0.697842  0.911469  0.907501  0.729634  0.094511  
Saved table: /Users/hbattey/Desktop/Rodwell/Output/csv/sensitivity_vs_spec_targets_cat_xgb.csv
Saved figure: /Users/hbattey/Desktop/Rodwell/Output/Images/sensitivity_vs_spec_targets_cat_xgb.png
```

## Appendix B. Reproducibility

- Notebook: `Code/Practicum_REVISION.ipynb` (header cell lists every change; changed lines tagged `[REV-…]` / `[R…]`). Kernel 'Python (global)': Python 3.12, catboost 1.2.7, xgboost 3.0.5, optuna 4.1.0, shap 0.46.0, scikit-learn 1.5.2, scipy 1.14.1, pandas 2.2.2. Full run ≈ 85 min on 12 CPU cores.
- Inputs: `Datasets/Checkpoints/Pakistan_imputed_minimal.xlsx` (modeling file, from `R scripts/Practicum data prep.Rmd`); `Datasets/TB_Antigen_Clinical_Data_Master_v3.0-08.30.24.xlsx` (TB_status_detailed, interview TB history, HIV, R3.2 cohort); `Datasets/master/TB_Antigen_Clinical_Data_Master_3.17.25.xlsx` (qXR score / contaminated-culture flag for R3.2).
- Original notebook and its outputs are unchanged; all revision outputs are `rev_*` files.
